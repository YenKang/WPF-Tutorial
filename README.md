<GroupBox
    Header="{Binding AutoRunVM.Title}"
    Visibility="{Binding IsAutoRunConfigVisible,
                         Converter={utilityConv:BoolToVisibilityConverter}}"
    Margin="0,12,0,0">

    <StackPanel x:Name="AutoRunPanel"
                Margin="12"
                DataContext="{Binding AutoRunVM}">

        <!-- Pattern Total -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Margin="0,0,8,0" />

            <xctk:IntegerUpDown Width="96"
                                Minimum="1"
                                Maximum="22"
                                Value="{Binding Total,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- Pattern Orders -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,2,0,2">

                        <TextBlock Width="120"
                                   VerticalAlignment="Center"
                                   Text="{Binding DisplayNo,
                                          StringFormat=Pattern Order: {0}}"/>

                        <ComboBox Width="96"
                                  ItemsSource="{Binding DataContext.PatternNumberOptions,
                                                        ElementName=AutoRunPanel}"
                                  SelectedItem="{Binding SelectedPatternNumber,
                                                         Mode=TwoWay,
                                                         UpdateSourceTrigger=PropertyChanged}"/>
                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                Margin="0,12,0,0"
                Command="{Binding ApplyCommand}" />

    </StackPanel>
</GroupBox>





＝＝＝＝＝＝


using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Diagnostics;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private const int TotalMin     = 1;
        private const int TotalMax     = 22;
        private const int TotalDefault = 22;

        private string _totalRegister;

        // JSON 給的 patternOrder.register 名稱，例如 BIST_PT_ORD0 ~ BIST_PT_ORD21
        private readonly List<string> _orderTargets = new List<string>();

        public IRegisterReadWriteEx RegControl { get; set; }

        public AutoRunVM()
        {
            Debug.WriteLine("[AutoRun] Ctor");

            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);

            // ComboBox 選項：0~21
            PatternNumberOptions = new ObservableCollection<int>();
            for (int i = 0; i <= 21; i++)
            {
                PatternNumberOptions.Add(i);
            }

            Total = TotalDefault;
        }

        #region Public Properties

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// Pattern Total：1~22，只決定要寫幾筆 ORD，不動 Orders 行數。
        /// </summary>
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                var raw = value;
                var v   = Clamp(raw, TotalMin, TotalMax);
                var old = GetValue<int>();

                if (!SetValue(v)) return;

                Debug.WriteLine($"[AutoRun] Total changed: old={old}, ui={raw}, clamped={v}");
            }
        }

        /// <summary>
        /// 每一列 Pattern Order slot（通常 22 列）
        /// </summary>
        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        /// <summary>
        /// ComboBox 選項：0~21
        /// </summary>
        public ObservableCollection<int> PatternNumberOptions { get; }

        public IRelayCommand ApplyCommand { get; }

        #endregion

        #region JSON Load

        public void LoadFrom(JObject autoRunControl)
        {
            Debug.WriteLine("===== [AutoRun] LoadFrom() start =====");

            if (autoRunControl == null)
            {
                Debug.WriteLine("[AutoRun] LoadFrom() aborted: autoRunControl == null");
                return;
            }

            Title = (string)autoRunControl["title"] ?? "Auto Run";
            Debug.WriteLine($"[AutoRun] Title = {Title}");

            // total 區
            var t = autoRunControl["total"] as JObject;
            if (t != null)
            {
                _totalRegister = (string)t["register"] ?? string.Empty;

                int def = t["default"] != null
                    ? t["default"].Value<int>()
                    : TotalDefault;

                Total = Clamp(def, TotalMin, TotalMax);

                Debug.WriteLine(
                    $"[AutoRun] TotalRegister={_totalRegister}, Default={def}, AfterClamp={Total}"
                );
            }
            else
            {
                Debug.WriteLine("[AutoRun] total section missing, use default.");
                _totalRegister = string.Empty;
                Total          = TotalDefault;
            }

            // patternOrder.register → 只拿 register 名稱，不再吃 JSON 的 default 值
            _orderTargets.Clear();
            Debug.WriteLine("[AutoRun] Parsing patternOrder.register...");

            var order    = autoRunControl["patternOrder"] as JObject;
            var regArray = order != null ? order["register"] as JArray : null;

            if (regArray != null)
            {
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;

                    var regName = prop.Name;  // e.g. BIST_PT_ORD0
                    _orderTargets.Add(regName);
                }
            }
            else
            {
                Debug.WriteLine("[AutoRun] patternOrder.register not found or empty.");
            }

            // 如果 JSON 沒給任何 register，就用預設 BIST_PT_ORD0~21
            if (_orderTargets.Count == 0)
            {
                Debug.WriteLine("[AutoRun] _orderTargets empty, fallback to BIST_PT_ORD0~21.");
                for (int i = 0; i < 22; i++)
                {
                    _orderTargets.Add($"BIST_PT_ORD{i}");
                }
            }

            // 一次性建立 Orders，並印出你截圖那種 log
            BuildOrdersFromTargets();

            Debug.WriteLine("===== [AutoRun] LoadFrom() end =====");
        }

        /// <summary>
        /// 根據 _orderTargets 一次性建立 Orders：
        ///  PatternOrder[1] → RegName=BIST_PT_ORD0, DefaultNum=0
        ///  PatternOrder[2] → RegName=BIST_PT_ORD1, DefaultNum=1
        ///  ...
        ///  PatternOrder[22] → RegName=BIST_PT_ORD21, DefaultNum=21
        /// </summary>
        private void BuildOrdersFromTargets()
        {
            Orders.Clear();

            int count = _orderTargets.Count;
            if (count > 22) count = 22;

            for (int i = 0; i < count; i++)
            {
                int displayIndex = i + 1; // PatternOrder[1..22]
                string regName   = _orderTargets[i];
                int defaultNum   = i;     // 0~21

                var slot = new OrderSlot
                {
                    DisplayNo             = displayIndex,
                    RegName               = regName,
                    SelectedPatternNumber = defaultNum
                };

                Orders.Add(slot);

                // 這行就是截圖裡的格式
                Debug.WriteLine(
                    $"[AutoRun] PatternOrder[{displayIndex}]  RegName={regName}, DefaultNum={defaultNum}"
                );
            }
        }

        #endregion

        #region Apply Write

        private void ApplyWrite()
        {
            Debug.WriteLine("===== [AutoRun] ApplyWrite() start =====");

            if (RegControl == null)
            {
                Debug.WriteLine("[AutoRun] RegControl == null");
                return;
            }

            Debug.WriteLine($"[AutoRun] ApplyWrite() with Total={Total}");

            // 1) TOTAL (Total - 1)
            if (!string.IsNullOrEmpty(_totalRegister))
            {
                int t = Clamp(Total, TotalMin, TotalMax) - 1;
                if (t < 0) t = 0;

                uint totalValue = (uint)(t & 0x1F); // 5 bits
                Debug.WriteLine(
                    $"[AutoRun] Write TOTAL: Reg={_totalRegister}, Value(dec)={totalValue}, Hex=0x{totalValue:X2}"
                );

                RegControl.SetRegisterByName(_totalRegister, totalValue);
            }
            else
            {
                Debug.WriteLine("[AutoRun] WARNING: _totalRegister is empty, TOTAL not written.");
            }

            // 2) ORD：寫前 Total 列
            int limit = Math.Min(Total, Orders.Count);
            Debug.WriteLine($"[AutoRun] ORD limit = {limit} (Orders.Count={Orders.Count})");

            for (int i = 0; i < limit; i++)
            {
                var slot = Orders[i];

                int num   = Clamp(slot.SelectedPatternNumber, 0, 21); // 0~21
                uint val  = (uint)(num & 0x3F);                       // 6 bits

                Debug.WriteLine(
                    $"[AutoRun] ORD[{i}] Reg={slot.RegName}, SelectedNum={slot.SelectedPatternNumber}, Clamped={num}, Write=0x{val:X2}"
                );

                RegControl.SetRegisterByName(slot.RegName, val);
            }

            Debug.WriteLine("===== [AutoRun] ApplyWrite() end =====");
        }

        #endregion

        private static int Clamp(int value, int min, int max)
        {
            if (value < min) return min;
            if (value > max) return max;
            return value;
        }
    }

    public sealed class OrderSlot : ViewModelBase
    {
        public int DisplayNo
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// ComboBox 選到的數字（0~21），直接拿來當暫存器值。
        /// </summary>
        public int SelectedPatternNumber
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// 對應 JSON patternOrder.register 的暫存器名稱（例如 BIST_PT_ORD0）。
        /// </summary>
        public string RegName
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }
    }
}

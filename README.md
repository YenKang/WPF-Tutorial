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
        private const int TotalMax     = 22;   // 新需求：1~22
        private const int TotalDefault = 22;   // 若 JSON 沒給，就預設 22

        private string _totalRegister;

        // patternOrder.register 讀進來的暫存器名稱（例如 BIST_PT_ORD0 ~ BIST_PT_ORD21）
        private readonly List<string> _orderTargets  = new List<string>();
        private readonly List<int>    _orderDefaults = new List<int>(); // JSON 給的預設號碼（圖片牆號碼）

        public IRegisterReadWriteEx RegControl { get; set; }

        public AutoRunVM()
        {
            Debug.WriteLine("[AutoRun] Ctor");

            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);

            // 1~22 給每列 ComboBox 使用
            PatternNumberOptions = new ObservableCollection<int>();
            for (int i = 1; i <= 22; i++)
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
        /// Pattern Total：1~22
        /// 不再控制 Orders 行數，只代表「實際要寫幾個 ORD」。
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
                // 新需求：不呼叫 ResizeOrders，行數固定由 JSON 的 patternOrder 決定
            }
        }

        /// <summary>
        /// 固定若干列（通常 22），每列對應一個 ORD 暫存器
        /// </summary>
        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        /// <summary>
        /// 給每列 ComboBox 選用的數字（1~22），代表圖片牆的編號，直接拿來下暫存器。
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

            // total 設定
            var t = autoRunControl["total"] as JObject;
            if (t != null)
            {
                _totalRegister = (string)t["register"] ?? string.Empty;
                Debug.WriteLine($"[AutoRun] Total Register Name = {_totalRegister}");

                var def = t["default"] != null ? t["default"].Value<int>() : TotalDefault;
                Debug.WriteLine($"[AutoRun] Total default from JSON = {def}");

                Total = Clamp(def, TotalMin, TotalMax);
                Debug.WriteLine($"[AutoRun] Total after clamp = {Total}");
            }
            else
            {
                Debug.WriteLine("[AutoRun] total section missing in JSON, using defaults.");
                _totalRegister = string.Empty;
                Total = TotalDefault;
            }

            // patternOrder.register
            _orderTargets.Clear();
            _orderDefaults.Clear();
            Debug.WriteLine("[AutoRun] Parsing patternOrder.register...");

            var order    = autoRunControl["patternOrder"] as JObject;
            var regArray = order != null ? order["register"] as JArray : null;

            if (regArray != null)
            {
                int index = 0;
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;

                    var regName = prop.Name;            // e.g. BIST_PT_ORD0
                    var raw     = (string)prop.Value;   // e.g. "1", "2", ...

                    int defaultNum;
                    if (!int.TryParse(raw, out defaultNum))
                        defaultNum = 1;

                    _orderTargets.Add(regName);
                    _orderDefaults.Add(defaultNum);

                    Debug.WriteLine($"[AutoRun] ORD[{index}] RegName={regName}, DefaultNum={defaultNum}");
                    index++;
                }
            }
            else
            {
                Debug.WriteLine("[AutoRun] patternOrder.register not found or empty.");
            }

            // 這版先 bypass PatOptions / fcnt1 / fcnt2

            // 依 JSON 給的 register 數量建立 Orders 列（通常 22）
            Debug.WriteLine($"[AutoRun] Building Orders rows, count = {_orderTargets.Count}");
            ResizeOrders(_orderTargets.Count);

            Debug.WriteLine("===== [AutoRun] LoadFrom() end =====");
        }

        #endregion

        #region Apply (寫暫存器)

        private void ApplyWrite()
        {
            Debug.WriteLine("===== [AutoRun] ApplyWrite() start =====");

            if (RegControl == null)
            {
                Debug.WriteLine("[AutoRun] ApplyWrite() aborted: RegControl == null");
                return;
            }

            Debug.WriteLine($"[AutoRun] ApplyWrite() with Total={Total}");

            // 1) TOTAL：寫入 (Total-1)，並用 5 bits 保護
            if (!string.IsNullOrEmpty(_totalRegister))
            {
                int t = Clamp(Total, TotalMin, TotalMax) - 1;
                if (t < 0) t = 0;

                uint totalValue = (uint)(t & 0x1F); // 5 bits
                Debug.WriteLine($"[AutoRun] Write TOTAL: Reg={_totalRegister}, Value(dec)={totalValue}, Hex=0x{totalValue:X2}");

                RegControl.SetRegisterByName(_totalRegister, totalValue);
            }
            else
            {
                Debug.WriteLine("[AutoRun] WARNING: _totalRegister is empty, TOTAL is not written.");
            }

            // 2) ORD 寫入：固定數量的 Orders，但只寫前 Total 列
            int maxCount = Math.Min(_orderTargets.Count, Orders.Count);
            int limit    = Math.Min(Total, maxCount);

            Debug.WriteLine($"[AutoRun] ORD: maxCount={maxCount}, limit (Total-based)={limit}");

            for (int i = 0; i < limit; i++)
            {
                var slot = Orders[i];

                string reg = (i < _orderTargets.Count) ? _orderTargets[i] : slot.RegName;
                if (string.IsNullOrEmpty(reg))
                {
                    Debug.WriteLine($"[AutoRun] ORD[{i}] skipped: empty regName.");
                    continue;
                }

                int num = slot.SelectedPatternNumber;
                int clampedNum = Clamp(num, 1, 22);
                uint ordValue  = (uint)(clampedNum & 0x3F); // 每個 ORD 只收 6 bits

                Debug.WriteLine(
                    $"[AutoRun] ORD[{i}] Reg={reg}, SelectedNum={num}, Clamped={clampedNum}, WriteValue(dec)={ordValue}, Hex=0x{ordValue:X2}"
                );

                RegControl.SetRegisterByName(reg, ordValue);
            }

            Debug.WriteLine("===== [AutoRun] ApplyWrite() end =====");
        }

        #endregion

        #region Helpers

        private void ResizeOrders(int count)
        {
            Debug.WriteLine($"[AutoRun] ResizeOrders(count={count})");

            if (count > 22) count = 22;

            Debug.WriteLine($"[AutoRun] ResizeOrders → actual row count={count}");

            // 增加到 count
            while (Orders.Count < count)
            {
                Orders.Add(new OrderSlot());
                Debug.WriteLine($"[AutoRun] Orders.Count increased to {Orders.Count}");
            }

            // 如果比 count 多，就砍掉多餘的（通常不會）
            while (Orders.Count > count)
            {
                Orders.RemoveAt(Orders.Count - 1);
                Debug.WriteLine($"[AutoRun] Orders.Count decreased to {Orders.Count}");
            }

            // 初始化每一列
            for (int i = 0; i < Orders.Count; i++)
            {
                var slot = Orders[i];

                slot.DisplayNo = i + 1;

                string rn = (i < _orderTargets.Count) ? _orderTargets[i] : string.Empty;
                slot.RegName = rn;

                int def = (i < _orderDefaults.Count) ? _orderDefaults[i] : 1;
                slot.SelectedPatternNumber = def;

                Debug.WriteLine(
                    $"[AutoRun] OrderSlot[{i}] → DisplayNo={slot.DisplayNo}, RegName={rn}, DefaultNum={def}"
                );
            }
        }

        private static int Clamp(int value, int min, int max)
        {
            if (value < min) return min;
            if (value > max) return max;
            return value;
        }

        private static int ParseHex(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);

            int val;
            if (int.TryParse(s,
                             System.Globalization.NumberStyles.HexNumber,
                             null, out val))
                return val;

            if (int.TryParse(s, out val))
                return val;

            return fallback;
        }

        #endregion
    }

    public sealed class OrderSlot : ViewModelBase
    {
        public int DisplayNo
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// 每一列 ComboBox 選到的圖片編號（1~22），直接拿來下暫存器。
        /// </summary>
        public int SelectedPatternNumber
        {
            get { return GetValue<int>(); }
            set
            {
                int old = GetValue<int>();
                if (!SetValue(value)) return;

                Debug.WriteLine($"[AutoRun] OrderSlot DisplayNo={DisplayNo} PatternNumber changed: old={old}, new={value}");
            }
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
＝＝＝＝

<GroupBox Header="{Binding AutoRunVM.Title,
                           RelativeSource={RelativeSource AncestorType=Window}}"
          Margin="0,12,0,12">

    <StackPanel Margin="12">

        <!-- Pattern Total -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,12">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Margin="0,0,12,0"/>

            <xctk:IntegerUpDown Width="90"
                                Minimum="1"
                                Maximum="22"
                                Value="{Binding AutoRunVM.Total,
                                                RelativeSource={RelativeSource AncestorType=Window},
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>


        <!-- Pattern Orders -->
        <ItemsControl ItemsSource="{Binding AutoRunVM.Orders,
                                            RelativeSource={RelativeSource AncestorType=Window}}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>

                    <StackPanel Orientation="Horizontal" Margin="0,0,0,6">

                        <TextBlock Text="{Binding DisplayNo}"
                                   VerticalAlignment="Center"
                                   Width="80"/>

                        <ComboBox Width="96"
                                  ItemsSource="{Binding AutoRunVM.PatternNumberOptions,
                                                        RelativeSource={RelativeSource AncestorType=Window}}"
                                  SelectedItem="{Binding SelectedPatternNumber,
                                                         Mode=TwoWay,
                                                         UpdateSourceTrigger=PropertyChanged}"/>
                    </StackPanel>

                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>


        <!-- Apply -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Margin="0,12,0,0"
                Command="{Binding AutoRunVM.ApplyCommand,
                                  RelativeSource={RelativeSource AncestorType=Window}}"/>

    </StackPanel>
</GroupBox>

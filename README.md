using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private const int TotalMin = 1;
        private const int TotalMax = 21;
        private const int TotalDefault = 5;

        private string _totalRegister;

        private readonly List<string> _orderTargets = new List<string>();
        private readonly List<int> _orderDefaults = new List<int>();

        private readonly FcntConfig _fcnt1 = new FcntConfig();
        private readonly FcntConfig _fcnt2 = new FcntConfig();

        private IRegisterControl _regControl;

        public void SetRegisterControl(IRegisterControl control)
        {
            _regControl = control;
        }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
            Total = TotalDefault;
        }

        #region Public Properties

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                ResizeOrders(value);
            }
        }

        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        public ObservableCollection<PatternOption> PatternOptions { get; }
            = new ObservableCollection<PatternOption>();

        public int FCNT1_Value
        {
            get { return GetValue<int>(); }
            set
            {
                var v = Clamp(value, _fcnt1.Min, _fcnt1.Max);
                SetValue(v);
                _fcnt1.Value = v;
            }
        }

        public int FCNT2_Value
        {
            get { return GetValue<int>(); }
            set
            {
                var v = Clamp(value, _fcnt2.Min, _fcnt2.Max);
                SetValue(v);
                _fcnt2.Value = v;
            }
        }

        public IRelayCommand ApplyCommand { get; }

        #endregion

        #region JSON Load

        public void LoadFrom(JObject autoRunControl)
        {
            if (autoRunControl == null) return;

            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // total
            var t = autoRunControl["total"] as JObject;
            if (t != null)
            {
                _totalRegister = (string)t["register"] ?? string.Empty;

                var def = t["default"] != null ? t["default"].Value<int>() : TotalDefault;
                Total = Clamp(def, TotalMin, TotalMax);
            }

            // patternOrder.register
            _orderTargets.Clear();
            _orderDefaults.Clear();

            var order = autoRunControl["patternOrder"] as JObject;
            var regArray = order != null ? order["register"] as JArray : null;

            if (regArray != null)
            {
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;

                    var regName = prop.Name;
                    var raw = (string)prop.Value;
                    int defaultIndex;

                    if (!int.TryParse(raw, out defaultIndex))
                        defaultIndex = 0;

                    _orderTargets.Add(regName);
                    _orderDefaults.Add(defaultIndex);
                }
            }

            // PatOptions
            PatternOptions.Clear();
            var optArray = autoRunControl["PatOptions"] as JArray;
            if (optArray != null)
            {
                foreach (var t2 in optArray.OfType<JObject>())
                {
                    var idx = t2["index"] != null ? t2["index"].Value<int>() : 0;
                    var name = (string)t2["name"] ?? idx.ToString();

                    PatternOptions.Add(new PatternOption
                    {
                        Index = idx,
                        Display = name
                    });
                }
            }

            // fcnt1
            var f1 = autoRunControl["fcnt1"] as JObject;
            if (f1 != null)
            {
                _fcnt1.RegName = (string)f1["register"] ?? string.Empty;
                _fcnt1.Min = ParseHex((string)f1["min"], 0x01E);
                _fcnt1.Max = ParseHex((string)f1["max"], 0x258);
                _fcnt1.Value = ParseHex((string)f1["default"], 0x03C);
                FCNT1_Value = _fcnt1.Value;
            }

            // fcnt2
            var f2 = autoRunControl["fcnt2"] as JObject;
            if (f2 != null)
            {
                _fcnt2.RegName = (string)f2["register"] ?? string.Empty;
                _fcnt2.Min = ParseHex((string)f2["min"], 0x01E);
                _fcnt2.Max = ParseHex((string)f2["max"], 0x258);
                _fcnt2.Value = ParseHex((string)f2["default"], 0x01E);
                FCNT2_Value = _fcnt2.Value;
            }

            // Apply JSON defaults → occupy Orders
            ResizeOrders(Total);
        }

        #endregion

        #region ApplyWrite (All via SetRegisterByName)

        private void ApplyWrite()
        {
            if (_regControl == null) return;

            // TOTAL: 寫入 total-1（原本BIST規格）
            if (!string.IsNullOrEmpty(_totalRegister))
            {
                _regControl.SetRegisterByName(_totalRegister, Total - 1);
            }

            // ORD0..ORDN
            var limit = Math.Min(Total, Orders.Count);
            for (int i = 0; i < limit; i++)
            {
                var slot = Orders[i];
                if (!string.IsNullOrEmpty(slot.RegName))
                {
                    _regControl.SetRegisterByName(slot.RegName, slot.SelectedIndex);
                }
            }

            // FCNT1
            if (!string.IsNullOrEmpty(_fcnt1.RegName))
            {
                _regControl.SetRegisterByName(_fcnt1.RegName, _fcnt1.Value);
            }

            // FCNT2
            if (!string.IsNullOrEmpty(_fcnt2.RegName))
            {
                _regControl.SetRegisterByName(_fcnt2.RegName, _fcnt2.Value);
            }

            // Trigger 已移除（你要求）
        }

        #endregion

        #region Helpers

        private void ResizeOrders(int count)
        {
            count = Clamp(count, TotalMin, TotalMax);

            while (Orders.Count < count)
            {
                Orders.Add(new OrderSlot());
            }

            while (Orders.Count > count)
            {
                Orders.RemoveAt(Orders.Count - 1);
            }

            for (int i = 0; i < Orders.Count; i++)
            {
                var slot = Orders[i];
                slot.DisplayNo = i + 1;

                if (i < _orderTargets.Count)
                    slot.RegName = _orderTargets[i];

                if (i < _orderDefaults.Count)
                    slot.SelectedIndex = _orderDefaults[i];
            }
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private static int ParseHex(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;

            var t = s.Trim();
            if (t.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                t = t.Substring(2);

            int result;
            return int.TryParse(t,
                                System.Globalization.NumberStyles.HexNumber,
                                null,
                                out result)
                ? result
                : fallback;
        }

        #endregion

        #region Inner Types

        public sealed class PatternOption
        {
            public int Index { get; set; }
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public string RegName
            {
                get { return GetValue<string>(); }
                set { SetValue(value); }
            }
        }

        private sealed class FcntConfig
        {
            public string RegName;
            public int Value;
            public int Min;
            public int Max;
        }

        #endregion
    }
}

====

<!-- 需要先在上層 UserControl / Window 加上：
     xmlns:xctk="http://schemas.xceed.com/wpf/xaml/toolkit" -->

<GroupBox Header="{Binding AutoRunVM.Title}"
          Margin="0,12,0,12">
    <!-- 這裡假設外層 DataContext 有 AutoRunVM 屬性 -->
    <GroupBox.DataContext>
        <Binding Path="AutoRunVM"/>
    </GroupBox.DataContext>

    <StackPanel Margin="12">

        <!-- Pattern Total：1 ~ 21 -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,12">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Margin="0,0,12,0"/>
            <xctk:IntegerUpDown Width="90"
                                Minimum="1"
                                Maximum="21"
                                Value="{Binding Total,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>

        <!-- Pattern Orders：每一列 ORD + 下拉 PatOptions -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
                        <!-- 顯示第幾筆：1,2,3... -->
                        <TextBlock Text="{Binding DisplayNo}"
                                   VerticalAlignment="Center"
                                   Width="32"/>

                        <!-- 對應 AutoRunVM.PatternOptions -->
                        <ComboBox Width="160"
                                  ItemsSource="{Binding DataContext.PatternOptions,
                                                        RelativeSource={RelativeSource AncestorType=GroupBox}}"
                                  DisplayMemberPath="Display"
                                  SelectedValuePath="Index"
                                  SelectedValue="{Binding SelectedIndex,
                                                          Mode=TwoWay,
                                                          UpdateSourceTrigger=PropertyChanged}"/>
                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

        <!-- FCNT1 -->
        <StackPanel Orientation="Horizontal" Margin="0,12,0,6">
            <TextBlock Text="FCNT1"
                       VerticalAlignment="Center"
                       Margin="0,0,12,0"/>
            <xctk:IntegerUpDown Width="120"
                                Value="{Binding FCNT1_Value,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>

        <!-- FCNT2 -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,12">
            <TextBlock Text="FCNT2"
                       VerticalAlignment="Center"
                       Margin="0,0,12,0"/>
            <xctk:IntegerUpDown Width="120"
                                Value="{Binding FCNT2_Value,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>

        <!-- 寫入 Auto Run -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Command="{Binding ApplyCommand}"/>

    </StackPanel>
</GroupBox>
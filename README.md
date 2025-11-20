## Cursor

```
using Newtonsoft.Json.Linq;
using System;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class CursorVM : ViewModelBase
    {
        private JObject _cfg;

        private string _regDiagonalEn = "BIST_DIAGONAL_EN";
        private string _regCursorEn   = "BIST_CURSOR_EN";
        private string _regHPos       = "BIST_CURSOR_HPOS";
        private string _regVPos       = "BIST_CURSOR_VPOS";

        private int _hMin = 0, _hMax = 6720, _hDefault = 960;
        private int _vMin = 0, _vMax = 2560, _vDefault = 360;

        public bool DiagonalEnable
        {
            get { return GetValue<bool>(); }
            set { SetValue(value); }
        }

        public bool CursorEnable
        {
            get { return GetValue<bool>(); }
            set { SetValue(value); }
        }

        public int HPos
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int VPos
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public ICommand ApplyCommand { get; private set; }

        public PageBase.ICStructure.Comm.IRegisterReadWriteEx RegControl { get; set; }

        public CursorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            var root = _cfg;
            if (root == null) return;

            var ctrl   = root["cursorControl"] as JObject ?? root;
            var fields = ctrl["fields"] as JObject;
            if (fields == null) return;

            // diagonalEn
            var diagonalNode = fields["diagonalEn"] as JObject;
            if (diagonalNode != null)
            {
                var regTok = diagonalNode["register"];
                if (regTok != null)
                {
                    var s = regTok.ToString().Trim();
                    if (!string.IsNullOrEmpty(s))
                        _regDiagonalEn = s;
                }

                int def = (int?)diagonalNode["default"] ?? 0;
                DiagonalEnable = def != 0;
            }

            // cursorEn
            var cursorEnNode = fields["cursorEn"] as JObject;
            if (cursorEnNode != null)
            {
                var regTok = cursorEnNode["register"];
                if (regTok != null)
                {
                    var s = regTok.ToString().Trim();
                    if (!string.IsNullOrEmpty(s))
                        _regCursorEn = s;
                }

                int def = (int?)cursorEnNode["default"] ?? 0;
                CursorEnable = def != 0;
            }

            // hPos
            var hPosNode = fields["hPos"] as JObject;
            if (hPosNode != null)
            {
                var regTok = hPosNode["register"];
                if (regTok != null)
                {
                    var s = regTok.ToString().Trim();
                    if (!string.IsNullOrEmpty(s))
                        _regHPos = s;
                }

                _hMin     = (int?)hPosNode["min"]     ?? _hMin;
                _hMax     = (int?)hPosNode["max"]     ?? _hMax;
                _hDefault = (int?)hPosNode["default"] ?? _hDefault;

                HPos = Clamp(_hDefault, _hMin, _hMax);
            }

            // vPos
            var vPosNode = fields["vPos"] as JObject;
            if (vPosNode != null)
            {
                var regTok = vPosNode["register"];
                if (regTok != null)
                {
                    var s = regTok.ToString().Trim();
                    if (!string.IsNullOrEmpty(s))
                        _regVPos = s;
                }

                _vMin     = (int?)vPosNode["min"]     ?? _vMin;
                _vMax     = (int?)vPosNode["max"]     ?? _vMax;
                _vDefault = (int?)vPosNode["default"] ?? _vDefault;

                VPos = Clamp(_vDefault, _vMin, _vMax);
            }

            System.Diagnostics.Debug.WriteLine(
                $"[Cursor Load] diagEn={DiagonalEnable}, curEn={CursorEnable}, " +
                $"HPos={HPos} [{_hMin},{_hMax}], VPos={VPos} [{_vMin},{_vMax}]");
        }

        private void ExecuteWrite()
        {
            if (RegControl == null)
            {
                MessageBox.Show("CursorVM: RegControl 尚未注入。");
                return;
            }

            int h = Clamp(HPos, _hMin, _hMax);
            int v = Clamp(VPos, _vMin, _vMax);
            int diagVal = DiagonalEnable ? 1 : 0;
            int curVal  = CursorEnable   ? 1 : 0;

            System.Diagnostics.Debug.WriteLine(
                $"[Cursor Apply] diagEn={diagVal}, curEn={curVal}, H={h}, V={v}");

            // 一律寫入（符合你之前「永遠寫」的規則）
            RegControl.SetRegisterByName(_regDiagonalEn, (uint)diagVal);
            RegControl.SetRegisterByName(_regCursorEn,   (uint)curVal);
            RegControl.SetRegisterByName(_regHPos,       (uint)h);
            RegControl.SetRegisterByName(_regVPos,       (uint)v);
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }
    }
}
```

## Cusor xaml

```
<GroupBox Header="Cursor Control" Margin="0,8,0,0">
    <StackPanel Margin="12">

        <!-- Diagonal -->
        <CheckBox Content="Diagonal Enable"
                  IsChecked="{Binding CursorVM.DiagonalEnable, Mode=TwoWay}" />

        <!-- Cursor Enable -->
        <CheckBox Content="Cursor Configurable Enable"
                  Margin="0,4,0,0"
                  IsChecked="{Binding CursorVM.CursorEnable, Mode=TwoWay}" />

        <!-- H Pos -->
        <StackPanel Orientation="Horizontal" Margin="0,8,0,0">
            <TextBlock Text="Cursor H Pos:" Width="100" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown Width="80"
                                Minimum="0"
                                Maximum="6720"
                                Value="{Binding CursorVM.HPos, Mode=TwoWay}"
                                IsEnabled="{Binding CursorVM.CursorEnable}" />
        </StackPanel>

        <!-- V Pos -->
        <StackPanel Orientation="Horizontal" Margin="0,4,0,0">
            <TextBlock Text="Cursor V Pos:" Width="100" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown Width="80"
                                Minimum="0"
                                Maximum="2560"
                                Value="{Binding CursorVM.VPos, Mode=TwoWay}"
                                IsEnabled="{Binding CursorVM.CursorEnable}" />
        </StackPanel>

        <!-- Set -->
        <Button Content="Set"
                Width="80"
                Height="30"
                Margin="0,10,0,0"
                HorizontalAlignment="Left"
                Command="{Binding CursorVM.ApplyCommand}" />
    </StackPanel>
</GroupBox>
```

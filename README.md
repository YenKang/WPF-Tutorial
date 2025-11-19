using Newtonsoft.Json.Linq;
using System;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    /// <summary>
    /// First Line Color：
    /// Enable = true  → White (寫 1)
    /// Enable = false → Black (寫 0)
    ///
    /// JSON:
    /// "firstLineSelControl": {
    ///     "register": "BIST_LINE_SEL",
    ///     "default": 0
    /// }
    /// </summary>
    public sealed class FirstLineVM : ViewModelBase
    {
        private JObject _cfg;

        // JSON 讀出的暫存器名稱 & 預設值
        private string _registerName = "BIST_LINE_SEL";
        private int _defaultValue = 0;

        /// <summary>
        /// CheckBox 綁定：true = White, false = Black
        /// </summary>
        public bool Enable
        {
            get { return GetValue<bool>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// 寫暫存器用 Command（綁 Set）
        /// </summary>
        public ICommand ApplyCommand { get; private set; }

        /// <summary>
        /// 暫存器讀寫介面（外部注入）
        /// </summary>
        public PageBase.ICStructure.Comm.IRegisterReadWriteEx RegControl { get; set; }

        public FirstLineVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        /// <summary>
        /// 從 JSON 載入：
        /// 可以丟整個 pattern node 或只丟 firstLineSelControl block。
        /// </summary>
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            var root = _cfg;
            if (root == null) return;

            // 1) { "firstLineSelControl": {...} }
            // 2) { "register": "...", "default": 0 }
            var ctrl = root["firstLineSelControl"] as JObject ?? root;

            // 讀 register 名稱
            var regTok = ctrl["register"];
            if (regTok != null)
            {
                var s = regTok.ToString().Trim();
                if (!string.IsNullOrEmpty(s))
                    _registerName = s;
            }

            // 讀 default：0 = Black, 1 = White
            int def = 0;
            var defTok = ctrl["default"];
            if (defTok != null)
            {
                int tmp;
                if (int.TryParse(defTok.ToString(), out tmp))
                    def = tmp;
            }

            _defaultValue = def;
            Enable = (_defaultValue != 0);

            System.Diagnostics.Debug.WriteLine(
                $"[FirstLine LoadFrom] register={_registerName}, default={_defaultValue}, Enable={Enable}");
        }

        /// <summary>
        /// 按下 Set：true→1、false→0，寫入 BIST_LINE_SEL
        /// </summary>
        private void ExecuteWrite()
        {
            if (_cfg == null)
            {
                MessageBox.Show("FirstLineVM: JSON 尚未載入。");
                return;
            }

            if (RegControl == null)
            {
                MessageBox.Show("FirstLineVM: RegControl 尚未注入。");
                return;
            }

            int val = Enable ? 1 : 0;

            System.Diagnostics.Debug.WriteLine(
                $"[FirstLine Apply] Enable={Enable}, val={val}, reg={_registerName}");

            RegControl.SetRegisterByName(_registerName, (uint)val);
        }
    }
}

=======

<GroupBox Header="First Line Color"
          Margin="0,8,0,0"
          Visibility="{Binding IsFirstLineVisible,
                               Converter={utilityConv:BoolToVisibilityConverter}}">
    <StackPanel Margin="12" Orientation="Vertical">

        <CheckBox Content="First Line Color = White (unchecked = Black)"
                  IsChecked="{Binding FirstLineVM.Enable, Mode=TwoWay}" />

        <Button Content="Set"
                Width="80"
                Height="30"
                Margin="0,8,0,0"
                HorizontalAlignment="Left"
                Command="{Binding FirstLineVM.ApplyCommand}" />
    </StackPanel>
</GroupBox>
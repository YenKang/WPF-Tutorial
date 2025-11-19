using Newtonsoft.Json.Linq;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class ExclamVM : ViewModelBase
    {
        private JObject _cfg;

        // 綁定 UI（勾 = White BG = 1）
        public bool Enable
        {
            get => GetValue<bool>();
            set => SetValue(value);
        }

        // 套用
        public ICommand ApplyCommand { get; }

        public ExclamVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        /// <summary>
        /// 從 JSON 載入 ExclamControl 節點
        /// {
        ///    "register": "BIST_EXCLAM_BG",
        ///    "default": 0
        /// }
        /// </summary>
        public void LoadFrom(object jsonCfg)
        {
            if (jsonCfg is JObject jo)
                _cfg = jo;
            else if (jsonCfg != null)
                _cfg = JObject.FromObject(jsonCfg);
            else
                return;

            int def = (int?)_cfg["default"] ?? 0;
            Enable = def != 0;

            System.Diagnostics.Debug.WriteLine($"[Exclam] Load: default={def}, Enable={Enable}");
        }

        /// <summary>
        /// 寫入暫存器
        /// </summary>
        private void ApplyWrite()
        {
            if (_cfg == null) return;

            string reg = (string)_cfg["register"];
            if (string.IsNullOrWhiteSpace(reg)) return;

            byte value = (byte)(Enable ? 1 : 0);
            RegControl.SetRegisterByName(reg, value);

            System.Diagnostics.Debug.WriteLine($"[Exclam] Write {reg} = {value}");
        }
    }
}

＝＝＝＝＝＝

<GroupBox Header="Exclamation Background"
          Margin="0,8,0,0"
          Visibility="{Binding IsExclamVisible, Converter={utilityConv:BoolToVisibilityConverter}}">
    <StackPanel Margin="12" Orientation="Vertical">

        <!-- 勾選 = White BG = 1 -->
        <CheckBox Content="Exclamation BG = White (unchecked = Black)"
                  IsChecked="{Binding ExclamVM.Enable, Mode=TwoWay}" />

        <Button Content="Set"
                Width="80"
                Height="30"
                Margin="0,8,0,0"
                Command="{Binding ExclamVM.ApplyCommand}" />
    </StackPanel>
</GroupBox>
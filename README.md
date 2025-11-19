using Newtonsoft.Json.Linq;
using PageBase.ICStructure.Comm;
using System;
using System.Diagnostics;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayReverseVM : ViewModelBase
    {
        private JObject _cfg;              // 記住 grayReverseControl JSON block
        private string _registerName = "BIST_HGRAY_REV"; // 預設，實際由 JSON 覆蓋

        // Checkbox 綁定的值（true = 啟用）
        public bool Enable
        {
            get { return GetValue<bool>(); }
            set { SetValue(value); }
        }

        public string Axis
        {
            get { return GetValue<string>(); }
            set
            {
                SetValue(value);
                Title = Axis == "V"
                    ? "Vertical Gray Reverse Enable"
                    : "Horizontal Gray Reverse Enable";
            }
        }

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// 對應 RegControl.SetRegisterByName 用的介面
        /// </summary>
        public IRegisterReadWriteEx RegControl { get; set; }

        public ICommand ApplyCommand { get; private set; }

        public GrayReverseVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);

            // 若尚未載入 JSON，先給預設值
            Axis = "H";
            Enable = false;
        }

        /// <summary>
        /// 從 grayReverseControl JSON 讀取 axis / default / register
        /// （呼叫端傳進來的 jsonCfg 應該就是 grayReverseControl 這個物件）
        /// </summary>
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null)
                return;

            // axis: "H" / "V"
            var axisToken = _cfg["axis"];
            var axis = axisToken != null
                ? axisToken.ToString().Trim().ToUpperInvariant()
                : "H";
            Axis = (axis == "V") ? "V" : "H";

            // default: 0 / 1
            int def = 0;
            var defTok = _cfg["default"];
            if (defTok != null)
            {
                int tmp;
                if (int.TryParse(defTok.ToString(), out tmp))
                    def = tmp;
            }
            Enable = (def != 0);

            // register: 直接指定要寫入的暫存器名稱
            var regTok = _cfg["register"];
            var regName = regTok != null ? regTok.ToString().Trim() : string.Empty;
            if (!string.IsNullOrEmpty(regName))
                _registerName = regName;
            else
                _registerName = "BIST_HGRAY_REV"; // 保底

            Debug.WriteLine(
                $"[GrayReverse LoadFrom] axis={Axis}, default={def}, register={_registerName}");
        }

        /// <summary>
        /// 將 Enable(0/1) 寫入 JSON 指定的 register
        /// 不再使用 RegMap / RMW / writes[]。
        /// </summary>
        private void ExecuteWrite()
        {
            if (_cfg == null)
            {
                MessageBox.Show("GrayReverse JSON 尚未載入。");
                return;
            }

            if (RegControl == null)
            {
                MessageBox.Show("RegControl 尚未注入。");
                return;
            }

            uint val = Enable ? 1u : 0u;

            Debug.WriteLine(
                $"[GrayReverse ExecuteWrite] axis={Axis}, enable={Enable}, register={_registerName}, val={val}");

            // 新版：直接透過 RegControl 寫入
            RegControl.SetRegisterByName(_registerName, val);
        }
    }
}
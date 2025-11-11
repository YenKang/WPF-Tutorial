using PageBase;

namespace BistMode
{
    public partial class BistModeView : NovaPageBase
    {
        // 內部持有 ViewModel
        private BistModeViewModel _vm;

        public BistModeView()
        {
            InitializeComponent();
        }

        public override bool InitializePage()
        {
            _vm = new BistModeViewModel(MainParameters);
            DataContext = _vm;
            return true;
        }

        // 🔹 對 IronPython 暴露的橋接方法
        public void SetRegDisplay(object reg)
        {
            _vm?.SetRegDisplay(reg);
        }
    }
}
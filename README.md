using System.Collections.Generic;
using System.Windows;
using OSDIconFlashMap.Model;
using OSDIconFlashMap.ViewModel;

namespace OSDIconFlashMap.View
{
    public partial class IconFlashMapWindow : Window
    {
        private readonly IconFlashMapViewModel _vm;

        public IconFlashMapWindow()
        {
            InitializeComponent();

            // 初始化 ViewModel
            _vm = new IconFlashMapViewModel();

            // 掛上資料上下文
            this.DataContext = _vm;
        }

        /// <summary>
        /// 外部可呼叫此方法，把舊畫面的 icon 列表 (IconModel) 傳進來。
        /// </summary>
        public void LoadCatalog(IEnumerable<IconModel> icons)
        {
            _vm.LoadCatalog(icons);
        }
    }
}
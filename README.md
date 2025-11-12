using OSDIconFlashMap.ViewModel;

public partial class IconToImageMapWindow : Window
{
    public IconToImageMapWindow()
    {
        InitializeComponent();

        // 若外部沒先塞，就自己建一個
        if (this.DataContext == null)
            this.DataContext = new IconToImageMapViewModel();

        var vm = (IconToImageMapViewModel)this.DataContext;
        vm.InitIconSlots();
        vm.LoadImagesFromFolder(@"Assets\TestIcons"); // 先用固定資料夾
    }
}
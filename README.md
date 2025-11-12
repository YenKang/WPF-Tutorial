public IconToImageMapWindow()
{
    try
    {
        InitializeComponent();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.ToString(), "XAML Load Error");
        throw;
    }

    if (DataContext == null) DataContext = new IconToImageMapViewModel();
    var vm = (IconToImageMapViewModel)DataContext;
    vm.InitIconSlots();
    vm.LoadImagesFromFolder(@"Assets\TestIcons");
}
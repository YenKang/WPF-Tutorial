public partial class App : Application
{
    public NovaCIDViewModel MainVM { get; private set; }

    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        MainVM = new NovaCIDViewModel();

        var mainWindow = new MainWindow();
        mainWindow.DataContext = MainVM;
        mainWindow.Show();
    }
}
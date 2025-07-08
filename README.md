public DrivePage()
{
    InitializeComponent();

    _viewModel = new DrivePageViewModel();
    this.DataContext = _viewModel;

    // 從設定檔讀取
    double x = Properties.Settings.Default.LeftKnobX;
    double y = Properties.Settings.Default.LeftKnobY;

    // 更新 ViewModel
    _viewModel.LeftKnobX = x;
    _viewModel.LeftKnobY = y;

    // 更新 Canvas Knob 位置
    double radius = LeftKnobEllipse.Width / 2;
    Canvas.SetLeft(LeftKnobEllipse, x - radius);
    Canvas.SetTop(LeftKnobEllipse, y - radius);
}
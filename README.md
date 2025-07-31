NovaCIDViewModel
│
├─ Raw data 進來 → FingerEventProcessor.Parse() → List<FingerStatus>
│
├─ IFingerEventRouter.Route(List<FingerStatus>)
│
└─ (根據 CurrentPage 轉發給實作了 IFingerHandler 的 Page ViewModel)
         └── DrivePageViewModel.OnFingerTouch()
         └── MusicPageViewModel.OnFingerTouch()


private void DrivePage_Loaded(object sender, RoutedEventArgs e)
{
    // 建立 ViewModel，傳入 Zoom 按鈕元件
    var vm = new DrivePageViewModel(App.Current.MainVM);
    vm.SetZoomButtons(ZoomInButton, ZoomOutButton); // 你剛剛做的 public 方法
    this.DataContext = vm;
}
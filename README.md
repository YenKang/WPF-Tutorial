NovaCIDViewModel
│
├─ Raw data 進來 → FingerEventProcessor.Parse() → List<FingerStatus>
│
├─ IFingerEventRouter.Route(List<FingerStatus>)
│
└─ (根據 CurrentPage 轉發給實作了 IFingerHandler 的 Page ViewModel)
         └── DrivePageViewModel.OnFingerTouch()
         └── MusicPageViewModel.OnFingerTouch()



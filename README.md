using Utility.MVVM;

namespace OSDIconFlashMap.Model
{
    public class IconSlotModel : ViewModelBase
    {
        // 1) Icon #N（唯讀顯示：建立時填 1..30）
        private int _iconIndex;
        public int IconIndex { get { return _iconIndex; } set { _iconIndex = value; RaisePropertyChanged(nameof(IconIndex)); } }

        // 2) Image Selection（選到的圖片）
        private ImageOption _selectedImage;
        public ImageOption SelectedImage
        {
            get { return _selectedImage; }
            set
            {
                if (Equals(_selectedImage, value)) return;
                _selectedImage = value;
                RaisePropertyChanged(nameof(SelectedImage));
                RaisePropertyChanged(nameof(SelectedImageName));
                RecomputeSramStartAddress(); // 選圖變更 → 重算 SRAM
            }
        }
        public string SelectedImageName { get { return _selectedImage == null ? "未選擇" : _selectedImage.Name; } }

        // 3) SRAM Start Address（工具計算 → 唯讀）
        private string _sramStartAddress = "-";
        public string SramStartAddress { get { return _sramStartAddress; } private set { _sramStartAddress = value; RaisePropertyChanged(nameof(SramStartAddress)); } }

        // 4) OSD Selection（1..30；0=未選）
        private int _osdTargetIndex;
        public int OsdTargetIndex { get { return _osdTargetIndex; } set { if (_osdTargetIndex != value) { _osdTargetIndex = value; RaisePropertyChanged(nameof(OsdTargetIndex)); } } }

        // 5) OSDx_EN（預設 false；之後你可跟 OsdTargetIndex 連動）
        private bool _isOsdEnabled;
        public bool IsOsdEnabled { get { return _isOsdEnabled; } set { if (_isOsdEnabled != value) { _isOsdEnabled = value; RaisePropertyChanged(nameof(IsOsdEnabled)); } } }

        // 6) HPos（先不嚴格卡，之後補 ValidationRule）
        private int _hPos = 1;
        public int HPos { get { return _hPos; } set { _hPos = value; RaisePropertyChanged(nameof(HPos)); } }

        // 7) VPos（先不嚴格卡）
        private int _vPos = 1;
        public int VPos { get { return _vPos; } set { _vPos = value; RaisePropertyChanged(nameof(VPos)); } }

        // 由 VM 注入的計算器（易於替換公式 / 因 IC 不同）
        public System.Func<IconSlotModel, string> CalcSramStartAddress { get; set; }
        private void RecomputeSramStartAddress()
        {
            SramStartAddress = (CalcSramStartAddress == null || SelectedImage == null) ? "-" : CalcSramStartAddress(this);
        }
    }
}
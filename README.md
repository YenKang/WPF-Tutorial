using Utility.MVVM;

namespace OSDIconFlashMap.Model
{
    // 一列 Icon 與目前選到的圖片
    public class IconSlotModel : ViewModelBase
    {
        public int IconIndex { get; set; } // 1..30

        private ImageOption _selectedImage;
        public ImageOption SelectedImage
        {
            get => _selectedImage;
            set
            {
                if (Set(ref _selectedImage, value))
                    RaisePropertyChanged(nameof(SelectedImageName));
            }
        }

        public string SelectedImageName => SelectedImage?.Name ?? "(未選擇)";
    }
}
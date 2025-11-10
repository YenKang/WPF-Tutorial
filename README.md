public class IconSlotModel : ViewModelBase
{
    /// <summary>
    /// Icon 序號 (1~30)
    /// </summary>
    public int IconIndex { get; set; }

    private ImageOption _selectedImage;

    /// <summary>
    /// 使用者目前選取的圖片
    /// </summary>
    public ImageOption SelectedImage
    {
        get { return _selectedImage; }
        set
        {
            // 1️⃣ 若值相同，不做多餘動作
            if (Equals(_selectedImage, value))
                return;

            // 2️⃣ 更新 backing field
            _selectedImage = value;

            // 3️⃣ 通知屬性改變
            RaisePropertyChanged(nameof(SelectedImage));

            // 4️⃣ 讓 UI 知道依附顯示文字也要重繪
            RaisePropertyChanged(nameof(SelectedImageName));
        }
    }

    /// <summary>
    /// 顯示用名稱（綁定在表格按鈕上）
    /// </summary>
    public string SelectedImageName
    {
        get
        {
            if (_selectedImage == null)
                return "(未選擇)";
            return _selectedImage.Name;
        }
    }
}
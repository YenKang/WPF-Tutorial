public ImageOption SelectedImage
{
    get { return GetValue<ImageOption>(); }
    set
    {
        // 1) 寫入本體
        SetValue(value);

        // 2) 同步顯示用文字（你原本就有）
        SelectedImageName = (value == null) ? "未選擇" : value.Name;

        // 3) ICON_DL_SEL summary 用
        IconDlSel = (value != null);

        // 4) 通知 ViewModel：圖片選擇有變，請重新算 SRAM
        if (OwnerViewModel != null)
        {
            OwnerViewModel.RecalculateSramAddressesByImageSize();
        }
    }
}
private void OnIconSlotPropertyChanged(object sender, PropertyChangedEventArgs e)
{
    // 1）Image Selection 變化 → 重算 SRAM
    if (e.PropertyName == nameof(IconSlotModel.SelectedImage))
    {
        RecalculateSramAddressesByImageSize();
    }

    // 2）OSD Icon Selection 變化 → 重建 OSDICButtonList
    if (e.PropertyName == nameof(IconSlotModel.OsdSelectedImage))
    {
        RebuildOsdIcButtonList();
    }
}
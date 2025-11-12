// 2) Image Selection（選到的圖片）
public ImageOption SelectedImage
{
    get { return GetValue<ImageOption>(); }
    set
    {
        var old = GetValue<ImageOption>();
        if (Equals(old, value)) return;

        // 1) 寫入本體（由 ViewModelBase.SetValue 觸發 PropertyChanged）
        SetValue(value);

        // 2) 同步對應顯示文字（改成「儲存欄位」而非即時計算）
        SelectedImageName = (value == null) ? "未選擇" : value.Name;

        // 3) 重新計算 SRAM
        RecomputeSramStartAddress();
    }
}

// 2') 給 DataGrid 顯示的名稱（改成「儲存屬性」風格，跟 SramStartAddress 一樣）
public string SelectedImageName
{
    get { return GetValue<string>() ?? "未選擇"; }
    private set { SetValue(value); }   // 建議設為 private，外界不要直接改
}
using System.ComponentModel;

// 4️⃣ 把 IconSlots 中每一列的 PropertyChanged 都掛到同一個 handler
public void HookSlotEvents()
{
    foreach (var slot in IconSlots)
    {
        // 先拔掉，避免重複掛
        slot.PropertyChanged -= OnIconSlotPropertyChanged;
        slot.PropertyChanged += OnIconSlotPropertyChanged;
    }
}

// 5️⃣ 只要有列的 SelectedImage 改變，就觸發重算 SRAM
private void OnIconSlotPropertyChanged(object sender, PropertyChangedEventArgs e)
{
    if (e.PropertyName == nameof(IconSlotModel.SelectedImage))
    {
        // 任何一列的圖片被改 → 全表 SRAM 重新計算
        RecalculateSramAddressesByImageSize();
    }
}
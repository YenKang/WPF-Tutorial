public void InitIconSlots()
{
    IconSlots.Clear();

    for (int i = 1; i <= 30; i++)
    {
        // 1) 先 new 一個 slot，把原本初始化的欄位都塞好
        var slot = new IconSlotModel
        {
            IconIndex          = i,
            SelectedImage      = null,  // 未選
            SramStartAddress   = "-",   // 初始全部顯示 "-"
            OsdIndex           = i,
            IsOsdEnabled       = false, // 未啟用
            IsOsdSramCrcEnabled= false, // 未啟用
            IsTtaleEnabled     = false, // 未啟用
            HPos               = 1,
            VPos               = 1,
        };

        // 2) ★關鍵：讓這個 slot 知道「它的 ViewModel 是誰」
        //    之後 SelectedImage 被改時，就能呼叫
        //    OwnerViewModel.RecalculateSramAddressesByImageSize()
        slot.OwnerViewModel = this;

        // 3) 加入集合裡
        IconSlots.Add(slot);
    }

    // 4) 通知 UI：IconSlots 內容已更新
    RaisePropertyChanged(nameof(IconSlots));
}
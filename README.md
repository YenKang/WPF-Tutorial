public void OpenAsilIconPicker(RegAsciiSlotModel slot, Window owner)
{
    if (slot == null || owner == null)
        return;

    // 1️⃣ 先從 OSDICButtonList 抓出「有選圖」的 OSD，照 OSDIndex 排序
    var osdButtons = OSDICButtonList
        .Where(b => b.OsdSelectedImage != null)
        .OrderBy(b => b.OsdIndex)
        .ToList();

    Debug.WriteLine($"[ASIL] 建立圖片牆：有效 OSD 數量 = {osdButtons.Count}");

    if (osdButtons.Count == 0)
    {
        MessageBox.Show(owner,
            "目前沒有任何 OSD 已選圖，請先在 Main 頁面為 OSD 選圖。",
            "提示",
            MessageBoxButton.OK,
            MessageBoxImage.Information);
        return;
    }

    // 2️⃣ 把 OSD 的圖片取出，變成圖片牆用的 candidates
    var candidates = osdButtons
        .Select(b => b.OsdSelectedImage)
        .ToList();

    Debug.WriteLine("[ASIL] 候選圖片列表：");
    for (int i = 0; i < candidates.Count; i++)
    {
        var img = candidates[i];
        var btn = osdButtons[i];
        var name = img?.Name ?? "(null)";
        Debug.WriteLine($"  idx={i}, OSD#{btn.OsdIndex}, IconName={name}");
    }

    // 3️⃣ 預設選中圖（可以是 null）
    string preKey = slot.SelectedImage?.Name;

    var picker = new ImagePickerWindow(
        candidates,     // 圖片清單
        preKey,         // 之前選過的圖名（可 null）
        null            // ASIL 不用 usedKeys
    );
    picker.Owner = owner;

    // 4️⃣ 開啟圖片牆，讓 ASIL 選圖
    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        // 4-1. ASIL 這一格 記住 選到的這張圖
        slot.SelectedImage = picker.Selected;

        // 4-2. 在 candidates 裡面找「這張圖是第幾個」
        int idx = candidates.IndexOf(picker.Selected);

        if (idx >= 0 && idx < osdButtons.Count)
        {
            var osdBtn = osdButtons[idx];

            // 4-3. 用同一個索引，在 osdButtons 找出對應的 OSD#
            slot.SelectedOsdIndex = osdBtn.OsdIndex;

            Debug.WriteLine(
                $"[ASIL] Reg={slot.RegName}, " +
                $"選到 idx={idx}, 圖={picker.Selected.Name}, 對應 OSD#{osdBtn.OsdIndex}");
        }
        else
        {
            // 理論上不該發生，如果發生就留 log
            slot.SelectedOsdIndex = 0;
            Debug.WriteLine(
                $"[ASIL][WARN] Reg={slot.RegName}, 圖={picker.Selected.Name}, " +
                $"在 candidates 裡找不到 IndexOf (idx={idx})");
        }
    }
    else
    {
        Debug.WriteLine($"[ASIL] Reg={slot.RegName}, 取消選圖或沒有選任何圖片。");
    }
}

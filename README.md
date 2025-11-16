using System.Diagnostics;
using System.Linq;

// 3️⃣ 重新計算所有列的 SRAM Start Address
public void RecalculateSramAddressesByImageSize()
{
    // 1. 先把全部列的 SramStartAddress 清成 "-"
    foreach (var slot in IconSlots)
    {
        slot.SramStartAddress = "-";
    }

    // 2. 把「有選圖片」的列抓出來，照 IconIndex 排序
    var selectedSlots = IconSlots
        .Where(s => s.SelectedImage != null)
        .OrderBy(s => s.IconIndex)
        .ToList();

    // 3. SRAM 位址從 0 開始累加
    uint currentAddr = 0;

    foreach (var slot in selectedSlots)
    {
        // 3-1. 取出這列圖片的 key，例如 "image1"
        var img = slot.SelectedImage;
        var key = img.Name;     // 或 img.Key，看你 UI 綁哪一個

        // 3-2. 從 _imageSizeMap 找出這張圖的 byte size
        uint size;
        if (!_imageSizeMap.TryGetValue(key, out size))
        {
            // 找不到就當 0，避免程式當掉
            size = 0;
        }

        // 3-3. 把目前累積位址填回這一列
        slot.SramStartAddress = string.Format("0x{0:X6}", currentAddr);

        // 3-4. 位址往後推：加上這張圖的 SRAM byte size
        currentAddr += size;
    }

    // Debug 用：把整張表 dump 出來
    Debug.WriteLine("==== SRAM MAP DUMP ====");
    foreach (var slot in IconSlots)
    {
        var imgName = slot.SelectedImage == null
            ? "未選擇"
            : slot.SelectedImage.Name;

        Debug.WriteLine(
            string.Format(
                "Icon#{0:00} | Img = {1} | SRAM = {2}",
                slot.IconIndex,
                imgName,
                slot.SramStartAddress));
    }
    Debug.WriteLine("==== END SRAM MAP ====");
}
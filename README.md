## Step 1：新增一個資料類別 IconImportItem.cs

```
public sealed class IconImportItem
{
    public int IconIndex { get; private set; }
    public string FlashImageName { get; private set; }
    public int SramStartAddress { get; private set; }

    public IconImportItem(int iconIndex, string flashImageName, int sramStartAddress)
    {
        IconIndex = iconIndex;
        FlashImageName = flashImageName ?? string.Empty;
        SramStartAddress = sramStartAddress;
    }
}
```


## Step 2：新增 Importer IconFlashMapJsonImporter.cs
- 放在 Parsers 或 Services 資料夾：

```
using System.Collections.Generic;
using Newtonsoft.Json.Linq;

public static class IconFlashMapJsonImporter
{
    public static List<IconImportItem> ImportIcons(string json)
    {
        if (string.IsNullOrWhiteSpace(json))
            return new List<IconImportItem>();

        JObject root;
        try { root = JObject.Parse(json); }
        catch { return new List<IconImportItem>(); }

        // 找 Icons array（容錯 Icons/icons）
        var arr = (root["Icons"] ?? root["icons"]) as JArray;
        if (arr == null) return new List<IconImportItem>();

        var list = new List<IconImportItem>(arr.Count);

        for (int i = 0; i < arr.Count; i++)
        {
            var obj = arr[i] as JObject;
            if (obj == null) continue;

            int iconIndex = ReadInt(obj, "IconIndex", -1);
            string flashImageName = ReadString(obj, "FlashImageName", "");
            int sramStart = ReadInt(obj, "SramStartAddress", -1);

            if (iconIndex <= 0) continue;

            list.Add(new IconImportItem(iconIndex, flashImageName, sramStart));
        }

        return list;
    }

    private static int ReadInt(JObject o, string key, int def)
    {
        var t = o[key];
        if (t == null || t.Type == JTokenType.Null) return def;

        int v;
        return int.TryParse(t.ToString(), out v) ? v : def;
    }

    private static string ReadString(JObject o, string key, string def)
    {
        var t = o[key];
        if (t == null || t.Type == JTokenType.Null) return def;
        return t.ToString();
    }
}
```

## Step 3：在你的 ViewModel 新增一個 Import 方法

- 假設你有 ObservableCollection<MainSetSlotModel> Slots（IconIndex=1~30 那種），你照這樣寫：

```
public void ImportFromJson(string jsonText)
{
    var imported = IconFlashMapJsonImporter.ImportIcons(jsonText);

    // 1) 先把現有 slots 清掉（你要保留舊值就把這段拿掉）
    for (int i = 0; i < Slots.Count; i++)
    {
        Slots[i].SelectedImage = null;
        Slots[i].SramStartAddress = 0;
    }

    // 2) 套用 import 結果到對應 IconIndex 的 slot
    for (int i = 0; i < imported.Count; i++)
    {
        var item = imported[i];

        // 找到同 IconIndex 的 slot
        var slot = FindSlotByIconIndex(item.IconIndex);
        if (slot == null) continue;

        // 2-1) 用 FlashImageName 找回 ImageOption
        slot.SelectedImage = FindImageOptionByName(item.FlashImageName);

        // 2-2) SRAM 若你要「照 JSON 直接顯示」就寫回去
        slot.SramStartAddress = item.SramStartAddress;
    }

    // 3) 如果你現在的架構是「SRAM 應該重算」
    //    就不要用 JSON 的 sramStartAddress，改成呼叫你既有的重算函式
    // RecalculateSramAddressesByImageSize();
}

private MainSetSlotModel FindSlotByIconIndex(int iconIndex)
{
    for (int i = 0; i < Slots.Count; i++)
    {
        if (Slots[i].IconIndex == iconIndex)
            return Slots[i];
    }
    return null;
}

// 你要改成你實際的 Images 集合型別/欄位
private ImageOption FindImageOptionByName(string flashImageName)
{
    if (string.IsNullOrEmpty(flashImageName))
        return null;

    for (int i = 0; i < Images.Count; i++)
    {
        // 這裡的比較欄位請換成你 ImageOption 真正存的名字
        if (Images[i].DisplayName == flashImageName
            || Images[i].FileName == flashImageName
            || Images[i].Key == flashImageName)
        {
            return Images[i];
        }
    }
    return null;
}
```

## Step 4：把 Import 接到 UI（最簡單版本）
```
private void ImportJson_Click(object sender, RoutedEventArgs e)
{
    string jsonText = File.ReadAllText(path);
    ViewModel.ImportFromJson(jsonText);
}
```

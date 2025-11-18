## Step1: 在 ViewModel 上面，建立 RegisterItem.cs

```
public class RegisterItem
{
    public string RegName { get; set; }
    public int Value { get; set; }
}
```

## Step2:建立registerTable.cs

```
public class RegisterTable
{
    public List<RegisterItem> Items { get; private set; }

    public RegisterTable()
    {
        Items = new List<RegisterItem>();
    }

    public void Add(string regName, int value)
    {
        Items.Add(new RegisterItem
        {
            RegName = regName,
            Value = value
        });
    }
}
```

## Step3:確認資料來源 (OSDSlotList)

```
public class OsdSlotModel
{
    public int OsdIndex { get; set; }       // 1-based
    public int IconIndex { get; set; }      // 1-based

    public bool IsOsdEnabled { get; set; }
    public int OsdIconSel { get; set; }

    public int HPos { get; set; }
    public int VPos { get; set; }

    public int SramStartAddr { get; set; }
    public int FlashStartAddr { get; set; }

    public bool SramCrcEn { get; set; }
    public bool TtaleEn { get; set; }

    public ImageModel SelectedImage { get; set; }
}
```

## Step4:貼入viewModel


```
public RegisterTable CollectRegisterTable()
{
    var table = new RegisterTable();
    var slots = this.OsdSlotList;

    // ----------------------------------------------------
    // 1. ICON_DL_SEL[29:0]  → bitmap
    // ----------------------------------------------------
    int dlSelBitmap = 0;
    foreach (var slot in slots)
    {
        if (slot.SelectedImage != null)
            dlSelBitmap |= (1 << (slot.IconIndex - 1));   // 1-based → bit(iconIndex-1)
    }
    table.Add("ICON_DL_SEL", dlSelBitmap);

    // ----------------------------------------------------
    // 2. ICON#_FLH_ADD_STA
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"ICON{slot.IconIndex}_FLH_ADD_STA";
        table.Add(reg, slot.FlashStartAddr);
    }

    // ----------------------------------------------------
    // 3. ICON#_SRAM_ADDR
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"ICON{slot.IconIndex}_SRAM_ADDR";
        table.Add(reg, slot.SramStartAddr & 0x7FFF);
    }

    // ----------------------------------------------------
    // 4. OSD#_ICON_SEL[5:0]
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"OSD{slot.OsdIndex}_ICON_SEL";
        table.Add(reg, slot.OsdIconSel & 0x3F);
    }

    // ----------------------------------------------------
    // 5. OSD#_EN
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"OSD{slot.OsdIndex}_EN";
        table.Add(reg, slot.IsOsdEnabled ? 1 : 0);
    }

    // ----------------------------------------------------
    // 6. OSD#_SRAMCRC_EN
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"OSD{slot.OsdIndex}_SRAMCRC_EN";
        table.Add(reg, slot.SramCrcEn ? 1 : 0);
    }

    // ----------------------------------------------------
    // 7. TTALE#
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"TTALE{slot.OsdIndex}";
        table.Add(reg, slot.TtaleEn ? 1 : 0);
    }

    // ----------------------------------------------------
    // 8. OSD#_V_POS
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"OSD{slot.OsdIndex}_V_POS";
        table.Add(reg, slot.VPos & 0x1FFF);
    }

    // ----------------------------------------------------
    // 9. OSD#_H_POS
    // ----------------------------------------------------
    foreach (var slot in slots)
    {
        string reg = $"OSD{slot.OsdIndex}_H_POS";
        table.Add(reg, slot.HPos & 0x1FFF);
    }

    return table;
}
```

```
public void DebugRegisterTable(RegisterTable table)
{
    var sb = new StringBuilder();
    sb.AppendLine("===== Register Table Dump =====");

    foreach (var item in table.Items)
    {
        sb.AppendLine($"{item.RegName} = 0x{item.Value:X} ({item.Value})");
    }

    sb.AppendLine("===== End =====");

    System.Diagnostics.Debug.WriteLine(sb.ToString());
}
```

## Step6: 按鈕中呼叫

```
private void OnWriteRegisterTable()
{
    var table = CollectRegisterTable();
    DebugRegisterTable(table);  // <-- Debug 用
}
```

```
<StackPanel Grid.Row="2"
            Orientation="Horizontal"
            HorizontalAlignment="Right"
            Margin="0,8,0,0">

    <!-- ✅ 新增：寫入 Register Table -->
    <Button Content="寫入 Register Table"
            Margin="0,0,8,0"
            Padding="16,4"
            Command="{Binding WriteRegisterTableCommand}" />

    <Button Content="Export"
            Margin="0,0,8,0"
            Padding="16,4"
            Command="{Binding ExportCommand}" />

    <Button Content="Cancel"
            Padding="16,4"
            IsCancel="True" />
</StackPanel>
```
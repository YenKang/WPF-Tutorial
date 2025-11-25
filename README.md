// 假設你在 ASIL 選圖的那段
slot.SelectedImage = picker.Selected;

// 從 OSDICButtonList 反查對應 OSD#
var match = vm.OSDICButtonList
    .FirstOrDefault(b =>
        b.OsdSelectedImage != null &&
        b.OsdSelectedImage.Name == picker.Selected.Name);

if (match != null)
{
    slot.SelectedOsdIndex = match.OsdIndex;
}

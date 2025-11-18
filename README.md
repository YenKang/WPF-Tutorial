if (picker.ShowDialog() == true && picker.Selected != null)
{
    // OSD 這一列要顯示的圖
    slot.OsdSelectedImage = picker.Selected;

    // 這一格候選圖是從哪個 Icon# 來的
    var cand = picker.Selected as OsdIconCandidate;
    if (cand != null)
    {
        slot.OsdIconSel = cand.SourceIconIndex;   // 1,4,5...
    }
}
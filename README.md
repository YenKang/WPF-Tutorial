private void BtnSetGrayLevel_Click(object sender, RoutedEventArgs e)
{
    // 1️⃣ 載入 JSON Profile
    var root = BistMode.Services.JsonConfigUiRunner.LoadRoot(@"Assets/Profiles/NT51365.profile.json");

    // 2️⃣ 取得 Red pattern 內的設定
    var rows = BistMode.Services.JsonConfigUiRunner.GetRows(root, "Red");
    var enumerator = rows.GetEnumerator();
    enumerator.MoveNext();
    var targetRow = enumerator.Current;

    // 3️⃣ 從 UI 元件讀取值（假設三個 ComboBox）
    targetRow.D2 = comboD2.SelectedIndex;
    targetRow.D1 = comboD1.SelectedIndex;
    targetRow.D0 = comboD0.SelectedIndex;

    // 4️⃣ 寫入暫存器（不改舊 code）
    BistMode.Services.JsonConfigUiRunner.ExecuteRow(
        root,
        targetRow,
        // 讀取委派
        addr => RegDisplayControl.ReadRegData(0, addr),
        // 寫入委派
        (addr, value) => RegDisplayControl.WriteRegData(0, addr, value)
    );
}
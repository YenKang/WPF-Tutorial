// 只允許輸入 0–9 與 A–F
private void TextBox_PreviewTextInput(object sender, TextCompositionEventArgs e)
{
    e.Handled = !System.Text.RegularExpressions.Regex.IsMatch(e.Text, "^[0-9A-Fa-f]+$");
}
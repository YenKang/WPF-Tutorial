// === 給 UI TextBox 使用：三位 HEX，可手動輸入 ===
public string FCNT2_Text
{
    get
    {
        // 例如 0x1E → "01E"
        return FCNT2_Value.ToString("X3");
    }
    set
    {
        if (string.IsNullOrWhiteSpace(value))
            return;

        var s = value.Trim();

        // 嘗試用 16 進位解析
        if (!int.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out var parsed))
        {
            // 非法字串（例如 ZZZ）就忽略
            return;
        }

        // 限制在 min/max 範圍
        parsed = Clamp(parsed, _min, _max);

        // 用你原本的 helper，把值拆回 D2/D1/D0
        SetFromValue(parsed);

        // 通知 Text 也更新成標準格式（X3）
        OnPropertyChanged(nameof(FCNT2_Text));
    }
}
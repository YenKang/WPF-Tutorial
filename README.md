/// <summary>
/// 將十進位字串轉成 byte（0~255）
/// </summary>
private bool TryParseDecimalToByte(string? text, out byte value)
{
    value = 0;

    if (string.IsNullOrWhiteSpace(text))
        return false;

    // 確保是十進位，不支援 0x 前綴
    return byte.TryParse(text.Trim(), NumberStyles.Integer, CultureInfo.InvariantCulture, out value);
}

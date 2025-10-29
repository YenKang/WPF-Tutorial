private JObject _cfg;
private JObject _jChessH;
private JObject _jChessV;

public void LoadFrom(JObject node)
{
    try
    {
        bool cfgChanged = !ReferenceEquals(_cfg, node);
        _cfg = node;

        if (_cfg?["fields"] is not JArray fields)
            return;

        // === 1️⃣ 若 JSON 結構改變才重建 options/default ===
        if (cfgChanged)
        {
            foreach (JObject f in fields.Cast<JObject>())
            {
                string key = (string)f["key"];
                int def = (int?)f["default"] ?? 0;
                int min = (int?)f["min"] ?? 0;
                int max = (int?)f["max"] ?? 0;

                switch (key)
                {
                    case "H_RES": HRes = def; break;
                    case "V_RES": VRes = def; break;
                    case "M": _mMin = min; _mMax = max; M = def; break;
                    case "N": _nMin = min; _nMax = max; N = def; break;
                }
            }

            // 重新建立 M / N ComboBox 選項
            BuildOptions();

            // 快取寄存器組節點，後續回讀會用
            _jChessH = _cfg?["chessH"] as JObject;
            _jChessV = _cfg?["chessV"] as JObject;
        }

        // === 2️⃣ 每次切圖都回讀暫存器，覆蓋顯示值 ===
        RefreshFromRegister();
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[ChessBoardVM] LoadFrom failed: " + ex.Message);
    }
}
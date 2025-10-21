using System;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;

/// <summary>
/// Auto-Run 設定 ViewModel：
/// - Total 決定要產生幾個「圖片順序」下拉
/// - Orders 逐一對應 BIST_PT_ORDx
/// - Trigger 在套用後啟動 Auto-Run
/// </summary>
public sealed class AutoRunVM : ViewModelBase
{
    // 常數
    private const int TotalMinDefault = 1;
    private const int TotalMaxDefault = 22;
    private const int TotalDefault    = 5;

    private const byte OrdValueMask   = 0x3F; // 每一格 ORD 只收 6 bits
    private const byte TotalValueMask = 0x1F; // TOTAL 只收 5 bits

    // 暫存器目標（由 JSON 載入）
    private string _totalTarget;                 // BIST_PT_TOTAL
    private string[] _orderTargets = new string[0]; // BIST_PT_ORD0..ORD21
    private string _triggerTarget;               // BIST_PT
    private byte _triggerValue;                  // e.g. 0x3F

    // 公開集合 (供 XAML 綁定)
    /// <summary>候選 pattern index（由外部 BistModeViewModel 填充）</summary>
    public ObservableCollection<int> PatternIndexOptions { get; } =
        new ObservableCollection<int>();

    /// <summary>可選總數 1..22（亦可從 JSON min/max 生成）</summary>
    public ObservableCollection<int> TotalOptions { get; } =
        new ObservableCollection<int>(Enumerable.Range(TotalMinDefault, TotalMaxDefault));

    /// <summary>依 Total 動態產生的「圖片順序」列（每列一個下拉）</summary>
    public ObservableCollection<OrderSlot> Orders { get; } =
        new ObservableCollection<OrderSlot>();

    // 可綁定屬性（使用 GetValue/SetValue）
    /// <summary>目前的圖片總數；改變時會同步調整 Orders 筆數與顯示序號</summary>
    public int Total
    {
        get { return GetValue<int>(); }
        set
        {
            if (!SetValue(value)) return;
            ResizeOrders(value);
        }
    }

    /// <summary>GroupBox 標題（預設 Auto Run）</summary>
    public string Title
    {
        get
        {
            var s = GetValue<string>();
            return string.IsNullOrEmpty(s) ? "Auto Run" : s;
        }
        private set { SetValue(string.IsNullOrEmpty(value) ? "Auto Run" : value); }
    }

    // 建構子
    public AutoRunVM()
    {
        // 直接指定屬性，讓 setter 觸發 ResizeOrders
        Total = TotalDefault;
    }

    /// <summary>由 Pattern JSON 讀取 autoRunControl 區塊。</summary>
    public void LoadFrom(JObject autoRunControl)
    {
        if (autoRunControl == null) return;

        Title = (string)autoRunControl["title"] ?? "Auto Run";

        // total
        var totalObj = autoRunControl["total"] as JObject;
        _totalTarget = (string)totalObj?["target"];
        var totalMin = totalObj?["min"]?.Value<int>() ?? TotalMinDefault;
        var totalMax = totalObj?["max"]?.Value<int>() ?? TotalMaxDefault;
        var totalDef = totalObj?["default"]?.Value<int>() ?? TotalDefault;

        Total = Clamp(totalDef, totalMin, totalMax);

        // orders
        var ordersObj = autoRunControl["orders"] as JObject;
        _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? new string[0];
        var defaults  = ordersObj?["defaults"]?.ToObject<int[]>() ?? Array.Empty<int>();

        Orders.Clear();
        for (int i = 0; i < Total; i++)
        {
            Orders.Add(new OrderSlot
            {
                DisplayNo     = i + 1,                        // 1-based 顯示
                SelectedIndex = (i < defaults.Length) ? defaults[i] : 0
            });
        }

        // trigger
        var trig = autoRunControl["trigger"] as JObject;
        _triggerTarget = (string)trig?["target"];
        _triggerValue  = ParseHexByte((string)trig?["value"], 0x3F);
    }

    /// <summary>將目前設定寫入暫存器並觸發 Auto-Run。</summary>
    public void Apply()
    {
        // Step 1: 寫入總數
        if (!string.IsNullOrEmpty(_totalTarget))
            RegMap.Write8(_totalTarget, (byte)(Total & TotalValueMask));

        // Step 2: 寫入各個 ORD
        var count = Math.Min(Total, _orderTargets.Length);
        for (int i = 0; i < count; i++)
        {
            var target = _orderTargets[i];
            if (string.IsNullOrEmpty(target)) continue;

            var code = (byte)(Orders[i].SelectedIndex & OrdValueMask);
            RegMap.Write8(target, code);
        }

        // Step 3: 觸發 Auto-Run
        if (!string.IsNullOrEmpty(_triggerTarget))
            RegMap.Write8(_triggerTarget, _triggerValue);
    }

    // 工具函式
    private void ResizeOrders(int desiredCount)
    {
        desiredCount = Clamp(desiredCount, TotalMinDefault, TotalMaxDefault);

        while (Orders.Count < desiredCount)
            Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

        while (Orders.Count > desiredCount)
            Orders.RemoveAt(Orders.Count - 1);

        for (int i = 0; i < Orders.Count; i++)
            Orders[i].DisplayNo = i + 1;
    }

    private static int Clamp(int v, int min, int max)
    {
        if (v < min) return min;
        if (v > max) return max;
        return v;
    }

    private static byte ParseHexByte(string hexOrNull, byte fallback)
    {
        if (string.IsNullOrWhiteSpace(hexOrNull)) return fallback;
        var s = hexOrNull.Trim();
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            s = s.Substring(2);
        byte b;
        return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out b) ? b : fallback;
    }

    /// <summary>一列「圖片順序」下拉所需資料。</summary>
    public sealed class OrderSlot : ViewModelBase
    {
        public int DisplayNo
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int SelectedIndex
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }
    }
}
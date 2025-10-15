using System.Collections.Generic;

namespace BistApp
{
    /// <summary>
    /// 對應 JSON 裡的 registers 區塊。
    /// 例如：
    /// "registers": {
    ///   "BIST_PT_LEVEL": { "addr": "0x0030", "width": 8 },
    ///   "BIST_CK_GY_FK_PT": { "addr": "0x0034", "width": 8 }
    /// }
    /// </summary>
    public sealed class RegDef
    {
        /// <summary>
        /// 暫存器位址字串，例如 "0x0030"
        /// </summary>
        public string addr { get; set; }

        /// <summary>
        /// 暫存器寬度 (通常為 8 或 16)
        /// </summary>
        public int width { get; set; }

        /// <summary>
        /// (可選) 人類閱讀用描述文字，例如 "GrayLevel[7:0]"
        /// </summary>
        public string desc { get; set; }
    }

    /// <summary>
    /// 對應 JSON 裡 row.fields 的每一個子項 (D2、D1、D0)
    /// 例如：
    /// "fields": {
    ///   "D2": { "min": 0, "max": 3, "default": 3 },
    ///   "D1": { "min": 0, "max": 15, "default": 15 },
    ///   "D0": { "min": 0, "max": 15, "default": 15 }
    /// }
    /// </summary>
    public sealed class FieldDef
    {
        /// <summary>
        /// 此欄位的最小值
        /// </summary>
        public int min { get; set; }

        /// <summary>
        /// 此欄位的最大值
        /// </summary>
        public int max { get; set; }

        /// <summary>
        /// 此欄位的預設值 (JSON key 為 "default")
        /// 在反序列化時自動對應，不需使用 @ 符號。
        /// </summary>
        public int defaultValue { get; set; }
    }

    /// <summary>
    /// 對應 JSON 裡 pattern.rows[] 的每一列設定。
    /// 例如：
    /// {
    ///   "id": "red.graylevel",
    ///   "title": "GrayLevel",
    ///   "fieldType": "UInt10Hex3",
    ///   "fields": {...},
    ///   "writes": [...]
    /// }
    /// </summary>
    public sealed class RowDef
    {
        /// <summary>
        /// 每列唯一識別 ID，例如 "red.graylevel"
        /// </summary>
        public string id { get; set; }

        /// <summary>
        /// 顯示在 UI 上的標題，例如 "GrayLevel"
        /// </summary>
        public string title { get; set; }

        /// <summary>
        /// 欄位類型，用來決定 UI 顯示模板，例如 "UInt10Hex3"、"Bool"、"Enum"
        /// </summary>
        public string fieldType { get; set; }

        /// <summary>
        /// 儲存欄位定義集合 (D2/D1/D0)
        /// </summary>
        public Dictionary<string, FieldDef> fields { get; set; }

        /// <summary>
        /// 寫入規則集合 (每個物件描述一次 register 寫入)
        /// JSON 內容會保留為原始 JToken，在送出時解析。
        /// </summary>
        public List<object> writes { get; set; }
    }

    /// <summary>
    /// 對應 JSON 裡 patterns[] 的每一個圖案。
    /// 例如：
    /// {
    ///   "name": "Red",
    ///   "index": 1,
    ///   "rows": [ {...} ]
    /// }
    /// </summary>
    public sealed class PatternDef
    {
        /// <summary>
        /// 圖案名稱 (例如 "Red")
        /// </summary>
        public string name { get; set; }

        /// <summary>
        /// 圖案索引值 (例如 1)
        /// </summary>
        public int index { get; set; }

        /// <summary>
        /// 該圖案所包含的多列設定
        /// </summary>
        public List<RowDef> rows { get; set; }
    }

    /// <summary>
    /// 對應 JSON 最外層物件。
    /// 例如：
    /// {
    ///   "chip": "NT51365",
    ///   "registers": {...},
    ///   "patterns": [...]
    /// }
    /// </summary>
    public sealed class ChipProfile
    {
        /// <summary>
        /// 晶片名稱 (例如 "NT51365")
        /// </summary>
        public string chip { get; set; }

        /// <summary>
        /// 暫存器定義集合 (key 為名稱，例如 "BIST_PT_LEVEL")
        /// </summary>
        public Dictionary<string, RegDef> registers { get; set; }

        /// <summary>
        /// 所有 pattern 的集合
        /// </summary>
        public List<PatternDef> patterns { get; set; }
    }
}
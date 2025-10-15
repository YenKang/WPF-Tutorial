using System;
using System.Collections.Generic;
using System.IO;
using Newtonsoft.Json.Linq;

namespace BistMode.Services
{
    /// <summary>
    /// 專門解析 JSON 中 configUIRaws 的執行器。
    /// 不修改舊 ProfileLoader，也不需要 IRegisterBus。
    /// 可直接使用委派 (Func/Action) 連接現有 RegDisplayControl。
    /// </summary>
    public static class JsonConfigUiRunner
    {
        /// <summary>
        /// 載入 JSON 檔並回傳 JObject。
        /// </summary>
        public static JObject LoadRoot(string path)
        {
            if (!File.Exists(path))
                throw new FileNotFoundException("Profile JSON not found.", path);

            string json = File.ReadAllText(path);
            return JObject.Parse(json);
        }

        /// <summary>
        /// 從指定 pattern 名稱讀取對應的 configUIRaws。
        /// </summary>
        public static IEnumerable<UiRow> GetRows(JObject root, string patternName)
        {
            var rows = new List<UiRow>();
            var patterns = root["patterns"] as JArray;
            if (patterns == null) return rows;

            foreach (JObject pattern in patterns)
            {
                // 找到指定 pattern（比對 name）
                if (!string.Equals((string)pattern["name"], patternName, StringComparison.OrdinalIgnoreCase))
                    continue;

                var rawList = pattern["configUIRaws"] as JArray;
                if (rawList == null) continue;

                foreach (JObject raw in rawList)
                {
                    var fields = raw["fields"] as JObject ?? new JObject();

                    rows.Add(new UiRow
                    {
                        Id = (string)raw["id"],
                        Title = (string)raw["title"],
                        D2 = (int?)(fields["D2"]?["default"]) ?? 0,
                        D1 = (int?)(fields["D1"]?["default"]) ?? 0,
                        D0 = (int?)(fields["D0"]?["default"]) ?? 0,
                        Raw = raw
                    });
                }
            }
            return rows;
        }

        /// <summary>
        /// 執行指定 row 內的 writes 規則。
        /// 不依賴 IRegisterBus，使用外部提供的 read8/write8 委派。
        /// </summary>
        public static void ExecuteRow(
            JObject root,
            UiRow row,
            Func<ushort, byte> read8,
            Action<ushort, byte> write8)
        {
            if (root == null || row == null || read8 == null || write8 == null)
                throw new ArgumentNullException("ExecuteRow 參數為 null。");

            var regs = root["registers"] as JObject;
            var writes = row.Raw["writes"] as JArray;
            if (regs == null || writes == null)
                return;

            // 組合 10-bit gray level
            int lowByte = ((row.D1 & 0x0F) << 4) | (row.D0 & 0x0F);
            int d2 = (row.D2 & 0x03);

            foreach (JObject w in writes)
            {
                string mode = (string)w["mode"];
                string target = (string)w["target"];
                if (string.IsNullOrEmpty(target)) continue;

                string addrStr = (string)regs[target];
                ushort addr = ToAddr(addrStr);

                if (string.Equals(mode, "write8", StringComparison.OrdinalIgnoreCase))
                {
                    // 直接寫入一個 byte
                    string src = (string)w["src"];
                    int value = (src == "LowByte") ? lowByte : (src == "D2" ? d2 : 0);
                    write8(addr, (byte)value);
                }
                else if (string.Equals(mode, "rmw", StringComparison.OrdinalIgnoreCase))
                {
                    // 讀-改-寫模式
                    int mask = ParseInt((string)w["mask"]);
                    int shift = (int)w["shift"];
                    string src = (string)w["src"];
                    int val = (src == "D2") ? d2 : 0;

                    byte cur = read8(addr);
                    byte next = (byte)((cur & ~mask) | ((val << shift) & mask));
                    write8(addr, next);
                }
            }
        }

        // ---------- 工具方法 ----------
        private static ushort ToAddr(string hex)
        {
            if (string.IsNullOrWhiteSpace(hex))
                throw new ArgumentException("Address is empty.");

            if (hex.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                hex = hex.Substring(2);

            return Convert.ToUInt16(hex, 16);
        }

        private static int ParseInt(string s)
        {
            if (string.IsNullOrWhiteSpace(s))
                throw new ArgumentException("ParseInt: empty string.");

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return Convert.ToInt32(s.Substring(2), 16);

            return int.Parse(s);
        }
    }

    /// <summary>
    /// 對應每一個 configUIRaw (例如 grayLevelControl)
    /// </summary>
    public sealed class UiRow
    {
        public string Id;
        public string Title;
        public int D2;
        public int D1;
        public int D0;
        internal JObject Raw;
    }
}
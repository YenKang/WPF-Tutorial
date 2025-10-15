using System;
using System.Collections.Generic;
using System.IO;
using Newtonsoft.Json.Linq;

namespace BistMode.Services
{
    /// <summary>
    /// 專門解析 JSON 中 configUIRaws 的執行器。
    /// 不改舊 ProfileLoader，可直接自動執行 write8 / rmw 寫暫存器。
    /// </summary>
    public static class JsonConfigUiRunner
    {
        public static JObject LoadRoot(string path)
        {
            if (!File.Exists(path)) throw new FileNotFoundException("profile json not found", path);
            return JObject.Parse(File.ReadAllText(path));
        }

        public static IEnumerable<UiRow> GetRows(JObject root, string patternName)
        {
            var rows = new List<UiRow>();
            var pats = root["patterns"] as JArray;
            if (pats == null) return rows;

            foreach (JObject p in pats)
            {
                if (!string.Equals((string)p["name"], patternName, StringComparison.OrdinalIgnoreCase))
                    continue;

                var rlist = p["configUIRaws"] as JArray;
                if (rlist == null) continue;

                foreach (JObject r in rlist)
                {
                    var f = r["fields"] as JObject ?? new JObject();
                    rows.Add(new UiRow
                    {
                        Id = (string)r["id"],
                        Title = (string)r["title"],
                        D2 = (int?)f?["D2"]?["default"] ?? 0,
                        D1 = (int?)f?["D1"]?["default"] ?? 0,
                        D0 = (int?)f?["D0"]?["default"] ?? 0,
                        Raw = r
                    });
                }
            }
            return rows;
        }

        public static void ExecuteRow(JObject root, UiRow row, IRegisterBus bus)
        {
            var regs = root["registers"] as JObject;
            var writes = row.Raw["writes"] as JArray;
            if (regs == null || writes == null) return;

            int lowByte = ((row.D1 & 0x0F) << 4) | (row.D0 & 0x0F);
            int d2 = (row.D2 & 0x03);

            foreach (JObject w in writes)
            {
                string mode = (string)w["mode"];
                string tgt = (string)w["target"];
                if (string.IsNullOrEmpty(tgt)) continue;

                string addrStr = (string)regs[tgt];
                ushort addr = ToAddr(addrStr);

                if (mode == "write8")
                {
                    string src = (string)w["src"];
                    int value = (src == "LowByte") ? lowByte : (src == "D2" ? d2 : 0);
                    bus.Write8(addr, (byte)value);
                }
                else if (mode == "rmw")
                {
                    int mask = ParseInt((string)w["mask"]);
                    int shift = (int)w["shift"];
                    string src = (string)w["src"];
                    int val = (src == "D2") ? d2 : 0;

                    byte cur = bus.Read8(addr);
                    byte next = (byte)((cur & ~mask) | ((val << shift) & mask));
                    bus.Write8(addr, next);
                }
            }
        }

        private static ushort ToAddr(string hex)
        {
            if (hex.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                hex = hex.Substring(2);
            return Convert.ToUInt16(hex, 16);
        }

        private static int ParseInt(string s)
        {
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return Convert.ToInt32(s.Substring(2), 16);
            return int.Parse(s);
        }
    }

    public sealed class UiRow
    {
        public string Id;
        public string Title;
        public int D2, D1, D0;
        internal JObject Raw;
    }

    /// <summary>
    /// 這是你的暫存器匯流介面（可直接包 RegDisplayControl）
    /// </summary>
    public interface IRegisterBus
    {
        byte Read8(ushort addr);
        void Write8(ushort addr, byte value);
    }
}
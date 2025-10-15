using System;
using System.IO;
using Newtonsoft.Json;

namespace BistApp
{
    /// <summary>
    /// 讀取 JSON Profile 並提供常用轉換工具。
    /// </summary>
    public static class ProfileLoader
    {
        /// <summary>
        /// 從指定路徑載入 JSON Profile 檔案，反序列化成 ChipProfile。
        /// 例：Assets/Profiles/NT51365.profile.json
        /// </summary>
        public static ChipProfile LoadChipProfile(string path)
        {
            // 1) 基本檢查：檔案是否存在
            if (!File.Exists(path))
            {
                throw new FileNotFoundException("找不到指定的 JSON Profile 檔案。", path);
            }

            // 2) 讀取檔案字串
            string json = File.ReadAllText(path);

            // 3) 反序列化為 ChipProfile 物件
            ChipProfile profile = JsonConvert.DeserializeObject<ChipProfile>(json);

            // 4) 檢查反序列化結果
            if (profile == null)
            {
                throw new InvalidOperationException("ChipProfile 反序列化失敗，請確認 JSON 格式是否正確。");
            }
            if (profile.registers == null)
            {
                throw new InvalidOperationException("JSON 中缺少 'registers' 欄位。");
            }
            if (profile.patterns == null)
            {
                throw new InvalidOperationException("JSON 中缺少 'patterns' 欄位。");
            }

            return profile;
        }

        /// <summary>
        /// 將十六進位位址字串（"0x0030" 或 "0030"）轉成 16-bit 位址。
        /// 來源：registers.*.addr
        /// </summary>
        public static ushort ToAddr(string hex)
        {
            if (string.IsNullOrWhiteSpace(hex))
            {
                throw new ArgumentException("位址字串為空白或無效。", "hex");
            }

            if (hex.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                hex = hex.Substring(2);
            }

            return Convert.ToUInt16(hex, 16);
        }

        /// <summary>
        /// 將十六進位或十進位字串轉成 int。
        /// 來源：writes[].mask / writes[].shift / 其它數值
        /// 例："0x03"→3、"12"→12
        /// </summary>
        public static int ParseInt(string s)
        {
            if (s == null)
            {
                throw new ArgumentNullException("s", "輸入字串為 null。");
            }

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                string hexPart = s.Substring(2);
                return Convert.ToInt32(hexPart, 16);
            }
            else
            {
                return int.Parse(s);
            }
        }
    }
}
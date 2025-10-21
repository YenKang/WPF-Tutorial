{
  "patterns": [
    {
      "name": "Chessboard",
      "fields": [
        { "key": "H_RES", "default": 1920 },
        { "key": "V_RES", "default": 720 },
        { "key": "M", "default": 5, "min": 2, "max": 40 },
        { "key": "N", "default": 7, "min": 2, "max": 40 }
      ],

      "chessH": {
        "writes": [
          { "mode": "write8", "target": "BIST_CHESSBOARD_H_TOGGLE_W_L", "src": "lowByte" },
          { "mode": "rmw",    "target": "BIST_CHESSBOARD_H_TOGGLE_W_H", "mask": "0x1F", "shift": 0, "src": "highByte" },
          { "mode": "write8", "target": "BIST_CHESSBOARD_H_PLUS1_BLKNUM","src": "lowByte" }
        ]
      },

      "chessV": {
        "writes": [
          { "mode": "write8", "target": "BIST_CHESSBOARD_V_TOGGLE_W_L", "src": "lowByte" },
          { "mode": "rmw",    "target": "BIST_CHESSBOARD_V_TOGGLE_W_H", "mask": "0x0F", "shift": 0, "src": "highByte" },
          { "mode": "write8", "target": "BIST_CHESSBOARD_V_PLUS1_BLKNUM","src": "lowByte" }
        ]
      },

      "patternSelect": {
        "writes": [
          { "mode": "rmw", "target": "BIST_PT", "mask": "0x3F", "shift": 0, "src": "lowByte" }
        ]
      }
    }
  ]
}


==============

// ChessBoardVM.cs
// 完整版 — 沿用前人 JSON writes 風格 (mode/target/mask/shift/src)

using System;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        private JObject _cfg;

        // 由 JSON 帶入的動態上下限
        private int _mMin = 2, _mMax = 40;
        private int _nMin = 2, _nMax = 40;

        // JSON 片段快取（避免每次 Apply 再找）
        private JArray _writesH;      // chessH.writes
        private JArray _writesV;      // chessV.writes
        private JArray _writesPatSel; // patternSelect.writes

        private const byte PATTERN_INDEX_CHESSBOARD = 0x0A;

        // ===== 綁定屬性 =====
        public int HRes
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int VRes
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int M
        {
            get { return GetValue<int>(); }
            set
            {
                if (value < _mMin) value = _mMin;
                if (value > _mMax) value = _mMax;
                SetValue(value);
                RaiseComputed();
            }
        }

        public int N
        {
            get { return GetValue<int>(); }
            set
            {
                if (value < _nMin) value = _nMin;
                if (value > _nMax) value = _nMax;
                SetValue(value);
                RaiseComputed();
            }
        }

        // ===== 衍生/除錯屬性（可不綁 UI） =====
        public int ToggleH { get { return (M > 0) ? (HRes / M) : 0; } }
        public int ToggleV { get { return (N > 0) ? (VRes / N) : 0; } }
        public int RemainderH { get { return HRes - (ToggleH * M); } }
        public int RemainderV { get { return VRes - (ToggleV * N); } }
        public int H_Plus1 { get { return (RemainderH == 0) ? 0 : 1; } }
        public int V_Plus1 { get { return (RemainderV == 0) ? 0 : 1; } }

        public ICommand ApplyCommand { get; private set; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteApply);
            // 給 UI 預設，不會影響 JSON 載入後的值
            HRes = 1920; VRes = 720; M = 5; N = 7;
        }

        // ===== 從 JSON 載入此 Pattern 節點 =====
        public void LoadFrom(JObject patternNode)
        {
            _cfg = patternNode;

            // fields
            var fields = patternNode["fields"] as JArray;
            if (fields != null)
            {
                foreach (JObject f in fields)
                {
                    string key = (string)f["key"];
                    int def = (int?)f["default"] ?? 0;
                    int min = (int?)f["min"] ?? 0;
                    int max = (int?)f["max"] ?? 0;

                    switch (key)
                    {
                        case "H_RES": HRes = def; break;
                        case "V_RES": VRes = def; break;
                        case "M":
                            _mMin = (min > 0) ? min : _mMin;
                            _mMax = (max > 0) ? max : _mMax;
                            M = def;
                            break;
                        case "N":
                            _nMin = (min > 0) ? min : _nMin;
                            _nMax = (max > 0) ? max : _nMax;
                            N = def;
                            break;
                    }
                }
            }

            // writes 區塊
            _writesH      = (patternNode["chessH"]?["writes"] as JArray) ?? new JArray();
            _writesV      = (patternNode["chessV"]?["writes"] as JArray) ?? new JArray();
            _writesPatSel = (patternNode["patternSelect"]?["writes"] as JArray) ?? new JArray();

            RaiseComputed();
        }

        // ===== 主要邏輯：按下 Set =====
        private void ExecuteApply()
        {
            if (_cfg == null)
            {
                System.Windows.MessageBox.Show("❌ Chessboard 未載入 JSON。");
                return;
            }

            // === 1) H 計算與寫入 ===
            int th = ToggleH;
            byte h_low   = (byte)(th & 0xFF);
            byte h_high  = (byte)((th >> 8) & 0xFF);
            byte h_plus1 = (byte)((RemainderH == 0) ? 0 : 1);

            // 1-1) 只跑 H_TOGGLE 的兩筆（低/高），略過 PLUS1
            ExecuteWritesFiltered(_writesH, low: h_low, high: h_high,
                includeTarget: t => !string.Equals(t, "BIST_CHESSBOARD_H_PLUS1_BLKNUM", StringComparison.Ordinal));

            // 1-2) 單獨跑 H_PLUS1（lowByte=plus1）
            ExecuteWritesFiltered(_writesH, low: h_plus1, high: 0,
                includeTarget: t => string.Equals(t, "BIST_CHESSBOARD_H_PLUS1_BLKNUM", StringComparison.Ordinal));

            // === 2) V 計算與寫入 ===
            int tv = ToggleV;
            byte v_low   = (byte)(tv & 0xFF);
            byte v_high  = (byte)((tv >> 8) & 0xFF);
            byte v_plus1 = (byte)((RemainderV == 0) ? 0 : 1);

            // 2-1) 只跑 V_TOGGLE 的兩筆（低/高），略過 PLUS1
            ExecuteWritesFiltered(_writesV, low: v_low, high: v_high,
                includeTarget: t => !string.Equals(t, "BIST_CHESSBOARD_V_PLUS1_BLKNUM", StringComparison.Ordinal));

            // 2-2) 單獨跑 V_PLUS1（lowByte=plus1）
            ExecuteWritesFiltered(_writesV, low: v_plus1, high: 0,
                includeTarget: t => string.Equals(t, "BIST_CHESSBOARD_V_PLUS1_BLKNUM", StringComparison.Ordinal));

            // === 3) 切換 Pattern：BIST_PT[5:0] = 0x0A ===
            ExecuteWritesFiltered(_writesPatSel, low: PATTERN_INDEX_CHESSBOARD, high: 0,
                includeTarget: _ => true);

            System.Windows.MessageBox.Show("✅ Chessboard applied.");
        }

        // ===== 依「前人格式」執行一批 writes（可用目標名稱過濾） =====
        private void ExecuteWritesFiltered(JArray writes, byte low, byte high, Func<string, bool> includeTarget)
        {
            if (writes == null) return;

            foreach (JObject w in writes)
            {
                string mode   = (string)w["mode"] ?? "write8";
                string target = (string)w["target"];
                if (string.IsNullOrEmpty(target)) continue;
                if (includeTarget != null && !includeTarget(target)) continue;

                int srcVal = ResolveSrc((string)w["src"], 0, 0, 0, low, high);

                if (string.Equals(mode, "write8", StringComparison.OrdinalIgnoreCase))
                {
                    RegMap.Write8(target, (byte)(srcVal & 0xFF));
                }
                else if (string.Equals(mode, "rmw", StringComparison.OrdinalIgnoreCase))
                {
                    int mask  = ParseInt((string)w["mask"]);
                    int shift = (int?)(w["shift"]) ?? 0;

                    byte cur  = RegMap.Read8(target);
                    byte next = (byte)((cur & ~(mask & 0xFF)) | (((srcVal << shift) & mask) & 0xFF));
                    RegMap.Write8(target, next);
                }
            }
        }

        // ===== ResolveSrc：沿用你的風格，支援 lowByte / highByte / 常數 =====
        private static int ResolveSrc(string src, int d2, int d1, int d0, byte low, byte high)
        {
            if (string.IsNullOrEmpty(src)) return 0;

            switch (src)
            {
                case "D2": return d2;
                case "D1": return d1;
                case "D0": return d0;
                case "lowByte":  return low;
                case "highByte": return high;
                default:
                    {
                        // 常數，如 "0x10" or "15"
                        if (TryParseInt(src, out int v)) return v;
                        return 0;
                    }
            }
        }

        private static bool TryParseInt(string s, out int v)
        {
            v = 0;
            if (string.IsNullOrWhiteSpace(s)) return false;

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                return int.TryParse(s.Substring(2),
                    System.Globalization.NumberStyles.HexNumber, null, out v);
            }
            return int.TryParse(s, out v);
        }

        private static int ParseInt(string s)
        {
            if (TryParseInt(s, out int v)) return v;
            return 0;
        }

        // ===== 批次通知（若 UI 需要顯示推算值） =====
        private void RaiseComputed()
        {
            RaisePropertyChanged(nameof(ToggleH));
            RaisePropertyChanged(nameof(ToggleV));
            RaisePropertyChanged(nameof(RemainderH));
            RaisePropertyChanged(nameof(RemainderV));
            RaisePropertyChanged(nameof(H_Plus1));
            RaisePropertyChanged(nameof(V_Plus1));
        }
    }
}
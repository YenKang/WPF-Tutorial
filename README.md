using System;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    /// <summary>
    /// Exclamation（驚嘆號）背景色切換 + 預留 Cursor 的 ViewModel
    /// 策略：優先使用快取（切頁回來立即還原 UI），按下 Set 才寫入暫存器
    /// </summary>
    public sealed class LineExclamCursorVM : ViewModelBase
    {
        private JObject _cfg;                // JSON 節點（由外部注入）
        private bool _isHydrating;           // 裝載 UI 中（避免觸發回寫或 cache 更新）

        // --- Exclamation 背景 bit mapping（由 JSON 載入；若無則走 fallback） ---
        private string _bgTarget = "BIST_Line_ExclamBG_Cursor";  // 既有 register 名稱（你 JSON 已有）
        private byte   _bgMask   = 0x01;     // 只占 1bit（預設）
        private int    _bgShift  = 0;        // 落在 bit[0]
        private int    _bgDefault = 0;       // 0=黑底, 1=白底（可由 JSON 覆寫）

        // === UI 綁定屬性 ===
        /// <summary>Exclamation BG 是否為白底（unchecked = 黑底）</summary>
        public bool WhiteBg
        {
            get { return GetValue<bool>(); }
            set
            {
                if (!SetValue(value) || _isHydrating) return;
                // 只改 UI 與快取；按下 Set 才寫暫存器
                LineExclamCursorCache.SaveExclam(value);
            }
        }

        /// <summary>GroupBox 標題（可由 JSON 覆寫）</summary>
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>按鈕：寫入暫存器（遵守「按下 Set 才寫」的 UX）</summary>
        public ICommand ApplyCommand { get; }

        public LineExclamCursorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
            Title = "Exclamation Background";
        }

        // 由外部傳入 JSON 的對應節點（通常是 Pattern config 的某個 section）
        public void LoadFrom(JObject node)
        {
            if (node == null) return;

            _cfg = node;
            _isHydrating = true;
            try
            {
                // 1) Title（可選）
                Title = (string)_cfg["title"] ?? Title;

                // 2) 解析 JSON 映射（彈性命名：支援 "exclam", "exclamation" 任一節點）
                //    結構預期：
                //    "exclam": {
                //      "default": 0 or 1,
                //      "targets": { "bg": "BIST_Line_ExclamBG_Cursor" },
                //      "mask": "0x01",
                //      "shift": 0
                //    }
                JObject ex = null;
                if      (_cfg["exclam"]       is JObject j1) ex = j1;
                else if (_cfg["exclamation"]  is JObject j2) ex = j2;

                if (ex != null)
                {
                    // default
                    _bgDefault = ParseHexOrInt((string)ex["default"], 0);

                    // targets.bg
                    if (ex["targets"] is JObject tg)
                        _bgTarget = (string)tg["bg"] ?? _bgTarget;

                    // mask / shift
                    _bgMask  = ParseHexByte((string)ex["mask"],  0x01);
                    _bgShift = ParseHexOrInt((string)ex["shift"], 0);
                }

                // 3) 邏輯：若有快取 → 直接恢復 UI；否則吃 JSON default
                if (LineExclamCursorCache.HasMemory)
                {
                    WhiteBg = LineExclamCursorCache.WhiteBg;
                }
                else
                {
                    WhiteBg = (_bgDefault != 0);
                    // 第一次載入就把 default 存進快取，確保切頁回來不紅框
                    LineExclamCursorCache.SaveExclam(WhiteBg);
                }
            }
            finally
            {
                _isHydrating = false;
            }
        }

        /// <summary>
        /// 寫入暫存器：依 WhiteBg（true=1, false=0），對 _bgTarget 做 RMW
        /// </summary>
        private void ApplyWrite()
        {
            try
            {
                var bit = WhiteBg ? 1 : 0;
                byte data = (byte)((bit << _bgShift) & _bgMask);

                // 你的底層已有 RegMap.Rmw8(target, mask, value)；這裡套用
                RegMap.Rmw8(_bgTarget, _bgMask, data);

                // 成功後把狀態存入快取（確保切頁回來維持打勾）
                LineExclamCursorCache.SaveExclam(WhiteBg);

                System.Diagnostics.Debug.WriteLine(
                    $"[Exclam] Write OK: target={_bgTarget}, mask=0x{_bgMask:X2}, value=0x{data:X2}, WhiteBg={WhiteBg}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Exclam] ApplyWrite error: " + ex.Message);
            }
        }

        // ====== 小工具 ======

        private static byte ParseHexByte(string hexOrNull, byte fallback)
        {
            if (string.IsNullOrWhiteSpace(hexOrNull)) return fallback;
            var s = hexOrNull.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
            return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var b) ? b : fallback;
        }

        private static int ParseHexOrInt(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                s = s.Substring(2);
                return int.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var v) ? v : fallback;
            }
            return int.TryParse(s, out var iv) ? iv : fallback;
        }
    }
}
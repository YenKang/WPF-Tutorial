using BistMode.Services;
using Newtonsoft.Json.Linq;
using System;
using System.Globalization;
using System.Windows.Input;

namespace BistMode.ViewModels
{
    public sealed class LineExclamCursorVM : ViewModelBase
    {
        // === Register target ===
        private string _target = "BIST_Line_ExclamBG_Cursor";

        // === Bit masks ===
        private byte _maskLineSel = 0x20;
        private byte _maskExclamBg = 0x10;
        private byte _maskDiagonal = 0x08;
        private byte _maskCursor = 0x04;

        // === Public bindable properties ===
        public string Title { get => GetValue<string>(); private set => SetValue(value); }

        public bool ShowLineSel { get => GetValue<bool>(); private set => SetValue(value); }
        public bool ShowExclamBg { get => GetValue<bool>(); private set => SetValue(value); }
        public bool ShowDiagonal { get => GetValue<bool>(); private set => SetValue(value); }
        public bool ShowCursor { get => GetValue<bool>(); private set => SetValue(value); }

        public bool LineSelWhite { get => GetValue<bool>(); set => SetValue(value); }
        public bool ExclamWhiteBG { get => GetValue<bool>(); set => SetValue(value); }
        public bool DiagonalEn { get => GetValue<bool>(); set => SetValue(value); }
        public bool CursorEn { get => GetValue<bool>(); set => SetValue(value); }

        // === Cursor Position ===
        public bool ShowHPos { get => GetValue<bool>(); private set => SetValue(value); }
        public bool ShowVPos { get => GetValue<bool>(); private set => SetValue(value); }

        public int HPos { get => GetValue<int>(); set => SetValue(value); }
        public int VPos { get => GetValue<int>(); set => SetValue(value); }

        private int _hMin, _hMax, _vMin, _vMax;
        private string _hLowTarget, _hHighTarget, _vLowTarget, _vHighTarget;
        private byte _hMask, _vMask;

        public ICommand ApplyCommand { get; }

        // === Constructor ===
        public LineExclamCursorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
            Reset();
        }

        private void Reset()
        {
            Title = "Line / Exclam / Cursor / Diagonal";
            ShowLineSel = ShowExclamBg = ShowDiagonal = ShowCursor = false;

            LineSelWhite = ExclamWhiteBG = DiagonalEn = CursorEn = false;
            ShowHPos = ShowVPos = false;
            HPos = VPos = 0;
        }

        // === JSON loader ===
        public void LoadFrom(object json)
        {
            Reset();
            if (!(json is JObject jsonObj)) return;

            Title = (string)jsonObj["title"] ?? Title;
            _target = (string)jsonObj["target"] ?? _target;

            var fields = jsonObj["fields"] as JObject;
            if (fields == null) return;

            // ---- Visibility ----
            ShowLineSel  = fields["lineSel"]?["visible"]?.Value<bool>() ?? false;
            ShowExclamBg = fields["exclamBg"]?["visible"]?.Value<bool>() ?? false;
            ShowDiagonal = fields["diagonalEn"]?["visible"]?.Value<bool>() ?? false;
            ShowCursor   = fields["cursorEn"]?["visible"]?.Value<bool>() ?? false;

            // ---- Bit Masks ----
            _maskLineSel  = ReadMask(fields["lineSel"], _maskLineSel);
            _maskExclamBg = ReadMask(fields["exclamBg"], _maskExclamBg);
            _maskDiagonal = ReadMask(fields["diagonalEn"], _maskDiagonal);
            _maskCursor   = ReadMask(fields["cursorEn"], _maskCursor);

            // ---- Default States ----
            LineSelWhite  = ReadDefault01(fields["lineSel"]) == 1;
            ExclamWhiteBG = ReadDefault01(fields["exclamBg"]) == 1;
            DiagonalEn    = ReadDefault01(fields["diagonalEn"]) == 1;
            CursorEn      = ReadDefault01(fields["cursorEn"]) == 1;

            // ---- Cursor Position (optional) ----
            if (fields["hPos"] != null)
                ParsePositionField(fields["hPos"], out ShowHPos, out HPos,
                    out _hMin, out _hMax, out _hLowTarget, out _hHighTarget, out _hMask, 0x1F);

            if (fields["vPos"] != null)
                ParsePositionField(fields["vPos"], out ShowVPos, out VPos,
                    out _vMin, out _vMax, out _vLowTarget, out _vHighTarget, out _vMask, 0x0F);
        }

        // === Write register ===
        private void ApplyWrite()
        {
            byte current = RegMap.Read8(_target);
            byte maskAll = 0;
            byte bits = 0;

            if (ShowLineSel) maskAll |= _maskLineSel;
            if (ShowExclamBg) maskAll |= _maskExclamBg;
            if (ShowDiagonal) maskAll |= _maskDiagonal;
            if (ShowCursor) maskAll |= _maskCursor;

            if (ShowLineSel && LineSelWhite) bits |= _maskLineSel;
            if (ShowExclamBg && ExclamWhiteBG) bits |= _maskExclamBg;
            if (ShowDiagonal && DiagonalEn) bits |= _maskDiagonal;
            if (ShowCursor && CursorEn) bits |= _maskCursor;

            byte next = (byte)((current & ~maskAll) | bits);
            RegMap.Write8(_target, next);

            // --- Cursor position write ---
            if (ShowCursor)
            {
                WriteCursorPos(_hLowTarget, _hHighTarget, _hMask, HPos, _hMin, _hMax);
                WriteCursorPos(_vLowTarget, _vHighTarget, _vMask, VPos, _vMin, _vMax);
            }
        }

        private static void WriteCursorPos(string lowTarget, string highTarget, byte mask,
                                           int value, int min, int max)
        {
            if (string.IsNullOrEmpty(lowTarget)) return;
            int val = Math.Max(min, Math.Min(value, max));

            RegMap.Write8(lowTarget, (byte)(val & 0xFF));

            if (!string.IsNullOrEmpty(highTarget))
            {
                byte hi = (byte)((val >> 8) & mask);
                RegMap.Rmw(highTarget, mask, hi);
            }
        }

        // === JSON parsing helpers ===
        private static byte ReadMask(JToken field, byte fallback)
        {
            var str = field?["mask"]?.Value<string>();
            if (string.IsNullOrWhiteSpace(str)) return fallback;
            str = str.Trim();
            if (str.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) str = str[2..];
            return byte.TryParse(str, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out var b) ? b : fallback;
        }

        private static int ReadDefault01(JToken field)
        {
            var t = field?["default"];
            if (t == null) return 0;
            if (t.Type == JTokenType.Integer) return t.Value<int>() != 0 ? 1 : 0;
            if (t.Type == JTokenType.String && int.TryParse((string)t, out var v)) return v != 0 ? 1 : 0;
            return 0;
        }

        private static int ReadIntDefault(JToken field)
        {
            var t = field?["default"];
            if (t == null) return 0;
            if (t.Type == JTokenType.Integer) return t.Value<int>();
            if (t.Type == JTokenType.String && int.TryParse((string)t, out var v)) return v;
            return 0;
        }

        private static void ParsePositionField(JToken field, out bool visible, out int def,
                                               out int min, out int max,
                                               out string low, out string high, out byte mask, byte fallbackMask)
        {
            visible = field["visible"]?.Value<bool>() ?? false;
            def = ReadIntDefault(field);
            min = field["min"]?.Value<int>() ?? 0;
            max = field["max"]?.Value<int>() ?? 0;
            low = field["low"]?.Value<string>();
            high = field["high"]?.Value<string>();
            mask = ReadMask(field, fallbackMask);
        }
    }
}
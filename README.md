using System;
using System.Globalization;
using System.Windows.Input;
using Newtonsoft.Json.Linq;

public sealed class LineExclamCursorVM : ViewModelBase
{
    private string _target = "BIST_Line_ExclamBG_Cursor";
    private byte _mLine = 0x20, _mExclam = 0x10, _mDiag = 0x08, _mCursor = 0x04;

    public string Title { get => GetValue<string>(); private set => SetValue(value); }

    public bool ShowLineSel  { get => GetValue<bool>(); private set => SetValue(value); }
    public bool ShowExclamBg { get => GetValue<bool>(); private set => SetValue(value); }
    public bool ShowDiagonal { get => GetValue<bool>(); private set => SetValue(value); }
    public bool ShowCursor   { get => GetValue<bool>(); private set => SetValue(value); }

    public bool LineSelWhite  { get => GetValue<bool>(); set => SetValue(value); } // 1=White first
    public bool ExclamWhiteBG { get => GetValue<bool>(); set => SetValue(value); } // 1=White BG
    public bool DiagonalEn    { get => GetValue<bool>(); set => SetValue(value); } // 1=Enable
    public bool CursorEn      { get => GetValue<bool>(); set => SetValue(value); } // 1=Enable

    public ICommand ApplyCommand { get; }

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
    }

    public void LoadFrom(object json)
    {
        Reset();

        if (json is not JObject jo) return;

        Title   = (string)jo["title"]  ?? Title;
        _target = (string)jo["target"] ?? _target;

        var f = jo["fields"] as JObject; 
        if (f is null) return;

        ShowLineSel  = f["lineSel"]   ?["visible"]?.Value<bool?>() ?? false;
        ShowExclamBg = f["exclamBg"]  ?["visible"]?.Value<bool?>() ?? false;
        ShowDiagonal = f["diagonalEn"]?["visible"]?.Value<bool?>() ?? false;
        ShowCursor   = f["cursorEn"]  ?["visible"]?.Value<bool?>() ?? false;

        _mLine   = ReadMask(f["lineSel"],    _mLine);
        _mExclam = ReadMask(f["exclamBg"],   _mExclam);
        _mDiag   = ReadMask(f["diagonalEn"], _mDiag);
        _mCursor = ReadMask(f["cursorEn"],   _mCursor);

        LineSelWhite  = ReadDefault01(f["lineSel"])    == 1;
        ExclamWhiteBG = ReadDefault01(f["exclamBg"])   == 1;
        DiagonalEn    = ReadDefault01(f["diagonalEn"]) == 1;
        CursorEn      = ReadDefault01(f["cursorEn"])   == 1;
    }

    private static byte ReadMask(JToken field, byte fb)
    {
        var s = (string)field?["mask"];
        if (string.IsNullOrWhiteSpace(s)) return fb;
        s = s.Trim();
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
        return byte.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out var b) ? b : fb;
    }

    private static int ReadDefault01(JToken field)
    {
        var t = field?["default"];
        if (t == null) return 0;
        if (t.Type == JTokenType.Integer) return t.Value<int>() != 0 ? 1 : 0;
        if (t.Type == JTokenType.String && int.TryParse((string)t, out var v)) return v != 0 ? 1 : 0;
        return 0;
    }

    private void ApplyWrite()
    {
        byte cur = RegMap.Read8(_target);

        byte maskAll = 0;
        if (ShowLineSel)  maskAll |= _mLine;
        if (ShowExclamBg) maskAll |= _mExclam;
        if (ShowDiagonal) maskAll |= _mDiag;
        if (ShowCursor)   maskAll |= _mCursor;

        byte bits = 0;
        if (ShowLineSel  && LineSelWhite)  bits |= _mLine;   // 1=White first
        if (ShowExclamBg && ExclamWhiteBG) bits |= _mExclam; // 1=White BG
        if (ShowDiagonal && DiagonalEn)    bits |= _mDiag;
        if (ShowCursor   && CursorEn)      bits |= _mCursor;

        byte next = (byte)((cur & ~maskAll) | bits);
        RegMap.Write8(_target, next);
    }
}
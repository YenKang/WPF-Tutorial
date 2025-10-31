namespace BistMode.ViewModels
{
    /// <summary>
    /// Exclamation + Cursor 共用快取（保存 UI 狀態）
    /// </summary>
    public static class LineExclamCursorCache
    {
        // 是否有記憶資料（用於 LoadFrom 判斷）
        public static bool HasMemory { get; private set; }

        // ===== Exclamation 背景區 =====
        public static bool WhiteBg { get; private set; } // true=白底, false=黑底

        // ===== Cursor 區 (預留未來用) =====
        public static int CursorX { get; private set; }  // 0~1023
        public static int CursorY { get; private set; }  // 0~1023

        // ===== 儲存行為 =====
        public static void SaveExclam(bool whiteBg)
        {
            WhiteBg = whiteBg;
            HasMemory = true;
        }

        public static void SaveCursor(int x, int y)
        {
            CursorX = x;
            CursorY = y;
            HasMemory = true;
        }

        // 可選：一次性全存
        public static void SaveAll(bool whiteBg, int x, int y)
        {
            WhiteBg = whiteBg;
            CursorX = x;
            CursorY = y;
            HasMemory = true;
        }

        public static void Clear()
        {
            HasMemory = false;
            WhiteBg = false;
            CursorX = 0;
            CursorY = 0;
        }
    }
}
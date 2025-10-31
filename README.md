// C# 7.3 相容
namespace BistMode.ViewModels
{
    public static class LineExclamCache
    {
        public static bool HasMemory { get; private set; }
        public static bool WhiteBg { get; private set; } // true=白底, false=黑底

        public static void Save(bool whiteBg)
        {
            WhiteBg = whiteBg;
            HasMemory = true;
        }

        public static void Clear()
        {
            HasMemory = false;
            WhiteBg = false; // 回到預設：不勾(黑底)
        }
    }
}

＝＝＝＝＝

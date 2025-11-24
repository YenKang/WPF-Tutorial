_controlParsers = new Dictionary<string, IPatternControlParser>(StringComparer.OrdinalIgnoreCase)
{
    { "bwLoopControl",         new BwLoopControlParser() },
    { "exclamControl",         new ExclamControlParser() },
    { "firstLineSelControl",   new FirstLineSelControlParser() },
    { "grayReverseControl",    new GrayReverseControlParser() },
    { "bgGrayLevelControl",    new BgGrayLevelControlParser() },
    { "grayLevelControl",      new GrayLevelControlParser() },
    { "grayColorControl",      new GrayColorControlParser() },
    { "flickerRGBControl",     new FlickerRgbControlParser() },
    { "chessBoardControl",     new ChessBoardControlParser() },
    { "cursorControl",         new CursorControlParser() },
    { "autoRunControl",        new AutoRunControlParser() },
};

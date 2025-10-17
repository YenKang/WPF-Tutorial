private static int ResolveSrc(string src, int r, int g, int b)
{
    if (string.IsNullOrEmpty(src))
        return 0;

    switch (src)
    {
        case "R":
            return r;
        case "G":
            return g;
        case "B":
            return b;
        default:
            int v;
            if (TryParseInt(src, out v))
                return v;
            return 0;
    }
}
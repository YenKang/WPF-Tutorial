if (fields["hPos"] != null)
    LoadPositionField(fields["hPos"], ref ShowHPos, ref HPos, ref _hMin, ref _hMax, ref _hLowTarget, ref _hHighTarget, ref _hMask, 0x1F);

private void LoadPositionField(JToken f, ref bool show, ref int def,
                               ref int min, ref int max,
                               ref string low, ref string high, ref byte mask, byte fallback)
{
    bool s; int d, mi, ma; string lo, hi; byte m;
    ParsePositionField(f, out s, out d, out mi, out ma, out lo, out hi, out m, fallback);
    show = s; def = d; min = mi; max = ma; low = lo; high = hi; mask = m;
}
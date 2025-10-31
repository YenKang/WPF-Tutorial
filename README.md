private void SetFcntDigitsFromValue(int value10bit, bool isFcnt1)
{
    int d2 = (value10bit >> 8) & 0x03;
    int d1 = (value10bit >> 4) & 0x0F;
    int d0 = (value10bit >> 0) & 0x0F;

    if (isFcnt1)
    {
        FCNT1_D2 = d2;
        FCNT1_D1 = d1;
        FCNT1_D0 = d0;
    }
}
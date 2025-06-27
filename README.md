private int ComputeDelta(int current, int previous)
{
    int delta = current - previous;

    if (delta > 128)
        delta -= 256;
    else if (delta < -128)
        delta += 256;

    return delta;
}
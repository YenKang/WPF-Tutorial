using System;

public sealed class RegDisplayBus : IRegisterBus
{
    private readonly RegisterReadWriteEx _hw;

    public RegDisplayBus(RegisterReadWriteEx hw)
    {
        if (hw == null) throw new ArgumentNullException(nameof(hw));
        _hw = hw;
    }

    public void Write(byte page, byte reg, byte value)
    {
        _hw.WriteRegData(page, reg, value);
    }

    public byte Read(byte page, byte reg)
    {
        return _hw.ReadRegData(page, reg);
    }
}

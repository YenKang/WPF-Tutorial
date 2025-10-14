using System;
using System.Collections.Generic;

public static class RegMap
{
    private static readonly Dictionary<string, ushort> _regs =
        new Dictionary<string, ushort>(StringComparer.OrdinalIgnoreCase);

    private static IRegisterBus _bus;

    public static void Init(IRegisterBus bus)
    {
        if (bus == null) throw new ArgumentNullException("bus");
        _bus = bus;
    }

    public static void LoadFrom(IcProfile profile)
    {
        if (profile == null) throw new ArgumentNullException("profile");

        _regs.Clear();
        foreach (var kv in profile.Registers)
        {
            // 防呆：避免空字串
            var hex = kv.Value == null ? "" : kv.Value.Replace("0x", "").Replace("0X", "");
            ushort val = Convert.ToUInt16(hex, 16);
            _regs[kv.Key] = val;
        }
    }

    private static Tuple<byte, byte> Split(ushort full)
    {
        var page = (byte)(full >> 8);
        var reg  = (byte)(full & 0xFF);
        return Tuple.Create(page, reg);
    }

    public static Tuple<byte, byte> Get(string name)
    {
        ushort full;
        if (!_regs.TryGetValue(name, out full))
            throw new KeyNotFoundException("Register name not found: " + name);
        return Split(full);
    }

    public static void Write8(string name, byte value)
    {
        EnsureBus();
        var pr = Get(name);
        _bus.Write(pr.Item1, pr.Item2, value);
    }

    public static byte Read8(string name)
    {
        EnsureBus();
        var pr = Get(name);
        return _bus.Read(pr.Item1, pr.Item2);
    }

    public static void Rmw8(string name, byte mask, byte valueShifted)
    {
        EnsureBus();
        var pr = Get(name);
        byte cur  = _bus.Read(pr.Item1, pr.Item2);
        byte next = (byte)((cur & ~mask) | (valueShifted & mask));
        _bus.Write(pr.Item1, pr.Item2, next);
    }

    private static void EnsureBus()
    {
        if (_bus == null)
            throw new InvalidOperationException("RegMap.Init(bus) must be called before using RegMap.");
    }
}

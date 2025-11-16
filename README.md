    // === Up / Down 命令 ===
    IncreaseFCNT2Command = CommandFactory.CreateCommand(_ => AdjustFCNT2(+1));
    DecreaseFCNT2Command = CommandFactory.CreateCommand(_ => AdjustFCNT2(-1));
}
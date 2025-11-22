public class RootObject
{
    public ChessBoardControl chessBoardControl { get; set; }
}

public class ChessBoardControl
{
    public int HRes { get; set; }
    public int VRes { get; set; }
    public int M { get; set; }
    public int N { get; set; }
    public ChessBoardRegister register { get; set; }
}

public class ChessBoardRegister
{
    public string HToggleW { get; set; }
    public string H_PLUS1 { get; set; }
    public string VToggleH { get; set; }
    public string V_PLUS1 { get; set; }
}
using System.Globalization;
using System.Windows.Controls;

public class IntRangeRule : ValidationRule
{
    public int Min { get; set; } = int.MinValue;
    public int Max { get; set; } = int.MaxValue;

    public override ValidationResult Validate(object value, CultureInfo cultureInfo)
    {
        if (value == null) return new ValidationResult(false, "Required");
        if (!int.TryParse(value.ToString(), out int v)) return new ValidationResult(false, "Number");
        if (v < Min || v > Max) return new ValidationResult(false, $"{Min}..{Max}");
        return ValidationResult.ValidResult;
    }
}
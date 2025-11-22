
## Model

```
namespace BistMode.Models
{
    public sealed class FlickerRGBModel
    {
        // ----- Page1 -----
        public int RPage1 { get; set; }
        public string RPage1Reg { get; set; }

        public int GPage1 { get; set; }
        public string GPage1Reg { get; set; }

        public int BPage1 { get; set; }
        public string BPage1Reg { get; set; }

        // ----- Page2 -----
        public int RPage2 { get; set; }
        public string RPage2Reg { get; set; }

        public int GPage2 { get; set; }
        public string GPage2Reg { get; set; }

        public int BPage2 { get; set; }
        public string BPage2Reg { get; set; }

        // ----- Page3 -----
        public int RPage3 { get; set; }
        public string RPage3Reg { get; set; }

        public int GPage3 { get; set; }
        public string GPage3Reg { get; set; }

        public int BPage3 { get; set; }
        public string BPage3Reg { get; set; }

        // ----- Page4 -----
        public int RPage4 { get; set; }
        public string RPage4Reg { get; set; }

        public int GPage4 { get; set; }
        public string GPage4Reg { get; set; }

        public int BPage4 { get; set; }
        public string BPage4Reg { get; set; }
    }
}
```

## json parse

```
var frgbNode = pnode["flickerRGBControl"] as JObject;
if (frgbNode != null)
{
    var model = new FlickerRGBModel();

    // Page1
    model.RPage1    = frgbNode["RPage1"]?["default"]?.Value<int>() ?? 0;
    model.RPage1Reg = frgbNode["RPage1"]?["register"]?.Value<string>() ?? string.Empty;

    model.GPage1    = frgbNode["GPage1"]?["default"]?.Value<int>() ?? 0;
    model.GPage1Reg = frgbNode["GPage1"]?["register"]?.Value<string>() ?? string.Empty;

    model.BPage1    = frgbNode["BPage1"]?["default"]?.Value<int>() ?? 0;
    model.BPage1Reg = frgbNode["BPage1"]?["register"]?.Value<string>() ?? string.Empty;

    // Page2
    model.RPage2    = frgbNode["RPage2"]?["default"]?.Value<int>() ?? 0;
    model.RPage2Reg = frgbNode["RPage2"]?["register"]?.Value<string>() ?? string.Empty;

    model.GPage2    = frgbNode["GPage2"]?["default"]?.Value<int>() ?? 0;
    model.GPage2Reg = frgbNode["GPage2"]?["register"]?.Value<string>() ?? string.Empty;

    model.BPage2    = frgbNode["BPage2"]?["default"]?.Value<int>() ?? 0;
    model.BPage2Reg = frgbNode["BPage2"]?["register"]?.Value<string>() ?? string.Empty;

    // Page3
    model.RPage3    = frgbNode["RPage3"]?["default"]?.Value<int>() ?? 0;
    model.RPage3Reg = frgbNode["RPage3"]?["register"]?.Value<string>() ?? string.Empty;

    model.GPage3    = frgbNode["GPage3"]?["default"]?.Value<int>() ?? 0;
    model.GPage3Reg = frgbNode["GPage3"]?["register"]?.Value<string>() ?? string.Empty;

    model.BPage3    = frgbNode["BPage3"]?["default"]?.Value<int>() ?? 0;
    model.BPage3Reg = frgbNode["BPage3"]?["register"]?.Value<string>() ?? string.Empty;

    // Page4
    model.RPage4    = frgbNode["RPage4"]?["default"]?.Value<int>() ?? 0;
    model.RPage4Reg = frgbNode["RPage4"]?["register"]?.Value<string>() ?? string.Empty;

    model.GPage4    = frgbNode["GPage4"]?["default"]?.Value<int>() ?? 0;
    model.GPage4Reg = frgbNode["GPage4"]?["register"]?.Value<string>() ?? string.Empty;

    model.BPage4    = frgbNode["BPage4"]?["default"]?.Value<int>() ?? 0;
    model.BPage4Reg = frgbNode["BPage4"]?["register"]?.Value<string>() ?? string.Empty;

    // 存進 PatternItem
    p.FlickerRGBModel = model;
}
```

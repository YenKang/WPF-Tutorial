## 1️⃣ BwLoopControlParser.cs

```
using System.Diagnostics;
using Newtonsoft.Json.Linq;
using BistMode.Services;
// using BistMode.ViewModels; // 如果 PatternItem 在這裡就打開

namespace BistMode.Parsers
{
    public sealed class BwLoopControlParser : IPatternControlParser
    {
        // 這兩個參數，就是你原本 ParsePatternNodeToModel(PatternItem p, Services.PatternItem node) 裡的 p / node
        public void Parse(PatternItem p, PatternItem node)
        {
            // 創建 bwLoopControl Model
            var bwLoopControlNode = node.BWLoopControl as JObject;
            if (bwLoopControlNode == null)
                return;

            // 取出 fcnt2 這個物件
            var fcnt2Obj = bwLoopControlNode["fcnt2"] as JObject;
            if (fcnt2Obj == null)
            {
                Debug.WriteLine("[BWLoop LoadFrom] fcnt2 node is null!");
                return;
            }

            // fcnt2
            int fcnt2Default = fcnt2Obj["default"]?.Value<int>() ?? 30;
            int fcnt2Max     = fcnt2Obj["max"]?.Value<int>()     ?? 600;
            int fcnt2Min     = fcnt2Obj["min"]?.Value<int>()     ?? 600;
            string regName   = fcnt2Obj["register"]?.ToString().Trim() ?? string.Empty;

            var bwLoopModel = new BWLoopModel
            {
                RegName = regName,
                Fcnt2   = fcnt2Default,
                Fcnt2Max = fcnt2Max,
                Fcnt2Min = fcnt2Min,
            };

            p.BWLoopModel = bwLoopModel;
        }
    }
}
```

## step2: IPatternControlParser

```
using BistMode.Services;

namespace BistMode.Parsers
{
    public interface IPatternControlParser
    {
        void Parse(PatternItem p, PatternItem node);
    }
}
```

## Step3:_controlParsers 註冊

```
using System;
using System.Collections.Generic;
using System.Diagnostics;
using BistMode.Services;
using BistMode.Parsers;

public class BistModeViewModel
{
    private readonly Dictionary<string, IPatternControlParser> _controlParsers;

    public BistModeViewModel()
    {
        _controlParsers = new Dictionary<string, IPatternControlParser>(StringComparer.OrdinalIgnoreCase)
        {
            { "bwLoopControl", new BwLoopControlParser() },
            // 之後會陸續加：
            // { "grayReverseControl", new GrayReverseControlParser() },
            // { "grayLevelControl",   new GrayLevelControlParser() },
            // ...
        };
    }

    #region 初始化解析 JSON Node
    public void ParsePatternNodeToModel(PatternItem p, PatternItem node)
    {
        if (node.RegControlType == null || node.RegControlType.Count == 0)
            return;

        foreach (var type in node.RegControlType)
        {
            IPatternControlParser parser;
            if (_controlParsers.TryGetValue(type, out parser))
            {
                parser.Parse(p, node);
            }
            else
            {
                Debug.WriteLine("[ParsePatternNode] Unknown control type: " + type);
            }
        }
    }
    #endregion
}
```

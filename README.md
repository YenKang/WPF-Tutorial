{
  "index": 90,
  "name": "Auto Run Config",
  "icon": "AutoRun.png",
  "regControlType": ["autoRunControl"],
  "autoRunControl": {
    "title": "Auto Run",
    "total": { "min": 1, "max": 22, "default": 22, "target": "BIST_PT_TOTAL" },

    "orders": {
      "targets": [
        "BIST_PT_ORD0","BIST_PT_ORD1","BIST_PT_ORD2","BIST_PT_ORD3","BIST_PT_ORD4",
        "BIST_PT_ORD5","BIST_PT_ORD6","BIST_PT_ORD7","BIST_PT_ORD8","BIST_PT_ORD9",
        "BIST_PT_ORD10","BIST_PT_ORD11","BIST_PT_ORD12","BIST_PT_ORD13","BIST_PT_ORD14",
        "BIST_PT_ORD15","BIST_PT_ORD16","BIST_PT_ORD17","BIST_PT_ORD18","BIST_PT_ORD19",
        "BIST_PT_ORD20","BIST_PT_ORD21"
      ],
      "defaults": [ 0, 1, 4, 4, 3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 ]
    },

    "options": [
      { "index": 0,  "name": "0 Black" },
      { "index": 1,  "name": "1 White" },
      { "index": 2,  "name": "2 Red" },
      { "index": 3,  "name": "3 Green" },
      { "index": 4,  "name": "4 Blue" },
      { "index": 5,  "name": "5 Horizontal 16 Gray Scale" },
      { "index": 6,  "name": "6 Vertical 16 Gray Scale" },
      { "index": 7,  "name": "7 Horizontal 256 Gray Scale" },
      { "index": 8,  "name": "8 Vertical 256 Gray Scale" },
      { "index": 9,  "name": "9 8 Color Bar" },
      { "index": 10, "name": "10 Chess Board with MN Block" },
      { "index": 11, "name": "11 Cursor" },
      { "index": 12, "name": "12 1-Pixel On/Off" },
      { "index": 13, "name": "13 2-Pixel On/Off" },
      { "index": 14, "name": "14 1 Sub-pixel On/Off" },
      { "index": 15, "name": "15 2 Sub-pixel On/Off" },
      { "index": 16, "name": "16 Crosstalk w/ Center Black" },
      { "index": 17, "name": "17 Crosstalk w/ Center White" },
      { "index": 18, "name": "18 Flicker (Pattern for FLK Level)" },
      { "index": 19, "name": "19 8 Color Bar + Gray Scale" },
      { "index": 20, "name": "20 3 Color Bar" },
      { "index": 21, "name": "21 4 Horizontal Bar (B↔W)" }
      // ...視你的專案有多少就列多少
    ],

    "trigger": { "target": "BIST_PT", "value": "0x3F" }
  }
}


=======


var optArray = autoRunControl["options"] as JArray;
if (optArray != null && optArray.Count > 0) {
    PatternOptions.Clear();
    foreach (var t in optArray.OfType<JObject>()) {
        var idx  = t["index"]?.Value<int>() ?? 0;
        var name = (string)t["name"] ?? idx.ToString();
        PatternOptions.Add(new PatternOption { Index = idx, Display = name });
    }
}
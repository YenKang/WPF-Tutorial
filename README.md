{
  "icName": "NT51365",

  "registers": {
    "BIST_PT":          "0x0017",
    "BIST_PT_LEVEL":    "0x0030",
    "BIST_CK_GY_FK_PT": "0x0034"
  },

  "constraints": {
    "gray10": { "min": 0, "max": 1023 },
    "digits": {
      "low":  { "min": 0, "max": 15 },
      "mid":  { "min": 0, "max": 15 },
      "high": { "min": 0, "max": 3  }
    }
  },

  "assets": {
    "patternIconBase": "pack://siteoforigin:,,,/Assets/Patterns/NT51365/"
  },

  "patterns": [
    { "index": 0,  "name": "Black", "icon": "P0_Black.png" },
    { "index": 1,  "name": "White", "icon": "P1_White.png" },
    { "index": 2,  "name": "Red",   "icon": "P2_Red.png" },
    { "index": 3,  "name": "Green", "icon": "P3_Green.png" },
    { "index": 4,  "name": "Blue",  "icon": "P4_Blue.png" },
    { "index": 12, "name": "1-Pixel On/Off",  "icon": "P12_1-Pixel-OnOff.png" },
    { "index": 13, "name": "2-Pixel On/Off",  "icon": "P13_2-Pixel-OnOff.png" },
    { "index": 14, "name": "1-Subpixel On/Off","icon": "P14_1-Sub-pixel-OnOff.png" },
    { "index": 15, "name": "2-Subpixel On/Off","icon": "P15_2-Sub-pixel-OnOff.png" },
    { "index": 16, "name": "Crosstalk Center Black", "icon": "P16_Crosstalk with Center Black.png" },
    { "index": 17, "name": "Crosstalk Center White", "icon": "P17_Crosstalk with Center White.png" },
    { "index": 23, "name": "Exclamation Mark", "icon": "P23_Exclamation Mark.png" },
    { "index": 24, "name": "Vertical Line 1px BW", "icon": "P24_Vertical Line with Black and White on 1 Pixel Base.png" },
    { "index": 25, "name": "Vertical Line 2px BW", "icon": "P25_Vertical Line with Black and White on 2 Pixel Base.png" },
    { "index": 26, "name": "Vertical Line Subpixel BW", "icon": "P26_Vertical Line with Black and White on Subpixel Base.png" },
    { "index": 27, "name": "Horizontal 1 Line BW", "icon": "P27_Horizontal 1 Line with Black and White.png" },
    { "index": 28, "name": "Horizontal 2 Line BW", "icon": "P28_Horizontal 2 Line with Black and White.png" },
    { "index": 29, "name": "Horizontal Sub-line BW", "icon": "P29_Horizontal sub-line with Black and White.png" },
    { "index": 32, "name": "Cyan",    "icon": "P32_Cyan.png" },
    { "index": 34, "name": "Yellow",  "icon": "P34_Yellow.png" }
  ],

  "grayControls": {
    "menu": [
      { "id": "PT_LEVEL",        "title": "Pattern_Level" },
      { "id": "FLK_LEVEL",       "title": "Flicker_Level" },
      { "id": "PIXEL_LEVEL",     "title": "Pixel_Level" },
      { "id": "LINE_LEVEL",      "title": "Line_Level" }
    ],
    "targets": [
      {
        "id": "PT_LEVEL",
        "name": "Pattern Level",
        "range": { "min": 0, "max": 1023 },
        "validPatterns": [ 1, 3, 4, 23, 32, 34 ]   // 只能用這 6 張
      },
      {
        "id": "FLK_LEVEL",
        "name": "Flicker Level",
        "range": { "min": 0, "max": 1023 },
        "validPatterns": [ 16, 17 ]
      },
      {
        "id": "PIXEL_LEVEL",
        "name": "Pixel Level",
        "range": { "min": 0, "max": 1023 },
        "validPatterns": [ 12, 13, 14, 15 ]
      },
      {
        "id": "LINE_LEVEL",
        "name": "Line Level",
        "range": { "min": 0, "max": 1023 },
        "validPatterns": [ 24, 25, 26, 27, 28, 29 ]
      }
    ]
  }
}

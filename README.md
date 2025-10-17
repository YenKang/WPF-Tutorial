{
  "index": 37,
  "name": "4 H-Gray Bar RGBW",
  "icon": "P37_H_Gray_Bar_RGBW.png",
  "regControlType": ["grayReverseControl"],
  "grayReverseControl": {
    "axis": "H",
    "default": 0,
    "writes": [
      { "mode": "rmw", "target": "BIST_GrayColor_VH_Reverse", "mask": "0x10", "shift": 4, "src": "enable" }
    ]
  }
}
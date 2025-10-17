{
  "index": 11,
  "name": "Cursor",
  "icon": "P11_Cursor.png",
  "regControlType": ["grayLevelControl", "bgGrayLevelControl"],

  "grayLevelControl": {
    "fields": {
      "D2": { "min": 0, "max": 3,  "default": 3 },
      "D1": { "min": 0, "max": 15, "default": 15 },
      "D0": { "min": 0, "max": 15, "default": 15 }
    },
    "writes": [
      { "mode": "write8", "target": "BIST_PT_LEVEL",    "src": "LowByte" },
      { "mode": "rmw",    "target": "BIST_CK_GY_FK_PT", "mask":"0x03", "shift":0, "src":"D2" }
    ]
  },

  "bgGrayLevelControl": {
    "fields": {
      "D2": { "min": 0, "max": 3,  "default": 1 },
      "D1": { "min": 0, "max": 15, "default": 8 },
      "D0": { "min": 0, "max": 15, "default": 0 }
    },
    "writes": [
      { "mode": "write8", "target": "BIST_BG_PT_LEVEL", "src": "LowByte" },
      { "mode": "rmw",    "target": "BIST_BG_CTL",      "mask":"0x03", "shift":0, "src":"D2" }
    ]
  }
}
{
  "patterns": [
    {
      "index": 0,
      "name": "Black",
      "icon": "P0_Black.png",
      "regControlType": []
    },
    {
      "index": 2,
      "name": "Red",
      "icon": "P2_Red.png",
      "regControlType": ["grayLevelControl"],
      "grayLevelControl": {
        "fields": {
          "D2": { "min": 0, "max": 3, "default": 3 },
          "D1": { "min": 0, "max": 15, "default": 15 },
          "D0": { "min": 0, "max": 15, "default": 15 }
        },
        "writes": [
          { "mode": "write8", "target": "BIST_PT_LEVEL", "src": "LowByte" },
          { "mode": "rmw", "target": "BIST_CK_GY_FK_PT", "mask": "0x03", "shift": 0, "src": "D2" }
        ]
      }
    }
  ]
}
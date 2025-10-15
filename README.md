{
  "chip": "NT51365",
  "registers": {
    "BIST_PT_LEVEL": "0x0030",
    "BIST_CK_GY_FK_PT": "0x0034"
  },
  "patterns": [
    {
      "name": "Red",
      "index": 1,
      "configUIRaws": [
        {
          "id": "red.graylevel",
          "title": "grayLevelControl",
          "kind": "grayLevelControl",
          "fields": {
            "D2": { "min": 0, "max": 3,  "default": 3 },
            "D1": { "min": 0, "max": 15, "default": 15 },
            "D0": { "min": 0, "max": 15, "default": 15 }
          },
          "writes": [
            { "mode": "write8", "target": "BIST_PT_LEVEL",    "src": "LowByte" },
            { "mode": "rmw",    "target": "BIST_CK_GY_FK_PT", "mask": "0x03", "shift": 0, "src": "D2" }
          ],
          "explain": "GrayLevel = (D2<<8) | (D1<<4) | D0"
        }
      ]
    }
  ]
}
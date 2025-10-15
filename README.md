{
  "version": "1.0",
  "chip": "NT51365",
  "registers": {
    "BIST_PT_LEVEL": { "addr": "0x0030", "width": 8, "desc": "GrayLevel[7:0] = (D1<<4)|D0" },
    "BIST_CK_GY_FK_PT": { "addr": "0x0034", "width": 8, "desc": "Holds high-2-bit of GrayLevel at pattern-specific bitfield" }
  },
  "patterns": [
    {
      "name": "Red",
      "index": 1,
      "rows": [
        {
          "id": "red.graylevel",
          "title": "GrayLevel",
          "fieldType": "UInt10Hex3",
          "fields": {
            "D2": { "min": 0, "max": 3,  "default": 3 },
            "D1": { "min": 0, "max": 15, "default": 15 },
            "D0": { "min": 0, "max": 15, "default": 15 }
          },
          "writes": [
            {
              "mode": "pack",
              "target": "BIST_PT_LEVEL",
              "parts": [
                { "src": "D1", "mask": "0x0F", "shift": 4 },
                { "src": "D0", "mask": "0x0F", "shift": 0 }
              ]
            },
            {
              "mode": "rmw",
              "target": "BIST_CK_GY_FK_PT",
              "mask":  "0x03",
              "shift": 0,
              "src":   "D2"
            }
          ]
        }
      ]
    },
    {
      "name": "Gray",
      "index": 2,
      "rows": [
        {
          "id": "gray.graylevel",
          "title": "GrayLevel",
          "fieldType": "UInt10Hex3",
          "fields": {
            "D2": { "min": 0, "max": 3,  "default": 3 },
            "D1": { "min": 0, "max": 15, "default": 15 },
            "D0": { "min": 0, "max": 15, "default": 15 }
          },
          "writes": [
            {
              "mode": "pack",
              "target": "BIST_PT_LEVEL",
              "parts": [
                { "src": "D1", "mask": "0x0F", "shift": 4 },
                { "src": "D0", "mask": "0x0F", "shift": 0 }
              ]
            },
            {
              "mode": "rmw",
              "target": "BIST_CK_GY_FK_PT",
              "mask":  "0x0C",
              "shift": 2,
              "src":   "D2"
            }
          ]
        }
      ]
    }
  ]
}
{
  "icName": "NT51365",

  "registers": {
    "BIST_PT":          "0x0017",
    "BIST_PT_LEVEL":    "0x0030",
    "BIST_CK_GY_FK_PT": "0x0034"
  },

  "patterns": [
    { "index": 0, "name": "Black", "icon": "P0_Black.png" },
    { "index": 1, "name": "White", "icon": "P1_White.png" },
    { "index": 2, "name": "Red",   "icon": "P2_Red.png" }
  ],

  "grayControls": {
    "targets": [
      {
        "id": "PT_LEVEL",
        "name": "Pattern Level",
        "range": { "min": 0, "max": 1023 },
        "validPatterns": [ 1, 2 ]
      }
    ]
  }
}

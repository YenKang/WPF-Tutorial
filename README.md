"writes": [
  { "mode": "write8", "target": "BIST_PT_LEVEL",    "src": "LowByte" },
  { "mode": "rmw",    "target": "BIST_CK_GY_FK_PT", "mask": "0x03", "shift": 0, "src": "D2" }
]
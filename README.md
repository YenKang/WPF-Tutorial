{
  "name": "Chessboard",
  "fields": [
    { "key": "H_RES", "default": 1920 },
    { "key": "V_RES", "default": 720 },
    { "key": "M", "default": 5, "min": 2, "max": 40 },
    { "key": "N", "default": 7, "min": 2, "max": 40 }
  ],
  "chessH": {
    "writes": [
      { "mode": "write8", "target": "BIST_CHESSBOARD_H_TOGGLE_W_L", "src": "lowByte" },
      { "mode": "rmw", "target": "BIST_CHESSBOARD_H_TOGGLE_W_H", "mask": "0x1F", "shift": 0, "src": "highByte" },
      { "mode": "write8", "target": "BIST_CHESSBOARD_H_PLUS1_BLKNUM", "src": "lowByte" }
    ]
  },
  "chessV": {
    "writes": [
      { "mode": "write8", "target": "BIST_CHESSBOARD_V_TOGGLE_W_L", "src": "lowByte" },
      { "mode": "rmw", "target": "BIST_CHESSBOARD_V_TOGGLE_W_H", "mask": "0x0F", "shift": 0, "src": "highByte" },
      { "mode": "write8", "target": "BIST_CHESSBOARD_V_PLUS1_BLKNUM", "src": "lowByte" }
    ]
  },
  "patternSelect": {
    "trigger": {
      "on": "Apply",
      "delay_ms": 0,
      "writes": [
        { "mode": "write8", "target": "BIST_PT", "src": "lowByte" }
      ]
    }
  }
}
{
  "name": "BWLoop",
  "regControlType": ["bwLoopControl"],
  "bwLoopControl": {
    "fcnt2": {
      "default": "0x01E",
      "min": "0x000",
      "max": "0x3FF",
      "targets": {
        "low":  "BIST_FCNT2_LO",
        "high": "BIST_FCNT2_HI",
        "mask": "0x0C",
        "shift": 2
      }
    }
  }
}
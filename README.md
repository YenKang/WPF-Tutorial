{
  "index": 11,
  "name": "Cursor",
  "regControlType": ["grayLevelControl", "grayColorControl"],

  "grayColorControl": {
    "title": "Gray Level Color",
    "fields": {
      "R": { "min": 0, "max": 1, "default": 1 },
      "G": { "min": 0, "max": 1, "default": 0 },
      "B": { "min": 0, "max": 1, "default": 0 }
    },
    "writes": [
      { "mode": "rmw", "target": "BIST_GRAY_COLOR", "mask": "0x01", "shift": 0, "src": "R" },  // bit0
      { "mode": "rmw", "target": "BIST_GRAY_COLOR", "mask": "0x02", "shift": 1, "src": "G" },  // bit1
      { "mode": "rmw", "target": "BIST_GRAY_COLOR", "mask": "0x04", "shift": 2, "src": "B" }   // bit2
    ]
  }
}
{
  "index": 11,
  "name": "Cursor Pattern",
  "icon": "P11_Cursor.png",
  "regControlType": ["lineExclamCursorControl"],
  "lineExclamCursorControl": {
    "title": "Cursor Control",
    "target": "BIST_Line_ExclamBG_Cursor",
    "fields": {
      "diagonalEn": { "mask": "0x08", "default": 0, "visible": true },
      "cursorEn":   { "mask": "0x04", "default": 0, "visible": true },

      "hPos": {
        "min": 0, "max": 1920, "default": 960, "visible": true,
        "write": {
          "low":  { "mode": "write8", "target": "BIST_CURSOR_HPOS_LO" },
          "high": { "mode": "rmw", "target": "BIST_CURSOR_HPOS_HI", "mask": "0x1F", "shift": 0 }
        }
      },
      "vPos": {
        "min": 0, "max": 1080, "default": 540, "visible": true,
        "write": {
          "low":  { "mode": "write8", "target": "BIST_CURSOR_VPOS_LO" },
          "high": { "mode": "rmw", "target": "BIST_CURSOR_VPOS_HI", "mask": "0x0F", "shift": 0 }
        }
      }
    }
  }
}
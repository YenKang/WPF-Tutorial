{
  "index": 38,
  "name": "4 V Gray Bar RGBW",
  "icon": "P38_4V.png",
  "regControlType": ["lineExclamCursorControl"],
  "lineExclamCursorControl": {
    "title": "First Line Color",
    "target": "BIST_Line_ExclamBG_Cursor",
    "fields": {
      "lineSel":  { "mask": "0x20", "default": 1, "visible": true },
      "exclamBg": { "mask": "0x10", "default": 0, "visible": false },
      "diagonalEn": { "mask": "0x08", "default": 0, "visible": false },
      "cursorEn":   { "mask": "0x04", "default": 0, "visible": false }
    }
  }
}
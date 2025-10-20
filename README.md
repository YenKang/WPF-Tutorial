{
  "index": 39,
  "name": "Exclamation Pattern",
  "icon": "P39_Exclam.png",

  "regControlType": ["lineExclamCursorControl"],

  "lineExclamCursorControl": {
    "title": "Exclamation Background",
    "target": "BIST_Line_ExclamBG_Cursor",
    "fields": {
      "exclamBg": { "mask": "0x10", "default": 1, "visible": true }  // 1=White, 0=Black
    }
  }
}
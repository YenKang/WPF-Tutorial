{
  "index": 90,
  "name": "Auto Run Config",
  "icon": "AutoRun.png",
  "regControlType": ["autoRunControl"],
  "autoRunControl": {
    "title": "Auto Run",
    "total": { "min": 1, "max": 22, "default": 5, "target": "BIST_PT_TOTAL" },

    "orders": {
      "targets": [
        "BIST_PT_ORD0","BIST_PT_ORD1","BIST_PT_ORD2","BIST_PT_ORD3","BIST_PT_ORD4",
        "BIST_PT_ORD5","BIST_PT_ORD6","BIST_PT_ORD7","BIST_PT_ORD8","BIST_PT_ORD9",
        "BIST_PT_ORD10","BIST_PT_ORD11","BIST_PT_ORD12","BIST_PT_ORD13","BIST_PT_ORD14",
        "BIST_PT_ORD15","BIST_PT_ORD16","BIST_PT_ORD17","BIST_PT_ORD18","BIST_PT_ORD19",
        "BIST_PT_ORD20","BIST_PT_ORD21"
      ],
      "defaults": [ 0, 1, 4, 4, 3 ]   // 範例：0黑、1白、4藍、4藍、3綠；超過 default 長度的槽位就用 0
    },

    "fcnt1": { "default": "0x3C", "target": "BIST_FCNT1" },
    "fcnt2": { "default": "0x1E", "target": "BIST_FCNT2" },

    "trigger": { "target": "BIST_PT", "value": "0x3F" }  // 啟動 auto-run
  }
}
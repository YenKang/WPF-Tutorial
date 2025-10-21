{
  "index": 90,
  "name": "Auto Run Config",
  "icon": "AutoRun.png",
  "regControlType": ["autoRunControl"],
  "autoRunControl": {
    "title": "Auto Run",
    "total": { "min": 1, "max": 22, "default": 22, "target": "BIST_PT_TOTAL" },

    "orders": {
      "targets": [
        "BIST_PT_ORD0","BIST_PT_ORD1","BIST_PT_ORD2","BIST_PT_ORD3","BIST_PT_ORD4",
        "BIST_PT_ORD5","BIST_PT_ORD6","BIST_PT_ORD7","BIST_PT_ORD8","BIST_PT_ORD9",
        "BIST_PT_ORD10","BIST_PT_ORD11","BIST_PT_ORD12","BIST_PT_ORD13","BIST_PT_ORD14",
        "BIST_PT_ORD15","BIST_PT_ORD16","BIST_PT_ORD17","BIST_PT_ORD18","BIST_PT_ORD19",
        "BIST_PT_ORD20","BIST_PT_ORD21"
      ],
      "defaults": [ 0, 1, 4, 4, 3 ]
    },

    "fcnt1": { "default": "0x03C", "targets": { "low":"BIST_FCNT1_LO", "high":"BIST_FCNT1_HI", "mask":"0x03", "shift":8 } },
    "fcnt2": { "default": "0x01E", "targets": { "low":"BIST_FCNT2_LO", "high":"BIST_FCNT2_HI", "mask":"0x0C", "shift":10 } },

    "trigger": { "target": "BIST_PT", "value": "0x3F" }
  }
}
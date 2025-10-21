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
      "defaults": [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21]
    },

    "fcnt1": {
      "min": 0,
      "max": 600,
      "default": 60,
      "write": {
        "low":  { "mode": "write8", "target": "BIST_FCNT1" },
        "high": { "mode": "rmw",    "target": "BIST_FCNT_HI", "mask": "0x03", "shift": 0 }
      }
    },

    "fcnt2": {
      "min": 0,
      "max": 600,
      "default": 30,
      "write": {
        "low":  { "mode": "write8", "target": "BIST_FCNT2" },
        "high": { "mode": "rmw",    "target": "BIST_FCNT_HI", "mask": "0x0C", "shift": 2 }
      }
    },

    "trigger": { "target": "BIST_PT", "value": "0x3F" }
  }
}
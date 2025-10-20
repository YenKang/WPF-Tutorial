{
  "index": 18,
  "name": "Flicker Pattern",
  "icon": "P18_Flicker.png",
  "regControlType": ["flickerMatrixControl"],
  "flickerMatrixControl": {
    "title": "Flicker Plane RGB Level",
    "defaults": {
      "P1": { "R": 5,  "G": 10, "B": 5  },
      "P2": { "R": 10, "G": 5,  "B": 10 },
      "P3": { "R": 5,  "G": 10, "B": 5  },
      "P4": { "R": 10, "G": 5,  "B": 10 }
    },
    "pairs": [
      { "target": "BIST_FLK_P1P2_R", "low": "P1.R", "high": "P2.R" },
      { "target": "BIST_FLK_P1P2_G", "low": "P1.G", "high": "P2.G" },
      { "target": "BIST_FLK_P1P2_B", "low": "P1.B", "high": "P2.B" },

      { "target": "BIST_FLK_P3P4_R", "low": "P3.R", "high": "P4.R" },
      { "target": "BIST_FLK_P3P4_G", "low": "P3.G", "high": "P4.G" },
      { "target": "BIST_FLK_P3P4_B", "low": "P3.B", "high": "P4.B" }
    ]
  }
}
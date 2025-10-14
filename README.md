{
  "icName": "NT51365",

  "registers": {
    "BIST_SW":        "0x0304",
    "BIST_PT":        "0x0017",
    "BIST_PT_LEVEL":  "0x0030",
    "BIST_FLK_LEVEL": "0x0031",
    "BIST_GRAY_LEVEL":"0x0032",
    "BIST_CK_GY_FK_PT":"0x0034"
  },

  "patterns": [
    { "index": 0,  "name": "Black",                                     "icon": "P0_Black.png" },
    { "index": 1,  "name": "White",                                     "icon": "P1_White.png" },
    { "index": 2,  "name": "Red",                                       "icon": "P2_Red.png" },
    { "index": 3,  "name": "Green",                                     "icon": "P3_Green.png" },
    { "index": 4,  "name": "Blue",                                      "icon": "P4_Blue.png" },

    { "index": 5,  "name": "Horizontal 16 Gray Scale",                  "icon": "P5_Horizontal_16_Gray_Scale.png" },
    { "index": 6,  "name": "Vertical 16 Gray Scale",                    "icon": "P6_Vertical_16_Gray_Scale.png" },
    { "index": 7,  "name": "Horizontal 256 Gray Scale",                 "icon": "P7_Horizontal_256_Gray_Scale.png" },
    { "index": 8,  "name": "Vertical 256 Gray Scale",                   "icon": "P8_Vertical_256_Gray_Scale.png" },

    { "index": 9,  "name": "8 Color Bar",                               "icon": "P9_8_Color_Bar.png" },
    { "index": 10, "name": "Chess Board with MN Block",                 "icon": "P10_Chess_Board_with_MN_Block.png" },
    { "index": 11, "name": "Black BG with White Outer Frame",           "icon": "P11_Black_BG_with_White_Outer_Frame.png" },

    { "index": 12, "name": "1-Pixel On/Off",                            "icon": "P12_1_Pixel_OnOff.png" },
    { "index": 13, "name": "2-Pixel On/Off",                            "icon": "P13_2_Pixel_OnOff.png" },
    { "index": 14, "name": "1 Sub-pixel On/Off",                        "icon": "P14_1_SubPixel_OnOff.png" },
    { "index": 15, "name": "2 Sub-pixel On/Off",                        "icon": "P15_2_SubPixel_OnOff.png" },

    { "index": 16, "name": "Crosstalk with Center Black",               "icon": "P16_Crosstalk_with_Center_Black.png" },
    { "index": 17, "name": "Crosstalk with Center White",               "icon": "P17_Crosstalk_with_Center_White.png" },

    { "index": 18, "name": "Flicker (Pattern for FLK Level)",           "icon": "P18_Flicker.png" },
    { "index": 19, "name": "8 Color Bar and 8 Gray Scale",              "icon": "P19_8_Color_Bar_and_8_Gray_Scale.png" },
    { "index": 20, "name": "3 Color Bar",                               "icon": "P20_3_Color_Bar.png" },

    { "index": 21, "name": "4 Horizontal Bar (Black→White)",            "icon": "P21_4_H_Bar_Black_White.png" },
    { "index": 22, "name": "4 Horizontal Bar (White→Black)",            "icon": "P22_4_H_Bar_White_Black.png" },
    { "index": 23, "name": "Exclamation Mark",                          "icon": "P23_Exclamation_Mark.png" },

    { "index": 24, "name": "Vertical Line B/W (1-Pixel Base)",          "icon": "P24_V_Line_BW_1px.png" },
    { "index": 25, "name": "Vertical Line B/W (2-Pixel Base)",          "icon": "P25_V_Line_BW_2px.png" },
    { "index": 26, "name": "Vertical Line B/W (Subpixel Base)",         "icon": "P26_V_Line_BW_Subpixel.png" },

    { "index": 27, "name": "Horizontal 1 Line B/W",                     "icon": "P27_H_1_Line_BW.png" },
    { "index": 28, "name": "Horizontal 2 Line B/W",                     "icon": "P28_H_2_Line_BW.png" },
    { "index": 29, "name": "Horizontal Sub-line B/W",                   "icon": "P29_H_Sub_Line_BW.png" },

    { "index": 30, "name": "Programmable Gray",                         "icon": "P30_Programmable_Gray.png" },
    { "index": 31, "name": "Black and White Loop",                      "icon": "P31_Black_and_White_Loop.png" },

    { "index": 32, "name": "Cyan",                                      "icon": "P32_Cyan.png" },
    { "index": 33, "name": "Magenta",                                   "icon": "P33_Magenta.png" },
    { "index": 34, "name": "Yellow",                                    "icon": "P34_Yellow.png" },

    { "index": 35, "name": "H Gray",                                    "icon": "P35_H_Gray.png" },
    { "index": 36, "name": "V Gray",                                    "icon": "P36_V_Gray.png" },
    { "index": 37, "name": "4 H-Gray Bar RGBW",                         "icon": "P37_4_H_Gray_Bar_RGBW.png" },
    { "index": 38, "name": "4 V-Gray Bar RGBW",                         "icon": "P38_4_V_Gray_Bar_RGBW.png" }
  ],

  "grayControls": {
    "targets": [
      {
        "id": "PT_LEVEL",
        "name": "Pattern Level",
        "range": { "min": 0, "max": 1023 },
        "validPatterns": [1, 2]  // White, Red 可調灰（你可再依需求調整）
      }
    ]
  }
}

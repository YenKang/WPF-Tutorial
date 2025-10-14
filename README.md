{
  "icName": "NT51365",

  "registers": {
    "BIST_PT_LEVEL":        "0x0030",
    "BIST_FLK_LEVEL":       "0x0031",
    "BIST_GRAY_LEVEL":      "0x0032",
    "BIST_CROSSTALK_LEVEL": "0x0033",
    "BIST_CK_GY_FK_PT":     "0x0034"
  },

  "grayControls": {
    "targets": [
      { "id": "PT_LEVEL",        "name": "Pattern Level",    "range": { "min": 0, "max": 1023 }, "controlFormatRef": "NT51365_GRAY10_PT" },
      { "id": "FLK_LEVEL",       "name": "Flicker Level",    "range": { "min": 0, "max": 1023 }, "controlFormatRef": "NT51365_GRAY10_FLK" },
      { "id": "GRAY_LEVEL",      "name": "Gray Level",       "range": { "min": 0, "max": 1023 }, "controlFormatRef": "NT51365_GRAY10_GY"  },
      { "id": "CROSSTALK_LEVEL", "name": "Crosstalk Level",  "range": { "min": 0, "max": 1023 }, "controlFormatRef": "NT51365_GRAY10_CK"  }
    ]
  },

  "controlFormats": {
    "NT51365_GRAY10_PT": {
      "valueBits": 10,
      "parts": [
        { "source": "LOW8", "write": { "type": "WRITE8", "addr": "BIST_PT_LEVEL" } },
        { "source": "HI2",  "write": { "type": "RMW8",   "addr": "BIST_CK_GY_FK_PT", "mask": "0x03", "shift": 0 } }
      ]
    },
    "NT51365_GRAY10_FLK": {
      "valueBits": 10,
      "parts": [
        { "source": "LOW8", "write": { "type": "WRITE8", "addr": "BIST_FLK_LEVEL" } },
        { "source": "HI2",  "write": { "type": "RMW8",   "addr": "BIST_CK_GY_FK_PT", "mask": "0x0C", "shift": 2 } }
      ]
    },
    "NT51365_GRAY10_GY": {
      "valueBits": 10,
      "parts": [
        { "source": "LOW8", "write": { "type": "WRITE8", "addr": "BIST_GRAY_LEVEL" } },
        { "source": "HI2",  "write": { "type": "RMW8",   "addr": "BIST_CK_GY_FK_PT", "mask": "0x30", "shift": 4 } }
      ]
    },
    "NT51365_GRAY10_CK": {
      "valueBits": 10,
      "parts": [
        { "source": "LOW8", "write": { "type": "WRITE8", "addr": "BIST_CROSSTALK_LEVEL" } },
        { "source": "HI2",  "write": { "type": "RMW8",   "addr": "BIST_CK_GY_FK_PT", "mask": "0xC0", "shift": 6 } }
      ]
    }
  }
}

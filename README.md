public string ThumbnailKey
{
    get
    {
        if (SelectedIcon != null)
        {
            return SelectedIcon.ThumbnailKey;
        }
        else
        {
            return "(待載入)";
        }
    }
}

public string FlashStartAddrHex
{
    get
    {
        if (SelectedIcon != null)
        {
            return SelectedIcon.FlashStartHex;
        }
        else
        {
            return "0x000000";
        }
    }
}
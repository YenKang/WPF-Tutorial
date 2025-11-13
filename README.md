private ICType _selectedIC = ICType.Primary;

public ICType SelectedIC
{
    get => _selectedIC;
    set
    {
        if (_selectedIC == value) return;
        _selectedIC = value;

        // 切換 IC → 載入對應資料集
        LoadIconSlotsForIC(_selectedIC);

        RaisePropertyChanged(nameof(SelectedIC));
    }
}

=====

// 每個 IC 都有一份自己的 IconSlots (30 rows)
private Dictionary<ICType, ObservableCollection<IconSlotModel>> _icSlotMap
    = new Dictionary<ICType, ObservableCollection<IconSlotModel>>();
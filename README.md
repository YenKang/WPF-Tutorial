using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using OSDIconFlashMap.Model;   // 為了用到 ICType, IconSlotModel

namespace OSDIconFlashMap.ViewModel
{
    public class IconToImageMapViewModel : ViewModelBase
    {
        // 目前畫面上 DataGrid 綁定的 30 列資料
        public ObservableCollection<IconSlotModel> IconSlots { get; } 
            = new ObservableCollection<IconSlotModel>();

        // 🔹 儲存「每一種 IC 各自的一份 IconSlot 列表」
        // 例如：Primary → List(30列), L1 → List(30列), …
        private readonly Dictionary<ICType, List<IconSlotModel>> _icSlotStorage
            = new Dictionary<ICType, List<IconSlotModel>>();

        // 🔹 UI 的 IC 下拉選單用：四種可選的 IC
        public ICType[] AvailableICs { get; } =
        {
            ICType.Primary,
            ICType.L1,
            ICType.L2,
            ICType.L3
        };

        // 🔹 目前選到哪一顆 IC（預設 Primary）
        private ICType _selectedIC = ICType.Primary;
        public ICType SelectedIC
        {
            get { return _selectedIC; }
            set
            {
                if (_selectedIC == value) return;
                _selectedIC = value;

                // 切換 IC ⇒ 載入對應的 30 列 IconSlotModel
                LoadIconSlotsForIC(_selectedIC);

                RaisePropertyChanged(nameof(SelectedIC));
            }
        }

        // ===== 建構式 =====
        public IconToImageMapViewModel()
        {
            // 這邊你原本應該有 InitIconSlots(), LoadImagesFromFolder() 等，
            // 現在只要在初始化時，先建立 Primary 的資料即可。

            // 1) 建立 Primary 預設 30 列（空白）
            var primarySlots = CreateDefaultIconSlots();
            _icSlotStorage[ICType.Primary] = primarySlots;

            // 2) 把 Primary 的 30 列顯示到畫面上
            IconSlots.Clear();
            foreach (var slot in primarySlots)
                IconSlots.Add(slot);

            // 3) 告訴 UI：IconSlots 已經準備好了
            RaisePropertyChanged(nameof(IconSlots));
        }
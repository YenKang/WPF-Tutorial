using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Linq;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.ViewModel
{
    public class IconFlashMapViewModel : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;
        private void Raise(string name) { PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name)); }

        // 全部可選 icon 清單（共用）
        public ObservableCollection<IconModel> Icons { get; } = new ObservableCollection<IconModel>();

        // 畫面綁定的當前 OSD 列表（會隨 SelectedIc 切換內容）
        public ObservableCollection<OsdModel> Osds { get; } = new ObservableCollection<OsdModel>();

        // IC 選項 + 當前選擇（注意：依照你的字樣 "Teritary"）
        public ObservableCollection<string> IcOptions { get; } =
            new ObservableCollection<string>(new[] { "Primary", "Secondary", "Teritary" });

        private string _selectedIc = "Primary";
        public string SelectedIc
        {
            get => _selectedIc;
            set
            {
                if (_selectedIc != value)
                {
                    _selectedIc = value;
                    Raise(nameof(SelectedIc));
                    // 切換目前顯示的 OSD 對應
                    ApplyMapForSelectedIc();
                }
            }
        }

        // 內部：每個 IC 對應一組 OSD 列
        private readonly Dictionary<string, ObservableCollection<OsdModel>> _icOsdMaps
            = new Dictionary<string, ObservableCollection<OsdModel>>();

        public IconFlashMapViewModel()
        {
            // 先放一些假 icon，或等外部 LoadCatalog 時覆蓋
            var mock = Enumerable.Range(1, 12).Select(i => new IconModel
            {
                Id = i,
                Name = $"icon_{i}",
                FlashStart = 0x18000UL + (ulong)((i - 1) * 0x2222),
                ThumbnailKey = $"icon_{i}_thumbnail"
            });
            LoadCatalog(mock);
        }

        /// <summary>
        /// 外部載入 icon catalog（共用），並以它建立每個 IC 的預設 OSD 對應集合。
        /// </summary>
        public void LoadCatalog(IEnumerable<IconModel> catalog)
        {
            Icons.Clear();
            foreach (var icon in catalog)
            {
                if (string.IsNullOrEmpty(icon.ThumbnailKey))
                    icon.ThumbnailKey = $"{icon.Name}_thumbnail"; // 規則：iconN → iconN_thumbnail
                Icons.Add(icon);
            }

            // 為每個 IC 建立獨立的 40 列 OSD 對應（可用不同初始化策略）
            _icOsdMaps.Clear();
            _icOsdMaps["Primary"]   = BuildOsdMap(strategy: 0);
            _icOsdMaps["Secondary"] = BuildOsdMap(strategy: 1);
            _icOsdMaps["Teritary"]  = BuildOsdMap(strategy: 2);

            ApplyMapForSelectedIc();
        }

        /// <summary>
        /// 依 "strategy" 建 40 列：不同 IC 可用不同預設 icon 分配邏輯
        /// 0: OSD_i -> icon_i（超出範圍則取第一個）
        /// 1: OSD_i -> icon_((i+4-1)%N+1)（往後位移 4）
        /// 2: OSD_i -> icon_N（全部同一個，若空則第一個）
        /// </summary>
        private ObservableCollection<OsdModel> BuildOsdMap(int strategy)
        {
            var list = new ObservableCollection<OsdModel>();
            int n = Icons.Count > 0 ? Icons.Count : 1;

            for (int i = 1; i <= 40; i++)
            {
                IconModel pick;
                switch (strategy)
                {
                    default: // Primary
                        pick = Icons.FirstOrDefault(x => x.Id == i) ?? Icons.First();
                        break;
                    case 1:  // Secondary: 位移 4
                        int idx = ((i + 4 - 1) % n) + 1; // 1..n
                        pick = Icons.FirstOrDefault(x => x.Id == idx) ?? Icons.First();
                        break;
                    case 2:  // Teritary: 全部用最後一個
                        pick = Icons.LastOrDefault() ?? Icons.First();
                        break;
                }

                list.Add(new OsdModel { OsdName = $"OSD_{i}", SelectedIcon = pick });
            }

            return list;
        }

        /// <summary>
        /// 將目前 SelectedIc 對應的 OSD 列拷貝到 Osds（供畫面顯示）。
        /// </summary>
        private void ApplyMapForSelectedIc()
        {
            var src = _icOsdMaps.ContainsKey(SelectedIc) ? _icOsdMaps[SelectedIc] : null;
            Osds.Clear();
            if (src != null) foreach (var row in src) Osds.Add(row);
            Raise(nameof(Osds));
        }
    }
}
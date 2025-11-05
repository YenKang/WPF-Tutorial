using System.Collections.ObjectModel;
using System.Linq;
using System.Collections.Generic;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.ViewModel
{
    public class IconFlashMapViewModel
    {
        public ObservableCollection<IconModel> Icons { get; } = new ObservableCollection<IconModel>();
        public ObservableCollection<OsdModel>  Osds  { get; } = new ObservableCollection<OsdModel>();

        public IconFlashMapViewModel()
        {
            // 先填假資料（之後會用外部資料覆蓋）
            LoadMock();
        }

        private void LoadMock()
        {
            var mock = new List<IconModel>
            {
                new IconModel{ Id=1, Name="icon_1", FlashStart=0x18000 },
                new IconModel{ Id=2, Name="icon_2", FlashStart=0x18222 },
                new IconModel{ Id=3, Name="icon_3", FlashStart=0x18880 },
                new IconModel{ Id=4, Name="icon_4", FlashStart=0x19000 },
            };
            LoadCatalog(mock);
        }

        // 給外部（舊畫面）餵資料用
        public void LoadCatalog(IEnumerable<IconModel> catalog)
        {
            Icons.Clear();
            foreach (var icon in catalog) Icons.Add(icon);

            Osds.Clear();
            for (int i = 1; i <= 40; i++)
            {
                var def = Icons.FirstOrDefault(x => x.Id == i) ?? Icons.FirstOrDefault();
                Osds.Add(new OsdModel { OsdName = $"OSD_{i}", SelectedIcon = def });
            }
        }
    }
}
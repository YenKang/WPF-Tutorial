using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.IO;
using System.Linq;
using Utility.MVVM;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.ViewModel
{
    public class IconToImageMapViewModel : ViewModelBase
    {
        public ObservableCollection<IconSlotModel> IconSlots { get; } =
            new ObservableCollection<IconSlotModel>();

        public List<ImageOption> Images { get; } = new List<ImageOption>();

        private IcProfile _selectedProfile = IcProfile.Primary;
        public IcProfile SelectedProfile
        {
            get { return _selectedProfile; }
            set
            {
                if (_selectedProfile != value)
                {
                    _selectedProfile = value;
                    RaisePropertyChanged(nameof(SelectedProfile));
                }
            }
        }

        public IconToImageMapViewModel()
        {
            // 30 列（NT51726）
            for (int i = 1; i <= 30; i++)
                IconSlots.Add(new IconSlotModel { IconIndex = i });
        }

        public void LoadImagesFromFolder(string folder)
        {
            Images.Clear();
            if (!Directory.Exists(folder)) return;

            var files = Directory.GetFiles(folder, "*.png");
            foreach (var f in files.OrderBy(x => x))
            {
                var name = Path.GetFileNameWithoutExtension(f); // image_1
                Images.Add(new ImageOption
                {
                    Key = name,
                    Name = name,
                    ImagePath = f
                });
            }
        }

        // 儲存 / 載入 INI（只作用於當前 SelectedProfile）
        public void SaveToIni(string filePath)
        {
            var dict = Util.IniHelper.ReadAsSections(filePath);
            var section = new Dictionary<string, string>();

            foreach (var slot in IconSlots)
            {
                var key = $"Icon{slot.IconIndex}";
                var val = slot.SelectedImage != null ? slot.SelectedImage.Key : "";
                section[key] = val;
            }

            dict[SelectedProfile.ToString()] = section;
            Util.IniHelper.WriteSections(filePath, dict);
        }

        public void LoadFromIni(string filePath, IcProfile profile)
        {
            var dict = Util.IniHelper.ReadAsSections(filePath);
            Dictionary<string, string> section;
            if (!dict.TryGetValue(profile.ToString(), out section)) return;

            foreach (var slot in IconSlots)
            {
                var key = $"Icon{slot.IconIndex}";
                string imageKey;
                if (section.TryGetValue(key, out imageKey) && !string.IsNullOrEmpty(imageKey))
                {
                    var hit = Images.FirstOrDefault(x => x.Key == imageKey);
                    slot.SelectedImage = hit;
                }
                else
                {
                    slot.SelectedImage = null;
                }
            }
        }
    }
}


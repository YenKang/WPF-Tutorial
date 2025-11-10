using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.IO;
using System.Linq;
using OSDIconFlashMap.Model;
using Utility.MVVM;

namespace OSDIconFlashMap.ViewModel
{
    public class IconToImageMapViewModel : ViewModelBase
    {
        public ObservableCollection<IconSlotModel> IconSlots { get; } = new();
        public ObservableCollection<ImageOption>   Images    { get; } = new();

        // 固定 NT51726：30 列
        public void InitIconSlots()
        {
            IconSlots.Clear();
            for (int i = 1; i <= 30; i++)
                IconSlots.Add(new IconSlotModel { IconIndex = i });
            RaisePropertyChanged(nameof(IconSlots));
        }

        public void LoadImages(params ImageOption[] options)
        {
            Images.Clear();
            foreach (var o in options) Images.Add(o);
            RaisePropertyChanged(nameof(Images));
        }

        // 從資料夾載入（僅取 7 張）
        public void LoadImagesFromFolder(string relativeFolder)
        {
            // 以執行檔資料夾為基準
            var baseDir = AppDomain.CurrentDomain.BaseDirectory;
            var folder  = Path.Combine(baseDir, relativeFolder);

            if (!Directory.Exists(folder))
                return;

            var filePatterns = new[] { "*.png", "*.jpg", "*.jpeg", "*.bmp" };
            var files = filePatterns.SelectMany(p => Directory.GetFiles(folder, p))
                                    .Take(7) // 只取 7 張
                                    .ToList();

            var list = new List<ImageOption>();
            int id = 1;
            foreach (var path in files)
            {
                var name = Path.GetFileNameWithoutExtension(path);
                list.Add(new ImageOption
                {
                    Id = id++,
                    Name = name,
                    Key  = name,
                    ThumbnailKey = $"{name}_thumbnail",
                    ImagePath = path
                });
            }

            LoadImages(list.ToArray());
        }
    }
}
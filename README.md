using OSDIconFlashMap.Model;
using System.Collections.ObjectModel;
using System.Linq;
using Utility.MVVM;

namespace OSDIconFlashMap.ViewModel
{
    public class IconToImageMapViewModel : ViewModelBase
    {
        public ObservableCollection<IconSlotModel> IconSlots { get; } = new ObservableCollection<IconSlotModel>();
        public ObservableCollection<ImageOption> Images { get; } = new ObservableCollection<ImageOption>();

        // 給「OSD Selection」下拉用：1..30
        public System.Collections.Generic.IReadOnlyList<int> OsdOptions { get; } =
            Enumerable.Range(1, 30).ToList();

        public IconToImageMapViewModel()
        {
            InitIconSlots();        // 你原本就呼叫
            // Images 仍可用 LoadImages(...) / LoadImagesFromFolder(...) 載入
        }

        // === 初始化 30 列（補齊預設值＋注入 SRAM 計算器） ===
        public void InitIconSlots()
        {
            IconSlots.Clear();
            for (int i = 1; i <= 30; i++)
            {
                IconSlots.Add(new IconSlotModel
                {
                    IconIndex = i,
                    SelectedImage = null,   // 未選
                    OsdTargetIndex = 0,     // 未選
                    IsOsdEnabled = false,   // 未啟用
                    HPos = 1,
                    VPos = 1,
                    CalcSramStartAddress = DemoSramCalc
                });
            }
            RaisePropertyChanged(nameof(IconSlots));
        }

        // === Demo：SRAM 起始位址公式（之後換正式規則即可） ===
        private string DemoSramCalc(IconSlotModel slot)
        {
            const int baseAddr = 0x000000; // demo
            const int stride   = 0x001000; // 每張 4KB（demo）
            var addr = baseAddr + (slot.IconIndex - 1) * stride;
            return "0x" + addr.ToString("X6");
        }

        // 你原本就有：
        // public void LoadImages(params ImageOption[] options) { ... }
        // public void LoadImagesFromFolder(string relativeFolder) { ... }
    }
}
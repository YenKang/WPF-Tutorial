using System.Linq;
// ...

namespace OSDIconFlashMap.ViewModel
{
    public class IconToImageMapViewModel : ViewModelBase
    {
        public ObservableCollection<IconSlotModel> IconSlots { get; private set; }
        public ObservableCollection<ImageOption> Images { get; private set; }

        // ... 你原本的 InitIconSlots(), LoadImagesFromFolder() 等

        /// <summary>
        /// 取得目前 Icon_DL_SEL = 1 的圖片清單，
        /// 也就是所有 SelectedImage != null 的 ImageOption，
        /// 並依 Key 去重複。
        /// </summary>
        public List<ImageOption> GetOsdCandidateImages()
        {
            return IconSlots
                .Where(slot => slot.SelectedImage != null)  // 有選圖的列
                .Select(slot => slot.SelectedImage)
                .Where(img => img != null)
                .GroupBy(img => img.Key)   // 依 Key 去重複
                .Select(g => g.First())
                .ToList();
        }
    }
}
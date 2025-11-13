        /// <summary>
        /// 切換 IC 時，把該 IC 對應的 30 列資料丟到 IconSlots 裡。
        /// 若該 IC 尚未有資料，就建立新的空白 30 列。
        /// </summary>
        private void LoadIconSlotsForIC(ICType ic)
        {
            // 1) 從 storage 裡找出這個 IC 的資料
            if (!_icSlotStorage.TryGetValue(ic, out var list))
            {
                // 第一次切換到這顆 IC → 建一份新的空白資料
                list = CreateDefaultIconSlots();
                _icSlotStorage[ic] = list;
            }

            // 2) 用這份 list 更新目前畫面上的 IconSlots
            //    注意：這裡是「複製參考」，不會 new 新物件，
            //    所以你在 UI 改某列的屬性，會直接反映在 list 裡。
            IconSlots.Clear();
            foreach (var slot in list)
                IconSlots.Add(slot);

            // 3) 通知 UI：IconSlots 內容已經切換（DataGrid 會重新刷新）
            RaisePropertyChanged(nameof(IconSlots));
        }
    }
}
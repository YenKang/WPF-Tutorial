        /// <summary>
        /// 建立一份「空的 30 列 IconSlotModel」，
        /// 給新 IC 第一次被切換時使用。
        /// </summary>
        private List<IconSlotModel> CreateDefaultIconSlots()
        {
            var list = new List<IconSlotModel>();

            for (int i = 1; i <= 30; i++)
            {
                list.Add(new IconSlotModel
                {
                    IconIndex = i
                    // 其他欄位用預設值即可（HPos/VPos 你之前有預設 1 的話，也可在此設定）
                    // HPos = 1,
                    // VPos = 1
                });
            }

            return list;
        }
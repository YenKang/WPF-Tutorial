public void ProcessFingerEvents(List<FingerStatus> fingersList)
{
    foreach (var current in fingersList)
    {
        int id = current.Id;

        // 取得上一筆狀態
        _fingerStatusMap.TryGetValue(id, out var previous);

        // Debug 印出前後狀態
        Debug.WriteLine($"🧠 FingerId={id}, Prev={previous?.Status}, Now={current.Status}");

        // 定義：從 None ➜ Move 才算一次 Click
        if ((previous == null || previous.Status == "None") && current.Status == "Move")
        {
            Debug.WriteLine($"✅ Finger {id} 被視為一次 Click");

            // 你可以在這裡進行 Click 邏輯，例如：
            HandleFingerClick(current);
        }

        // 更新 Map 中該筆資料
        _fingerStatusMap[id] = current;
    }

    // Optional: 移除已消失的手指 id
    CleanupFingerMap(fingersList);
}
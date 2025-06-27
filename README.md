using System;
using System.Collections.Generic;
using System.Diagnostics;
using NovaCID.Knob;

namespace NovaCID.ViewModel
{
    public class NovaCIDViewModel
    {
        private readonly KnobEventRouter _router;
        private readonly KnobEventProcessor _processor;

        // ✅ constructor
        public NovaCIDViewModel()
        {
            // 1️⃣ 建立 Router，轉發給目前頁面的 ViewModel
            _router = new KnobEventRouter(() => CurrentPageViewModel);

            // 2️⃣ 建立 Processor，並注入 Router
            _processor = new KnobEventProcessor(_router);

            // 3️⃣ 測試接收模擬資料（可移除）
            //OnRawKnobDataReceived("Knob_0,Role=Driver,Touch=True,Counter=10,Press=False");
        }

        // 取得目前畫面的 ViewModel（供 Router 使用）
        public object CurrentPageViewModel
        {
            get
            {
                return _viewModelCache.TryGetValue(_currentPageType, out var vm) ? vm : null;
            }
        }

        // ✅ 這是唯一對外開放的 Knob 入口
        public void OnRawKnobDataReceived(string rawData)
        {
            Debug.WriteLine($"📩 收到 Raw Knob Data:\n{rawData}");
            _processor.ParseAndProcess(rawData); // 交給 Processor 處理
        }

        // 🔒 預設的頁面 / ViewModel 綁定邏輯與 Cache（略）
        private readonly Dictionary<PageType, object> _viewModelCache = new();
        private PageType _currentPageType;
    }
}
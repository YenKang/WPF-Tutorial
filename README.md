using Newtonsoft.Json.Linq;
using System;
using System.Collections.ObjectModel;
using System.Windows.Input;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private string _totalTarget;
        private string[] _ordTargets;
        private JObject _fcnt1Obj, _fcnt2Obj;
        private string _triggerTarget;
        private byte _triggerValue;

        // === Properties ===
        public string Title { get; private set; } = "Auto Run Configuration";

        public ObservableCollection<int> TotalOptions { get; } =
            new ObservableCollection<int>(System.Linq.Enumerable.Range(1, 22));

        public int Total
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        public ObservableCollection<int> Orders { get; } = new ObservableCollection<int>();

        public int FCNT1 { get => GetValue<int>(); set => SetValue(value); }
        public int FCNT2 { get => GetValue<int>(); set => SetValue(value); }

        public ICommand ApplyCommand { get; }

        // === Constructor ===
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // === Load from JSON ===
        public void LoadFrom(JObject jo)
        {
            if (jo == null) return;

            Title = (string)jo["title"] ?? "Auto Run";

            // 1️⃣ 解析 total
            var totalObj = jo["total"] as JObject;
            _totalTarget = (string)totalObj?["target"];
            Total = totalObj?["default"]?.Value<int>() ?? 22;

            // 2️⃣ 解析 orders
            var orders = jo["orders"] as JObject;
            _ordTargets = orders?["targets"]?.ToObject<string[]>();
            var defaults = orders?["defaults"]?.ToObject<int[]>() ?? new int[0];
            Orders.Clear();
            for (int i = 0; i < Total; i++)
            {
                int val = (i < defaults.Length) ? defaults[i] : 0;
                Orders.Add(val);
            }

            // 3️⃣ FCNT1 / FCNT2
            _fcnt1Obj = jo["fcnt1"] as JObject;
            _fcnt2Obj = jo["fcnt2"] as JObject;
            FCNT1 = _fcnt1Obj?["default"]?.Value<int>() ?? 60;
            FCNT2 = _fcnt2Obj?["default"]?.Value<int>() ?? 30;

            // 4️⃣ Trigger
            var trigger = jo["trigger"] as JObject;
            _triggerTarget = (string)trigger?["target"];
            var vStr = (string)trigger?["value"];
            _triggerValue = string.IsNullOrWhiteSpace(vStr)
                ? (byte)0x3F
                : Convert.ToByte(vStr.Replace("0x", ""), 16);
        }

        // === Apply ===
        private void Apply()
        {
            // 1️⃣ 寫入總數
            if (!string.IsNullOrEmpty(_totalTarget))
                RegMap.Write8(_totalTarget, (byte)Total);

            // 2️⃣ 寫入 ORD0~ORD(Total-1)
            if (_ordTargets != null)
            {
                for (int i = 0; i < Total && i < _ordTargets.Length; i++)
                {
                    RegMap.Write8(_ordTargets[i], (byte)(Orders[i] & 0x3F));
                }
            }

            // 3️⃣ 寫入 FCNT1 / FCNT2
            WriteCountFromJson(FCNT1, _fcnt1Obj);
            WriteCountFromJson(FCNT2, _fcnt2Obj);

            // 4️⃣ 觸發
            if (!string.IsNullOrEmpty(_triggerTarget))
                RegMap.Write8(_triggerTarget, _triggerValue);
        }

        // === Helper ===
        private static void WriteCountFromJson(int value, JObject fcntObj)
        {
            if (fcntObj == null) return;
            var write = fcntObj["write"] as JObject;
            if (write == null) return;

            // 低位
            var low = write["low"] as JObject;
            if (low != null)
            {
                string lowTarget = (string)low["target"];
                if (!string.IsNullOrEmpty(lowTarget))
                {
                    byte lo = (byte)(value & 0xFF);
                    RegMap.Write8(lowTarget, lo);
                }
            }

            // 高位
            var high = write["high"] as JObject;
            if (high != null)
            {
                string hiTarget = (string)high["target"];
                string maskStr = (string)high["mask"];
                int shift = (int?)high["shift"] ?? 0;
                if (!string.IsNullOrEmpty(hiTarget) && !string.IsNullOrWhiteSpace(maskStr))
                {
                    byte mask = Convert.ToByte(maskStr.Replace("0x", ""), 16);
                    int hi2 = (value >> 8) & 0x03;
                    byte val = (byte)((hi2 << shift) & mask);
                    RegMap.Rmw(hiTarget, mask, val);
                }
            }
        }
    }
}
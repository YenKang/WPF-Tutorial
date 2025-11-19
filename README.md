using Newtonsoft.Json.Linq;
using PageBase.ICStructure.Comm;
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Diagnostics;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    /// <summary>
    /// Flicker Plane RGB Level ViewModel
    /// 對應 UI：P1~P4 × R/G/B，各自 0~15 (0~F) 的 ComboBox。
    /// 新版 JSON: flickerRGBControl.Registers.*
    /// </summary>
    public sealed class FlickerRGBVM : ViewModelBase
    {
        public ObservableCollection<PlaneVM> Planes { get; } =
            new ObservableCollection<PlaneVM>
            {
                new PlaneVM(1),
                new PlaneVM(2),
                new PlaneVM(3),
                new PlaneVM(4)
            };

        public ICommand ApplyCommand { get; }

        private JObject _cfg; // 整個 flickerRGBControl 區塊
        private readonly List<ChannelEntry> _entries = new List<ChannelEntry>();

        /// <summary>
        /// 由外部注入的暫存器讀寫介面
        /// </summary>
        public IRegisterReadWriteEx RegControl { get; set; }

        public FlickerRGBVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        /// <summary>
        /// 由 PatternItem.FlickerRGBControl 傳入 JSON block
        /// 新版結構：
        /// {
        ///   "flickerRGBControl" : {
        ///       "Registers" : { ... }
        ///   }
        /// }
        /// 這裡的 jsonCfg 就是 flickerRGBControl 那一層。
        /// </summary>
        public void LoadFrom(object jsonCfg)
        {
            _entries.Clear();

            if (jsonCfg is JObject jo)
                _cfg = jo;
            else if (jsonCfg != null)
                _cfg = JObject.FromObject(jsonCfg);
            else
            {
                _cfg = null;
                return;
            }

            var regs = _cfg["Registers"] as JObject;
            if (regs == null)
            {
                Debug.WriteLine("[FLK LoadFrom] Registers node is null.");
                return;
            }

            Debug.WriteLine("[FLK LoadFrom] ---");

            // P1
            SetupChannel(regs, "Page1Red",   Planes[0], p => p.R, (p, v) => p.R = v);
            SetupChannel(regs, "Page1Green", Planes[0], p => p.G, (p, v) => p.G = v);
            SetupChannel(regs, "Page1Blue",  Planes[0], p => p.B, (p, v) => p.B = v);

            // P2
            SetupChannel(regs, "Page2Red",   Planes[1], p => p.R, (p, v) => p.R = v);
            SetupChannel(regs, "Page2Green", Planes[1], p => p.G, (p, v) => p.G = v);
            SetupChannel(regs, "Page2Blue",  Planes[1], p => p.B, (p, v) => p.B = v);

            // P3
            SetupChannel(regs, "Page3Red",   Planes[2], p => p.R, (p, v) => p.R = v);
            SetupChannel(regs, "Page3Green", Planes[2], p => p.G, (p, v) => p.G = v);
            SetupChannel(regs, "Page3Blue",  Planes[2], p => p.B, (p, v) => p.B = v);

            // P4
            SetupChannel(regs, "Page4Red",   Planes[3], p => p.R, (p, v) => p.R = v);
            SetupChannel(regs, "Page4Green", Planes[3], p => p.G, (p, v) => p.G = v);
            SetupChannel(regs, "Page4Blue",  Planes[3], p => p.B, (p, v) => p.B = v);

            foreach (var e in _entries)
            {
                Debug.WriteLine(
                    $"[FLK LoadFrom] {e.Name}: reg={e.Register}, range={e.Min}~{e.Max}, def={e.Default}");
            }
        }

        /// <summary>
        /// 將 Registers.* 轉成 ChannelEntry，並初始化 PlaneVM 的 R/G/B 預設值
        /// </summary>
        private void SetupChannel(
            JObject regs,
            string key,
            PlaneVM plane,
            Func<PlaneVM, int> getter,
            Action<PlaneVM, int> setter)
        {
            var node = regs[key] as JObject;
            if (node == null)
            {
                Debug.WriteLine($"[FLK SetupChannel] node '{key}' not found.");
                return;
            }

            var entry = new ChannelEntry
            {
                Name     = key,
                Plane    = plane,
                Getter   = getter,
                Setter   = setter,
                Register = ReadString(node, "Register", key),
                Min      = ReadInt(node, "min", 0),
                Max      = ReadInt(node, "max", 15),
                Default  = ReadInt(node, "default", 0)
            };

            // 初始化 UI 顯示值（Clamp default）
            var def = Clamp(entry.Default, entry.Min, entry.Max);
            setter(plane, def);

            _entries.Add(entry);
        }

        private void ExecuteWrite()
        {
            if (_cfg == null)
            {
                MessageBox.Show("FlickerRGB JSON 尚未載入。");
                return;
            }

            if (RegControl == null)
            {
                MessageBox.Show("RegControl 尚未注入。");
                Debug.WriteLine("[FLK ExecuteWrite] RegControl is NULL.");
                return;
            }

            if (_entries.Count == 0)
            {
                Debug.WriteLine("[FLK ExecuteWrite] No channel entries.");
                return;
            }

            Debug.WriteLine("[FLK ExecuteWrite] ---");

            foreach (var entry in _entries)
            {
                int raw = entry.Getter(entry.Plane);
                int clamped = Clamp(raw, entry.Min, entry.Max);

                if (clamped != raw)
                    entry.Setter(entry.Plane, clamped); // 修正 UI

                Debug.WriteLine(
                    $"[FLK] Write {entry.Register} <= 0x{clamped:X1} ({entry.Name})");

                RegControl.SetRegisterByName(entry.Register, (uint)clamped);
            }
        }

        #region helpers

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private static int ReadInt(JObject obj, string key, int fallback)
        {
            if (obj == null) return fallback;

            var tok = obj[key];
            if (tok == null) return fallback;

            int v;
            if (int.TryParse(tok.ToString(), out v))
                return v;

            return fallback;
        }

        private static string ReadString(JObject obj, string key, string fallback)
        {
            if (obj == null) return fallback;
            var tok = obj[key];
            if (tok == null) return fallback;
            var s = tok.ToString().Trim();
            return string.IsNullOrEmpty(s) ? fallback : s;
        }

        /// <summary>
        /// 一個 channel (P1.R / P2.G / ...) 的設定與寫入資訊
        /// </summary>
        private sealed class ChannelEntry
        {
            public string Name;               // ex: "Page1Red"
            public PlaneVM Plane;
            public Func<PlaneVM, int> Getter;
            public Action<PlaneVM, int> Setter;
            public string Register;           // ex: "BIST_FLK_P1_R"
            public int Min;
            public int Max;
            public int Default;
        }

        #endregion

        /// <summary>
        /// UI 上每一列 P1~P4
        /// </summary>
        public sealed class PlaneVM : ViewModelBase
        {
            public int Index { get; private set; }

            public ObservableCollection<int> LevelOptions { get; } =
                new ObservableCollection<int>(Generate(0, 15));

            public int R
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public int G
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public int B
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public PlaneVM(int index)
            {
                Index = index;
                R = 0;
                G = 0;
                B = 0;
            }

            private static int[] Generate(int min, int max)
            {
                var list = new List<int>();
                for (int i = min; i <= max; i++)
                    list.Add(i);
                return list.ToArray();
            }
        }
    }
}
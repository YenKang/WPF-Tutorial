using System;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Globalization;
using System.Runtime.CompilerServices;
using System.Windows.Input;

namespace NovaCID.ViewModels
{
    public class BistPtLevelViewModel : INotifyPropertyChanged
    {
        // 0x000 ~ 0x3FF
        public const int Min = 0x000;
        public const int Max = 0x3FF;

        // ===== 下拉清單資料源（對應 XAML: ItemsSource）=====
        // D[9:8] 只能 0~3
        public ObservableCollection<string> HexDigitList { get; } =
            new ObservableCollection<string>(MakeHexList(0, 15)); // 給 D[7:4] 用（0~F）
        public ObservableCollection<string> HighDigitList { get; } =
            new ObservableCollection<string>(MakeHexList(0, 3));  // 給 D[9:8] 用（0~3）
        public ObservableCollection<string> LowDigitList  { get; } =
            new ObservableCollection<string>(MakeHexList(0, 15)); // 給 D[3:0] 用（0~F）

        // ===== 三個選中值（對應 XAML: SelectedItem=Digit1/2/3）=====
        private string _digit1 = "0"; // D[9:8]，只允許 "0"~"3"
        private string _digit2 = "0"; // D[7:4]，允許 "0"~"F"
        private string _digit3 = "0"; // D[3:0]，允許 "0"~"F"

        public string Digit1
        {
            get => _digit1;
            set
            {
                if (value == null) return;
                if (!HighDigitList.Contains(value)) return; // 保證 0~3
                if (SetField(ref _digit1, value))
                    RecomputeFromDigits();
            }
        }

        public string Digit2
        {
            get => _digit2;
            set
            {
                if (value == null) return;
                if (!HexDigitList.Contains(value)) return; // 0~F
                if (SetField(ref _digit2, value))
                    RecomputeFromDigits();
            }
        }

        public string Digit3
        {
            get => _digit3;
            set
            {
                if (value == null) return;
                if (!LowDigitList.Contains(value)) return; // 0~F
                if (SetField(ref _digit3, value))
                    RecomputeFromDigits();
            }
        }

        // ===== 組合值 =====
        private int _grayLevel;
        public int GrayLevel
        {
            get => _grayLevel;
            private set
            {
                var v = Math.Max(Min, Math.Min(Max, value));
                if (SetField(ref _grayLevel, v))
                {
                    OnPropertyChanged(nameof(FullHexValue));
                }
            }
        }

        // 對應 XAML: Text="{Binding FullHexValue}"
        public string FullHexValue => GrayLevel.ToString("X3"); // 3 位大寫 16 進位

        // ===== Set 按鈕（對應 XAML: Command="{Binding SetBistPTLevelCmd}"）=====
        public ICommand SetBistPTLevelCmd { get; }
        public Action<int>? OnSet { get; set; } // 由外部注入實際寫入：OnSet?.Invoke(GrayLevel);

        public BistPtLevelViewModel()
        {
            // 預設值 3FFh
            LoadFromInt(0x3FF);

            SetBistPTLevelCmd = new RelayCommand(
                _ => OnSet?.Invoke(GrayLevel),
                _ => OnSet != null);
        }

        // ===== 內部：由三個十六進位字元回推整數 =====
        private void RecomputeFromDigits()
        {
            int d1 = ParseHex(_digit1); // 0~3
            int d2 = ParseHex(_digit2); // 0~15
            int d3 = ParseHex(_digit3); // 0~15

            GrayLevel = (d1 << 8) | (d2 << 4) | d3; // 10-bit 中高位會自然受限
            // 更新 Command CanExecute（Set 按鈕可用狀態）
            (SetBistPTLevelCmd as RelayCommand)?.RaiseCanExecuteChanged();
        }

        // 由整數載入，反推三位字串（若你之後想支援外部寫入/同步用）
        public void LoadFromInt(int value)
        {
            value = Math.Max(Min, Math.Min(Max, value));
            _digit1 = ((value >> 8) & 0x3).ToString("X");
            _digit2 = ((value >> 4) & 0xF).ToString("X");
            _digit3 = ( value       & 0xF).ToString("X");

            OnPropertyChanged(nameof(Digit1));
            OnPropertyChanged(nameof(Digit2));
            OnPropertyChanged(nameof(Digit3));

            GrayLevel = value;
        }

        private static int ParseHex(string s) =>
            int.Parse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture);

        private static string[] MakeHexList(int from, int to)
        {
            var arr = new string[to - from + 1];
            for (int i = from; i <= to; i++) arr[i - from] = i.ToString("X");
            return arr;
        }

        // ===== INotifyPropertyChanged =====
        public event PropertyChangedEventHandler? PropertyChanged;
        protected void OnPropertyChanged([CallerMemberName] string? name = null)
            => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        protected bool SetField<T>(ref T field, T value, [CallerMemberName] string? name = null)
        {
            if (Equals(field, value)) return false;
            field = value;
            OnPropertyChanged(name);
            return true;
        }
    }

    // 最小版 RelayCommand
    public class RelayCommand : ICommand
    {
        private readonly Action<object?> _exec;
        private readonly Predicate<object?>? _can;
        public RelayCommand(Action<object?> exec, Predicate<object?>? can = null)
        { _exec = exec; _can = can; }
        public bool CanExecute(object? p) => _can?.Invoke(p) ?? true;
        public void Execute(object? p) => _exec(p);
        public event EventHandler? CanExecuteChanged;
        public void RaiseCanExecuteChanged() => CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}
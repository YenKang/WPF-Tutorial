using System;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using System.Windows.Input;

namespace NovaCID.ViewModels
{
    /// <summary>
    /// 0x000 ~ 0x3FF (10-bit) Gray Level 控制
    /// - GrayLevel (int) 與 Hex3 (string, 3 位大寫十六進位) 互相同步
    /// - 可選定 High/Mid/Low 某一位十六進位做 +1/-1
    /// - Increment/Decrement 對整體數值做 +1/-1（含邊界）
    /// - SetCommand 透過 IRegisterBus 寫入（可接你現有的 SPI 實作）
    /// </summary>
    public class GrayLevelViewModel : INotifyPropertyChanged
    {
        public const int Min = 0x000;
        public const int Max = 0x3FF;

        private int _grayLevel = Max; // 預設 3FFh
        private bool _isBusy;
        private Digit _selectedDigit = Digit.High;

        // ====== 可選：把目標暫存器位置抽象化 ======
        public uint TargetAddress { get; set; } = 0x0000; // 依你的 IC 實際位址調整
        public IRegisterBus? RegisterBus { get; set; }    // 由外部注入 SPI 實作

        // ====== Label / 標題（對應 bit 範圍）=======
        public string Title => "BIST_PT_Level[9:0]";
        public string HighLabel => "High D2 (bits 9..8)";
        public string MidLabel  => "Mid  D1 (bits 7..4)";
        public string LowLabel  => "Low  D0 (bits 3..0)";

        public event PropertyChangedEventHandler? PropertyChanged;

        public GrayLevelViewModel()
        {
            IncrementCommand = new RelayCommand(_ => Increment(), _ => CanIncrement);
            DecrementCommand = new RelayCommand(_ => Decrement(), _ => CanDecrement);
            DigitIncrementCommand = new RelayCommand(_ => IncrementDigit(), _ => CanIncrementDigit);
            DigitDecrementCommand = new RelayCommand(_ => DecrementDigit(), _ => CanDecrementDigit);
            SetCommand = new AsyncRelayCommand(SetAsync, _ => !IsBusy);
        }

        // ====== 主值（整數）=======
        public int GrayLevel
        {
            get => _grayLevel;
            set
            {
                var clamped = Math.Max(Min, Math.Min(Max, value));
                if (clamped != _grayLevel)
                {
                    _grayLevel = clamped;
                    OnPropertyChanged();
                    OnPropertyChanged(nameof(Hex3));
                    OnPropertyChanged(nameof(HighDigit));
                    OnPropertyChanged(nameof(MidDigit));
                    OnPropertyChanged(nameof(LowDigit));
                    RaiseCommandCanExecutes();
                }
            }
        }

        // ====== 三位十六進位（大寫，固定 3 碼）=======
        public string Hex3
        {
            get => GrayLevel.ToString("X3");
            set
            {
                if (string.IsNullOrWhiteSpace(value)) return;
                var s = value.Trim().ToUpperInvariant();
                // 容忍不足 3 碼輸入，會自動左補零
                if (s.Length > 3) return;
                if (!int.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var parsed)) return;
                GrayLevel = parsed;
            }
        }

        // ====== 選定要調整的十六進位位數 ======
        public Digit SelectedDigit
        {
            get => _selectedDigit;
            set
            {
                if (_selectedDigit != value)
                {
                    _selectedDigit = value;
                    OnPropertyChanged();
                    RaiseCommandCanExecutes();
                }
            }
        }

        // ====== 各位十六進位（0~3 / 0~15 / 0~15）=======
        public int HighDigit   // bits 9..8
        {
            get => (GrayLevel >> 8) & 0x3;
            set
            {
                var v = Math.Max(0, Math.Min(3, value));
                var next = (v << 8) | (MidDigit << 4) | LowDigit;
                GrayLevel = next;
            }
        }

        public int MidDigit    // bits 7..4
        {
            get => (GrayLevel >> 4) & 0xF;
            set
            {
                var v = Math.Max(0, Math.Min(0xF, value));
                var next = (HighDigit << 8) | (v << 4) | LowDigit;
                GrayLevel = next;
            }
        }

        public int LowDigit    // bits 3..0
        {
            get => GrayLevel & 0xF;
            set
            {
                var v = Math.Max(0, Math.Min(0xF, value));
                var next = (HighDigit << 8) | (MidDigit << 4) | v;
                GrayLevel = next;
            }
        }

        // ====== 狀態 ======
        public bool IsBusy
        {
            get => _isBusy;
            private set
            {
                if (_isBusy != value)
                {
                    _isBusy = value;
                    OnPropertyChanged();
                    RaiseCommandCanExecutes();
                }
            }
        }

        // ====== 命令 ======
        public ICommand IncrementCommand { get; }
        public ICommand DecrementCommand { get; }
        public ICommand DigitIncrementCommand { get; }
        public ICommand DigitDecrementCommand { get; }
        public IAsyncCommand SetCommand { get; }

        private bool CanIncrement => GrayLevel < Max;
        private bool CanDecrement => GrayLevel > Min;

        private bool CanIncrementDigit =>
            SelectedDigit switch
            {
                Digit.High => HighDigit < 3,
                Digit.Mid  => MidDigit < 0xF,
                Digit.Low  => LowDigit < 0xF,
                _ => false
            };

        private bool CanDecrementDigit =>
            SelectedDigit switch
            {
                Digit.High => HighDigit > 0,
                Digit.Mid  => MidDigit > 0,
                Digit.Low  => LowDigit > 0,
                _ => false
            };

        private void Increment()
        {
            if (CanIncrement) GrayLevel += 1;
        }

        private void Decrement()
        {
            if (CanDecrement) GrayLevel -= 1;
        }

        private void IncrementDigit()
        {
            switch (SelectedDigit)
            {
                case Digit.High: if (HighDigit < 3) HighDigit += 1; break;
                case Digit.Mid:  if (MidDigit  < 0xF) MidDigit  += 1; break;
                case Digit.Low:  if (LowDigit  < 0xF) LowDigit  += 1; break;
            }
        }

        private void DecrementDigit()
        {
            switch (SelectedDigit)
            {
                case Digit.High: if (HighDigit > 0) HighDigit -= 1; break;
                case Digit.Mid:  if (MidDigit  > 0) MidDigit  -= 1; break;
                case Digit.Low:  if (LowDigit  > 0) LowDigit  -= 1; break;
            }
        }

        private async Task SetAsync()
        {
            if (RegisterBus == null) return; // 尚未注入實體寫入通道
            try
            {
                IsBusy = true;
                // 10-bit 值寫入：若裝置需要對齊/遮罩，這裡可再調整
                ushort data = (ushort)(GrayLevel & 0x03FF);
                await RegisterBus.WriteAsync(TargetAddress, data);
            }
            finally
            {
                IsBusy = false;
            }
        }

        private void RaiseCommandCanExecutes()
        {
            (IncrementCommand as RelayCommand)?.RaiseCanExecuteChanged();
            (DecrementCommand as RelayCommand)?.RaiseCanExecuteChanged();
            (DigitIncrementCommand as RelayCommand)?.RaiseCanExecuteChanged();
            (DigitDecrementCommand as RelayCommand)?.RaiseCanExecuteChanged();
            (SetCommand as AsyncRelayCommand)?.RaiseCanExecuteChanged();
        }

        protected void OnPropertyChanged([CallerMemberName] string? name = null)
            => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }

    public enum Digit { High, Mid, Low }

    // ====== ICommand / IAsyncCommand 簡易實作 ======
    public class RelayCommand : ICommand
    {
        private readonly Action<object?> _execute;
        private readonly Predicate<object?>? _canExecute;
        public event EventHandler? CanExecuteChanged;

        public RelayCommand(Action<object?> execute, Predicate<object?>? canExecute = null)
        {
            _execute = execute ?? throw new ArgumentNullException(nameof(execute));
            _canExecute = canExecute;
        }

        public bool CanExecute(object? parameter) => _canExecute?.Invoke(parameter) ?? true;
        public void Execute(object? parameter) => _execute(parameter);
        public void RaiseCanExecuteChanged() => CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }

    public interface IAsyncCommand : ICommand
    {
        Task ExecuteAsync(object? parameter);
        void RaiseCanExecuteChanged();
    }

    public class AsyncRelayCommand : IAsyncCommand
    {
        private readonly Func<Task> _execute;
        private readonly Predicate<object?>? _canExecute;
        private bool _isRunning;

        public event EventHandler? CanExecuteChanged;

        public AsyncRelayCommand(Func<Task> execute, Predicate<object?>? canExecute = null)
        {
            _execute = execute ?? throw new ArgumentNullException(nameof(execute));
            _canExecute = canExecute;
        }

        public bool CanExecute(object? parameter) =>
            !_isRunning && (_canExecute?.Invoke(parameter) ?? true);

        public async void Execute(object? parameter) => await ExecuteAsync(parameter);

        public async Task ExecuteAsync(object? parameter)
        {
            if (!CanExecute(parameter)) return;
            try
            {
                _isRunning = true;
                RaiseCanExecuteChanged();
                await _execute();
            }
            finally
            {
                _isRunning = false;
                RaiseCanExecuteChanged();
            }
        }

        public void RaiseCanExecuteChanged() => CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }

    /// <summary>
    /// 你現有 SPI 實作可實作這個介面，並在 ViewModel 建構後注入
    /// </summary>
    public interface IRegisterBus
    {
        Task WriteAsync(uint address, ushort data);
    }
}
## 3️⃣ 建立 RandomBmpViewModel（產圖核心）


```
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Windows;
using System.Windows.Input;
using YourApp.MVVM;

namespace YourApp.ViewModels
{
    public sealed class RandomBmpViewModel : ViewModelBase
    {
        private readonly Random _rand = new Random();

        // H 方向
        private int _hStart = 1916;
        public int HStart { get { return _hStart; } set { _hStart = value; RaisePropertyChanged(); } }

        private int _hEnd = 1920;
        public int HEnd   { get { return _hEnd; }  set { _hEnd  = value; RaisePropertyChanged(); } }

        private int _hStep = 1;
        public int HStep  { get { return _hStep; } set { _hStep = value; RaisePropertyChanged(); } }

        // V 方向
        private int _vStart = 2000;
        public int VStart { get { return _vStart; } set { _vStart = value; RaisePropertyChanged(); } }

        private int _vEnd = 2005;
        public int VEnd   { get { return _vEnd; }  set { _vEnd  = value; RaisePropertyChanged(); } }

        private int _vStep = 1;
        public int VStep  { get { return _vStep; } set { _vStep = value; RaisePropertyChanged(); } }

        // 輸出路徑
        private string _outputFolder = @"D:\RandomBmp";
        public string OutputFolder
        {
            get { return _outputFolder; }
            set { _outputFolder = value; RaisePropertyChanged(); }
        }

        public ICommand GenerateCommand { get; private set; }

        public RandomBmpViewModel()
        {
            GenerateCommand = new RelayCommand(_ => Generate(), _ => CanGenerate());
        }

        private bool CanGenerate()
        {
            return HStep > 0 &&
                   VStep > 0 &&
                   HEnd >= HStart &&
                   VEnd >= VStart &&
                   !string.IsNullOrWhiteSpace(OutputFolder);
        }

        private void Generate()
        {
            try
            {
                Directory.CreateDirectory(OutputFolder);

                int count = 0;

                for (int h = HStart; h <= HEnd; h += HStep)
                {
                    for (int v = VStart; v <= VEnd; v += VStep)
                    {
                        using (var bmp = new Bitmap(h, v, PixelFormat.Format24bppRgb))
                        {
                            // 隨機純色
                            int r = _rand.Next(256);
                            int g = _rand.Next(256);
                            int b = _rand.Next(256);
                            var color = System.Drawing.Color.FromArgb(r, g, b);

                            using (var gdi = Graphics.FromImage(bmp))
                            {
                                gdi.Clear(color);
                            }

                            // 檔名：H{寬}_V{高}.bmp
                            string fileName = $"H{h}_V{v}.bmp";
                            string fullPath = Path.Combine(OutputFolder, fileName);

                            bmp.Save(fullPath, ImageFormat.Bmp);
                            count++;
                        }
                    }
                }

                MessageBox.Show($"完成，共產出 {count} 張圖片。",
                    "完成", MessageBoxButton.OK, MessageBoxImage.Information);
            }
            catch (Exception ex)
            {
                MessageBox.Show("產出失敗：" + ex.Message,
                    "錯誤", MessageBoxButton.OK, MessageBoxImage.Error);
            }
        }
    }
}
```

## 4️⃣ 建立 RandomBmpWindow（專門 for 產圖）

```
<Window x:Class="YourApp.RandomBmpWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:YourApp.ViewModels"
        Title="Random BMP Generator" Height="260" Width="430">

    <Window.DataContext>
        <vm:RandomBmpViewModel />
    </Window.DataContext>

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- H / V 像 Excel 黃色欄位 -->
        <Grid Grid.Row="0">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="80"/>
                <ColumnDefinition Width="80"/>
                <ColumnDefinition Width="80"/>
            </Grid.ColumnDefinitions>

            <!-- 標題列 -->
            <TextBlock Grid.Row="0" Grid.Column="1" Text="Start"
                       HorizontalAlignment="Center" Margin="0,0,0,4"/>
            <TextBlock Grid.Row="0" Grid.Column="2" Text="End"
                       HorizontalAlignment="Center" Margin="0,0,0,4"/>
            <TextBlock Grid.Row="0" Grid.Column="3" Text="Step"
                       HorizontalAlignment="Center" Margin="0,0,0,4"/>

            <!-- H Pixel -->
            <TextBlock Grid.Row="1" Grid.Column="0" Text="H Pixel"
                       VerticalAlignment="Center" Margin="0,0,4,0"/>
            <TextBox Grid.Row="1" Grid.Column="1" Width="60"
                     Text="{Binding HStart, UpdateSourceTrigger=PropertyChanged}"/>
            <TextBox Grid.Row="1" Grid.Column="2" Width="60"
                     Text="{Binding HEnd,   UpdateSourceTrigger=PropertyChanged}"/>
            <TextBox Grid.Row="1" Grid.Column="3" Width="60"
                     Text="{Binding HStep,  UpdateSourceTrigger=PropertyChanged}"/>

            <!-- V Pixel -->
            <TextBlock Grid.Row="2" Grid.Column="0" Text="V Pixel"
                       VerticalAlignment="Center" Margin="0,4,4,0"/>
            <TextBox Grid.Row="2" Grid.Column="1" Width="60" Margin="0,4,0,0"
                     Text="{Binding VStart, UpdateSourceTrigger=PropertyChanged}"/>
            <TextBox Grid.Row="2" Grid.Column="2" Width="60" Margin="0,4,0,0"
                     Text="{Binding VEnd,   UpdateSourceTrigger=PropertyChanged}"/>
            <TextBox Grid.Row="2" Grid.Column="3" Width="60" Margin="0,4,0,0"
                     Text="{Binding VStep,  UpdateSourceTrigger=PropertyChanged}"/>
        </Grid>

        <!-- Output Folder -->
        <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="0,8,0,0">
            <TextBlock Text="Output Folder:" VerticalAlignment="Center" Margin="0,0,4,0"/>
            <TextBox Width="260"
                     Text="{Binding OutputFolder, UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>

        <!-- Generate 按鈕 -->
        <Button Grid.Row="2"
                Content="Generate BMP"
                Width="130" Height="32"
                Margin="0,8,0,0"
                HorizontalAlignment="Left"
                Command="{Binding GenerateCommand}"/>
    </Grid>
</Window>
```

## 6️⃣ 在 OSD 視窗 XAML 加按鈕（Click）


```
<Button Content="Random BMP 測試"
        Padding="8,4"
        Click="OpenRandomBmpWindow_Click"/>
```

## 7️⃣ 在 OSD 視窗 .xaml.cs 實作 Click 事件

```
private void OpenRandomBmpWindow_Click(object sender, RoutedEventArgs e)
{
    var win = new RandomBmpWindow();
    win.Owner = this;   // 可選
    win.Show();         // 開新的非模態視窗
}
```


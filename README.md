```csharp
using System.Collections.ObjectModel;
using System.ComponentModel;

namespace NovaCID.ViewModel
{
    public class MusicAlbum
    {
        public string Artist { get; set; }
        public string SongName { get; set; } // ← 修改這裡
        public string CoverImagePath { get; set; }
    }

    public class MusicPageViewModel : INotifyPropertyChanged
    {
        public ObservableCollection<MusicAlbum> AlbumList { get; set; }

        private int _currentAlbumIndex;
        public int CurrentAlbumIndex
        {
            get => _currentAlbumIndex;
            set
            {
                if (_currentAlbumIndex != value)
                {
                    _currentAlbumIndex = value;
                    OnPropertyChanged(nameof(CurrentAlbumIndex));
                    OnPropertyChanged(nameof(CurrentAlbum));
                }
            }
        }

        public MusicAlbum CurrentAlbum => AlbumList[CurrentAlbumIndex];

        public MusicPageViewModel()
        {
            AlbumList = new ObservableCollection<MusicAlbum>
            {
                new MusicAlbum { Artist = "周杰倫", SongName = "晴天", CoverImagePath = "/Assets/jay.png" },
                new MusicAlbum { Artist = "周蕙", SongName = "好想好想", CoverImagePath = "/Assets/zhouhui.png" },
                new MusicAlbum { Artist = "劉若英", SongName = "後來", CoverImagePath = "/Assets/rene.png" },
            };

            CurrentAlbumIndex = 0;
        }

        public void NextSong()
        {
            CurrentAlbumIndex = (CurrentAlbumIndex + 1) % AlbumList.Count;
        }

        public void PreviousSong()
        {
            CurrentAlbumIndex = (CurrentAlbumIndex - 1 + AlbumList.Count) % AlbumList.Count;
        }

        public event PropertyChangedEventHandler PropertyChanged;
        protected void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```


```xaml
<UserControl x:Class="NovaCID.Views.MusicPage"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="clr-namespace:NovaCID.ViewModel"
             Width="500" Height="500">

    <UserControl.DataContext>
        <vm:MusicPageViewModel/>
    </UserControl.DataContext>

    <StackPanel Orientation="Vertical" HorizontalAlignment="Center" VerticalAlignment="Center">
        <Image Source="{Binding CurrentAlbum.CoverImagePath}" Width="200" Height="200" Margin="10"/>
        <TextBlock Text="{Binding CurrentAlbum.Artist}" FontSize="20" Margin="5" HorizontalAlignment="Center"/>
        <TextBlock Text="{Binding CurrentAlbum.SongName}" FontSize="16" Margin="5" HorizontalAlignment="Center"/> <!-- 修改這裡 -->

        <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="10">
            <Button Content="上一首" Click="PreviousSong_Click" Width="80" Margin="5"/>
            <Button Content="下一首" Click="NextSong_Click" Width="80" Margin="5"/>
        </StackPanel>
    </StackPanel>
</UserControl>
```

```csharp
using System.Windows;
using System.Windows.Controls;
using NovaCID.ViewModel;

namespace NovaCID.Views
{
    public partial class MusicPage : UserControl
    {
        public MusicPage()
        {
            InitializeComponent();
        }

        private void NextSong_Click(object sender, RoutedEventArgs e)
        {
            if (DataContext is MusicPageViewModel vm)
            {
                vm.NextSong();
                System.Diagnostics.Debug.WriteLine($"[Music] 下一首：{vm.CurrentAlbum.Artist} - {vm.CurrentAlbum.SongName}");
            }
        }

        private void PreviousSong_Click(object sender, RoutedEventArgs e)
        {
            if (DataContext is MusicPageViewModel vm)
            {
                vm.PreviousSong();
                System.Diagnostics.Debug.WriteLine($"[Music] 上一首：{vm.CurrentAlbum.Artist} - {vm.CurrentAlbum.SongName}");
            }
        }
    }
}
```
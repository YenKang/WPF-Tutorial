public void OnLeftKnobRotated(int delta)
{
    if (delta > 0)
    {
        NextSong();
    }
    else if (delta < 0)
    {
        PreviousSong();
    }

    // 設定圖片路徑（你的 icon 檔名請自行調整）
    LeftKnobInnerImage = "/Assets/switch_song_icon.png";
    IsLeftKnobInnerVisible = true;

    // 自動延遲 500ms 消失
    Task.Delay(500).ContinueWith(_ =>
    {
        IsLeftKnobInnerVisible = false;
    }, TaskScheduler.FromCurrentSynchronizationContext());
}

<Canvas x:Name="leftKnobInner"
        Width="2560"  <!-- 整個畫布總寬，可依實際螢幕調整 -->
        Height="1440"> <!-- 整個畫布總高，可依實際螢幕調整 -->

    <!-- Knob 背景，可依實際需求放 -->
    <!-- <Ellipse Fill="Gray" Width="300" Height="300" Canvas.Left="..." Canvas.Top="..."/> -->

    <!-- 切換歌曲 icon -->
    <Image Source="{Binding LeftKnobInnerImage}"
           Width="220"
           Height="220"
           Visibility="{Binding IsLeftKnobInnerVisible, Converter={StaticResource BoolToVis}}"
           Canvas.Left="313"
           Canvas.Top="1020"/>
</Canvas>
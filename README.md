xmlns:conv="clr-namespace:OSDIconFlashMap.Converter"

<Window.Resources>
  <!-- bool → "1"/"0"；你已有可跳過 -->
  <conv:BoolTo01Converter x:Key="Bool01"/>
  <!-- bool → 顏色（綠/灰）。若尚未建立 OsdEnColorConverter.cs，照我之前給的檔案新增一個 -->
  <conv:OsdEnColorConverter x:Key="OsdColor"/>
</Window.Resources>
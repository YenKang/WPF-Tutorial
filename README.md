xmlns:services="clr-namespace:YourProjectNamespace.Services"
xmlns:conv="clr-namespace:YourProjectNamespace.Converters"


<Window.Resources>
    <conv:KnobRoleToColorConverter x:Key="KnobRoleToColorConverter"/>
    <conv:RoleToOpacityConverter x:Key="RoleToOpacityConverter"/>
</Window.Resources>

<Canvas>
    <Ellipse
        Width="372"
        Height="372"
        Canvas.Left="1949"  <!-- ⚠️ 右 Knob 的座標，依實際調整 -->
        Canvas.Top="922"
        Stroke="Transparent"
        Fill="Transparent">
        <Ellipse.Effect>
            <DropShadowEffect 
                Color="{Binding Source={x:Static services:KnobGlowStatus.Instance}, Path=RightKnobRole, Converter={StaticResource KnobRoleToColorConverter}}" 
                BlurRadius="60"
                ShadowDepth="0"
                Opacity="{Binding Source={x:Static services:KnobGlowStatus.Instance}, Path=RightKnobRole, Converter={StaticResource RoleToOpacityConverter}}"/>
        </Ellipse.Effect>
    </Ellipse>
</Canvas>
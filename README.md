<Ellipse.Effect>
    <DropShadowEffect
        Color="{Binding Source={x:Static services:KnobGlowStatus.Instance}, Path=RightKnobRole, Converter={StaticResource KnobRoleToColorConverter}}"
        BlurRadius="100"
        ShadowDepth="0"
        Opacity="{Binding Source={x:Static services:KnobGlowStatus.Instance}, Path=RightKnobRole, Converter={StaticResource RoleToOpacityConverter}}"/>
</Ellipse.Effect>
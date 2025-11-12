<Window
    ...
    xmlns:conv="clr-namespace:OSDIconFlashMap.Converters"
    xmlns:scm="clr-namespace:System.ComponentModel;assembly=WindowsBase">

    <Window.Resources>
        <!-- 讓摘要表從 30 → 1 顯示 -->
        <CollectionViewSource x:Key="IconSlotsDesc" Source="{Binding IconSlots}">
            <CollectionViewSource.SortDescriptions>
                <scm:SortDescription PropertyName="IconIndex" Direction="Descending"/>
            </CollectionViewSource.SortDescriptions>
        </CollectionViewSource>

        <!-- bool → "1"/"0" -->
        <conv:BoolTo01Converter x:Key="Bool01"/>
    </Window.Resources>
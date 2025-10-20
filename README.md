var types = p?.RegControlType ?? new List<string>();

IsLineExclamCursorVisible = types.Exists(t =>
    string.Equals(t?.Trim(), "lineExclamCursorControl", StringComparison.OrdinalIgnoreCase));

if (IsLineExclamCursorVisible && p?.LineExclamCursorControl != null)
    LineExclamCursorVM.LoadFrom(p.LineExclamCursorControl);
else
    LineExclamCursorVM.LoadFrom(null); // 清空
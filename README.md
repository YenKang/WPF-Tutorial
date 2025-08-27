// 類別欄位（加入兩個 Debouncer；lazy 建立，避免改 constructor）
private GlowDebouncer _leftGlowDebouncer;
private GlowDebouncer _rightGlowDebouncer;

// （可選）若 VM 沒有這些屬性就補上；若已有可略過
// bool LeftKnobGlowOn / RightKnobGlowOn
// KnobRole LeftKnobRole / RightKnobRole
// 建議 setter 內加等值判斷，避免重複刷新（可稍後再做）

private GlowDebouncer GetLeftDebouncer()
{
    if (_leftGlowDebouncer == null)
        _leftGlowDebouncer = new GlowDebouncer(TimeSpan.FromMilliseconds(180), () =>
        {
            // 到時自動關（回到 None + Glow Off）
            _vmNovaCID.LeftKnobRole = KnobRole.None;
            _vmNovaCID.LeftKnobGlowOn = false;
        });
    return _leftGlowDebouncer;
}

private GlowDebouncer GetRightDebouncer()
{
    if (_rightGlowDebouncer == null)
        _rightGlowDebouncer = new GlowDebouncer(TimeSpan.FromMilliseconds(180), () =>
        {
            _vmNovaCID.RightKnobRole = KnobRole.None;
            _vmNovaCID.RightKnobGlowOn = false;
        });
    return _rightGlowDebouncer;
}

// === 這裡是你原本的 UpdateGlowStatus，直接覆蓋即可 ===
private void UpdateGlowStatus(KnobStatus current /*, KnobStatus previous 可有可無 */)
{
    // 1) 依 KnobId 取對應 Debouncer
    var d = (current.KnobId == KnobId.Left) ? GetLeftDebouncer() : GetRightDebouncer();

    // 2) 只要偵測到「有活動」（例如 IsTouched=true / 有 Rotate / Press），就：亮起 + 重置倒數
    if (current.IsTouched /* 或者你要擴充：|| current.Press || current.RotateDelta != 0 */)
    {
        if (current.KnobId == KnobId.Left)
        {
            if (_vmNovaCID.LeftKnobRole != current.Role) _vmNovaCID.LeftKnobRole = current.Role;
            if (!_vmNovaCID.LeftKnobGlowOn) _vmNovaCID.LeftKnobGlowOn = true;
        }
        else
        {
            if (_vmNovaCID.RightKnobRole != current.Role) _vmNovaCID.RightKnobRole = current.Role;
            if (!_vmNovaCID.RightKnobGlowOn) _vmNovaCID.RightKnobGlowOn = true;
        }

        d.Bump(); // 180ms 內若沒有新活動，就會自動關
    }
    else
    {
        // 3) 沒活動：不要立刻關，啟動最後 180ms 的倒數（避免閃爍）
        d.Bump();
    }

    // （可選）如果你「有」明確的 release 事件想要立即關，就加這段：
    // if (previous?.IsTouched == true && current.IsTouched == false)
    //     d.StopNow();
}
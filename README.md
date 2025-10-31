// Step 2.6：FCNT2 寫入（若 JSON 有提供 mapping）
var f2Val = PackFcnt2FromUI();
WriteFcnt(_fc2LowTarget, _fc2HighTarget, _fc2Mask, _fc2Shift, f2Val);
// ---- pending cache (先記，再在本次 LoadFrom 結尾套用) ----
private bool _hasPendingCache;
private int _pendingTotal;
private int[] _pendingOrderIdx = new int[0];
private int _pendingFcnt1, _pendingFcnt2;
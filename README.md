Debug.WriteLine($"🔍 比對狀態：{newStatus.Id}, Role={newStatus.Role}");
Debug.WriteLine($"🔄 Counter: {oldStatus.Counter} → {newStatus.Counter}");
Debug.WriteLine($"👆 Touch: {oldStatus.IsTouched} → {newStatus.IsTouched}");
Debug.WriteLine($"🟢 Press: {oldStatus.IsPressed} → {newStatus.IsPressed}");

if (_router == null)
{
    Debug.WriteLine("❌ _router 尚未注入！");
    return;
}
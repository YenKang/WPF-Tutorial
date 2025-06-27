using System;
using System.Diagnostics;

namespace NovaCID.Knob
{
    public class KnobEventRouter : IKnobEventRouter
    {
        private readonly Func<object> _getCurrentPageViewModel;

        public KnobEventRouter(Func<object> getCurrentPageViewModel)
        {
            _getCurrentPageViewModel = getCurrentPageViewModel;
        }

        public void Route(KnobEvent e)
        {
            var vm = _getCurrentPageViewModel();

            if (vm is not IKnobHandler handler)
            {
                Debug.WriteLine($"⚠️ [Router] 當前頁面沒有實作 IKnobHandler，事件無法處理。Event = {e.Type}, Role = {e.Role}");
                return;
            }

            Debug.WriteLine($"📬 [Router] 收到事件：Type = {e.Type}, Role = {e.Role}");

            if (e.Type == KnobEventType.Rotate)
            {
                if (e.Role == KnobRole.Driver)
                {
                    Debug.WriteLine("➡️ 呼叫 OnDriverKnobRotated()");
                    handler.OnDriverKnobRotated(e);
                }
                else
                {
                    Debug.WriteLine("➡️ 呼叫 OnPassengerKnobRotated()");
                    handler.OnPassengerKnobRotated(e);
                }
            }
            else if (e.Type == KnobEventType.Press)
            {
                if (e.Role == KnobRole.Driver)
                {
                    Debug.WriteLine("➡️ 呼叫 OnDriverKnobPressed()");
                    handler.OnDriverKnobPressed(e);
                }
                else
                {
                    Debug.WriteLine("➡️ 呼叫 OnPassengerKnobPressed()");
                    handler.OnPassengerKnobPressed(e);
                }
            }
            else
            {
                Debug.WriteLine($"❓ 未知事件類型：{e.Type}");
            }
        }
    }
}
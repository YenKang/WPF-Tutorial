NovaCIDViewModel.cs
│
├── KnobEventProcessor → Parse(raw) → KnobEvent
│
└── KnobEventRouter → Route(e)
      │
      └── CurrentPageViewModel as IKnobHandler
              ├── OnDriverKnobRotated(e)
              └── OnPassengerKnobPressed(e)
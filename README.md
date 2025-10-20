feat(bist): add JSON-driven Line/Exclam/Cursor control with dynamic H/V position

• Implemented LineExclamCursorVM for lineSel/exclamBg/diagonal/cursor control  
• Added dynamic cursor position (HPos/VPos) parsing from JSON  
• Refactored ParsePositionField for low/high mask+shift extraction  
• Updated XAML to show H/V position fields conditionally (CursorEn=true)  
• Enhanced RegMap write sequence (Write8 + RMW mixed)  
• Expanded NT51365.profile.json with hPos/vPos configuration
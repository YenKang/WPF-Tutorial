RaisePropertyChanged(nameof(AsilImageSource));

        // debug 看一下
        Debug.WriteLine("==== ASIL ImageSource ====");
        foreach (var img in _asilImageSource)
        {
            int osd;
            _imageKeyToOsdIndex.TryGetValue(img.Key, out osd);
            Debug.WriteLine($"OSD#{osd:00} -> {img.DisplayName} -> {img.Key}");
        }
        Debug.WriteLine("==== END ====");
string[] parts = input.Split('=');
bool isDriver = parts.Length == 2 && parts[0].Trim() == "Driver" && bool.TryParse(parts[1].Trim(), out var result) ? result : false;
public static ProfileLoadResult<ProfileRoot> LoadV2(string path, IDictionary<string,string> registers = null)
{
    if (!File.Exists(path))
        throw new FileNotFoundException("Profile json not found.", path);

    var json = File.ReadAllText(path);
    var root = Newtonsoft.Json.JsonConvert.DeserializeObject<ProfileRoot>(json);
    if (root == null) throw new InvalidDataException("Failed to deserialize profile.");

    var result = new ProfileLoadResult<ProfileRoot> { Profile = root };

    // 輕量校驗
    Validate(root, registers, result.Warnings);

    return result;
}
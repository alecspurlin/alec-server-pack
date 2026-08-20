# Minecraft 26.2 Resource Pack Notes

This repository publishes the server resource pack used by Alec's Minecraft server.

## Working pack

- Published archive: `AlecServerPack26_2.zip`
- Minecraft version: 26.2
- Resource-pack format: 88
- Server download URL: `https://raw.githubusercontent.com/alecspurlin/alec-server-pack/master/AlecServerPack26_2.zip`
- The ZIP must contain `pack.mcmeta` and `assets/` directly at its root. Do not put them inside an extra folder.

## Problems encountered and fixes

### Wrong texture directory

The old pack used:

`assets/minecraft/textures/blocks/burnt_torch.png`

Minecraft 26.2 expects the singular `block` directory. The final pack directly overrides the vanilla off-redstone-torch texture at:

`assets/minecraft/textures/block/redstone_torch_off.png`

Directly overriding the vanilla texture avoids unnecessary model JSON files. Both floor and wall off-redstone-torch models already reference this texture.

### Incomplete or outdated `pack.mcmeta`

The working 26.2 metadata uses format 88:

```json
{
  "pack": {
    "pack_format": 88,
    "min_format": 88,
    "max_format": 88,
    "description": "Alec's Burnt Torch - Minecraft 26.2"
  }
}
```

Do not add `supported_formats`. Minecraft reports that key as deprecated starting with pack format 65 and marks the pack incompatible.

### Windows ZIP path-separator bug

Do not build the published ZIP with PowerShell `Compress-Archive`. On this machine it stored asset entries with Windows backslashes, such as:

`assets\minecraft\textures\block\redstone_torch_off.png`

Minecraft could read `pack.mcmeta`, but ignored those asset entries. This made the server pack appear loaded while vanilla textures remained visible.

Build the ZIP with the JDK `jar` tool so entries use forward slashes:

```powershell
& 'C:\Program Files\Java\jdk-26\bin\jar.exe' `
  --create `
  --file 'AlecServerPack26_2.zip' `
  --no-manifest `
  -C 'PATH_TO_UNPACKED_PACK' .
```

Expected entry:

`assets/minecraft/textures/block/redstone_torch_off.png`

### GitHub Pages caching

GitHub Pages continued serving an older ZIP after commits were pushed, and a renamed file temporarily returned 404 while Pages deployment lagged. The server therefore uses the repository's `raw.githubusercontent.com` URL, which was verified to serve the current committed file immediately.

### Server cache and configuration

Whenever the ZIP changes:

1. Calculate its SHA-1 hash.
2. Set `resource-pack-sha1` in `server.properties` to that hash.
3. Assign a new UUID to `resource-pack-id` so clients cannot reuse an older cached pack.
4. Fully restart the server; changing `server.properties` while it is running is not enough.
5. Reconnect and confirm the client log shows the new pack ID and hash.

## Local testing and diagnostics

- Test an unpacked pack folder in `.minecraft/resourcepacks` first. This eliminates ZIP-format problems.
- Use `F3+T` to reload enabled packs.
- Check `.minecraft/logs/latest.log` for metadata errors, missing textures, download failures, and mismatched hashes.
- Confirm the pack is enabled in `.minecraft/options.txt` under `resourcePacks`.
- For this texture, test a placed, powered-off redstone torch. Lit redstone torches and the inventory item use different resources and remain vanilla.


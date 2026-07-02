# 🔊 BetterSound

> A master audio module for Roblox. 3D positional audio, tweened fades, adaptive music, cue points, ducking, pooling, reverb zones, ambient soundscapes, and more — through one consistent API.

---

## What is it?

Roblox's built-in `Sound` instance covers the basics — play, stop, loop. Everything past that gets rebuilt from scratch in every project. BetterSound packages all of that into one drop-in module so you can focus on your game's sound *design* instead of your sound *engine*.

No more copy-pasting the same fade-in helper. No more "why are there 40 footstep Sound instances alive at once." No more manually wiring `SoundService.AmbientReverb` every time a player walks into a cave.

---

## Features

| Category | What's included |
|---|---|
| **Playback** | 2D & 3D via `Play` / `PlaySpatial`, sound pooling, priority-based channel limiting |
| **Fading** | Tween-driven fade in/out with a built-in easing library, no TweenService dependency |
| **Mixing** | SoundGroup buses (Music/SFX/Ambient/Voice/UI), auto-ducking, manual crossfading |
| **Music** | Beat-locked adaptive stem layers, shuffled/repeating playlists with crossfade |
| **Environment** | Reverb zones, raycast-based occlusion muffling, ambient soundscape zones |
| **Sync** | Cue points, `WaitForTimestamp`, dialogue/subtitle sync |
| **Workflow** | Central SoundBank registry, preloading, per-player volume settings w/ DataStore |
| **Network** | Replicated "play for client(s)" helper |
| **Debug** | Draggable, filterable live overlay showing every active sound |

---

## Installation

1. Copy the setup script from the [Releases](#) page
2. Open Studio's **Command Bar** (`View → Command Bar`) and run it once
3. It builds the full module tree under `ReplicatedStorage` — safe to re-run, it always rebuilds clean

---

## Quick Start

```lua
-- Server Script (ServerScriptService) AND LocalScript (StarterPlayerScripts)
-- Both sides need to call Init()

local BetterSound = require(game.ReplicatedStorage:WaitForChild("BetterSound"))
BetterSound:Init()

-- Register your sounds once, reference them by name everywhere
BetterSound.SoundBank.Register("Explosion", {
    Id     = "rbxassetid://9120386436",
    Volume = 0.8,
    Group  = "SFX",
})

-- Play it
local handle = BetterSound:Play("Explosion", {
    Pooled   = true,
    Priority = "High",
})

-- Fade it out after 3 seconds
task.delay(3, function()
    handle:FadeOut(1.5)
end)
```

---

## Demo

### Basic 2D Playback
```lua
BetterSound:Play("UI_Click")
```

### Fade In / Out
```lua
local handle = BetterSound:Play("Music_Calm", { FadeIn = 2, Volume = 0.7 })

task.delay(6, function()
    handle:FadeOut(3) -- fades over 3s, stops and cleans up automatically
end)
```

### 3D Spatial Audio
```lua
-- At a fixed world position
BetterSound:PlaySpatial("Explosion", Vector3.new(10, 5, 0))

-- Attached to a moving part
BetterSound:PlaySpatial("Engine_Loop", somePart, { Looped = true })
```

### Cue Points & WaitForTimestamp
```lua
local handle = BetterSound:Play("BossTheme")

-- Fires a callback the moment playback crosses 12.5s
handle:AddCue(12.5, function()
    -- trigger a cutscene, spawn an enemy, flash the screen
end)

-- Or yield a coroutine until a timestamp is hit
task.spawn(function()
    handle:WaitForTimestamp(30)
    print("Phase 2 starts now")
end)
```

### Ducking (auto-lower music during dialogue)
```lua
-- Duck the Music bus to 30% for 4 seconds, then auto-restore
BetterSound.Ducking.Duck(BetterSound.Groups, "Music", 0.3, 0.3, 4)
BetterSound:Play("Voice_Intro", { Group = "Voice" })
```

### Adaptive / Layered Music
```lua
-- All stems play in sync underneath; crossfade between them on demand
local music = BetterSound:CreateMusicLayers({
    Calm  = calmSoundInstance,
    Tense = tenseSoundInstance,
})

music:SetLayer("Calm", 0)   -- start instantly

task.delay(10, function()
    music:SetLayer("Tense", 2)  -- beat-locked 2s crossfade when combat starts
end)
```

### Shuffled Playlist
```lua
local playlist = BetterSound:CreatePlaylist(
    { "Track1", "Track2", "Track3" },
    { Shuffle = true, RepeatMode = "All", CrossfadeTime = 2 }
)
playlist:Play()
```

### Reverb Zones
```lua
-- Tag any BasePart as a reverb region
BetterSound:AddReverbZone(workspace.CaveRegion, Enum.ReverbType.Cave)

BetterSound:StartReverbTracking(function()
    return player.Character.HumanoidRootPart.Position
end)
```

### Ambient Soundscapes
```lua
-- Randomly fires one-shot sounds at random positions inside a region
BetterSound:AddAmbientZone({
    Region      = workspace.ForestRegion,
    Sounds      = { "Bird_Call", "Wind_Rustle" },
    MinInterval = 4,
    MaxInterval = 12,
    VolumeRange = { 0.3, 0.6 },
})
BetterSound:StartAmbientZone(workspace.ForestRegion)
```

### Occlusion (muffling through walls)
```lua
local handle = BetterSound:Play("Machine_Hum", { Volume = 0.8 })

BetterSound.Occlusion.Apply(
    handle:GetSound(),
    function() return character.HumanoidRootPart.Position end,
    function() return machinePart.Position end
)
```

### Dialogue & Subtitles
```lua
BetterSound.Dialogue.SubtitleShown:Connect(function(text, duration)
    mySubtitleLabel.Text = text  -- hook this to your own UI
end)

BetterSound.Dialogue.PlayLine(voiceSound, {
    { time = 0,   text = "Hey, welcome to the facility.",   duration = 2.5 },
    { time = 2.5, text = "Mind the floor — it's slippery.", duration = 2   },
})
```

### Volume Settings (DataStore-backed)
```lua
-- Set sliders
BetterSound.Settings.SetVolume("Music", 0.6, BetterSound.Groups)
BetterSound.Settings.SetVolume("SFX",   0.8, BetterSound.Groups)

-- Save to DataStore (yields - task.spawn if needed)
task.spawn(BetterSound.Settings.Save)

-- Load on next session
task.spawn(function()
    BetterSound.Settings.Load(BetterSound.Groups)
end)
```

### Networked Playback
```lua
-- Server: play a sound for every connected client
BetterSound.Network.PlayForAll("Explosion", { Priority = "High" })

-- Client: receive and play it
BetterSound.Network.OnReceive(function(soundName, opts)
    BetterSound:Play(soundName, opts)
end)
```

### Debug Overlay
```lua
-- Client only - shows a draggable, filterable live overlay of every
-- active sound, its group, volume, and playback position
BetterSound.Debug.Show()
```

---

## Module Structure

```
ReplicatedStorage
└── BetterSound          ← require this
    ├── Signal           lightweight event emitter
    ├── Tween            property tweening, no TweenService dependency
    ├── Pool             reusable Sound instance pool
    ├── SoundBank        name → config registry
    ├── Groups           SoundGroup bus management
    ├── Spatial          3D positioning, rolloff curves, doppler
    ├── Ducking          auto-duck one group while another plays
    ├── Crossfade        fade between two Sound instances
    ├── Timestamps       cue points, WaitForTimestamp
    ├── Occlusion        raycast-based muffling
    ├── Music            adaptive/layered stem manager
    ├── Playlist         shuffle/repeat queue with crossfade
    ├── Dialogue         voice lines + subtitle sync
    ├── Preloader        ContentProvider wrapper
    ├── Settings         per-player prefs with DataStore save/load
    ├── Network          replicated play helper
    ├── Priority         channel-limiting / sound stealing
    ├── Reverb           zone-based environment reverb
    ├── Ambient          randomized ambient soundscape zones
    └── Debug            draggable live overlay
```

---

## Notes

- A **server Script** must call `BetterSound:Init()` for `Network` and `Settings` (DataStore) to work. Client-only features (playback, fades, music, debug overlay) work fine without it.
- `SoundBank` is per Lua VM — register sounds on both server and client if both sides call `:Play()` directly.
- Reverb and Ambient use a **single global listener** — fine for most games, not designed for true per-player split-screen.
- DataStore persistence requires **Enable Studio Access to API Services** (`Game Settings → Security`) to test locally.

---

## License

MIT — free to use in any project, commercial or not. Credit appreciated but not required.

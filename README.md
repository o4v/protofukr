# protofukr

A lo-fi destruction VST3 effect for FL Studio (and any VST3 host). It combines warping and time-stretching, 2000s-internet codec emulation, tape hiss and transport, transient shaping, compression and two groovebox-style insert FX slots in one window. Every control is a host parameter, so FL Studio can automate all of it.

## Get the plugin (Windows)

You don't need a compiler. GitHub builds it for you:

1. Create a new repository on GitHub and upload this whole folder to it (keep the `.github` folder).
2. Open the repo's **Actions** tab. The *Build protofukr (Windows VST3)* job starts automatically; if it doesn't, click **Run workflow**.
3. When it turns green (about 10–15 min), open the run and download **protofukr-windows-vst3** from *Artifacts*.
4. Unzip it and copy `protofukr.vst3` into `C:\Program Files\Common Files\VST3\`.
5. In FL Studio: **Options ▸ Manage plugins ▸ Find more plugins**. protofukr then appears under *Effects*.

If you push a tag such as `v1.0.0`, the zip is also attached to a GitHub Release.

To build it yourself instead: install Visual Studio 2022 (with the "Desktop development with C++" workload) and CMake, then run:

```
cmake -S . -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release --target protofukr_VST3
```

The output is `build/protofukr_artefacts/Release/VST3/protofukr.vst3`.

## Signal flow

```
IN gain → WARP → transient shaper → compressor → FX 1 → FX 2 → TAPE ⇄ LOFI (order switchable) → width / mono → OUT gain
                                                                        (global MIX blends the latency-compensated dry signal)
```

## Sections

### WARP

These are original implementations of the ideas behind Ableton Live's warp modes. No Ableton code is used.

| Algorithm | What it does |
|---|---|
| **Beats** | Plays slices between transients (or 1/16, 1/8, 1/4 grid slices). **Transient Loop** controls what happens when a slice ends early: *Loop Off* goes silent, *Loop Forward* loops the slice's tail forward, *Loop Back-and-Forth* bounces it back and forth. **Envelope** fades each slice, for gated stutters. |
| **Tones** | Pitch-preserving granular stretch. **Grain** sets the grain size. |
| **Texture** | Four-voice granular stretch. **Flux** randomises grain position and length. |
| **Re-Pitch** | Varispeed, where pitch follows speed. Use this for the screwed, tape-slowed sound. |
| **Complex** | Phase-vocoder stretch with transient phase reset. **Pitch** transposes independently of speed. |

There are two sources:

- **Capture** records the live input. Every *Capture Length* (synced to FL's grid), playback restarts at "now" and plays at **Speed**. At 0.75× you get a chopped-and-screwed slowdown that stays in time. **Freeze** loops the last window. **Direction** can be Forward, Reverse or Ping-Pong.
- **Sample** plays a file you load. Drag a WAV/AIFF/FLAC/OGG/MP3 onto the plugin or click *LOAD SAMPLE*. It is locked to FL's transport at Speed × tempo, and plays only while FL plays unless **Free Run** is on. **Smp BPM** is guessed on load; type the real value if the guess is wrong.

### LOFI

Codec types:

- **ZOH**: no-filter sample-and-hold (aliasing crunch). Zero latency.
- **Resample**: band-limited converter (dull, smooth). Zero latency.
- **MP3 '08**: an MDCT coder modelled on low-bitrate MP3 and 2008-era YouTube FLV audio. It produces swirly highs, pre-echo, spectral holes and the encoder low-pass. *YouTube '08 SQ* = 22.05 kHz / 64 kbps / mono.
- **AAC**: the higher `fmt=18` tier. Smoother and wider band.
- **Real '99**: coarse-band streaming codec (metallic, narrow).
- **GSM Phone**: 300–3400 Hz, mu-law crunch.

The **Rate** (500 Hz–48 kHz), **Bitrate** (8–320 kbps) and **Bits** (1–24) faders have typeable readouts: click the green number and type an exact value (e.g. `22050` or `22.05k`). The ▼ menus next to them offer common values. **Swirl** adds time-varying "underwater" allocation artifacts. **Frame** sets the block size: longer frames smear more; *Low Latency* is about 6 ms. MDCT codecs report their latency to FL, so plugin delay compensation keeps everything in sync.

### TAPE

Hiss voicings:

- Cheap Ferric (UR-ish, bright and gritty)
- Ferric Normal
- Chrome CrO2
- Metal
- Microcassette
- VHS Linear (with head-switch buzz)
- CRT Broadcast (static plus 15.734 kHz flyback whine)
- Vinyl Dust

**Hiss Sidechain** sets what the hiss level follows:

- Constant
- All Input
- Duck Input (hiss rises in the gaps)
- Low Band (kick)
- High Band (hats)
- Sidechain Bus (the plugin's external sidechain input)

**Transport** adds wow, flutter and tape drive.

### DYNAMICS

**Transient** modes:

- Sharp (crisp high band)
- Fat (low band plus saturation)
- Balanced
- Punch
- Snap
- Smack

Each mode has Attack, Sustain and Focus controls.

The **Compressor** has six styles: Clean VCA, Opto, FET, Glue, Brick and Pump. *PRESETS* offers: Screw Tape Glue, Soundcloud Smash, Lofi Bus, Vapor Pump, Drum Punch, Parallel NY Crush, Gentle Opto, Vocal Sit and Boom-Bap Knock.

### INSERT FX (×2)

These are groovebox-style performance effects in the spirit of the SP-404 / SP-202 banks:

| Group | Effects |
|---|---|
| EQ and filters | Isolator, Filter+Drive, Super Filter, Auto Wah |
| Lo-fi and sims | Lo-Fi, Vinyl Sim A/B, Cassette Sim, Crusher |
| Performance | Stutter (beat-synced repeat with chance and gate), DJ Looper (−2× to +2×, reverse), Downer, Tape Stop / Back Spin |
| Delay and reverb | Sync Delay, Tape Echo, Reverb |
| Modulation | Chorus, Flanger, Phaser, Slicer, Tremolo/Pan |
| Tone and pitch | Ring Mod, Resonator, Overdrive, Distortion, Pitch Shift |

Each effect has three macro controls (A/B/C), which are relabelled per effect, plus Balance. Stutter, DJ Looper and Tape Stop are made to be automated: for example, automate *Hold* or *Trigger* in FL.

### Global

- Stereo / Mono switch, Width (0–200 %)
- In / Out gain
- Global Mix (dry is delay-compensated)
- Chain order: Tape → Codec (hiss gets crushed, like a tape ripped to YouTube) or Codec → Tape

## Factory presets

Init, YouTube '08 SQ, YouTube '08 HQ, Signalwave Broadcast, Mediafire Rip, Plaza Slowdown, Chopped & Screwed, Maxell Tape Dub, Soundcloud Smash, Late Night Sleepcore, Glitter Static, Mystic Dust (Texture), Mallsoft Hallway, RealPlayer '99, Phone Call, Beats Stutter Gate, Tape Stop Throw.

The presets also appear in FL's preset menu on the plugin wrapper.

## FL Studio tips

- **Automation:** move any control, then use *Tools ▸ Last tweaked ▸ Create automation clip*, or right-click it in the wrapper's *Browse parameters* list. All 69 parameters are exposed.
- **Sidechain the hiss:** protofukr has a stereo *Sidechain* input. Route the source track to protofukr's mixer insert as a sidechain, assign it to the plugin's sidechain input in the wrapper's processing/port settings, then set *Hiss Sidechain* to *Sidechain Bus*.
- **Latency:** the MP3/AAC/Real/GSM codecs use 6–93 ms of look-behind, depending on Frame. It is reported to FL, so leave PDC on.
- **CPU and RAM:** everything is pre-allocated when playback starts. Nothing allocates on the audio thread. The capture buffer is about 10 s, and loaded samples are capped at 10 min.

## The secret

Click the plugin window and type `qaeru`. If FL's *Typing keyboard to piano* is on and steals the keys, toggle it off first.

## Credits

Fonts: Dela Gothic One and VT323 (SIL Open Font License; see `Resources/OFL-*.txt`). Built with JUCE 8.

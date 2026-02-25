# SF2 to SFZ Converter

This Python tool converts SoundFont 2 (SF2) files into SFZ format. It extracts sample and instrument data from an SF2 file, then creates a separate SFZ file for each preset along with dedicated sample folders. The exported regions include detailed information such as pitch key center, tuning, key and velocity ranges, volume envelopes, modulation envelopes, LFOs, filters, pan, attenuation, effects sends, exclusive classes, and loop settings.

I made this because I kept finding SF2 files that I wanted to use in Serum 2. You can copy the folder it outputs directly into your `Documents\Xfer\Serum 2 Presets\Multisamples\User` folder, and it should work.

The script is fully standalone -- no third-party libraries needed, just Python 3. It parses the SF2 binary format directly and maps all SF2 generator parameters to their correct SFZ equivalents, aiming for 1:1 sound fidelity with the original soundfont.

If you want a feature, add it and make a pull request.

## Features

- **Standalone:**
  No external dependencies. The SF2 binary parser is built-in -- just Python 3 and the standard library.
- **Preset-Specific Output:**
  For each preset, the tool creates:
  - A dedicated sample folder named:
    `<OUTPUT_BASE> <PresetName> Samples`
  - A separate SFZ file named:
    `<OUTPUT_BASE> <PresetName>.sfz`
- **Correct SF2-to-SFZ Parameter Mapping:**
  All SF2 generator parameters are converted with proper unit conversions verified against the SF2 2.01 spec and FluidSynth:
  - Volume envelope (ampeg_attack/hold/decay/sustain/release)
  - Modulation envelope mapped to both filter and pitch envelopes (fileg_* and pitcheg_*)
  - LFO parameters (amplfo, fillfo, pitchlfo -- delay, freq, depth)
  - Filter cutoff and resonance (with fil_type=lpf_2p)
  - Initial attenuation (volume in dB)
  - Pan (-100 to +100)
  - Reverb/chorus effects sends
  - Coarse/fine tuning and sample pitch correction
  - Scale tuning (pitch_keytrack)
  - Exclusive class (group/off_by for drum muting groups)
  - Loop mode, start, and end points with offset support
- **Generator Layering:**
  Properly merges generators across all four SF2 zone levels (instrument global, instrument split, preset global, preset split) per SF2 spec Section 9.4.
- **ROM Sample Handling:**
  SF2 files that reference ROM samples (hardware-resident waveforms with no embedded PCM) are handled gracefully -- ROM samples are skipped with a warning, and presets that only use ROM samples are cleaned up automatically.
- **Sample Naming:**
  Samples are exported with filenames using the true (sanitized) sample names from the SF2 file. Duplicate names are automatically disambiguated.
- **Region Tags:**
  Each SFZ region includes:
  - `sample`: The exported sample file name.
  - `pitch_keycenter`: Derived from the overriding root key generator if available; otherwise, the sample's original pitch.
  - `tune`: Computed from fine tuning generator plus sample pitch correction.
  - `transpose`: From coarse tuning generator (semitones).
  - `lokey` and `hikey`: The key range of the region (or `key` for single-key drum mappings with `lochan=10 hichan=10`).
  - `lovel` and `hivel`: The velocity range of the region (when not the full 0-127 default).
  - Loop parameters if the sample is flagged for looping:
    `loop_mode` (loop_sustain or loop_continuous), `loop_start`, `loop_end`.
- **Control Section:**
  Each SFZ file begins with a `<control>` block setting `default_path` to the preset's sample folder (as a relative path).

## Requirements

- Python 3.x

That's it. No pip installs needed.

## Usage

Place the `sf2-to-sfz.py` script in your working directory and run:

```bash
python sf2-to-sfz.py input.sf2 output
```

Where:
- `input.sf2` is the path to your SF2 file.
- `output` is the base output name. The script uses this for naming the base folder, sample folders, and SFZ files.

For example, if you run:

```bash
python sf2-to-sfz.py mySound.sf2 mySound
```

The tool will create a base folder named `mySound` containing:
- Sample folders like `mySound Instrument1 Samples/`, `mySound Instrument2 Samples/`, etc.
- SFZ files like `mySound Instrument1.sfz`, `mySound Instrument2.sfz`, etc.

## Output Structure

After running the tool, your output directory structure will look similar to:

```
mySound/
├── mySound Instrument1 Samples/
│    ├── mySound-Instrument1-TrueSampleName.wav
│    └── ...
├── mySound Instrument1.sfz
├── mySound Instrument2 Samples/
│    └── ...
└── mySound Instrument2.sfz
```

Each SFZ file contains region definitions like:

```sfz
// Instrument1
// Converted from SF2 to SFZ by bash explode

<control>
default_path=mySound Instrument1 Samples/

<region>
sample=mySound-Instrument1-TrueSampleName.wav
lokey=21 hikey=108
pitch_keycenter=60
ampeg_attack=0.030012
ampeg_hold=1.149362
ampeg_decay=28.246496
ampeg_sustain=97.7237
ampeg_release=1.105731
cutoff=578.92
fil_type=lpf_2p
volume=-7.40
effect1=37.0
tune=35
loop_mode=loop_sustain
loop_start=53
loop_end=12955
```

## Customization

- **Mapping Generators:**
  The SF2-to-SFZ conversion logic lives in the `generators_to_sfz_opcodes()` function. You can modify how SF2 generator values map to SFZ opcodes by editing that function.
- **Loop Settings:**
  Loop parameters (mode, start, end) are automatically added if the sample is flagged for looping. Loop points include fine and coarse offset support and are clamped for safety.
- **Additional Parameters:**
  Extend the region block in `process_preset()` to include more parameters if desired.

## License

This tool is provided "as-is" without any warranty. Feel free to modify and distribute it as needed.

## Acknowledgments

- SoundFont 2.01 Technical Specification for the binary format details.
- [FluidSynth](https://github.com/FluidSynth/fluidsynth) source code for verifying unit conversion formulas.
- SFZ format documentation at [sfzformat.com](https://sfzformat.com/) for opcode reference.

---

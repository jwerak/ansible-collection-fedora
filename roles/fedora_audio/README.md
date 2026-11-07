fedora_audio
============

Makes laptop speakers usable on Fedora. Two independent halves:

1. **ALSA mixer controls** reapplied on every boot via a system-wide systemd
   unit.
2. **EasyEffects** installed, with an output preset and a systemd *user* unit
   that loads it at login.

Background
----------

Some mixer controls exposed by SOF topologies are off by default and are only
remembered by `alsactl` on a best-effort basis: `alsa-state.service` saves
whatever the state happened to be at shutdown, so a single stray toggle makes
the setting disappear permanently. This role makes the desired values
authoritative instead.

The motivating case is the ThinkPad X1 Carbon Gen 12 (Realtek ALC287 on
`sof-hda-generic-2ch.tplg`), where `Post Mixer Analog Playback DRC switch`
defaults to `off`. Enabling that compressor helps, but the larger gap is that
Linux plays the raw signal while Windows and macOS apply heavy per-model DSP.
EasyEffects closes most of the rest.

Requirements
------------

Fedora with `systemd` and `dnf`. The role installs `alsa-utils` and, when
EasyEffects is enabled, `easyeffects`.

The EasyEffects half writes into the user's home and manages a systemd user
unit, so it must run **unprivileged** with facts gathered — the role escalates
per task for the parts that need root. Import it from a play that does not set
`become: true` globally.

Role Variables
--------------

### ALSA mixer controls

| Variable | Default | Description |
| --- | --- | --- |
| `fedora_audio_card` | `sofhdadsp` | ALSA card id as listed in `/proc/asound/cards`. |
| `fedora_audio_mixer_controls` | `[]` | Controls to reapply at boot. Empty means this half does nothing. |
| `fedora_audio_unit_name` | `alsa-mixer-restore.service` | Name of the generated system unit. |

Each entry of `fedora_audio_mixer_controls` takes:

- `name` — exact simple-control name from `amixer -c <card> scontrols`
- `value` — argument passed to `amixer sset`, such as `"on"`, `"off"`, `"100%"`

Quote `"on"` and `"off"`, otherwise YAML parses them as booleans.

Controls that the running hardware does not expose are filtered out before the
unit is written, so the same variable can be set for a group containing
machines with different sound hardware. If nothing matches, no unit is
installed.

### EasyEffects

| Variable | Default | Description |
| --- | --- | --- |
| `fedora_audio_easyeffects_enabled` | `false` | Install EasyEffects and deploy the preset plus user unit. |
| `fedora_audio_easyeffects_preset` | `X1Carbon-Speakers` | Preset base name; `files/<name>.json` must exist. |
| `fedora_audio_easyeffects_unit` | `easyeffects-preset.service` | Name of the generated systemd user unit. |

Because presets are tuned to a specific chassis, scope this by hardware rather
than by host:

```yaml
fedora_audio_easyeffects_enabled: >-
  {{ ansible_facts['product_version'] | default('') is search('X1 Carbon Gen 12') }}
```

Notes on the preset
-------------------

The preset format is version-specific and fails quietly. EasyEffects 8.x reads
plugin settings at `json["output"]["<plugin>#<n>"]` — there is no `plugins`
wrapper — and `plugins_order` is parsed separately. A wrong layout still builds
the plugin chain from `plugins_order` while every parameter silently falls back
to its default, which is indistinguishable from success unless the values are
checked. Watch the journal for:

```
presets_manager.cpp: A parsing error occurred while trying to load the <plugin> plugin
```

Other traps: band quality is `q`, not `quality`; `.at("left")`, `.at("right")`
and `.at("bandN")` throw when absent, so all 32 bands are emitted; the
compressor has a nested `sidechain` object while the limiter's sidechain keys
are flat siblings.

EasyEffects restores the plugin *chain* from its own db but not the parameter
*values*, which live only in the preset file. The user unit therefore loads the
preset explicitly once `easyeffects_sink` exists, otherwise the chain comes up
flat at every login.

Filter nodes (`ee_soe_equalizer` and friends) only exist while audio is
playing; an empty chain in `pw-cli ls Node` on an idle machine is normal.

Example Playbook
----------------

```yaml
- name: Setup user config
  hosts: laptops
  gather_facts: true
  tasks:
    - name: Configure audio
      ansible.builtin.import_role:
        name: jwerak.fedora.fedora_audio
```

With group vars:

```yaml
fedora_audio_card: sofhdadsp
fedora_audio_mixer_controls:
  - name: Post Mixer Analog Playback DRC switch
    value: "on"
fedora_audio_easyeffects_enabled: true
```

License
-------

GPL-2.0-or-later

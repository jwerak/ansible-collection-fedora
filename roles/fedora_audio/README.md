fedora_audio
============

Reapplies ALSA mixer controls on every boot via a generated systemd unit.

Some mixer controls exposed by SOF topologies are off by default and are only
remembered by `alsactl` on a best-effort basis: `alsa-state.service` saves
whatever the state happened to be at shutdown, so a single stray toggle makes
the setting disappear permanently. This role makes the desired values
authoritative instead.

The motivating case is the ThinkPad X1 Carbon Gen 12 (Realtek ALC287 on
`sof-hda-generic-2ch.tplg`), where `Post Mixer Analog Playback DRC switch`
defaults to `off`. Enabling the compressor noticeably raises perceived
loudness from the internal speakers.

Requirements
------------

Fedora with `systemd` and `dnf`. The role installs `alsa-utils` itself.

Role Variables
--------------

| Variable | Default | Description |
| --- | --- | --- |
| `fedora_audio_card` | `sofhdadsp` | ALSA card id as listed in `/proc/asound/cards`. |
| `fedora_audio_mixer_controls` | `[]` | Controls to reapply at boot. Empty means the role does nothing. |
| `fedora_audio_unit_name` | `alsa-mixer-restore.service` | Name of the generated unit. |

Each entry of `fedora_audio_mixer_controls` takes:

- `name` — exact simple-control name from `amixer -c <card> scontrols`
- `value` — argument passed to `amixer sset`, such as `"on"`, `"off"`, `"100%"`

Quote `"on"` and `"off"`, otherwise YAML parses them as booleans.

Controls that the running hardware does not expose are filtered out before the
unit is written, so the same variable can be set for a group containing
machines with different sound hardware. If nothing matches, no unit is
installed.

Example Playbook
----------------

```yaml
- name: Setup Fedora machine
  hosts: laptops
  become: true
  tasks:
    - name: Configure audio
      ansible.builtin.import_role:
        name: jwerak.fedora.fedora_audio
```

With group or host vars:

```yaml
fedora_audio_card: sofhdadsp
fedora_audio_mixer_controls:
  - name: Post Mixer Analog Playback DRC switch
    value: "on"
```

License
-------

GPL-2.0-or-later

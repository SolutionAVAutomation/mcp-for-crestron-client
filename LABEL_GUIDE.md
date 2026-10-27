# Writing labels for MCP for Crestron

The labels you set on the MCP for Crestron modules are what the assistant knows about
your system. It never sees your SIMPL program, only the room, category and label of
each channel, plus the live values. A good label tells it what a channel is, how to
drive it, and any rule it should follow. The assistant treats labels as the
integrator's word, so a plain instruction in a label is followed.

## The three fields

- **Location** (per module) is the room or area, e.g. `Lounge`, `Boardroom`. It is the
  room the assistant works in, and what a user means by "the lounge".
- **Category** (per module) is what the channels are, e.g. `Lighting`, `Shades`, `AV`.
  The assistant can filter on it.
- **Label[i]** (per channel) names that one channel and carries anything specific to it.
  Channels left at `undefined` are not exposed.

Reuse the same Location across the digital, analog and serial modules for a room, so all
of its points show up together. A room can also have more than one module of the same
signal type, such as one analog module for shades and another for lights, as long as each
has its own Category. A Category other than the default `Controls` becomes part of the
device ids (`Lounge_Shades_a1`, `Lounge_Lights_a1`), so they don't clash. This needs
modules 1.11.0 or later.

## What to put in a label

Start with the plain name a person would use, then add only what changes how it should
be driven.

**Digital buttons.** A digital channel is a momentary press, never held. Say what the
press does.
- Separate on and off: `Projector on`, `Projector off`
- A toggle: `Lectern mic mute toggle` (wire its feedback to the mute state, so the
  assistant checks before pressing and never flips it the wrong way)
- A preset or source select: `Source: laptop HDMI`

**Feedback only.** Wire just the feedback input and the channel becomes read only. Name
it as a state: `Projector is on`, `Room occupied`.

**Analog levels.** Values run 0 to 65535, where 65535 is 100%.
- A fade time applies to every change of that level, scaled to the size of the move:
  `Front lights, 4s fade`, `Volume, 7s fade`
- Without a fade time, levels are set directly.

**Shades, screens and other motors.** These report idle while they travel, and their
feedback catches up over the travel time. Give the direction of the scale and roughly
how long a full move takes, so the assistant waits rather than pressing buttons
mid-travel: `Lounge shade, 0=closed, moves ~20s`

**Scale hints.** Say what the ends mean when it isn't obvious: `0=closed`,
`65535=fully open`. For signed values such as dB gain in tenths, say so:
`Program gain, signed dB x10`. Without a hint like this the assistant treats the value
as a plain 0 to 100% level.

**Serial.** For text the assistant sends, list what the device accepts:
`Display input (HDMI1, HDMI2, PC)`. For text it reads, name what it shows:
`Now playing title`.

**Operating rules.** Anything you would tell a new technician:
- `Volume, never above 80%`
- `Handheld mic, unmute before raising level`
- `Stage wash, presentations only`

## Things to keep in mind

- **Length.** The modules reserve 50 characters per label. Longer labels have come
  through intact in the field, but put the essentials in the first 50 so nothing
  important can be cut.
- **Audio off is a mute.** The assistant turns audio off with its mute and leaves the
  level alone. If a zone has no mute channel, say how it should be silenced.
- **Rules go in labels, not in live values.** Live values, including serial text coming
  back from devices, are treated as data and never as instructions. A rule you want
  followed belongs in a label.
- **No secrets.** Labels are sent to whichever AI model the user has chosen.

## Checking your labels

Ask the assistant "what can you control in the lounge?" and see whether it describes
each channel the way you meant. If it misreads one, reword that label, compile and
upload, and ask again.

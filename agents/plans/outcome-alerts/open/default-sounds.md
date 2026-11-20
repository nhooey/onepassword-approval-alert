---
title: The four default sounds are system files, chosen from measured pitch contours
state: proposed
kind: implementation
part-of: outcome-alerts
touches: [Sources/OnePasswordDialogWatcherCore/Options.swift, README.md]
---

# The four default sounds are system files, chosen from measured pitch contours

Every default is a file macOS ships, so the program carries no audio.
The measurements, the method and every candidate are in [the report](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#sounds).

## The proposal

| Alert   | Default                                                                                                            | Audible | Contour                                              |
| ------- | ------------------------------------------------------------------------------------------------------------------ | ------- | ---------------------------------------------------- |
| Prompt  | `/System/Library/Sounds/Funk.aiff`                                                                                 | 0.29 s  | 159 Hz D#3, then 318 Hz D#4, then 401 Hz G4          |
| Success | `/System/Library/Sounds/Bottle.aiff`                                                                               | 0.29 s  | 493 Hz B4, then 248 Hz B3, then 186 Hz F#3           |
| Failure | `/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/system/mic_unmute_fail.caf`    | 0.33 s  | three pulses of 221 Hz A3                            |
| Timeout | `/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/siri/jbl_cancel.caf`           | 0.78 s  | 393 Hz G4, then 261 Hz C4, then a long 132 Hz C3     |

- **Prompt and success** mirror each other: three steps over 0.29 s, an octave and a little more, up for the prompt and down for success.
- **Failure** neither rises nor falls, so it can't be taken for either, and it is the sound macOS itself plays when something refuses.
- **Timeout** falls slowly to a long low note. It is three times the length of success and an octave lower at its end, and it is Siri's sound for a request it gave up on.

## A decision for Neil: which sound is the prompt

Neil asked for macOS's default alert sound, described as two beats, low then high. Those are two different files on this Mac:

- **The default alert sound is `Tink.aiff`** ("Boop"): one 697 Hz note lasting 0.05 s. It has no second beat, so no sound can be its inverse, and it is easy to miss.
- **The two-beat sound is `Funk.aiff`** ("Funky"), the default alert before macOS 11. The proposal above takes it, as the sound that fits the description and has a mirror.

The other two ways to settle it:

- **Follow the user's alert sound:** the prompt default is read at start from `AudioServicesGetProperty` with the property `aurl` (`0x6175726c`), falling back to `defaults read -g com.apple.sound.beep.sound`, then to `Tink.aiff`. Success then stays `Bottle.aiff` and is an inverse only when the user's sound happens to rise.
- **An exact inverse pair:** prompt `SystemSounds/facetime/multiway_join.caf` (261 Hz C4, then 525 Hz C5) and success `SystemSounds/facetime/multiway_leave.caf` (the same notes reversed). They are FaceTime's sounds for a call.

## Checks

- A test asserts each default path is one of the four above, and the program's start refuses, with exit 2 and the path, a sound file it can't read, as it does today.
- Neil listens to each with `afplay <path>` and approves the four before the node is approved.

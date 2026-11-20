---
title: The default sounds are system files, chosen by ear from measured pitch contours
state: proposed
kind: implementation
part-of: outcome-alerts
touches: [Sources/OnePasswordDialogWatcherCore/Options.swift, README.md]
---

# The default sounds are system files, chosen by ear from measured pitch contours

Every default is a file a clean install of macOS ships under `/System`, so the program carries no audio.
The measurements, the method and every candidate are in [the report](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#sounds).

Paths below are shortened:

- `SS` is `/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds`;
- `SR` is `/System/Library/PrivateFrameworks/ScreenReader.framework/Versions/A/Resources/Sounds`.

## The defaults

| Alert                            | Default                                    | Audible | Contour                                          |
| -------------------------------- | ------------------------------------------ | ------- | ------------------------------------------------ |
| Prompt                           | `SS/facetime/multiway_join.caf`            | 0.43 s  | 261 Hz C4, then 525 Hz C5                        |
| Success                          | `SS/facetime/multiway_leave.caf`           | 0.24 s  | 525 Hz C5, then 261 Hz C4                        |
| Failure, first and third onwards | `SS/system/head_gestures_double_shake.caf` | 0.39 s  | 261 Hz C4, a double shake                        |
| Failure, second in a row         | `SR/XcodeLineError.aiff`                   | 0.26 s  | not measured                                     |
| Timeout                          | `SS/siri/jbl_cancel.caf`                   | 0.78 s  | 393 Hz G4, then 261 Hz C4, then a long 132 Hz C3 |
| Tick, each second a prompt is open | `SR/MouseDown.aiff`, at volume 0.25      | 0.02 s  | not measured                                     |

- **Prompt and success** are the same two notes an octave apart, up for the prompt and down for success.
- **Failure** is the AirPods head-shake "no", at the prompt's own low C4, so it belongs with the pair and can't be taken for either.
- **A second failure in a row** plays VoiceOver's error-line sound instead, so a retry that fails again sounds different from the first refusal. The third and later failures in a row go back to the first failure sound. A success or a timeout ends the run, and the next failure is a first again.
- **Timeout** falls slowly to a long low note, Siri's sound for a request it gave up on.
- **Tick** is a mouse-button click, played once a second from the prompt sound until the prompt ends, at a quarter of full volume so it counts the seconds without drowning out the other sounds. It stops when the prompt ends, before the outcome sound plays.

## Checks

- A test asserts each default path is one of the six above, the tick's volume is 0.25,, and the program's start refuses, with exit 2 and the path, a sound file it can't read, as it does today.
- The alerter's tests cover a run of three failures playing the first, second and first failure sounds, and a success between two failures resetting the run.

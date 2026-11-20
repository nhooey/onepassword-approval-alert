---
title: Alert sounds, prompt outcomes and the prompt timeout, measured on one Mac
state: current
as-of: nhooey/shell-skills dd41ef5 (the merge of shell-skills#21), macOS 27.2 (26B123), 1Password 8.12.46
---

# Alert sounds, prompt outcomes and the prompt timeout, measured on one Mac

This report holds the research the plans in `agents/plans/` rest on.
Everything in it was read or measured on 2026-11-20 on one Mac, without playing a sound.

## The short version

- **The default alert sound is `Tink.aiff`, and it isn't a two-beat sound.** macOS resolves the alert sound to `/System/Library/Sounds/Tink.aiff`, shown as "Boop" in System Settings. It is one 697 Hz blip that lasts 0.05 s.
- **The two-beat, low-then-high sound is `Funk.aiff`,** shown as "Funky": 159 Hz, then 318 Hz, then 401 Hz, over 0.29 s. It was macOS's default alert before macOS 11.
- **Its closest inverse is `Bottle.aiff`,** shown as "Pebble": 493 Hz, then 248 Hz, then 186 Hz, over 0.29 s. An exact inverse pair also exists: `multiway_join.caf` (261 Hz then 525 Hz) and `multiway_leave.caf` (525 Hz then 261 Hz).
- **1Password ends an unanswered prompt after 59.6 s to 60.0 s.** Its own log on this Mac holds 450 SSH prompts that timed out, 439 of them between 59.30 s and 60.00 s after they started.
- **The window list can tell a timeout, and can't tell an approval from a cancel.** Both close the prompt early. 1Password's log tells them apart for SSH prompts.

## The program today

`onepassword-dialog-watcher` is in nhooey/shell-skills at `commands/onepassword-dialog-watcher/`, added by one commit, 0babd60 "Sound an alert only while a 1Password approval dialog is up", merged in shell-skills#21.

- **Detection:** twice a second it reads `CGWindowListCopyWindowInfo` with `.optionAll`. A window is an approval prompt when 1Password owns it, it is on screen, its layer is above 0, and it isn't Quick Access.
- **Titles:** the window list leaves a window's name out without Screen Recording permission, which the program never asks for. Quick Access is then told by its 550-point width.
- **Alerts:** a new prompt plays `--first-sound` (Antic) and notifies. One still open after 60 s plays `--second-sound` (Text-Message-Acknowledgement-Exclamation) and notifies once more.
- **What it keeps:** nothing between runs. In a run it keeps each prompt's window id and when it first saw it.
- **Wiring in shell-skills:** `flake.nix` builds it from `package.nix` and runs `check-native-arm64.sh` as a check. Four source comments point at a plan node named `onepassword-dialog-watcher`, which is not in shell-skills at `origin/master`.

## Sounds

### How they were measured

- **Decoding:** `afconvert --file WAVE --data LEI16@22050 --channels 1` wrote each file as mono 16-bit audio to a temporary folder. Nothing was played.
- **Pitch over time:** a Python script with numpy, run from a temporary `nix shell`, took a 512-sample Hann window (23 ms) every 128 samples (6 ms), and in each took the strongest spectral peak between 90 Hz and 6 kHz. When a peak at a half, a third or a quarter of that frequency held at least 40% of its magnitude, the lower one was taken as the pitch.
- **Notes:** neighbouring windows within a semitone of each other form one note. Windows quieter than 5% of the file's loudest window count as silence. A note is given as its start and end in seconds, its median frequency, its note name, and its level against the file's loudest note.
- **Lengths:** "file" is the length `afinfo` reports. "Audible" is the span above the silence threshold, which is what a listener hears.
- **Limits:** the pitch of a noisy or percussive sound, such as `Basso.aiff`, is a rough figure. Each note's time is good to about 0.02 s.

167 files were measured: `/System/Library/Sounds`, every alert tone under `ToneLibrary.framework`, the system sounds under `CoreAudio.component`, and the call and lock sounds in `TelephonyUtilities.framework` and `SecurityInterface.framework`.

### Where macOS keeps alert sounds

| Folder                                                                                      | Holds                                                               |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `/System/Library/Sounds/`                                                                   | the 14 alert sounds System Settings offers                          |
| `~/Library/Sounds/`, `/Library/Sounds/`                                                     | the user's and the machine's own alert sounds                       |
| `/System/Library/PrivateFrameworks/ToneLibrary.framework/Versions/A/Resources/AlertTones/`  | 59 notification tones, in `Classic/`, `Modern/`, `EncoreInfinitum/` |
| `/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/`       | interface sounds, in `system/`, `siri/`, `facetime/`, `finder/`     |
| `/System/Library/PrivateFrameworks/TelephonyUtilities.framework/Versions/A/Resources/`      | call sounds                                                         |

Below, `Sounds/` stands for the first folder, `AlertTones/` for the third, `SystemSounds/` for the fourth and `Telephony/` for the fifth.

### The default alert sound, and how to read the user's choice

- **The setting:** the user's choice is the key `com.apple.sound.beep.sound` in the global preferences domain, holding a file path. `defaults read -g com.apple.sound.beep.sound` prints it. On this Mac the key is unset.
- **The effective sound:** `AudioServicesGetProperty` with the property `0x6175726c` (`aurl`), no specifier and an 8-byte buffer returns the alert sound's URL, whether the key is set or not. This is the call the Sound settings extension makes, read from its disassembly. On this Mac it returns `/System/Library/Sounds/Tink.aiff`. The property isn't in the public headers, so a macOS release can change it.
- **Display names:** `AlertSounds.loctable` in `/System/Library/ExtensionKit/Extensions/Sound.appex` maps file names to the names System Settings shows: Tink is "Boop", Funk is "Funky", Bottle is "Pebble", Basso is "Mezzo", Submarine is "Submerge".
- **What `Tink.aiff` is:** one note, 697 Hz (F5), audible for 0.05 s of a 0.56 s file, then digital silence. It has no second beat, so nothing can be its inverse.

Neil described the default alert as a two-beat sound, low then high.
`Tink.aiff` isn't that sound, and `Funk.aiff` is: which of the two the prompt uses is his to decide after listening.

### Low then high: candidates for the prompt

| File                                                 | File length | Audible | Contour                                                           |
| ---------------------------------------------------- | ----------- | ------- | ----------------------------------------------------------------- |
| `Sounds/Funk.aiff`                                   | 2.16 s      | 0.29 s  | 0.00–0.10 159 Hz D#3, 0.08–0.17 318 Hz D#4, 0.16–0.29 401 Hz G4   |
| `Sounds/Tink.aiff`                                   | 0.56 s      | 0.05 s  | 0.00–0.05 697 Hz F5, one note                                     |
| `Sounds/Morse.aiff`                                  | 0.70 s      | 0.12 s  | 0.00–0.09 258 Hz C4, 0.08–0.12 649 Hz E5                          |
| `SystemSounds/facetime/multiway_join.caf`            | 0.65 s      | 0.43 s  | 0.00–0.12 261 Hz C4, 0.28–0.43 525 Hz C5                          |
| `AlertTones/EncoreInfinitum/Slide-EncoreInfinitum.caf` | 1.20 s    | 0.53 s  | 0.01–0.23 293 Hz D4, 0.23–0.46 587 Hz D5, then a faint 293 Hz     |
| `SystemSounds/system/Screen Capture.aif`             | 0.73 s      | 0.36 s  | 0.00–0.18 261 Hz C4, 0.22–0.36 393 Hz G4                          |
| `SystemSounds/system/screen_sharing_started.caf`     | 1.49 s      | 0.50 s  | 0.00–0.11 350 Hz F4, 0.17–0.50 524 Hz C5                          |
| `SystemSounds/system/end_record.caf`                 | 0.43 s      | 0.20 s  | 0.00–0.10 880 Hz A5, 0.08–0.20 1174 Hz D6                         |
| `AlertTones/Text-Message-Acknowledgement-ThumbsUp.caf` | 0.50 s    | 0.17 s  | 0.00–0.10 880 Hz A5, 0.08–0.17 1760 Hz A6                         |
| `AlertTones/EncoreInfinitum/Antic-EncoreInfinitum.caf` | 0.96 s    | 0.45 s  | 0.00–0.12 369 Hz F#4 faint, 0.15–0.25 587 Hz D5, 0.23–0.45 371 Hz F#4; today's first sound |

### High then low: candidates for success

| File                                                 | File length | Audible | Contour                                                           |
| ---------------------------------------------------- | ----------- | ------- | ----------------------------------------------------------------- |
| `Sounds/Bottle.aiff`                                 | 0.77 s      | 0.29 s  | 0.00–0.06 493 Hz B4, 0.05–0.15 248 Hz B3, 0.14–0.29 186 Hz F#3    |
| `Sounds/Submarine.aiff`                              | 1.49 s      | 0.54 s  | 0.00–0.10 651 Hz E5, 0.09–0.54 326 Hz E4                          |
| `SystemSounds/facetime/multiway_leave.caf`           | 0.44 s      | 0.24 s  | 0.00–0.12 525 Hz C5, 0.11–0.24 261 Hz C4                          |
| `Telephony/let-me-join.caf`                          | 0.98 s      | 0.45 s  | 0.00–0.10 659 Hz E5, 0.12–0.45 522 Hz C5                          |
| `AlertTones/Text-Message-Acknowledgement-HaHa.caf`   | 0.23 s      | 0.21 s  | 0.00–0.07 1397 Hz F6, 0.07–0.21 1174 Hz D6                        |
| `AlertTones/Text-Message-Acknowledgement-ThumbsDown.caf` | 0.50 s  | 0.20 s  | 0.00–0.08 1319 Hz E6, 0.08–0.20 880 Hz A5                         |
| `AlertTones/Modern/Input.m4r`                        | 1.20 s      | 1.09 s  | 0.02–0.18 560 Hz C#5, 0.17–1.09 468 Hz A#4                        |

**How each pair mirrors:**

- **`Funk.aiff` and `Bottle.aiff`:** both are three steps over 0.29 s, from the same folder and at about the same level (peaks of −11 and −13 dBFS). Funk rises an octave and then a major third; Bottle falls an octave and then a fourth. The order is mirrored and the intervals nearly so.
- **`Funk.aiff` and `Submarine.aiff`:** Submarine falls exactly an octave, in two notes, but its second note rings for 0.45 s, so the pair is uneven in length.
- **`multiway_join.caf` and `multiway_leave.caf`:** the same two notes, C4 and C5, in opposite order. This is the only exact inverse pair found. They are FaceTime's sounds for a person joining and leaving a call, which a listener may take for a call.

### Candidates for failure

| File                                                 | File length | Audible | Contour                                                           |
| ---------------------------------------------------- | ----------- | ------- | ----------------------------------------------------------------- |
| `Sounds/Basso.aiff`                                  | 0.77 s      | 0.17 s  | 0.03–0.17 about 207–226 Hz G#3–A3, one low thud                   |
| `SystemSounds/system/mic_unmute_fail.caf`            | 0.35 s      | 0.33 s  | three pulses of 221 Hz A3: 0.00–0.10, 0.09–0.20, 0.20–0.33        |
| `SystemSounds/system/payment_failure.aif`            | 2.19 s      | 0.24 s  | 0.11–0.16 1435 Hz F6, 0.14–0.19 1130 Hz C#6, 0.17–0.35 713 Hz F5  |
| `Telephony/mute_fail.caf`                            | 0.38 s      | 0.23 s  | 0.08–0.15 659 Hz E5 faint, 0.16–0.31 522 Hz C5                    |
| `SystemSounds/system/burn failed.aif`                | 0.95 s      | 0.73 s  | 293 Hz D4, 439 Hz A4, 587 Hz D5 rising; the same audio as `burn complete.aif` |

### Candidates for timeout

| File                                                 | File length | Audible | Contour                                                           |
| ---------------------------------------------------- | ----------- | ------- | ----------------------------------------------------------------- |
| `SystemSounds/siri/jbl_cancel.caf`                   | 1.96 s      | 0.78 s  | 0.20–0.30 393 Hz G4, 0.28–0.42 261 Hz C4, 0.41–0.98 132 Hz C3     |
| `Telephony/end_call_tone_cept.caf`                   | 1.20 s      | 1.02 s  | three beeps of 425 Hz G#4: 0.00–0.21, 0.39–0.62, 0.78–1.02        |
| `Telephony/vc~ended.caf`                             | 0.90 s      | 0.48 s  | 0.00–0.16 415 Hz G#4, 0.16–0.33 328 Hz E4, then faint echoes      |
| `AlertTones/Text-Message-Acknowledgement-QuestionMark.caf` | 0.50 s | 0.16 s | a glide up: 764 Hz G5, 848 Hz, 945 Hz, 1031 Hz C6                 |
| `AlertTones/Text-Message-Acknowledgement-Exclamation.caf` | 1.05 s | 0.34 s  | 0.00–0.34 880 Hz A5, one note; today's second sound               |

### Recommendation

| Alert   | Recommended                                  | Why                                                                                                  |
| ------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Prompt  | `/System/Library/Sounds/Funk.aiff`           | the two-beat low-then-high sound Neil described; the default alert sound itself is a 0.05 s blip     |
| Success | `/System/Library/Sounds/Bottle.aiff`         | the closest mirror of Funk: same length, same folder, same level, order reversed                     |
| Failure | `SystemSounds/system/mic_unmute_fail.caf`    | three flat low pulses: no rise or fall to mistake for the other three, and it is macOS's own "that didn't work" |
| Timeout | `SystemSounds/siri/jbl_cancel.caf`           | Siri's sound for giving up on a request: a slow fall to a long, low C3, longer and lower than Bottle  |

- **If the prompt must follow the user's chosen alert sound,** it reads the `aurl` property at start, and success can't be its inverse: no file is the inverse of an arbitrary sound.
- **If an exact inverse matters more than the sound's character,** the prompt is `multiway_join.caf` and success is `multiway_leave.caf`.
- **Basso for failure** is the conventional macOS error sound, but its 0.17 s thud is easy to miss beside the others.

Full paths, for `afplay`:

```text
/System/Library/Sounds/Funk.aiff
/System/Library/Sounds/Tink.aiff
/System/Library/Sounds/Bottle.aiff
/System/Library/Sounds/Submarine.aiff
/System/Library/Sounds/Basso.aiff
/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/facetime/multiway_join.caf
/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/facetime/multiway_leave.caf
/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/system/mic_unmute_fail.caf
/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/system/payment_failure.aif
/System/Library/Components/CoreAudio.component/Contents/SharedSupport/SystemSounds/siri/jbl_cancel.caf
/System/Library/PrivateFrameworks/TelephonyUtilities.framework/Versions/A/Resources/end_call_tone_cept.caf
/System/Library/PrivateFrameworks/TelephonyUtilities.framework/Versions/A/Resources/vc~ended.caf
/System/Library/PrivateFrameworks/TelephonyUtilities.framework/Versions/A/Resources/let-me-join.caf
```

## How a prompt ends

### What 1Password's log shows

1Password writes a log to `~/Library/Group Containers/2BUA8C4S2C.com.1password/Library/Application Support/1Password/Data/logs/`, as `1Password_rCURRENT.log` and numbered older files of 500 kB each.
The files on this Mac run from 2026-11-04 to 2026-11-20.
Each duration below runs from the first line of the burst of biometry lines that opens a prompt to the line that ends it.

| Log line                                                              | Count | Durations                                                        |
| --------------------------------------------------------------------- | ----- | ---------------------------------------------------------------- |
| `ssh authorization prompt timed out` (`op-ssh-agent`)                 | 450   | 439 between 59.30 s and 60.00 s, median 59.63 s, deviation 0.05 s |
| `AppCancel invoked by a timed-out prompt` (`op-system-auth`)          | 213   | 193 between 59.53 s and 60.00 s, median 59.99 s                  |
| `Session was not authorized` (`op-ssh-agent`)                         | 351   | 212 between 55 s and 60 s; 110 under 10 s                        |
| `received error from SSH auth prompt` (`ssh_agent.rs`)                | 53    | spread from 0 s to 60 s, median 31 s                             |
| `prompt error response from CLI rich auth prompt` (`cli_integration`) | 21    | spread from 0 s to 59 s, median 18 s                             |

- **A timed-out SSH prompt writes two lines:** `ssh authorization prompt timed out`, then `Session was not authorized` within a millisecond.
- **An approved prompt writes nothing** at the log's level. No line marks a success.
- **The timeout is 1Password's own,** about 59.6 s for an SSH prompt and 60.0 s for the system's Touch ID sheet. Today's second sound, at 60 s, therefore fires as the prompt goes, or never.

### Known, and guessed

| Statement                                                                                                    | Status   | From                                                        |
| ------------------------------------------------------------------------------------------------------------ | -------- | ----------------------------------------------------------- |
| The window list gives each window's owner, layer, size, position and on-screen flag, twice a second          | known    | the program's source                                        |
| The window list gives no title without Screen Recording permission                                           | known    | the program's source and README                             |
| An unanswered SSH prompt ends 59.3 s to 60.0 s after it starts                                               | known    | 439 log entries on this Mac                                 |
| A timed-out or refused SSH request is logged as `Session was not authorized`                                 | known    | the log                                                     |
| An approval leaves no log line                                                                               | known    | the log                                                     |
| The CLI reports `authorization timeout` and `authorization prompt dismissed` to its caller                   | reported | 1Password community threads, from users, not documentation  |
| A prompt's window closes when it times out                                                                   | guessed  | the log says the prompt ended; no window sample was taken   |
| A `Session was not authorized` under 10 s is the user cancelling                                             | guessed  | the durations fit a person's reaction; nothing labels them  |
| A failed Touch ID read or a wrong password keeps the same window open, with a shake or a message inside it   | guessed  | how macOS's own sheet behaves; not observed for 1Password   |
| After failed Touch ID reads the prompt offers a password field, and the window changes size                  | guessed  | not observed                                                |
| A client retries after a failure, so the same prompt reappears within seconds                                | guessed  | `ssh` tries its next key; `op` and SecretSpec don't retry   |
| The prompt's title, readable with Screen Recording permission, names the request                             | guessed  | not observed                                                |
| A launchd agent can read 1Password's log folder without a privacy prompt                                     | guessed  | a terminal session could; macOS guards other apps' group containers |

The community threads: ["authorization timeout when using cli"](https://www.1password.community/discussions/developers/authorization-timeout-when-using-cli/159177), ["Authorization prompt dismissed error [Linux]"](https://1password.community/discussion/comment/689224) and ["Why are my SSH attempts failing?"](https://1password.community/discussions/developers/why-are-my-ssh-attempts-failing/145064).

### What follows for the program

- **Timeout** is the one outcome the window list shows by itself: a prompt that closes 59.6 s after it appeared.
- **Success and cancel** both close the prompt early, and the window list can't tell them apart. The log can, for SSH: a `Session was not authorized` line at the close is a refusal.
- **A failed attempt** closes nothing, so a window that stays is all the window list shows of it. Whether its size changes has to be observed.
- **The guesses need one sitting with Neil at the keyboard:** a dry run recording every window change while he approves, cancels, fails Touch ID, types a wrong password and lets a prompt run out, for an SSH prompt and an `op` prompt each.

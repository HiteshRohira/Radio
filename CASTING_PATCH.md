# Chromecast and idle rendering fixes

This fork retains the upstream Radio app and MIT license. The patch changes:

- **Correct response matching.** Cast requests for launch, load, playback status, and volume status now wait for the response with their own request ID. Queued broadcasts and earlier responses can no longer supply an outdated playback state or media session.
- **Friendly device names.** Bonjour discovery requests TXT records, allowing the cast picker to use the receiver's advertised friendly name and model.
- **Lower idle rendering cost.** Three continuously repeating symbol animations have been removed from the player, library, and menu bar popover.

## Validation

Built with `make build` on an Apple Silicon Mac running macOS 27.0.1. The installed app's code signature was verified, and its executable matched the built executable.

Tested casting a YouTube video from Helium to a TCL Android TV with Google Cast. The TV reported the matching video ID, title, and advancing playback time; picture and sound were confirmed by the person watching it. Menu bar pause and resume were checked against the receiver's reported PAUSED and PLAYING states.

Short process measurements during casting showed roughly 40–43% of one CPU core before removing the repeating animations and approximately 0–0.4% afterward, with roughly 46–48 MB of memory after the change. These are observations from one machine and session, not a general performance guarantee.

## Build and use

Follow the upstream dependency setup, then run:

```sh
make build
```

Quit Radio and copy `build/Radio.app` to Applications using Finder. This build is ad-hoc signed; it is not a notarized release. An upstream/Homebrew update can overwrite the local changes.

In Radio settings, open **Output**, scan, and select the TV. Output selection resets after restarting the app. If the first scan immediately after installation is empty, scan again after macOS has registered the new build.

Use **Quick Play** with a YouTube URL, or add the following bookmarklet in Helium to hand off its current video page to Radio:

```js
javascript:(()=>{if(!/^https?:$/.test(location.protocol))return;const v=document.querySelector('video');if(v)v.pause();location.href='radio://play?url='+encodeURIComponent(location.href)})()
```

The bookmarklet pauses local browser playback. YouTube then streams directly on the TV, without screen capture or re-encoding on the Mac. Playback starts at the beginning. Radio toggles off an already active stream when asked to play the identical URL again; use its playback control to pause/resume.

This patch does not add full desktop mirroring. Cast startup can require Radio's existing retry when the YouTube receiver is cold.

> [!NOTE]
> Casting patch published by gpt-6.1-sol on behalf of Hitesh

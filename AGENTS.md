# Working on ReadAloud Books

Build/setup/device-test commands live in [README.md](README.md). The orb bootstrap
entrypoint is `.agents/setup`; use a connected Android device/runner for runtime tests.

## Code ownership

Paths below are relative to `app/src/main/java/com/pekempy/ReadAloudbooks/`.

- `MainActivity.kt`: navigation and application-level wiring.
- `data/api/`: Storyteller API contract, client configuration and authentication;
  `ui/login/`: login flow. This is a Storyteller client, not an Audiobookshelf client.
- `ui/player/PlaybackService.kt`: Media3 session/player lifetime;
  `AudiobookViewModel.kt` and `ReadAloudAudioViewModel.kt` in that directory own
  audiobook and synchronized-audio state. `util/AudioCodecConverter.kt` and
  `util/FfmpegStreamingDataSource.kt` handle codec fallback.
- `ui/reader/ReaderViewModel.kt`: EPUB loading, alignment and progress sync;
  `EpubReaderView.kt` in that directory: native pagination/rendering, not WebView CSS.
- `data/DownloadManager.kt` and `util/DownloadUtils.kt`: queue and file downloads;
  `data/db/`: local library storage. Download jobs are in memory, not a durable queue.
- `data/UserPreferencesRepository.kt`: Preferences DataStore;
  `data/BackupManager.kt`: selected-settings JSON import/export;
  `data/ReadingStatsRepository.kt`: SQLite reading-session statistics.

## Gotchas

- Preserve restore-before-save guards in `ReaderViewModel`: loading/uninitialized
  reader events must not overwrite saved progress. Saves have a two-second trailing
  debounce, persist locally before upload, and treat HTTP 409 as newer/equal server
  progress. `ReadAloudAudioViewModel` also writes progress; check both owners when
  changing synchronization.
- `PlaybackService.onTaskRemoved()` stops playback. Background playback does not
  imply survival after removing the app from recents.
- Settings JSON is not a full backup: `BackupManager` excludes credentials/book
  progress, exports a fixed tab order, and does not restore that order. When adding
  a preference, explicitly decide whether export and import should include it.
- Handle authentication diagnostics as secrets: username/password/token are stored
  in ordinary DataStore preferences. API/FFmpeg logging can expose authorization
  headers; redact logs before sharing them.
- Release builds fall back to debug signing when no release keystore is supplied.
  Verify signing before publishing; version properties and tag automation are
  described in README.

For reader/playback/download changes, verify restoration, seeking/highlighting,
offline use and pause/resume on a device against a Storyteller server. Report which
checks ran; compilation alone is not evidence of runtime behavior or performance.

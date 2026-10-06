# ReadAloud Books

An Android client for a Storyteller server: browse your library, listen to audiobooks,
read EPUBs, and follow synchronized text highlighting in ReadAloud books. Built with
Kotlin, Jetpack Compose, Material 3, and Media3.

## Use the app

Install an APK from [GitHub Releases](https://github.com/pricci1/ReadaloudBooks-fork/releases)
on Android 7.0 (API 24) or later. Sign in with your Storyteller server URL, username,
and password. Download books before reading or listening offline.

The app includes chapter navigation, playback speed and sleep timer controls,
reader font/theme settings, and download/storage management. The EPUB reader is
native and paginated; synchronized highlighting requires a book with alignment data.

## Screenshots

These existing screenshots illustrate the app; layouts may differ from the current build.

<details>
<summary><strong>Click to expand screenshots</strong></summary>
<br>

<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/c56de205-5fa4-456c-9095-58791a745021" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Login Screen
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/829a922b-6804-493f-930e-6f23f49d5d33" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Home Screen (now Shelf)
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/0f074336-4c54-4737-93bd-4cdb1b254821" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Series View
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/e8629f42-9798-4014-b9ce-8f96546d4c2c" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Book Details
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/5a5645da-d77c-46b3-b933-2ea75a5d6237" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Readaloud View
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/60cec593-a8a6-4489-994e-321ce0572eb7" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Download viewer / queueing
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/29ecda12-347e-4cd1-aa44-842bede3b9ab" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Storage management
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/5a4c1838-f748-49a1-a43e-ebee4ba2e26e" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Settings page
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/f52e0201-9b08-4388-a839-5b1d2a110958" width="230" style="border:1px solid #ccc; border-radius:12px;"><br/>
      Theming
    </td>
  </tr>
</table>

</details>

## Build and test

Use JDK 17 and Android SDK 34 with accepted SDK licenses. Set `ANDROID_HOME` to
your SDK directory, or set `sdk.dir` in an untracked `local.properties` file.
In an Amp orb, run [`.agents/setup`](.agents/setup) to provision the toolchain and
warm Gradle plugins and dependency artifacts for builds and tests. Setup also
installs Google's official `android` CLI and its Amp skill, and persists tool
paths for new login shells. Orb Gradle defaults use one worker and an in-process
Kotlin compiler to fit small machines; explicit user settings are preserved.
Warm reruns reuse installed packages and caches;
setup reaches future orbs once these files are on the project's default branch.
No resume hook or server is needed: this app has no local backing service, and
setup performs no user authentication. Device tests need a connected Android
device or runner (orbs lack KVM); setup installs no emulator or system image.

From the repository root:

```sh
./gradlew assembleDebug
./gradlew testDebugUnitTest lintDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

Launch ReadAloud Books on the device and sign in to your server. For connected
instrumentation tests, run `./gradlew connectedDebugAndroidTest`.
See [AGENTS.md](AGENTS.md) for code ownership and maintenance gotchas.

### Chapter utility

[`tools/generate_chapters.py`](tools/generate_chapters.py) copies M4B chapter markers
into adjacent `*(readaloud).epub` files **in place**. Back up books before running:

```sh
.venv/bin/python tools/generate_chapters.py /path/to/book-copies
```

Orb setup supplies FFmpeg/FFprobe and the Python virtual environment. Outside
orbs, install FFmpeg and create a venv with `tools/requirements.txt`. The tool
uses `tqdm`; it does not run speech recognition or need Torch/Whisper.

## Releases

The [release workflow](.github/workflows/android_release.yml) builds and publishes
APKs when a `v*` tag is pushed. It supplies `versionName` from the tag (without
the leading `v`), `versionCode` from the workflow run number, and enables ABI splits
plus a universal APK. Pull requests produce debug APK artifacts through the
[PR workflow](.github/workflows/android_pr_debug.yml).

For a local release:

```sh
./gradlew assembleRelease -PversionName=<version> -PversionCode=<integer>
```

Replace the placeholders with your release version and integer version code.
Configure signing through `keystore.properties` or
`KEYSTORE_FILE`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, and `KEY_PASSWORD`. CI additionally
uses `KEYSTORE_BASE64` to provision the keystore. **Without a release keystore,
the build falls back to debug signing**; verify the signing identity before publishing.
Test on a real device against your Storyteller server before tagging a release.

## License

See [LICENSE](LICENSE) for the GNU Affero General Public License v3.0.

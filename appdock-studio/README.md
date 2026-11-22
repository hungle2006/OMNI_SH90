# AppDock Arduino Studio

Arduino Studio integrates a browser-based editor, GitHub Actions Arduino CLI compilation, firmware download and USB serial tools into AppDock.

**Live UI:** https://anh-app-hub.vercel.app/arduino

**Compiler workflow:** [AppDock Arduino Compiler](../.github/workflows/appdock-arduino.yml)

**Deployment architecture:** AppDock main project on Vercel routes `/arduino` and `/arduino/api/arduino` to the dedicated Vercel project `appdock-arduino-studio`. The main AppDock homepage is not replaced. `index.html` in this directory is the deployed HTML snapshot; future edits must be deployed to the Vercel project to become live.

## Compile workflow

1. User edits a single `.ino` sketch (up to 36 KB), selects board and signs in with an access passphrase.
2. Browser calls Vercel `/arduino/api/arduino?action=compile`. The Vercel server-side API holds the **GitHub token**. **Do not commit a GitHub token into this repository or expose it in client JavaScript.**
3. Vercel dispatches the `AppDock Arduino Compiler` workflow on `main`, passing an allowlisted board ID and a base64 sketch.
4. Arduino CLI installs the selected board core, compiles, and uploads a seven-day firmware artifact.
5. Vercel API polls the workflow ID and sends back a HEX or ZIP firmware download. USB flashing runs locally in the browser.

## Vercel configuration (required before Verify works)

Project: `appdock-arduino-studio`, not the original `anh-app-hub`.

Environment variables (set in Vercel dashboard, **never send keys in chat**):

- `ARDUINO_STUDIO_ACCESS_KEY` — a 32+ character password used by the AppDock Studio sign-in dialog. Configured in the initial deployment. Keep it secret and rotate if necessary.
- `GITHUB_ACTIONS_TOKEN` — a GitHub fine-grained personal access token scoped to `hungle2006/OMNI_SH90` with **Actions: Read and write** permission (and repository metadata read). **Not configured as part of the initial deployment**. After adding it, redeploy the Vercel project.
- `ARDUINO_GITHUB_REPO` optional, default `hungle2006/OMNI_SH90`.

Navigate to https://vercel.com/hope-1091/appdock-arduino-studio/settings/environment-variables to set the token.

## Device compatibility

| Device | Compile | Direct Web Serial flashing |
| --- | --- | --- |
| Uno R3, ATmega328P Nano (new/old bootloader), compatible RF-Nano | Yes | STK500v1 proof-of-concept |
| Mega 2560 / Leonardo | Yes | Download HEX; needs board-specific protocol |
| ESP32 DevKit / ESP8266 NodeMCU | Yes (platform installs on Actions) | Download complete artifact ZIP; use an appropriate flasher |

Web Serial requires a compatible secure desktop browser (Chrome/Edge). Native USB uploading and live serial monitoring **have not been validated against real hardware**.

## Verification history

- GitHub Actions smoke build completed successfully, Blink/Uno: https://github.com/hungle2006/OMNI_SH90/actions/runs/45025790781
- GitHub artifact `appdock-firmware-smoke` created.
- Vercel frontend `appdock-arduino-studio` production deployed and HTTP 200 confirmed.
- AppDock original `/` and new `/arduino` each returned HTTP 200 after routing promotion.
- End-to-end web-to-GitHub compile cannot be considered verified until `GITHUB_ACTIONS_TOKEN` is configured. Hardware flashing is also unverified.

## Safety

Compile only trusted sketches. The GitHub runner executes compiler/toolchain code, so access to the API must be private and rate-limited. Use only authorized board types and confirm USB port and target MCU before uploading. The firmware ZIP artifact expires after seven days.

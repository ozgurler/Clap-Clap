# Clap! Clap!

A hand-clapping rhythm game, packaged with Capacitor for iOS and Android.

- Game source: `www/index.html` (single file — edit here, then run `npx cap sync`)
- iOS native project: `ios/` (Capacitor 8, Swift Package Manager — no CocoaPods)
- Android native project: `android/`
- CI: `.github/workflows/ios.yml` builds and uploads to TestFlight from GitHub Actions (no Mac needed)

## One-time setup

### 1. Push this repo to GitHub

```bash
git init
git add .
git commit -m "Clap! Clap! v1"
git remote add origin https://github.com/YOURNAME/clap-clap.git
git push -u origin main
```

Tip: a public repo gets unlimited free Actions minutes; private repos consume
free minutes at 10x for macOS runners (~200 real minutes/month on the free tier).

### 2. Apple setup (browser + Git Bash, no Mac)

a. **Signing certificate** — in Git Bash:

```bash
openssl genrsa -out dist.key 2048
openssl req -new -key dist.key -out dist.csr \
  -subj "/emailAddress=you@example.com/CN=Emre Distribution/C=US"
```

At developer.apple.com → Certificates → "+" → **Apple Distribution** →
upload `dist.csr` → download `distribution.cer`, then:

```bash
openssl x509 -in distribution.cer -inform DER -out distribution.pem
openssl pkcs12 -export -inkey dist.key -in distribution.pem -out cert.p12
```

(If CI later rejects the .p12 with a format error, re-run the last command
with `-legacy` added.)

b. **App ID** — developer.apple.com → Identifiers → "+" → App ID →
bundle ID `com.emre.clapclap` (must match `capacitor.config.json`; change
both if you want a different ID).

c. **App Store Connect API key** — appstoreconnect.apple.com → Users and
Access → Integrations → App Store Connect API → "+", role **App Manager**.
Download the `.p8` (one chance!) and note the **Key ID** and **Issuer ID**.

d. **App record** — App Store Connect → Apps → "+" → New App →
"Clap! Clap!" with your bundle ID.

### 3. GitHub secrets

Repo → Settings → Secrets and variables → Actions → New repository secret:

| Secret              | Value                                          |
|---------------------|------------------------------------------------|
| `P12_BASE64`        | output of `base64 -w0 cert.p12`                |
| `P12_PASSWORD`      | password you chose for the .p12                |
| `KEYCHAIN_PASSWORD` | anything random                                |
| `APPSTORE_KEY_ID`   | API Key ID from step 2c                        |
| `APPSTORE_ISSUER_ID`| Issuer ID from step 2c                         |
| `APPSTORE_P8`       | full text contents of the .p8 file             |
| `TEAM_ID`           | developer.apple.com → Membership → Team ID     |

### 4. Run it

GitHub → Actions tab → "iOS TestFlight build" → **Run workflow**.
~10–15 minutes later the build appears in App Store Connect → TestFlight
(allow a few extra minutes of processing). Install the TestFlight app on
your iPhone to play-test, then submit for review from App Store Connect.

## Troubleshooting

- **Xcode/SDK version error in the Archive step**: Apple requires current-SDK
  builds. Add a step before Archive:
  `sudo xcode-select -s /Applications/Xcode_<version>.app`
  (check the actions/runner-images release notes for installed versions).
- **Signing errors**: almost always a bundle ID mismatch between the Apple
  Developer portal and `capacitor.config.json`.
- **`.p12` import fails**: regenerate with the `-legacy` OpenSSL flag.

## TODO before store submission

- [ ] Bundle the Fredoka font locally (download the .woff2 files, place in
  `www/fonts/`, replace the Google Fonts `<link>` with a local `@font-face`)
  so the game is fully offline.
- [ ] App icon + splash screen: put a 1024x1024 `icon.png` (and optionally
  `splash.png`) in an `assets/` folder and run
  `npm install -D @capacitor/assets && npx capacitor-assets generate`.
- [ ] Privacy policy URL (a simple "this app collects no data" page).

## Android (for Google Play)

The `android/` project is included. Build an .aab locally with Android
Studio, or ask for a matching Android workflow — Google Play uploads can
also be automated from Actions on the free Linux runners.

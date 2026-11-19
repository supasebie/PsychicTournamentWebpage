# psychictournament.online

Marketing site for **Psychic Tournament**, an intuition-training app for Android and iOS
(Flutter — see the `PsychicTournament` repo). Served by GitHub Pages from `main`; the
custom domain lives in `CNAME`.

## Structure

| File | Purpose |
|---|---|
| `index.html` | The landing page. All content, plus ~120 lines of inline JS (sticky nav, mobile menu, scroll reveal, starfield). |
| `style.css` | The whole design system. Tokens at the top mirror `lib/theme/app_theme.dart` in the app. |
| `privacy-policy.html` | Static privacy policy. |
| `account-deletion.html` | Account deletion instructions Google Play links to. `noindex`, not in the sitemap. |
| `get/index.html` | QR code target (`/get/`). Sends Android to Google Play and iPhone/iPad to the App Store; desktop and unknown devices see both buttons. Logs a `qr_redirect` analytics event. `noindex`, not in the sitemap. |
| `imgs/qr-get-app.svg`, `.png` | QR code for `https://psychictournament.online/get/` (error correction H). Shown in the home page's bottom-right "Get the app" card (`.qr-card`), which only appears on screens ≥1000px with a mouse and remembers its close button in `localStorage`. Regenerate with `segno` if the URL changes, and keep the SVG's `viewBox` so it scales. |
| `404.html` | GitHub Pages serves this for any missing path. |
| `robots.txt`, `sitemap.xml` | Crawl rules and the URL list for Search Console. Bump `<lastmod>` when a page changes. |
| `site.webmanifest` | Icons and the two store listings as `related_applications`. |
| `terms-of-service.html` | Shell that renders `terms-of-service.md` client-side — edit the Markdown, not the HTML. |
| `imgs/logo.svg` | Logo mark (also the favicon). |
| `imgs/apple-touch-icon.png`, `icon-*.png`, `favicon-48.png` | PNG icons, resized from the app's 1024px iOS icon with `sips`. |
| `imgs/og-cover.png` | 1200×630 social share card. |
| `imgs/screens/*.webp` | App screenshots, captured from the Android emulator. |

No build step, no dependencies, no framework. Open `index.html` or run:

```bash
python3 -m http.server 4311
```

## Design notes

The site deliberately mirrors the app's visual language rather than inventing its own:
near-black indigo surfaces (`#0E0B1F`), violet (`#6B2DD9`) and deep blue (`#1256C4`)
accents, electric cyan (`#00D4FF`) highlights, gold for exceptional results, glass panels,
and Sora throughout. Each game mode carries its in-app colour — Zener violet, Dowsing mint,
Star Seed rose, Lodestar fuchsia into sapphire (`#F26BE3` → `#5B8DFF`).

Two things worth not breaking:

- **`.reveal` is gated on `html.js`.** An inline script in `<head>` adds the class. Without
  it the hidden state never applies, so a visitor with JS disabled sees the full page
  instead of a column of invisible boxes. There is also a 3-second failsafe in case the
  IntersectionObserver never fires.
- **No `scroll-snap` on `.rail`.** A snap container with centre-aligned children makes the
  browser scroll the *document* to the rail on first layout, landing visitors at the footer.

Both the starfield and the CSS animations respect `prefers-reduced-motion`.

## Updating screenshots

Screenshots come straight off a running Android emulator. Clean up the status bar first:

```bash
adb shell settings put global sysui_demo_allowed 1
adb shell am broadcast -a com.android.systemui.demo -e command enter
adb shell am broadcast -a com.android.systemui.demo -e command clock -e hhmm 0941
adb shell am broadcast -a com.android.systemui.demo -e command battery -e level 100 -e plugged false
adb shell am broadcast -a com.android.systemui.demo -e command network -e wifi show -e level 4
adb shell am broadcast -a com.android.systemui.demo -e command network -e mobile hide
adb shell am broadcast -a com.android.systemui.demo -e command notifications -e visible false
```

Capture, then crop off the gesture pill and encode. The `1280 2776` crop keeps every shot
at the same aspect ratio, which the phone frames rely on:

```bash
adb exec-out screencap -p > shot.png
cwebp -crop 0 0 1280 2776 -resize 640 0 -q 84 -m 6 shot.png -o imgs/screens/name.webp
```

Exit demo mode with `adb shell am broadcast -a com.android.systemui.demo -e command exit`.

### From the iOS Simulator

The 1.1 shots (Lodestar, the Mindsight menu, Focus Training, the drawer) come from an iPhone 17
Pro simulator. Release builds can't run on a simulator, so build debug with the tester surface
switched off. `TESTER_TOOLS=false` hides the tester-only controls (the drawer's Options row,
Focus Training's Session options) without changing anything else:

```bash
scripts/build.sh dev ios --simulator --debug --dart-define=TESTER_TOOLS=false
```

Run that in the app repo, install `build/ios/iphonesimulator/Runner.app`, then pin the status bar
and capture:

```bash
xcrun simctl status_bar booted override --time "9:41" --batteryState charged --batteryLevel 100 --cellularMode notSupported --wifiBars 3 --dataNetwork wifi
```

```bash
xcrun simctl io booted screenshot --type=png shot.png
```

The simulator captures at 1206×2622. Cropping to 1206×2616 gives the same aspect ratio as the
Android crop:

```bash
cwebp -crop 0 0 1206 2616 -resize 640 0 -q 84 -m 6 shot.png -o imgs/screens/name.webp
```

Wait for the "Log in to set a username!" snackbar to clear before you shoot the menus.

## Store links

Both listings are live. The store URLs appear in several places, so change all of them together:

- the two badge groups in `index.html` (hero and closing CTA) and the footer's Download column;
- `installUrl`, `downloadUrl` and `sameAs` in the JSON-LD block;
- `site.webmanifest`, and the `apple-itunes-app` meta tag (App Store id only).

A small script puts the visitor's own store first (Android → Google Play, iPhone/iPad → App
Store) and sends a `store_click` event to Google Analytics.

## SEO notes

- The FAQ appears twice: as visible `<details>` in the page and as `FAQPage` JSON-LD in the
  head. Google requires the two to match, so edit both.
- There is no `aggregateRating` in the JSON-LD on purpose. Only add one from real store ratings.

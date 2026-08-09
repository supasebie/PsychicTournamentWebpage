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
| `terms-of-service.html` | Shell that renders `terms-of-service.md` client-side — edit the Markdown, not the HTML. |
| `imgs/logo.svg` | Logo mark (also the favicon). |
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
Star Seeds rose.

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

## Store links

Both store buttons are currently non-linking "Coming soon" badges (`.store` in `style.css`,
two instances in `index.html` — hero and closing CTA). When a listing goes live, give the
badge a real `href` and change `Coming soon to` → `Get it on` / `Download on the`.

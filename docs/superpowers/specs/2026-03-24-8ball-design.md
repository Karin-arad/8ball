# 8ball — Magic Crystal Ball PWA

## Overview

A mobile-only PWA that simulates a crystal ball. The user thinks of a question, shakes their phone, and gets a randomized answer. Retro 80s neon arcade aesthetic.

## Platform

- **Type:** Progressive Web App (PWA)
- **Target:** Mobile only (iOS Safari, Android Chrome, tablets with accelerometer)
- **Desktop:** Shows a "open on your phone" screen with QR code
- **Hosting:** Static files — deployable to Netlify, Vercel, GitHub Pages, or any static host
- **Language:** Hebrew, RTL, gender-neutral language throughout (infinitive form, plural form, or genderless phrasing)

## Tech Stack

- Single HTML file for the app (embedded CSS and JS), plus PWA support files (manifest, service worker, icons)
- No framework, no build step, no external dependencies
- PWA manifest + service worker for "Add to Home Screen" capability
- DeviceMotionEvent API for shake detection
- QR code: pre-generated static SVG embedded in the HTML (no JS library needed)

## Screens

### 0. iOS Permission Screen (iOS only, first visit)

- Same dark neon background as idle screen
- Crystal ball visible but dimmed
- Centered button: "לחצו להתחיל" (tap to start) — neon styled button
- On tap: calls `DeviceMotionEvent.requestPermission()`
- If granted → transition to idle screen
- If denied → transition to fallback screen (see Screen 5)
- Only shown once; permission is remembered by the browser

### 1. Idle Screen (default)

- Dark background with gradient (`#0d001a` → `#1a002e`)
- `<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">`
- CSS: `touch-action: manipulation` to prevent double-tap zoom; `overscroll-behavior: none` to prevent pull-to-refresh
- Portrait orientation preferred (CSS handles landscape gracefully but no lock enforced)
- **"8BALL"** title in large neon-glowing text (solid pink `#ff00cc` with text-shadow glow)
- **Subtitle:** "לשאול. לנער. לגלות." — small, subtle, gender-neutral infinitive
- **Crystal ball** centered:
  - ~160px diameter translucent sphere (scales with viewport: `min(40vw, 160px)`)
  - Radial gradient simulating glass depth (lighter top-left, darker bottom-right)
  - Neon ring around the ball (pink `#ff00cc` with box-shadow glow)
  - Animated fog/mist inside (CSS animation — two overlapping divs with radial gradients, slow translateX/Y keyframes, 8-12s loop, `rgba` purples and blues at low opacity)
  - Subtle light glint on upper-left (blurred white pseudo-element)
- **Instruction text:** "לנער חזק ולשאול בלב" with neon cyan (`#00d4ff`) glow
- **Secondary text:** "חשבו על שאלה ותנו לכדור לענות" — small, `#666`
- Decorative neon gradient line near bottom (pink → purple → cyan, with box-shadow glow)
- Three small dots at bottom (pink, purple, cyan) as decorative element

### 2. Shaking State

- Triggered by DeviceMotionEvent exceeding acceleration threshold
- Ball vibrates/trembles (CSS class `shaking` added — rapid `translate(±2px, ±2px)` keyframes at 50ms)
- Fog inside intensifies and swirls (animation-duration shortened, rotation added)
- Neon ring pulses brighter (box-shadow intensity increase)
- Duration: ~1.5 seconds of animation after shake detected, then auto-transition to reveal

### 3. Reveal State

- Fog clears (opacity transition to near-transparent, 0.6s)
- Answer text fades in at center of ball (`opacity: 0→1` + `translateY: 10px→0`, 0.8s ease-out)
- Text style: white with subtle purple glow, `clamp(14px, 3.5vw, 18px)`
- Answer remains visible until next shake
- Instruction text changes to: "לנער שוב לשאלה חדשה"
- **No consecutive duplicate answers** — track last answer index, re-roll if same

### 4. Desktop Fallback

- Same dark neon background
- Message: "הכדור עובד רק בטלפון 📱"
- QR code: pre-generated SVG inline, pointing to the app URL
- **Detection logic:** Check if `DeviceMotionEvent` is supported AND (on iOS) if `DeviceMotionEvent.requestPermission` exists. Do NOT use screen width — tablets with accelerometers should work.

### 5. Permission Denied / No Accelerometer Fallback

- Same dark neon background, crystal ball visible
- **Tap-the-ball fallback:** user can tap the crystal ball to trigger the shake animation and get an answer
- Small text: "אין גישה לחיישן? לחצו על הכדור"
- Also activates if no shake detected within 10 seconds of idle (show hint: "אפשר גם פשוט ללחוץ על הכדור")

## Shake Detection

```
Listen to DeviceMotionEvent ('devicemotion')
Read accelerationIncludingGravity (x, y, z)
Calculate magnitude: sqrt(x² + y² + z²)
Subtract gravity (~9.8) to get net acceleration
If net acceleration > threshold (12) for 2+ events within 500ms → trigger shake
Cooldown: 2 seconds between triggers (prevent rapid re-fire)

iOS flow:
  if (typeof DeviceMotionEvent.requestPermission === 'function')
    → show "tap to start" button → on tap, call requestPermission()
    → if 'granted' → start listening
    → if 'denied' → enable tap-the-ball fallback

Android: no permission needed, start listening immediately
```

## Answer Pool

Large array of Hebrew strings, served randomly with no consecutive duplicates. Gender-neutral (infinitive/plural forms). **These are examples — implementation should include 50+ total answers maintaining these approximate ratios:**

**Positive (~30%):**
- כן
- בהחלט כן
- הכוכבים אומרים כן
- בלי שום ספק
- זה הולך לקרות
- אפשר לסמוך על זה
- בדרך אליך
- ללא ספק
- חד משמעית כן
- התשובה חיובית

**Negative (~20%):**
- לא
- ממש לא
- לא בחיים האלה
- בגלגול הבא אולי
- תשכחו מזה
- הכוכבים צוחקים
- לא הפעם
- שלילי לגמרי

**Evasive/Maybe (~25%):**
- אולי מחר
- צריך לחשוב שוב
- שאלתם את אמא?
- הרוחות שותקות
- לא עכשיו
- תנערו שוב, לא שמעתי
- הערפל כבד מדי
- לשאול שוב מאוחר יותר
- לא ברור עדיין
- ממממ... לא בטוח
- קשה להגיד
- תחזרו מחר

**Philosophical/Cheeky (~25%):**
- התשובה כבר ידועה, לא?
- מספיק לשאול. הגיע הזמן לבצע
- התשובה בפנים, לא בכדור
- למה לשאול אותי? תסתכלו במראה
- אם צריך לשאול, כנראה כן
- הכדור לא יודע הכל, אבל יודע את זה
- תפסיקו לשאול ותתחילו לעשות
- שאלה טובה. תשובה? לא בטוח
- הלב יודע. הכדור רק מאשר
- אין תשובות קלות לשאלות טובות

Total: 50+ answers. Easily expandable — just add strings to the array.

## Animations (CSS only, no JS animation libraries)

1. **Fog drift (idle):** Two overlapping `<div>`s with radial-gradient backgrounds (`rgba(180,100,255,0.1)` and `rgba(100,200,255,0.08)`), animated with `translateX`/`translateY` keyframes, 8-12s loop, `alternate` direction
2. **Ball tremble (shaking):** CSS class adds keyframe with rapid `translate(±2px, ±2px) rotate(±1deg)` at 50ms intervals
3. **Fog swirl (shaking):** `animation-duration` reduced to 1s, `rotate` keyframe added
4. **Text reveal:** `opacity: 0→1` + `translateY: 10px→0` over 0.8s ease-out
5. **Neon pulse (shaking):** `box-shadow` keyframe increasing glow intensity on the ring
6. **Title glow (idle):** Subtle `text-shadow` pulse, 3s loop (low priority, nice-to-have)
7. **`prefers-reduced-motion`:** Respect the user preference — disable fog animation, ball tremble, and neon pulse. Keep text reveal (opacity only, no translateY).

## PWA Configuration

- `manifest.json`: name "8Ball", short_name "8Ball", display "standalone", orientation "portrait", theme_color "#0d001a", background_color "#0d001a", lang "he", dir "rtl"
- Service worker strategy: **cache-first with version check** — cache index.html on install, serve from cache, check for updates in background. If new version found, update cache (user gets new version on next open).
- Icons: simple 8ball icon in neon style (192x192, 512x512) — can be generated SVG or simple PNG

## File Structure

```
/tmp/8ball/
├── index.html          # The entire app (HTML + CSS + JS)
├── manifest.json       # PWA manifest
├── sw.js              # Service worker
├── icons/
│   ├── icon-192.png   # App icon
│   └── icon-512.png   # App icon large
└── docs/
    └── superpowers/specs/
        └── 2026-03-24-8ball-design.md
```

## Not In Scope (v1)

- Text input field for question
- Answer history
- Social media sharing
- Sound effects
- Analytics
- Backend/API
- Multiple themes
- Settings page
- Accessibility beyond `prefers-reduced-motion` (v2 candidate: `aria-live` for answer reveal)
- Install prompt/banner (user can use browser's native "Add to Home Screen")

# 8ball — Magic Crystal Ball PWA

## Overview

A mobile-only PWA that simulates a crystal ball. The user thinks of a question, shakes their phone, and gets a randomized answer. Retro 80s neon arcade aesthetic.

## Platform

- **Type:** Progressive Web App (PWA)
- **Target:** Mobile only (iOS Safari, Android Chrome)
- **Desktop:** Shows a "open on your phone" screen with QR code pointing to the app URL
- **Hosting:** Static files — deployable to Netlify, Vercel, GitHub Pages, or any static host
- **Language:** Hebrew, RTL, gender-neutral language throughout

## Tech Stack

- Single HTML file with embedded CSS and JavaScript
- No framework, no build step, no dependencies
- PWA manifest + service worker for "Add to Home Screen" capability
- DeviceMotionEvent API for shake detection

## Screens

### 1. Idle Screen (default)

- Dark background with gradient (`#0d001a` → `#1a002e`)
- **"8BALL"** title in large neon-glowing text (pink/purple/cyan gradient or solid pink with glow)
- **Subtitle:** "שאלי. נערי. גלי." — small, subtle (NOTE: may change to gender-neutral)
- **Crystal ball** centered:
  - ~160px diameter translucent sphere
  - Radial gradient simulating glass depth (lighter top-left, darker bottom-right)
  - Neon ring around the ball (pink, `#ff00cc`, with box-shadow glow)
  - Animated fog/mist inside (CSS animation, slow-moving radial gradients)
  - Subtle light glint on upper-left
- **Instruction text:** "לנער חזק ולשאול בלב" with neon cyan glow
- **Secondary text:** "חשבו על שאלה ותנו לכדור לענות" — small, gray
- Decorative neon gradient line near bottom (pink → purple → cyan)
- Three small dots at bottom (pink, purple, cyan) as decorative element

### 2. Shaking State

- Triggered by DeviceMotionEvent exceeding acceleration threshold (~15m/s²)
- Ball vibrates/trembles (CSS transform with rapid small translations)
- Fog inside intensifies and swirls (faster CSS animation)
- Neon ring pulses brighter
- Duration: ~1.5 seconds of shaking animation after shake detected

### 3. Reveal State

- Fog clears/parts (opacity transition)
- Answer text fades in at center of ball
- Text style: white/light with subtle glow, 16-18px
- Answer remains visible until next shake
- Instruction text changes to: "לנער שוב לשאלה חדשה"

### 4. Desktop Fallback

- Same dark neon background
- Message: "הכדור עובד רק בטלפון 📱"
- QR code pointing to the app URL
- Detected via: absence of DeviceMotionEvent support or screen width > 768px

## Shake Detection

```
Listen to DeviceMotionEvent
Calculate total acceleration: sqrt(x² + y² + z²)
If acceleration > threshold (15) for 2+ events within 500ms → trigger
Cooldown: 2 seconds between triggers (prevent rapid re-fire)
iOS quirk: Requires user gesture to request permission (DeviceMotionEvent.requestPermission)
  → Show a "tap to start" button on first visit on iOS
```

## Answer Pool

Large array of Hebrew strings, categorized but served randomly. Gender-neutral where possible. Mix of tones:

**Positive (~30%):**
- כן
- בהחלט כן
- הכוכבים אומרים כן
- בלי שום ספק
- זה הולך לקרות
- סמוך/סמכי על זה

**Negative (~20%):**
- לא
- ממש לא
- לא בחיים הבאים
- בגלגול הבא אולי
- תשכחו מזה
- הכוכבים צוחקים

**Evasive/Maybe (~25%):**
- אולי מחר
- צריך לחשוב שוב
- שאלת את אמא שלך?
- הרוחות שותקות
- לא עכשיו
- תנערו שוב, לא שמעתי
- הערפל כבד מדי

**Philosophical/Cheeky (~25%):**
- את/ה יודע/ת לבד
- מספיק לשאול. הגיע הזמן לבצע
- התשובה בפנים, לא בכדור
- מכירה אותך, את כבר יודעת — (NOTE: gendered, consider alternative)
- למה את/ה שואל/ת אותי? תסתכלו במראה
- אם צריך לשאול, כנראה כן

Total: ~50+ answers. Easily expandable — just add strings to the array.

## Animations (CSS only, no JS animation libraries)

1. **Fog drift:** Two overlapping radial-gradient divs with slow `translateX`/`translateY` keyframes (8-12s loop)
2. **Ball tremble (on shake):** Rapid `translate(±2px, ±2px)` keyframes (50ms intervals)
3. **Fog swirl (on shake):** Speed up fog animation + add rotation
4. **Text reveal:** `opacity: 0→1` + slight `translateY: 10px→0` over 0.8s ease-out
5. **Neon pulse:** `box-shadow` intensity oscillation on the ring (on shake)
6. **Title glow:** Subtle `text-shadow` pulse on idle (optional, low priority)

## PWA Configuration

- `manifest.json`: app name "8Ball", display "standalone", theme_color "#0d001a", background_color "#0d001a"
- Service worker: cache the single HTML file for offline use
- Icons: simple 8ball icon in neon style (192x192, 512x512)

## File Structure

```
/tmp/8ball/
├── index.html          # The entire app (HTML + CSS + JS)
├── manifest.json       # PWA manifest
├── sw.js              # Service worker (minimal, caches index.html)
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

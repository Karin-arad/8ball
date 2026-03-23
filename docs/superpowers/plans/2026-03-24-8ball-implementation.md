# 8ball Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a mobile-only PWA crystal ball that responds to phone shaking with randomized Hebrew answers in a retro neon aesthetic.

**Architecture:** Single index.html with embedded CSS and JS. PWA support files (manifest.json, sw.js) alongside. No build step, no dependencies. State managed via CSS classes toggled by JS.

**Tech Stack:** Vanilla HTML/CSS/JS, DeviceMotionEvent API, PWA (manifest + service worker)

**Spec:** `docs/superpowers/specs/2026-03-24-8ball-design.md`

---

## File Structure

```
/tmp/8ball/
├── index.html          # Complete app: HTML structure + CSS + JS
├── manifest.json       # PWA manifest
├── sw.js              # Service worker (cache-first with version check)
└── icons/
    ├── icon-192.png   # App icon (generated SVG→PNG or inline SVG)
    └── icon-512.png   # App icon large
```

---

### Task 1: PWA Infrastructure (manifest + service worker + icons)

**Files:**
- Create: `/tmp/8ball/manifest.json`
- Create: `/tmp/8ball/sw.js`
- Create: `/tmp/8ball/icons/icon-192.png`
- Create: `/tmp/8ball/icons/icon-512.png`

**Can run in parallel with Task 2.**

- [ ] **Step 1: Create manifest.json**

```json
{
  "name": "8Ball",
  "short_name": "8Ball",
  "description": "כדור בדולח דיגיטלי — לנער חזק ולשאול בלב",
  "start_url": "/index.html",
  "display": "standalone",
  "orientation": "portrait",
  "theme_color": "#0d001a",
  "background_color": "#0d001a",
  "lang": "he",
  "dir": "rtl",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

- [ ] **Step 2: Create sw.js**

```javascript
const CACHE_NAME = '8ball-v1';
const ASSETS = ['/index.html', '/manifest.json'];

self.addEventListener('install', (e) => {
  e.waitUntil(caches.open(CACHE_NAME).then((cache) => cache.addAll(ASSETS)));
  self.skipWaiting();
});

self.addEventListener('activate', (e) => {
  e.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => k !== CACHE_NAME).map((k) => caches.delete(k)))
    )
  );
  self.clients.claim();
});

self.addEventListener('fetch', (e) => {
  e.respondWith(
    caches.match(e.request).then((cached) => {
      const fetchPromise = fetch(e.request).then((response) => {
        if (response.ok) {
          const clone = response.clone();
          caches.open(CACHE_NAME).then((cache) => cache.put(e.request, clone));
        }
        return response;
      }).catch(() => cached);
      return cached || fetchPromise;
    })
  );
});
```

- [ ] **Step 3: Create app icons**

Generate simple SVG-based icons — a dark circle with "8" in neon pink, export as PNG at 192x192 and 512x512. Use an inline SVG converted to canvas→PNG, or create minimal SVG files.

- [ ] **Step 4: Commit**

```bash
cd /tmp/8ball
git add manifest.json sw.js icons/
git commit -m "feat: add PWA manifest, service worker, and icons"
```

---

### Task 2: HTML Structure + CSS (Visual Layer)

**Files:**
- Create: `/tmp/8ball/index.html`

**Can run in parallel with Task 1.** This task creates the complete HTML and CSS. JavaScript is added in Task 3.

- [ ] **Step 1: Create index.html with full HTML structure**

The HTML should contain:
- `<!DOCTYPE html>` with `lang="he" dir="rtl"`
- `<head>`: viewport meta (no-scale, no-zoom), theme-color, manifest link, PWA meta tags
- `<body>` with these containers:
  - `#app` — main app wrapper
  - `#ios-permission` — iOS permission request screen (hidden by default)
  - `#idle` — idle state with ball and instructions
  - `#desktop-fallback` — desktop message with QR (hidden by default)
  - `#no-sensor-hint` — tap fallback hint (hidden by default)
  - `.crystal-ball` — the ball element with `.fog-1`, `.fog-2` inside, and `#answer-text`
  - `.neon-line` — decorative bottom line
  - `.dots` — three decorative dots

- [ ] **Step 2: Write all CSS**

Embedded in `<style>` tag. Must include:
- Dark gradient background (`#0d001a` → `#1a002e`)
- Crystal ball: radial gradient, neon ring (`#ff00cc` box-shadow), glint pseudo-element
- Fog animation: two `@keyframes fogDrift1/fogDrift2` with translateX/Y, 8-12s
- `.shaking` class: `@keyframes tremble` with rapid translate ±2px
- `.shaking .fog-1, .shaking .fog-2`: faster animation, rotation added
- `.shaking .crystal-ball`: pulsing box-shadow
- `.reveal #answer-text`: opacity 0→1, translateY 10→0
- Neon text effects: text-shadow with multiple layers for glow
- `@media (prefers-reduced-motion: reduce)`: disable fog, tremble, pulse; keep opacity-only reveal
- Title "8BALL" in `#ff00cc` with neon glow
- All text sizing with `clamp()` for responsive
- `.neon-line` gradient line
- Screen-specific show/hide classes

- [ ] **Step 3: Verify visual in browser**

Open `/tmp/8ball/index.html` in browser. Verify:
- Dark background renders
- Crystal ball visible with fog animation
- Neon glow on title and ring
- "לנער חזק ולשאול בלב" text visible
- RTL layout correct

- [ ] **Step 4: Commit**

```bash
cd /tmp/8ball
git add index.html
git commit -m "feat: add HTML structure and CSS with neon crystal ball design"
```

---

### Task 3: JavaScript (Interaction Layer)

**Files:**
- Modify: `/tmp/8ball/index.html` (add `<script>` block at end of body)

**Depends on Task 2.** If running parallel, this task writes JS as a standalone block that gets inserted into index.html after Task 2 completes.

- [ ] **Step 1: Write the answer pool**

Array of 50+ Hebrew strings, categorized by tone. Gender-neutral. See spec for full list.

```javascript
const answers = [
  // Positive (~30%)
  'כן', 'בהחלט כן', 'הכוכבים אומרים כן', 'בלי שום ספק',
  'זה הולך לקרות', 'אפשר לסמוך על זה', 'בדרך אליך', 'ללא ספק',
  'חד משמעית כן', 'התשובה חיובית', 'סימנים טובים', 'יותר כן מלא',
  'הכל מצביע על כן', 'בהחלט', 'אני הייתי הולך על זה',
  // Negative (~20%)
  'לא', 'ממש לא', 'לא בחיים האלה', 'בגלגול הבא אולי',
  'תשכחו מזה', 'הכוכבים צוחקים', 'לא הפעם', 'שלילי לגמרי',
  'אפילו לא שווה לנסות', 'חד משמעית לא',
  // Evasive (~25%)
  'אולי מחר', 'צריך לחשוב שוב', 'שאלתם את אמא?', 'הרוחות שותקות',
  'לא עכשיו', 'תנערו שוב, לא שמעתי', 'הערפל כבד מדי',
  'לשאול שוב מאוחר יותר', 'לא ברור עדיין', 'ממממ... לא בטוח',
  'קשה להגיד', 'תחזרו מחר', 'יכול להיות שכן ויכול להיות שלא',
  // Philosophical/Cheeky (~25%)
  'התשובה כבר ידועה, לא?', 'מספיק לשאול. הגיע הזמן לבצע',
  'התשובה בפנים, לא בכדור', 'למה לשאול אותי? תסתכלו במראה',
  'אם צריך לשאול, כנראה כן', 'תפסיקו לשאול ותתחילו לעשות',
  'שאלה טובה. תשובה? לא בטוח', 'הלב יודע. הכדור רק מאשר',
  'אין תשובות קלות לשאלות טובות', 'הכדור לא יודע הכל, אבל יודע את זה',
  'תנסו ותראו', 'מי שלא מנסה לא יודע', 'זה תלוי רק בך'
];
```

- [ ] **Step 2: Write shake detection logic**

```javascript
let lastAnswer = -1;
let cooldown = false;
let shakeCount = 0;
let lastShakeTime = 0;
const THRESHOLD = 12;
const SHAKE_WINDOW = 500;
const COOLDOWN_MS = 2000;

function handleMotion(e) {
  if (cooldown) return;
  const { x, y, z } = e.accelerationIncludingGravity || {};
  if (x == null) return;
  const magnitude = Math.sqrt(x * x + y * y + z * z);
  const net = Math.abs(magnitude - 9.8);
  if (net > THRESHOLD) {
    const now = Date.now();
    if (now - lastShakeTime < SHAKE_WINDOW) {
      shakeCount++;
    } else {
      shakeCount = 1;
    }
    lastShakeTime = now;
    if (shakeCount >= 2) {
      triggerShake();
      shakeCount = 0;
    }
  }
}
```

- [ ] **Step 3: Write state management (idle → shaking → reveal)**

```javascript
function triggerShake() {
  cooldown = true;
  const ball = document.querySelector('.crystal-ball');
  const app = document.getElementById('app');
  app.classList.remove('reveal');
  app.classList.add('shaking');

  setTimeout(() => {
    app.classList.remove('shaking');
    app.classList.add('reveal');
    showAnswer();
    cooldown = false;
  }, 1500);

  setTimeout(() => { cooldown = false; }, COOLDOWN_MS);
}

function showAnswer() {
  let idx;
  do {
    idx = Math.floor(Math.random() * answers.length);
  } while (idx === lastAnswer && answers.length > 1);
  lastAnswer = idx;
  document.getElementById('answer-text').textContent = answers[idx];
  document.getElementById('instruction').textContent = 'לנער שוב לשאלה חדשה';
}
```

- [ ] **Step 4: Write iOS permission flow**

```javascript
function initMotion() {
  if (typeof DeviceMotionEvent !== 'undefined' &&
      typeof DeviceMotionEvent.requestPermission === 'function') {
    // iOS — show permission button
    document.getElementById('ios-permission').style.display = 'flex';
    document.getElementById('ios-btn').addEventListener('click', async () => {
      const perm = await DeviceMotionEvent.requestPermission();
      document.getElementById('ios-permission').style.display = 'none';
      if (perm === 'granted') {
        window.addEventListener('devicemotion', handleMotion);
      } else {
        enableTapFallback();
      }
    });
  } else if (typeof DeviceMotionEvent !== 'undefined') {
    // Android — just listen
    window.addEventListener('devicemotion', handleMotion);
  } else {
    // Desktop — show fallback
    document.getElementById('app').style.display = 'none';
    document.getElementById('desktop-fallback').style.display = 'flex';
  }
}
```

- [ ] **Step 5: Write tap fallback + no-sensor hint**

```javascript
function enableTapFallback() {
  document.querySelector('.crystal-ball').addEventListener('click', () => {
    if (!cooldown) triggerShake();
  });
  document.getElementById('no-sensor-hint').style.display = 'block';
}

// Show tap hint after 10s if no shake detected
let hintTimeout = setTimeout(() => {
  if (!document.getElementById('app').classList.contains('reveal')) {
    document.getElementById('no-sensor-hint').style.display = 'block';
    enableTapFallback();
  }
}, 10000);
```

- [ ] **Step 6: Wire up initialization + service worker registration**

```javascript
document.addEventListener('DOMContentLoaded', () => {
  initMotion();
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js');
  }
});
```

- [ ] **Step 7: Test on mobile**

Open on phone (via local network IP or deploy). Verify:
- iOS: permission button appears, tap grants access, shake works
- Android: shake works immediately
- Shake → ball trembles → answer appears
- No consecutive duplicate answers
- "לנער שוב לשאלה חדשה" appears after reveal
- Tap fallback works after 10s or after permission denied
- Desktop: shows fallback message

- [ ] **Step 8: Commit**

```bash
cd /tmp/8ball
git add index.html
git commit -m "feat: add shake detection, answer logic, iOS permission, and tap fallback"
```

---

### Task 4: Desktop Fallback with QR Code

**Files:**
- Modify: `/tmp/8ball/index.html` (add QR SVG and desktop fallback section)

**Can run in parallel with Task 3** as a separate HTML section.

- [ ] **Step 1: Add desktop fallback HTML with inline QR SVG**

The `#desktop-fallback` div should contain:
- "הכדור עובד רק בטלפון 📱" heading
- "סרקו את הקוד ופתחו בטלפון" subtitle
- Inline SVG QR code (placeholder — actual QR generated once URL is known)
- Same dark neon background as main app

- [ ] **Step 2: Style desktop fallback**

Centered layout, neon styling consistent with app. QR code with subtle neon border.

- [ ] **Step 3: Commit**

```bash
cd /tmp/8ball
git add index.html
git commit -m "feat: add desktop fallback with QR code"
```

---

### Task 5: Final Polish + Integration

**Files:**
- Modify: `/tmp/8ball/index.html`

**Depends on Tasks 1-4 being complete.**

- [ ] **Step 1: Verify all CSS animations work together**

Check idle fog → shaking tremble + swirl → reveal text. Verify transitions are smooth.

- [ ] **Step 2: Test prefers-reduced-motion**

Enable reduced motion in OS settings. Verify fog stops, tremble stops, text still appears (opacity only).

- [ ] **Step 3: Test PWA install flow**

Open in mobile Safari/Chrome. Verify "Add to Home Screen" works. Open from home screen — verify standalone mode, correct theme color, no browser chrome.

- [ ] **Step 4: Final commit**

```bash
cd /tmp/8ball
git add -A
git commit -m "feat: final polish and integration verification"
```

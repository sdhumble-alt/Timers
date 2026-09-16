# Classroom Smart Timers

A single-file, zero-dependency, touch-optimized web application built for interactive classroom displays (Smart Boards, Promethean Boards, touch panels) and desktop web browsers.

---

## 1. Executive Summary

**Classroom Smart Timers** was developed to solve common pain points educators face when managing classroom pacing on interactive displays:
* **No Keyboard Required:** Every operational setting (durations, round counts, precision, sounds, volume, themes, and navigation) is adjustable via large, high-contrast touch targets.
* **Responsive Canvas & Collapsible Drawer:** Features an adaptive layout with a floating drawer toggle button (`☰`) and automatic media queries that tuck the sidebar away when viewport widths drop below 900px, maximizing workspace during split-screen presentation.
* **Anti-Throttling Engine:** Background tab throttling is bypassed via an isolated Web Worker heartbeat thread, ensuring completion alerts chime on time even when teaching from another window.
* **100% Offline & Portable:** Delivered as a single `.html` file with zero external assets, npm packages, CDN dependencies, or hosted sound files.

---

## 2. Timer Modes & Functional Specifications

### Countdown
* **Purpose:** Standard timed classroom tasks, quizzes, sustained silent reading, and structured transitions.
* **Controls:**
  * Three dedicated steppers (`▲` / `▼`) for **Hours**, **Minutes**, and **Seconds**.
  * Dynamic timestamp formatting: automatically displays `MM:SS` for durations under one hour and expands to `HH:MM:SS` when an hour or more is loaded.
* **Precision Mode:** Toggle between **Whole Second** and **Precise** (hundredths of a second, `.XX`).
* **Auto-Start Presets:** Tapping any preset button (15s to 60m) instantly sets the duration and starts the countdown.

### Turn Timer
* **Purpose:** Rotational classroom tasks, station work, debate speaking windows, group rotations, and student presentations.
* **Turn Counter:**
  * Located in a distinctly highlighted purple container for clear visibility across the room.
  * Adjusts with dedicated `▲` / `▼` stepper buttons.
  * **Infinite Mode:** Stepping down below 1 turn toggles the target to **Infinity (`∞`)**, allowing turns to cycle continuously without a preset end round.
* **Progress Tracking:**
  * Live status pill indicates current stage (`Turn X of Y` or `Turn X of ∞`).
  * Dedicated progress bar smoothly depletes over the course of each active turn.
* **Sound Feedback:**
  * Plays an ascending **Harp Chime** when each turn expires and automatically rolls directly into the next turn.
  * Tolls the primary completion chime twice only when the final turn has elapsed (in finite mode).
* **Safe Presets:** Tapping a preset loads the turn duration without auto-starting, ensuring instructions can be delivered before beginning.

### Visual Timer
* **Purpose:** Non-numeric and analog time tracking, ideal for early learners, special education accommodations, or low-stress pacing.
* **Display Modes:**
  * **Pie Mode:** Solid circular chart that visually drains counter-clockwise from 12 o'clock, showing elapsed time in red (`#ef4444`) and remaining time in green (`#10b981`).
  * **Bar Mode:** Full-width horizontal gauge draining smoothly from right to left.
* **Anti-Overlap Typography:** Time digits are positioned cleanly underneath the graphic elements rather than layered over the center, ensuring readability across all window aspect ratios.

---

## 3. Audio & Subsystem Architecture

### Synthetic Audio Engine (Web Audio API)
To remove reliance on external MP3/WAV files that can fail due to network restrictions or CORS policies, all sound effects are synthesized mathematically in real time:

> **Audio Signal Pipeline:**
> `[Oscillators]` ➔ `[Pre-Gain Stage (2.5x Booster)]` ➔ `[Master Volume (0–100%)]` ➔ `[Dynamics Compressor (-12dB)]` ➔ `[Hardware Speakers]`

* **Pre-Gain Stage:** Boosts sound generation by **2.5×** ahead of the master volume control so alerts remain audible on large, low-powered classroom interactive displays.
* **Dynamics Compressor:** Ensures high-gain tones and dual-bell strikes do not clip or distort through built-in flat-panel speakers.
* **Built-in Sound Library:**
  * **Bell Toll:** Dual low-frequency church bell strikes with natural decay.
  * **Harp Chime:** Ascending 6-note glissando.
  * **Digital Beep:** Rapid triple-pulse square wave.
  * **Marimba Chime:** Warm 4-note wooden acoustic chord.
  * **8-Bit Level Up:** Vintage arcade coin sweep.
  * **Zen Singing Bowl:** Sustained dual-frequency gong with acoustic beating.
  * **School Bell:** High-frequency alternating striker bell simulation.
  * **Space Radar Ping:** Resonant sonar pulse with extended reverberation tail.

### Background Tab Execution (Web Worker)
Web browsers routinely throttle or suspend background tabs, which stops standard JavaScript timers (`setInterval`, `requestAnimationFrame`).

* **Implementation:** The application instantiates an inline Web Worker via a `Blob` URL (`URL.createObjectURL`).
* **Execution:** The worker runs on an isolated background thread that browsers do not throttle, emitting a steady 100 ms tick pulse.
* **Synchronization:** Time calculations compute against real hardware clock deltas (`Date.now()`). If a teacher switches tabs to present slides or take attendance, the alert still sounds precisely on time.

---

## 4. UI/UX, Layout & Responsive Drawer

* **Floating Sidebar Toggle (`☰`):** An accessible control button fixed at the top-left permits teachers to collapse the navigation drawer completely.
* **Auto-Collapse Behavior:** Utilizing pure CSS media queries (`@media (max-width: 900px)`), the sidebar automatically stows off-screen on narrower screens or split-screen configurations, while sliding in as an overlay when summoned.
* **Touch-Friendly Modals:** Theme and chime selections trigger dedicated touch-grid modal windows with oversized tiles and integrated sound previews instead of cramped system dropdowns.

### Available Themes
| Theme ID | Name | Background | Primary Accent | Typography |
| :--- | :--- | :--- | :--- | :--- |
| `default` | **Default Dark** | Slate Navy (`#131722`) | Vivid Blue (`#3b82f6`) | Modern System Sans |
| `chalkboard` | **Classroom Chalkboard** | Chalkboard Green (`#1c2e24`) | Chalk Yellow (`#facc15`) | Casual Classroom Script |
| `neon` | **Digital Neon** | Deep Black (`#0a0a0f`) | Cyan (`#00ffff`) | Monospace Digital |
| `pastel` | **Warm Pastel** | Muted Plum (`#24222f`) | Soft Lavender (`#a78bfa`) | Rounded Sans-Serif |
| `arcade` | **Retro Arcade** | 8-Bit Void (`#0b0819`) | Coin-Op Amber (`#f59e0b`) | Pixel / Terminal Monospace |
| `cafe` | **Cozy Cafe** | Espresso Brown (`#1f1b18`) | Warm Sand (`#ddb892`) | Classic Editorial Serif |
| `space` | **Deep Space** | Nebula Black (`#060814`) | Cosmic Indigo (`#6366f1`) | Clean Geometric Sans |
| `primary-edu` | **Elementary Light** | Clean White (`#ffffff`) | Royal Blue (`#2563eb`) | Bold Rounded Schoolhouse |
| `high-contrast`| **High Contrast** | Solid Black (`#000000`) | Pure Yellow (`#ffff00`) | Ultra-Bold Display Sans |

---

## 5. Local Storage Schema

User selections persist automatically in the browser's `localStorage` to retain preferences between reboots:

* `timer_volume`: Float (`0.0` to `1.0`, default: `0.8`)
* `timer_chime`: String identifier (`bell`, `harp`, `beep`, `marimba`, `arcade`, `zen`, `school`, `sonar`)
* `timer_theme`: String theme key (`default`, `chalkboard`, `neon`, etc.)
* `timer_visual_mode`: Active graphic state (`pie` or `bar`)
* `timer_ms_cd`: Precision toggle boolean for Countdown (`true` / `false`)
* `timer_ms_turn`: Precision toggle boolean for Turn Timer (`true` / `false`)
* `timer_ms_vis`: Precision toggle boolean for Visual Timer (`true` / `false`)

---

## 6. Quick Presets Reference

All presets appear along the bottom of the interface in a wrapping touch grid:

> **Available Durations:**
> `15s` • `30s` • `45s` • `1m` • `2m` • `3m` • `4m` • `5m` • `10m` • `15m` • `20m` • `25m` • `30m` • `45m` • `60m`

---

## 7. Version History

* **V1.0.0 (09-06-2026)**
  * Initial production release.
  * Added Countdown, Turn Timer (finite & infinite modes), and Visual Timer (pie & bar charts).
  * Implemented Web Audio synthesizer engine with 8 selectable chimes and pre-gain boosting.
  * Added Web Worker background thread execution to prevent background tab sleep.
  * Integrated 9 classroom themes with full touch-modal selection interfaces.
  * Added collapsible navigation drawer with automated responsive rules below 900px viewport width.
  * Separated Visual Timer typography below graphic viewport to guarantee zero text collision.
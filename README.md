# Deskhelp — Tactile Grounding Adaptive Lamp

An interactive, browser-based prototype of **Deskhelp**, a concept desk lamp that senses stress through heart rate variability (HRV) and responds with a coordinated set of calming interventions: breathing light, generative sound, haptic grounding, and aromatherapy.

Everything runs in a single HTML file. No build step, no backend, no dependencies to install.

> ⚠️ **This is a design and interaction prototype, not a medical device.** The HRV signal is simulated and the stress scoring is illustrative. It is not intended to diagnose, treat, or monitor any health condition.

---

## Features

- **3D product model** built with Three.js: articulated lamp arm, glowing LED ring head, aroma diffuser vent, and a low-profile dome-shaped tactile pad. Drag to orbit the model.
- **Five adaptive stress levels** (Calm → Severe), each with its own light colour, sound, haptic, and aroma behaviour.
- **Breath-synced behaviour.** A single master breathing wave drives the light intensity, ring scale, dome glow, and sound volume together.
- **Generative audio** via the Web Audio API. All sounds are synthesised in the browser with no audio files.
- **Haptic pad simulation** with slow dome rotation, micro-vibration, pulsing ripples, and cycling surface textures (ridges, spokes, nodules, wave fins).
- **Aroma diffuser simulation** with a 2 s spray / 3 s pause cycle and colour-tinted mist particles from the tower vent.
- **HRV detector panel** with simulated R-R intervals, a moving-average filter, a rolling RMSSD window, and a live stress trend graph that drives the lamp in real time.
- **Manual override** to toggle each hardware component independently.
- **Emergency contact flow.** At Level 5, a modal offers to send a pre-written message via WhatsApp or SMS deep link.

---

## Stress levels

| Level | State    | Light                | Sound                              | Haptic | Aroma |
|:-----:|----------|----------------------|------------------------------------|:------:|:-----:|
| 1     | Calm     | Soft yellow, breathing | Silence                          | Off    | Off   |
| 2     | Mild     | Warm amber           | Theta-wave style drone (120 / 180 Hz) | Off | Off   |
| 3     | Moderate | Amber                | Rain + forest noise soundscape     | Off    | On    |
| 4     | High     | Deep orange          | Ocean waves + wind                 | On (light) | On |
| 5     | Severe   | Dim, low-intensity   | Om-style drone (136.1 Hz harmonics) | On (strong) | On |

At higher levels, the breathing cycle slows down (Level 5 uses a slow 5-in / 5-out rhythm), the status ring pulses more slowly, and the haptic texture cycles faster.

---

## How the stress pipeline works

```
Simulated PPG R-R intervals
        │
        ▼
Moving-average filter
        │
        ▼
RMSSD (rolling window)
        │
        ▼
Stress score (1–5)  ──►  setLevel()  ──►  light · sound · haptic · aroma
```

RMSSD thresholds used for scoring in the simulator:

| RMSSD (ms) | Stress level |
|------------|:------------:|
| > 60       | 1 |
| 45 – 60    | 2 |
| 32 – 45    | 3 |
| 20 – 32    | 4 |
| ≤ 20       | 5 |

In the current prototype, the lamp follows the **Stress slider** directly so that moving it always produces a visible response. RMSSD and heart rate are still computed from the noisy simulated signal to make the charts behave like a real detector.

---

## Getting started

### Run locally

1. Save the file as `index.html` (or clone this repo).
2. Open it in a modern browser (Chrome, Edge, Firefox, Safari).

That's it. An internet connection is needed on first load for the CDN scripts and fonts.

Alternatively, serve it with any static server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

### Using the demo

1. **Pick a stress level** in the right sidebar (1–5), or open the **HRV DETECTOR** panel at the bottom and move the *Stress* slider.
2. **Click anywhere on the page once** if you hear no sound. Browsers require a user gesture before audio can start.
3. **Drag** the 3D view to orbit the lamp.
4. Use **Manual override** to switch the LED, speaker, haptic pad, diffuser, or status ring on or off individually.
5. Add an **Emergency Contact** (name, phone, custom message) and press *Save contact*. Switching to Level 5 will then show an alert modal.

---

## Tech stack

- [Three.js r128](https://threejs.org/) with `OrbitControls` (loaded from cdnjs / jsDelivr)
- Web Audio API for oscillators, filtered noise, and LFO-modulated ocean waves
- Canvas 2D for the mist particles, haptic ripples, and HRV charts
- Vanilla HTML, CSS, and JavaScript
- [DM Sans](https://fonts.google.com/specimen/DM+Sans) and [DM Mono](https://fonts.google.com/specimen/DM+Mono) from Google Fonts

---

## Project structure

```
.
├── index.html   # entire prototype: markup, styles, 3D scene, audio, HRV simulator
└── README.md
```

Inside `index.html`, the main sections are:

| Section | Purpose |
|---------|---------|
| `LEVELS` config | Per-level colours, sound type, haptic rate, and which components are active |
| Three.js scene setup | Lamp geometry, materials, lights, haptic dome, and diffuser vent |
| Overlay canvas | Aroma mist particles and haptic ripple effects |
| `animate()` | Main render loop with breath wave, light, dome rotation, and texture cycling |
| Sound engine | `startRainSound`, `startOceanSound`, and oscillator-based tone drones |
| Emergency contact system | Save contact, level-5 modal, and WhatsApp/SMS deep links |
| HRV simulator | Signal generation, filtering, RMSSD, stress scoring, and trend graph |

---

## Customisation

- **Change level behaviour:** edit the `LEVELS` object (colours, frequencies, volume, haptic pulse rate, active components).
- **Change breathing pace:** edit `BREATH_SPEEDS` (radians per second).
- **Change dome rotation speed:** edit `DOME_ROT_SPEEDS`.
- **Change aroma timing:** edit `AROMA_SPRAY_DUR` and `AROMA_PAUSE_DUR`.
- **Change haptic texture interval:** edit `HAPTIC_TEXTURE_INTERVALS`.
- **Change stress thresholds:** edit `stressFromRMSSD()`.
- **Change trend sampling rate:** edit `TREND_SAMPLE_INTERVAL`.

---

## Privacy

The emergency contact details are held in memory only and are never sent to a server or stored. When you tap *Open WhatsApp* or *Open SMS*, the message is passed to your device's own app through a standard `wa.me` or `sms:` link.

---

## Limitations and roadmap

- HRV data is **simulated**. Integrating a real PPG sensor (for example over Web Bluetooth) is the natural next step.
- The stress model is a simple RMSSD threshold and is not clinically validated.
- Aroma, haptics, and light are visual and audio simulations of hardware that does not yet exist.
- Level 5 alerts require the user to confirm and send manually. There is no automatic dispatch.

Ideas for future work:

- [ ] Web Bluetooth heart-rate / PPG sensor input
- [ ] Personalised baselines instead of fixed RMSSD thresholds
- [ ] Persisted settings and emergency contact (localStorage)
- [ ] Guided breathing overlay
- [ ] Physical hardware build (ESP32, LED ring, haptic actuator, ultrasonic diffuser)

---

## Browser support

Works in current versions of Chrome, Edge, Firefox, and Safari. WebGL and Web Audio support are required.

---

## License

Add your preferred license here (for example MIT) and include a `LICENSE` file.

---

## Acknowledgements

Built with [Three.js](https://threejs.org/). Typography by Google Fonts.

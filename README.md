# Gymmer — my gym roadmap

A single-page, phone-first gym plan: 3-phase beginner→intermediate training roadmap
(with form videos and anatomical muscle heatmaps **per workout and per exercise**),
nutrition targets for a 75 kg lean bulk — including plain-language explainers for
protein, carbs, fat, fiber and calories, and a reason for every Serbian budget food —
an evidence-based supplement guide (Serbian shops/prices), and a tap-to-log
training consistency tracker.

Layout styled after [Overlord Gym Tracker](https://github.com/strahinja0212-ctrl/Overlord-Gym-Tracker); six Space Marine chapter themes —
**Ultramarines · Blood Angels · Dark Angels · Space Wolves · Imperial Fists · Black Templars**.

Everything lives in **`index.html`** — no build step, no dependencies.
Only the YouTube form videos and the Google Fonts stylesheet need internet;
offline it falls back to system fonts and everything still works.

## Open it locally

Just double-click `index.html`, or:

```powershell
start index.html
```

## Live

**https://strahinja0212-ctrl.github.io/gymmer/** — served by GitHub Pages from `main` / root.
Open it on your phone and **Add to Home Screen**; it behaves like an app.

## Notes

- Tracker data (trained days, logged weights, start date) is stored in the browser's
  `localStorage` — **per device**. Use *Tracker → Copy backup / Import* to move it
  between phone and PC.
- Repo is public by default on GitHub Pages free tier — the page contains no personal
  data (your logs stay in your browser).

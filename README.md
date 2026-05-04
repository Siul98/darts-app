# Darts Auto-Score

Web-Prototyp für eine Darts-App mit automatischem Score-Erkennen via Kamera. Geplantes Endziel: native iOS-App. Diese Web-Version dient als schneller Iterations-Prototyp für die Kamera-Erkennung.

Roadmap und Kontext: siehe `~/.claude/plans/es-gibt-so-dartsapps-tender-rabin.md`.

## Aktueller Stand (Phase 0)

- Single-File `index.html`, kein Build-Step.
- Kamera-Zugriff via `getUserMedia` (rückwärtige Kamera bevorzugt).
- 4-Punkt-Kalibrierung (12/3/6/9 Uhr Außenkante Doppel) mit Perspektiv-Transform via Homographie.
- Test-Modus: Tap aufs Bild → korrekter Dart-Score (Single/Double/Triple/Bull/Bullseye/Miss).
- SVG-Overlay zeigt erkanntes Board (Rings + Segment-Linien) live über dem Kamerabild.
- Self-Test der Score-Logik im DevTools-Console (9/9 Cases).

## Phase 1 (geplant)

- Frame-Diff-Erkennung neuer Pfeile.
- Pfeilspitzen-Detection.
- Live-Score-Anzeige für 3 Würfe.

## Lokale Entwicklung

Die `.claude/launch.json` im Parent-Ordner enthält eine `darts`-Config:

```bash
# manuell:
python3 -m http.server 8766
# dann http://localhost:8766/ öffnen
```

**Wichtig für iPhone-Tests:** `getUserMedia` braucht HTTPS. Lokal entweder
- via `ngrok http 8766` einen HTTPS-Tunnel öffnen, oder
- nach GitHub Pages deployen (siehe unten).

## Deploy via GitHub Pages

```bash
# einmalig:
gh repo create darts-app --private --source=. --remote=origin
git add .
git commit -m "Phase 0: Skelett mit Kalibrierung und Score-Test"
git push -u origin main

# Pages aktivieren (Branch: main, Folder: /):
gh api -X POST repos/{owner}/darts-app/pages -f source[branch]=main -f source[path]=/
```

Nach ~30s ist die App live unter `https://<owner>.github.io/darts-app/`. Auf dem iPhone öffnen → Kamera-Permission erlauben → loslegen.

## Architektur (Single-File-Pattern)

```
darts-app/
└── index.html      # Alles inline: HTML, CSS, JS
```

Globale Strukturen in `<script>`:
- `S` — Runtime-State (Stream, Video, Kalibrierungspunkte, Homographie, Modus).
- `SEGMENTS` — Board-Sektoren-Reihenfolge (im Uhrzeigersinn ab 20 oben).
- `RINGS` — Ring-Radien normiert auf Außenkante Doppel = 1.0.
- `computeHomography()` — 4-Punkt-Perspektivtransform via Gauß-Elimination.
- `scoreFromBoardCoord(bx, by)` — Polarkoordinaten → Sektor + Ring → Score.
- `applyH()`, `invertMatrix3()` — Homographie-Anwendung in beide Richtungen.

Kein Build, keine NPM-Deps. Wenn OpenCV.js / TensorFlow.js dazukommen (Phase 1+): über `<script>`-Tag als externe Resource laden, kein Bundling.

## Design-Entscheidungen

- **iOS-First UI:** Liquid-Glass, dunkles Theme, Safe-Area-aware (`env(safe-area-inset-*)`), `viewport-fit=cover`.
- **Kein Build-Step:** maximale Iterations-Geschwindigkeit, einfacher Deploy.
- **Tap-to-Test-Modus:** vor dem Auto-Detect eine deterministische Verifikation der Kalibrierung möglich (User tippt → sieht den Score, den ein Dart an dieser Pixelposition hätte).

# 3D Solar System - Design Spec

**Date**: 2026-06-15
**Status**: Approved
**Type**: Single-file HTML application

---

## Overview

A realistic 3D interactive solar system model built with Three.js, delivered as a single self-contained HTML file. All resources loaded from public CDNs. Zero local dependencies.

---

## Architecture

### Tech Stack
- **3D Engine**: Three.js r160+ via ES importmap CDN
- **Camera Controls**: OrbitControls (same CDN)
- **Textures**: Canvas procedural generation (primary) + Three.js example textures (fallback)
- **UI**: Pure CSS frosted glass panels + native DOM events
- **Effects**: Custom shaders (sun glow) + CSS animations

### Component Tree
```
HTML Document
├── Canvas #three-canvas — Three.js render target
├── UI Overlay (pointer-events: none container)
│   ├── Control Panel #control-panel — top-right, frosted glass
│   │   ├── Orbit Toggle Button
│   │   ├── Zoom Preset Buttons (3 levels)
│   │   ├── Reset View Button
│   │   ├── Speed Slider + Value Display
│   │   └── Star Twinkle Toggle
│   └── Info Modal #info-modal — dynamically positioned, frosted glass
│       ├── Celestial body name
│       ├── Diameter / Rotation period / Orbital period
│       └── Description text
└── Three.js Scene
    ├── Star particle system (BufferGeometry, multi-layer)
    ├── Ambient light + Point light (at sun position)
    ├── Sun (sphere + glow Sprite + flare)
    ├── Orbit rings (TorusGeometry, semi-transparent)
    ├── 8 Planets (spheres, procedural textures, rotation + revolution)
    │   ├── Earth → cloud layer sphere (slightly larger, semi-transparent)
    │   └── Saturn → multi-layer rings (RingGeometry)
    └── Moon (orbits Earth)
```

### Data Flow
```
User Action → Event Handler → State Update → Render
├── Mouse drag/scroll → OrbitControls → Camera transform
├── Canvas click → Raycaster → Hit test
│   ├── Hit → Highlight body + Show modal (with data)
│   └── Miss → Clear highlight + Hide modal
├── Panel buttons → State toggle → Scene property update
│   ├── Orbit toggle → orbits.visible
│   ├── Zoom level → Camera position (animated)
│   ├── Reset → Camera defaults restore
│   ├── Speed slider → global timeScale
│   └── Twinkle toggle → enabled/disabled flag
└── rAF loop → Update rotation/revolution × timeScale → Render
```

### Texture Strategy
All planet textures generated procedurally via Canvas 2D API:
- **Sun**: Orange-red gradient + bright spot noise
- **Mercury/Moon**: Gray crater noise
- **Venus**: Pale yellow cloud texture
- **Earth**: Blue-green simplified continents + white cloud layer
- **Mars**: Red-brown rocky texture
- **Jupiter**: Orange-brown banded gradient
- **Saturn**: Pale yellow bands + RingGeometry rings
- **Uranus**: Pale cyan-blue
- **Neptune**: Deep blue

### UI Styling
- Frosted glass: `backdrop-filter: blur(20px)` + semi-transparent dark bg + thin border
- Sci-fi theme: Cyan/blue accent colors, rounded corners, subtle glow borders
- Button feedback: hover color shift + `scale(0.95)` click press
- Modal: fade-in animation, positioned near clicked body or centered

### Interaction Spec
- **Left drag**: Rotate view (OrbitControls with damping)
- **Scroll**: Zoom in/out
- **Right drag**: Pan view
- **Click planet/sun/moon**: Show info modal + highlight glow
- **Click empty space**: Close modal + clear highlight
- **Re-click same body**: Close modal (toggle behavior)
- **Panel controls**: All mutually compatible, no conflicts

### Planet Data Table
| Body | Relative Size | Orbit Radius | Color |
|------|-------------|-------------|-------|
| Sun | 5.0 | 0 | Orange |
| Mercury | 0.4 | 8 | Gray |
| Venus | 0.9 | 12 | Yellowish |
| Earth | 1.0 | 16 | Blue-green |
| Mars | 0.5 | 20 | Red-brown |
| Jupiter | 3.0 | 28 | Orange-brown bands |
| Saturn | 2.5 | 36 | Pale yellow + rings |
| Uranus | 1.8 | 44 | Cyan |
| Neptune | 1.7 | 50 | Deep blue |
| Moon | 0.27 | Earth orbit + 2.5 | Gray |

---

## Code Structure (Sections)
1. HTML structure (canvas, UI elements)
2. CSS styles (frosted glass panels, animations)
3. Constants & Planet data configuration
4. Scene initialization (renderer, scene, camera, controls)
5. Lighting setup
6. Starfield generation
7. Planet texture generation (Canvas procedurals)
8. Celestial body creation (sun, planets, moon, rings)
9. Orbit ring creation
10. Animation loop
11. Raycaster interaction (click, highlight, modal)
12. Control panel event handlers
13. Window resize handler

---

## Acceptance Criteria
1. ✅ Opens in browser, no console errors, no local files needed
2. ✅ All 8 planets + sun + moon visible with correct relative sizes
3. ✅ Orbits drawn, planets revolve at different speeds
4. ✅ Mouse controls work (rotate, zoom, pan)
5. ✅ Click celestial body → info modal appears
6. ✅ Control panel: orbit toggle, zoom presets, reset, speed slider, twinkle toggle
7. ✅ All features work simultaneously without conflicts
8. ✅ Smooth 60fps animation

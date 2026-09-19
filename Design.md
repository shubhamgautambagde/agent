# Design System & UI Specification (Design.md)

This document specifies the visual design language, theme contract, typography, color palette, and component design patterns used across the **Rapida Voice AI** web console.

---

## 1. Visual Philosophy & Theme Contract

Rapida's design language conveys precision, enterprise reliability, and real-time responsiveness. It uses an adaptive dual-theme system governed by the `data-color-mode` HTML attribute.

### 1.1 Theme Contract
Theme switching is managed at the root element:
```html
<!-- Dark Mode -->
<html data-color-mode="dark">
<!-- Light Mode -->
<html data-color-mode="light">
```
Tailwind CSS custom variant:
```css
@custom-variant dark (&:where([data-color-mode='dark'], [data-color-mode='dark'] *));
```

---

## 2. Typography

The platform utilizes **IBM Plex** to balance clean readability with technical precision.

| Role | Font Family | Weight / Variable | Usage |
| :--- | :--- | :--- | :--- |
| **Primary Sans** | `'IBM Plex Sans', sans-serif` | Regular (400)<br>Medium (450)<br>Semibold (500)<br>Bold (700) | Main UI labels, forms, tables, card headers, and button text. |
| **Code & Data** | `'IBM Plex Mono', monospace` | Regular (400)<br>Medium (500) | Transcripts, latency timers, audio timestamps, JSON payloads, API keys, and code editors. |

### Hierarchy Guidelines
- **Page Titles**: `text-2xl font-semibold tracking-tight`
- **Section Headers**: `text-lg font-medium text-foreground`
- **Body Text**: `text-sm font-normal text-foreground leading-relaxed`
- **Secondary / Helper Text**: `text-xs font-normal text-muted`
- **Technical Badges / Metrics**: `font-mono text-xs font-medium`

---

## 3. Color Palette & Semantic Tokens

### 3.1 Core Semantic Tokens

| Token | CSS Variable / Value | Dark Mode Role | Light Mode Role |
| :--- | :--- | :--- | :--- |
| `primary` | `var(--brand-primary)` | Accent buttons, active tabs, selected states | Main brand focus |
| `surface` | `var(--cds-background)` | Root app background canvas (`#0d0d0d` or `#161616`) | Root app canvas (`#ffffff` or `#f8f8f8`) |
| `shell` / `layer` | `var(--cds-layer-01)` | Sidebar, navbar, container card backgrounds | Card containers, sidebar panels |
| `layer-hover` | `var(--cds-layer-hover-01)` | Hovered row, menu item, or button surface | Hovered row or surface |
| `foreground` | `var(--cds-text-primary)` | High-contrast white text (`#fcfcfc`) | Deep slate/black text (`#1a1a1a`) |
| `muted` | `var(--cds-text-secondary)` | Subtitle text, secondary labels (`#808080`) | Subtitle text, muted metadata (`#666666`) |
| `border-subtle`| `var(--cds-border-subtle-01)` | Dividers, card borders, table line separators | Subtle boundaries |
| `border-strong`| `var(--cds-border-strong-01)` | Active input borders, focused modal outlines | Input borders on focus |

### 3.2 Status & Feedback Accents

| Status | Color Name | Hex / Mix | Usage |
| :--- | :--- | :--- | :--- |
| **Live / Active** | Emerald Green | `#10b981` | Real-time call in progress, active WebSocket connection, healthy node. |
| **Speaking (TTS)**| Indigo / Violet | `#6366f1` | Assistant active speech stream, audio synthesis waveform. |
| **Listening (STT)**| Cyan / Sky | `#0ea5e9` | Microphone active, user speech detected (VAD). |
| **Warning** | Amber | `#f59e0b` | Provider latency degradation, near rate limit. |
| **Error / Interrupted**| Rose Red | `#ef4444` | Upstream failure, call drop, VAD interruption. |

---

## 4. UI Components & Layout Patterns

### 4.1 Shell & Navigation
- **Sidebar**: Fixed-width, collapsible left navigation containing Project switcher, Voice Assistants, Phone Numbers, Knowledge, Endpoints, and Settings.
- **Top Bar**: Tenant/Organization breadcrumbs, live environment status, theme toggle, and user profile avatar.
- **Content Area**: Responsive container with subtle inner border and comfortable 24px padding (`p-6`).

### 4.2 Real-Time Voice Visualizers
- **Waveform Sphere / Ring**: Central visualizer indicating voice conversation state:
  - *Idle*: Subtle breathing pulse (border opacity transitions).
  - *Listening*: Radial sound wave reacting to user microphone input.
  - *Processing (LLM)*: Rotating gradient halo (`spin` / `morph` keyframes).
  - *Speaking (TTS)*: Dynamic audio equalizer bars with smooth heights.
- **Barge-in Flash**: Micro-animation flash indicating instant audio cutoff when user interrupts.

### 4.3 Call Logs & Transcript Timeline
- **Split-Pane Layout**: Audio wave visualizer and playback scrubber on top; conversational transcript timeline below.
- **Transcript Bubbles**:
  - *User Message*: Left-aligned or right-aligned bubble with timestamp, audio snippet replay, and STT confidence score.
  - *Assistant Message*: Styled bubble with model tag (`gpt-4o`, `claude-3.5-sonnet`), latency breakdown tooltip (TTFA, STT, LLM, TTS), and tool execution badges.

### 4.4 Form Controls & Modals
- **Inputs & Dropdowns**: Sleek dark-mode inputs with `bg-layer`, `border-border-subtle`, and focus ring in `var(--brand-primary)`.
- **Modals & Drawers**: Slide-over sheets and glassmorphic overlays with blurred backdrops (`backdrop-blur-sm`).

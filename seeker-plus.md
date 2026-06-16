# PQCanvass support for Seeker+

## Overview
Built a multi-resolution frequency-domain spectrogram and added new measure stripcharts for a power quality monitoring web app (PQCanvass), to support a new device platform's (Seeker+) higher resolution data. The spectrogram combines four overlapping data layers: fundamental, harmonics, interharmonics (5 Hz bins), and supraharmonics (200 Hz bands). The work spans the UI, the PHP middle tier, a Python daemon, and a data-processing library.

## My Role
Full-stack ownership of the project. Coordinated with backend developers who were implementing parallel changes in the daemon and the library, but the UI, PHP endpoint, performance optimization across all 4 repos, and the test strategy were my work. 

**Owned end-to-end:**
- Designed and built the spectrogram component using Plotly.js
- Added new measure definitons and per-channel stripchart templates for around 10 new measures
- Built a new middle-tier endpoint for harmonics-style data, including raw response passthrough optimization that cut the per-request latency in half
- Diagnosed and fixed several bugs and regressions.
- Created a comprehensive testing checklist to hand off to other developers and QA testers.

**Coordinated with backend team**
- New path for retrieving harmonic stripchart data from the Python data daemon
- Python and C library work for binary file parsing
- Desing specs and data validation from manager

## Tech Stack
**Frontend:**
- Preact
- Redux + thunk middleware
- Plotly.js for heatmap rendering
- LESS for styling
- Webpack 3.x
- JavaScript ES2019

**Middle tier:**
- PHP (custom internall "Still" framework
- Socket I/O to Python daemon

**Backend (coordinated but didn't own):**
- Python data daemon (custom)
- Python wrapper around a C/Rust library for binary file parsing

**Dev/test tooling:**
- Browser DevTools
- Git history archaeology across 4 repos
- Custom in-app benchmark UI

## The Problem:
The new device platform records power quality data at a significantly higher frequency resolution than legacy devices: 70 individual harmonics, 600 interharmonic 5 Hz bins (5 Hz-3 kHz), and 740 supraharmonic 200 Hz bands (2 kHz-150 kHz). The existing UI handled harmonics well, but was blind to anything above around 3 kHz, and didn't have any way to view them as a heatmap. 
**The ask:** A single view, frequency on Y, time on X, color = magnitude, al four resolution tiers composable (inlcuding the fundamental). Plus all the new device's measures need to graph through the existing stripchart system. 

Three constraints made this harder than a generic visualization problem:

1. **Data volume.** A request for all 740 supraharmonic bands × N time samples per band exceeded the middle-tier daemon's 128 MB memory ceiling on full recording fetches.
2. **Browser limits.** HTTP/1.1 caps simultaneous connections at 6 per origin. Aggressive parallelization didn't actually show much improvement, if any.
3. **No backend yet.** When I started working on this, the daemon's harmonic endpoint didn't exist. UI scaffolding had to be built against synthetic data and integrated as the backend caught up.

## Solution

### Architectural overview

```mermaid
flowchart LR
    UI[Browser: Spectrogram Component] -->|HTTP| MID[PHP Middle Tier]
    MID -->|Socket: JSON| DMN[Python Data Daemon]
    DMN -->|Calls into| LIB[Data Library<br/>C/Rust + Python]
    LIB -->|Reads| FILE[(Binary Recording File)]

    FILE -.->|Bytes| LIB
    LIB -.->|Decimated JSON| DMN
    DMN -.->|Raw bytes| MID
    MID -.->|Pass-through| UI
```

Each interface had to do something *non-obvious* to make the whole pipeline performant.

### Frontend: composing four resolutions into one heatmap

Each of the four data layers (fundamental, harmonics, interharmonics, and supraharmonics) has different bin counts and time resolutions. Instead of trying to combine them into one dataset that could be graphed, I chose to rendereach as a separate Plotly heatmap trace with its own native bin/time resolution and stack them on top of eachother, ensuring the finer data was stacked on top wherever there were overlaps (mainly harmonics and interharmonics). This also had the added benefit of allowing the users to toggle individual layers on and off if they wanted to. 

```mermaid
flowchart TB
    subgraph render["Render Pipeline"]
        direction TB
        FUND[Fundamental row<br/>60 Hz only]
        HARM[Harmonics<br/>70 traces]
        IHARM[Interharmonics<br/>600 bins]
        SHARM[Supraharmonics<br/>740 bands]
    end
    FUND --> COLOR[Single shared color scale<br/>log10 of ratio to fundamental]
    HARM --> COLOR
    IHARM --> COLOR
    SHARM --> COLOR
    COLOR --> PLOT[Plotly heatmap layers, stacked]
```


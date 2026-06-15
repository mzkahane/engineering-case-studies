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

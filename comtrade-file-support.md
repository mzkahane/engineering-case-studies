# Adding COMTRADE file support to PQCanvass

## Overview
Added support for COMTRADE waveform files to our already existing power quality monitoring site.

## My Role
- Designed a frontend adapter to allow existing systems to handle the new file type
- Created a backend endpoint to retrieve additional data that comtrade files needed
- Coordinated with other engineers who were concurrently working on an extractor, file upload and storage, and surrounding frontend changes

## Tech Stack
- Preact/Redux
- PHP middle tier
- Python heavyd service
- Rust extraction service (indirectly)

## The Problem
COMTRADE is an IEEE standard file format with a fundamentally different data model than the existing PMI format. The Old code shoehorned it into the PMI structure, resulting in an incomplete and messy experience.

## Solution
- Rewrote code to build datasets directly from the COMTRADE waveform api instead of forcing data through a PMI template
- Modify the backend to proxy to heavyd to get COMTRADE channel info
- Updated FFT functions to be inclusive of COMTRADE recordings without breaking PMI recordings
- Added isComtrade flag to that gates PMI-specific behavior: skips redundant data fetching, disables features that don't apply, and fixes channel pair detection to work with both measure ID schemes
- Changed dataset grouping from label (serial number) to request.recording (recording ID), which freed up label for displaying COMTRADE channel identifiers in graph legends.
- Surfaced real server error messages in the import modal instead of generic "something went wrong" strings

## Challenges
Debugging across three layers made pinpointing issues and implementing safe fixes difficult. Adapting old coding choices that were not the best to something that is a little more appropriate. 

## Lessons Learned
- It is really important and helpful to understand existing architecture before changing it.
- Shoehorning new data into existing structures can be dangerous and fragile.
- Redux field whitelists can silently drop data
- Surfacing real error messages makes debugging a lot easier.

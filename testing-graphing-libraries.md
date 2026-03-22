# Evaluating different graphing libraries

## Overview
We needed a way to graph spectrograms on our app. We had utilities to create line and bar graphs, but nothing yet for heatmaps/spectrograms.

## My Role
- researching available graphing libraries that had heatmap capabilities
- evaluating each of the libraries for speed, ease of use, customizablilty, and looks
	- creating a small test site with some fake data to do so
- ultimately integrating the tool and feeding our data into it

## Tech Stack
### Graphing tools
- Plotly.js
- Chart.js
- Canvas / WebGL
### Test site
- TODO

## The Problem
We needed a tool that could be drawn and redrawn quickly, was easy to use for the user as well as the deveopers, and was customizable enough to fit in and around the rest of the elements on the page.

## The Solution
- created a little website where all three graphing tools could be loaded on seperate pages
- fed the same realistically sized test dataset to all three tools
- objectively evaluated each one by creating load and reload timers.
- subjectively evaluated their ease of use, since I would be the one using them
- Evaluated customizability according to specs provided by my manager

## Challenges
Each utility had its strengths and weaknesses. There wasn't a clear winner in my testing. It required me and my colleagues to determine which aspects were more important than others. I ultimately decided on Plotly.js because, even though it was slower than drawing my own using canvas/WebGL, it was had a lot more customization and built in features, and it was equally as easy to use as Chart.js.

## Lessons Learned
- Load times should be tested on devices of various specs/speeds. They can differ more than expected
- developer ease of use can be just as important as user ease of use
- Learned how to inspect load times of individual parts of the site using the flame chart in the dev tools' performance tab

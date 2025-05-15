# Guac Shock

# Guac Shock

Original article: https://www.cfr.org/article/guac-shock-understanding-tariffs-avocados

An [Observable Framework](https://observablehq.com/framework/) app that visualizes how tariffs affect avocado prices.

## Description

A scrollytelling data visualization showing how tariffs impact avocado prices from Mexico. This interactive experience uses D3.js to animate a stacked-bar breakdown of import cost, tariff, and profit as users scroll through the narrative, and is fully reusable across datasets.

## Features

- Scroll-driven interactions & animations
- Dynamic stacked-bar transitions
- Responsive, mobile-friendly layout
- Wrapped annotations
- Persistent chart display

## Technologies Used

- D3.js for data visualization
- Intersection Observer API for scroll tracking
- CSS Flexbox for responsive layout
- SVG for vector graphics rendering

## Quick Start

```bash
npm install
npm run dev
```

## Project Structure
- **src/**: Source files including pages (Markdown files)
- **src/data/**: Static data files
- **observablehq.config.js**: App configuration

## Commands

| Command          | Description                   |
| ---------------- | ----------------------------- |
| `npm install`    | Install dependencies          |
| `npm run dev`    | Start local preview server    |
| `npm run build`  | Build static site to `./dist` |
| `npm run deploy` | Deploy to Observable          |
| `npm run clean`  | Clear data loader cache       |
| `npm run deploy:gh-pages` | Deploy to GitHub Pages        |

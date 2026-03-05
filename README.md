# Calculator

A responsive (RWD) calculator built with **Vue 3** + **TailwindCSS v4** + **Vite**.

## Features

- Responsive design that works on mobile, tablet, and desktop
- Basic arithmetic operations: +, −, ×, ÷
- Percentage and sign toggle
- Expression history display
- Dark theme with orange accent (no blue/purple)
- CI/CD via GitHub Actions

## Tech Stack

- **Vue 3** (Composition API)
- **TailwindCSS v4**
- **Vite 7**

## Development

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
```

## CI/CD

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| CI       | Push / PR to any branch | Build verification |
| CD       | Push to `main` | Deploy to GitHub Pages |
| Release  | Push to `main` / manual | Bump version, create tag & GitHub Release |

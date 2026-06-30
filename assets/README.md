# Assets Directory

This directory is reserved for static assets used across the project.

## Current Contents

This project uses **zero external image assets**. All visual elements are implemented as:

- **Inline SVG icons** — Vector-based, resolution-independent, zero HTTP requests
- **CSS gradients** — For backgrounds, cards, and visual accents
- **CSS-generated UI elements** — Avatar initials, badges, status indicators

## Why No Image Assets?

1. **Performance**: Every image adds an HTTP request and increases page weight. By using SVGs and CSS, the page achieves a sub-25KB total weight.
2. **Responsiveness**: SVGs scale perfectly to any screen size without pixelation.
3. **Maintainability**: Colors and sizes can be changed via CSS variables without editing image files.

## Future Use

If OrthoNow provides brand assets (logo PNG/SVG, doctor photographs, clinic photos), place them here with the following naming convention:

```
assets/
├── logo/
│   ├── orthonow-logo-primary.svg
│   ├── orthonow-logo-white.svg
│   └── orthonow-logo-favicon.png
├── doctors/
│   ├── dr-rajesh-malhotra.webp
│   └── dr-[name].webp
├── clinics/
│   ├── clinic-koramangala.webp
│   └── clinic-[location].webp
└── icons/
    └── custom-icons.svg (sprite)
```

**Image format recommendations:**
- Photos: WebP (with JPEG fallback for older browsers)
- Logos: SVG (vector, scalable)
- Icons: Inline SVG (no sprite sheet needed for < 20 icons)
- Maximum dimensions: 1200px wide for photos, compressed to < 100KB each

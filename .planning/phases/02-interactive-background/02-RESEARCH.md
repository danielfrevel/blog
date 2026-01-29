# Phase 2: Interactive Background - Research

**Researched:** 2026-01-29
**Domain:** HTML5 Canvas particle systems with Perlin/Simplex noise flow fields
**Confidence:** HIGH

## Summary

Canvas-based particle systems with flow fields driven by Perlin or Simplex noise are a well-established pattern in creative coding. The standard approach uses vanilla JavaScript with requestAnimationFrame, a noise library (noisejs being the most common), and careful performance optimizations for mobile devices.

For this phase, the implementation will be plain vanilla JS without frameworks. The noise library noisejs provides both Perlin and Simplex noise functions at 10M queries/sec. Key performance concerns include particle count limits (200-300 on desktop, 50-80 on mobile), proper canvas scaling for high-DPI displays using devicePixelRatio, and implementing Page Visibility API to pause animations when tabs are hidden.

The accessibility requirements (prefers-reduced-motion) are straightforward: use CSS media queries to detect the preference and either render a static gradient or disable the animation entirely. The single configuration object approach is standard practice for creative coding projects.

**Primary recommendation:** Use vanilla JS with noisejs for Simplex noise (fewer artifacts than Perlin), implement object pooling for particles to reduce garbage collection, and use requestAnimationFrame with delta time for frame-independent animation. Target 60fps desktop / 30fps mobile with adaptive quality settings.

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| noisejs | Latest (GitHub) | 2D/3D Perlin & Simplex noise | Most popular JS noise library, 10M queries/sec, MIT licensed, based on Stefan Gustavson's implementation |
| Vanilla JS | ES6+ | Animation loop, particle system | No framework needed for canvas work, maximum performance |
| requestAnimationFrame | Browser API | Animation timing | Standard for all canvas animations, auto-pauses in background tabs |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Page Visibility API | Browser API | Pause animations when tab hidden | Battery optimization (required per BG-05) |
| matchMedia | Browser API | Detect prefers-reduced-motion | Accessibility requirement (BG-03) |
| devicePixelRatio | Browser API | High-DPI display support | Crisp rendering on Retina/4K displays |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| noisejs | Custom Perlin implementation | Reinventing the wheel, slower, more bugs |
| noisejs | WebGL shader noise | 10-100x faster but requires WebGL setup, overkill for this use case |
| Vanilla JS | p5.js library | Adds 900KB for features we don't need, slower |
| Canvas 2D | WebGL/Three.js | Much faster but complex setup, unnecessary for 50-300 particles |

**Installation:**
```bash
# noisejs is not on npm, use direct include
# Download from: https://github.com/josephg/noisejs
# Or use CDN: https://cdn.jsdelivr.net/gh/josephg/noisejs/perlin.js
```

## Architecture Patterns

### Recommended Project Structure
```
static/js/
├── flow-field-background.js    # Main animation file with config object
└── perlin.js                   # noisejs library (vendored)
```

### Pattern 1: Single Configuration Object
**What:** All tweakable values at top of file in one object
**When to use:** Always for creative coding projects, makes values obvious to adjust
**Example:**
```javascript
// Configuration - adjust these values to customize the animation
const CONFIG = {
  // Particle settings
  particleCount: 200,        // Number of particles (reduce for mobile)
  particleSize: 2,           // Radius in pixels
  particleColor: '#3b82f6',  // CSS color
  particleAlpha: 0.5,        // Opacity 0-1

  // Flow field settings
  noiseScale: 0.003,         // Zoom level of noise (smaller = larger patterns)
  noiseStrength: 2,          // Force multiplier for flow vectors
  timeIncrement: 0.001,      // Speed of flow evolution

  // Mouse interaction
  mouseRadius: 150,          // Pixels of influence around cursor
  mouseStrength: 5,          // Force multiplier for mouse repulsion

  // Performance
  maxSpeed: 2,               // Particle velocity cap
  trailLength: 0.05,         // Canvas fade amount (lower = longer trails)
  targetFPS: 60              // Desktop target (mobile will auto-throttle)
};
```

### Pattern 2: Object Pool for Particles
**What:** Pre-allocate particle objects, reuse instead of creating/destroying
**When to use:** When animating 50+ objects per frame
**Example:**
```javascript
class ParticlePool {
  constructor(size) {
    this.pool = [];
    this.active = [];

    // Pre-allocate particles
    for (let i = 0; i < size; i++) {
      this.pool.push({
        x: 0, y: 0,
        vx: 0, vy: 0,
        active: false
      });
    }
  }

  get() {
    const particle = this.pool.pop() || this.createParticle();
    particle.active = true;
    this.active.push(particle);
    return particle;
  }

  release(particle) {
    particle.active = false;
    this.pool.push(particle);
    this.active.splice(this.active.indexOf(particle), 1);
  }
}
```

### Pattern 3: Delta Time for Frame Independence
**What:** Scale movement by elapsed time, not frame count
**When to use:** Always with requestAnimationFrame for smooth cross-device animation
**Example:**
```javascript
// Source: https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame
let lastTime = 0;

function animate(timestamp) {
  // Calculate delta in seconds
  const deltaTime = (timestamp - lastTime) / 1000;
  lastTime = timestamp;

  // Update particles using delta
  particles.forEach(p => {
    p.x += p.vx * deltaTime * 60; // Normalize to 60fps baseline
    p.y += p.vy * deltaTime * 60;
  });

  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

### Pattern 4: High-DPI Canvas Scaling
**What:** Scale canvas for devicePixelRatio to avoid blur on Retina displays
**When to use:** Always for production canvas work
**Example:**
```javascript
// Source: https://www.kirupa.com/canvas/canvas_high_dpi_retina.htm
function setupCanvas(canvas) {
  const rect = canvas.getBoundingClientRect();
  const dpr = window.devicePixelRatio || 1;

  // Scale canvas backing store
  canvas.width = rect.width * dpr;
  canvas.height = rect.height * dpr;

  // Scale CSS size back down
  canvas.style.width = rect.width + 'px';
  canvas.style.height = rect.height + 'px';

  // Scale drawing context
  const ctx = canvas.getContext('2d');
  ctx.scale(dpr, dpr);

  return ctx;
}
```

### Pattern 5: Page Visibility API for Battery Optimization
**What:** Pause animation when tab is not visible
**When to use:** Always for canvas animations (requirement BG-05)
**Example:**
```javascript
// Source: https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API
let animationId;
let isAnimating = false;

function startAnimation() {
  if (!isAnimating) {
    isAnimating = true;
    animationId = requestAnimationFrame(animate);
  }
}

function stopAnimation() {
  if (isAnimating) {
    isAnimating = false;
    cancelAnimationFrame(animationId);
  }
}

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    stopAnimation();
  } else {
    startAnimation();
  }
});
```

### Pattern 6: Prefers-Reduced-Motion Detection
**What:** Detect user's motion preference and provide static fallback
**When to use:** Always for accessibility (requirement BG-03)
**Example:**
```javascript
// Source: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');

if (prefersReducedMotion.matches) {
  // Show static gradient fallback
  canvas.style.display = 'none';
  document.body.classList.add('static-background');
} else {
  // Run animation
  startAnimation();
}

// Listen for preference changes
prefersReducedMotion.addEventListener('change', (e) => {
  if (e.matches) {
    stopAnimation();
    canvas.style.display = 'none';
    document.body.classList.add('static-background');
  } else {
    canvas.style.display = 'block';
    document.body.classList.remove('static-background');
    startAnimation();
  }
});
```

```css
/* CSS fallback gradient */
body.static-background::before {
  content: '';
  position: fixed;
  inset: 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  opacity: 0.1;
  z-index: -1;
}
```

### Anti-Patterns to Avoid
- **Using setInterval/setTimeout instead of requestAnimationFrame:** These don't sync with browser repaints and waste battery in background tabs
- **Clearing entire canvas with fillRect instead of clearRect:** fillRect is slower and can cause compositing issues
- **Using non-default globalCompositeOperation:** This is extremely expensive and will tank performance with many particles
- **Ignoring devicePixelRatio:** Canvas will look blurry on Retina/4K displays
- **Creating new particle objects every frame:** Causes garbage collection spikes, use object pooling
- **DOM manipulation inside animation loop:** Keep DOM queries outside the loop, cache references

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Noise generation | Custom Perlin/Simplex | noisejs library | Extremely hard to get right, performance-critical, well-tested implementation exists |
| Animation timing | Custom delta time with Date.now() | requestAnimationFrame timestamp parameter | Browser-provided, synchronized across frames, handles visibility automatically |
| High-DPI detection | Manual pixel ratio math | devicePixelRatio API | Handles edge cases, browser updates automatically |
| Motion preference | Custom user settings | prefers-reduced-motion media query | System-level preference, WCAG standard, works across sites |
| Tab visibility | Focus/blur events | Page Visibility API | More reliable, handles all visibility cases (minimize, tab switch, etc) |

**Key insight:** Canvas performance optimization is full of subtle edge cases. The browser APIs exist because they handle these correctly. Custom implementations almost always have bugs or poor performance.

## Common Pitfalls

### Pitfall 1: Particle Count Too High for Mobile
**What goes wrong:** Animation runs at 60fps on desktop, drops to 5fps on mobile, drains battery
**Why it happens:** Mobile CPUs are 5-10x slower than desktop, mobile browsers have more overhead
**How to avoid:**
- Detect mobile and reduce particle count (use `matchMedia('(pointer: coarse)')` or check screen width)
- Target 30fps minimum on mobile, 60fps on desktop
- Desktop: 200-300 particles max
- Mobile: 50-80 particles max
**Warning signs:** Users report "laggy" site, battery drain complaints, mobile lighthouse score drops

### Pitfall 2: Not Using Delta Time
**What goes wrong:** Animation speed varies wildly across devices with different refresh rates (60Hz vs 120Hz vs variable)
**Why it happens:** Assuming 60fps means each frame moves particles by fixed amount, but 120Hz displays call your function twice as often
**How to avoid:** Always multiply movement by delta time from requestAnimationFrame timestamp
**Warning signs:** Animation runs at different speeds on different monitors, user reports "too fast" or "too slow"

### Pitfall 3: Ignoring Garbage Collection
**What goes wrong:** Animation stutters every few seconds with visible frame drops
**Why it happens:** Creating/destroying objects in animation loop triggers GC pauses
**How to avoid:**
- Use object pooling for particles
- Avoid array.push/splice in hot paths
- Pre-allocate arrays at maximum size
- Reuse objects instead of creating new ones
**Warning signs:** Periodic stuttering (not continuous slow), Chrome DevTools shows GC spikes in performance timeline

### Pitfall 4: Mouse Event Flood
**What goes wrong:** CPU spikes when moving mouse over canvas, animation drops frames
**Why it happens:** mousemove fires 100+ times per second, each triggering expensive calculations
**How to avoid:**
- Store mouse position in variable, read it in animation loop (don't process in event)
- OR throttle mousemove handler to ~16ms (60fps) maximum
- Avoid heavy calculations in mouse event handlers
**Warning signs:** Animation smooth until mouse moves, then drops frames

### Pitfall 5: Canvas Blur on Retina Displays
**What goes wrong:** Canvas looks crisp on 1080p monitors, blurry on MacBook/4K displays
**Why it happens:** Canvas uses CSS pixels by default, doesn't scale for devicePixelRatio
**How to avoid:** Use High-DPI pattern (scale canvas.width/height by devicePixelRatio, scale context, set CSS size)
**Warning signs:** Users report "blurry graphics", canvas looks worse than CSS elements

### Pitfall 6: Simplex vs Perlin Choice
**What goes wrong:** Visible grid artifacts in flow field, or patent concerns
**Why it happens:** Perlin noise has directional artifacts, Simplex has patent issues (use OpenSimplex)
**How to avoid:**
- For 2D flow fields, Simplex has fewer artifacts
- noisejs provides both, use simplex2() for this use case
- Perlin is fine if you notice no artifacts in your specific case
**Warning signs:** Visible horizontal/vertical alignment in flow patterns

### Pitfall 7: Not Testing prefers-reduced-motion
**What goes wrong:** Users with motion sensitivity get headaches/nausea, accessibility audit fails
**Why it happens:** Forgetting to test the setting or implementing it incorrectly
**How to avoid:**
- Test with system preference enabled (macOS: System Preferences > Accessibility > Display > Reduce motion)
- Ensure static fallback is visible and looks good
- Don't just disable animation, provide alternative visual
**Warning signs:** Accessibility complaints, WCAG 2.1 audit failures

## Code Examples

Verified patterns from official sources:

### Flow Field Calculation with Simplex Noise
```javascript
// Based on: https://github.com/josephg/noisejs
const noise = new Noise(Math.random());

function getFlowAngle(x, y, time) {
  // Sample 3D noise (x, y, time) for evolving patterns
  const noiseValue = noise.simplex3(
    x * CONFIG.noiseScale,
    y * CONFIG.noiseScale,
    time
  );

  // Map noise (-1 to 1) to angle (0 to 2π)
  return noiseValue * Math.PI * 2;
}

function updateParticle(particle, time) {
  // Get flow field angle at particle position
  const angle = getFlowAngle(particle.x, particle.y, time);

  // Apply force in flow direction
  particle.vx += Math.cos(angle) * CONFIG.noiseStrength;
  particle.vy += Math.sin(angle) * CONFIG.noiseStrength;

  // Limit velocity
  const speed = Math.sqrt(particle.vx ** 2 + particle.vy ** 2);
  if (speed > CONFIG.maxSpeed) {
    particle.vx = (particle.vx / speed) * CONFIG.maxSpeed;
    particle.vy = (particle.vy / speed) * CONFIG.maxSpeed;
  }

  // Update position
  particle.x += particle.vx;
  particle.y += particle.vy;

  // Wrap around screen edges
  if (particle.x < 0) particle.x = canvas.width;
  if (particle.x > canvas.width) particle.x = 0;
  if (particle.y < 0) particle.y = canvas.height;
  if (particle.y > canvas.height) particle.y = 0;
}
```

### Mouse Interaction with Distance Check
```javascript
let mouseX = -1000;
let mouseY = -1000;

// Store mouse position (don't process here)
canvas.addEventListener('mousemove', (e) => {
  const rect = canvas.getBoundingClientRect();
  mouseX = e.clientX - rect.left;
  mouseY = e.clientY - rect.top;
});

canvas.addEventListener('mouseleave', () => {
  mouseX = -1000;
  mouseY = -1000;
});

// Apply mouse force in animation loop
function applyMouseForce(particle) {
  const dx = particle.x - mouseX;
  const dy = particle.y - mouseY;
  const distSq = dx * dx + dy * dy;
  const radiusSq = CONFIG.mouseRadius * CONFIG.mouseRadius;

  if (distSq < radiusSq && distSq > 0) {
    // Repel from mouse
    const dist = Math.sqrt(distSq);
    const force = (1 - dist / CONFIG.mouseRadius) * CONFIG.mouseStrength;
    particle.vx += (dx / dist) * force;
    particle.vy += (dy / dist) * force;
  }
}
```

### Canvas Fade Trail Effect
```javascript
// Instead of clearRect (no trail) or not clearing (full trail)
// Use semi-transparent fillRect for fade effect
function clearWithFade() {
  ctx.fillStyle = `rgba(255, 255, 255, ${CONFIG.trailLength})`;
  ctx.fillRect(0, 0, canvas.width, canvas.height);
}

// In animation loop
function animate(timestamp) {
  clearWithFade();

  // Draw particles
  ctx.fillStyle = CONFIG.particleColor;
  ctx.globalAlpha = CONFIG.particleAlpha;

  particles.forEach(p => {
    ctx.beginPath();
    ctx.arc(p.x, p.y, CONFIG.particleSize, 0, Math.PI * 2);
    ctx.fill();
  });

  ctx.globalAlpha = 1.0;

  requestAnimationFrame(animate);
}
```

### Mobile Detection and Adaptive Quality
```javascript
// Detect mobile (coarse pointer = touch)
const isMobile = window.matchMedia('(pointer: coarse)').matches;

// Adjust config for device
const CONFIG = {
  particleCount: isMobile ? 60 : 200,
  targetFPS: isMobile ? 30 : 60,
  particleSize: isMobile ? 3 : 2, // Larger particles easier to see
  // ... other settings
};

// Optional: FPS-based throttling
let lastFrameTime = 0;
const frameInterval = 1000 / CONFIG.targetFPS;

function animate(timestamp) {
  const elapsed = timestamp - lastFrameTime;

  if (elapsed > frameInterval) {
    lastFrameTime = timestamp - (elapsed % frameInterval);

    // Render frame
    updateAndDraw();
  }

  requestAnimationFrame(animate);
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| setInterval for animation | requestAnimationFrame | ~2015 | 60fps sync with display, auto-pause in background |
| Perlin noise only | Simplex noise preferred | ~2018 | Fewer directional artifacts, better visual quality |
| Ignore devicePixelRatio | Scale for high-DPI | ~2016 | Crisp rendering on Retina/4K displays |
| Always animate | Page Visibility API | ~2017 | Battery optimization on mobile |
| Ignore motion preferences | prefers-reduced-motion | ~2019 (WCAG 2.1) | Accessibility requirement, legal compliance |
| Manual noise implementation | Use noisejs library | Established | Faster, fewer bugs, maintained |
| Fixed frame rate | Delta time | ~2015 | Works across 60/120/144Hz displays |

**Deprecated/outdated:**
- **p5.js for simple canvas work:** Adds 900KB for features not needed, slower than vanilla
- **jQuery for canvas:** No benefits, just overhead
- **Classic Perlin (perlin2/perlin3):** Use Simplex (simplex2/simplex3) for fewer artifacts in 2D
- **Date.now() for delta time:** Use requestAnimationFrame timestamp parameter instead

## Open Questions

Things that couldn't be fully resolved:

1. **Exact particle count for "noticeable but calm"**
   - What we know: User wants clearly visible particles, not subtle/hidden
   - What's unclear: Exact density to achieve this on typical 1920x1080 display
   - Recommendation: Start with 150 particles desktop / 60 mobile, make easily adjustable in CONFIG

2. **Static gradient specific colors**
   - What we know: Should maintain visual interest without motion
   - What's unclear: Which colors match the "minimal/clean aesthetic" from Phase 1
   - Recommendation: Use very subtle gradient (low opacity), let Claude choose based on Phase 1 color scheme

3. **Mouse interaction: attract or repel?**
   - What we know: Particles should "react to mouse movement"
   - What's unclear: Should they move toward or away from cursor
   - Recommendation: Repel is more common and less "busy" (particles don't cluster), fits "calm" requirement

4. **Mobile threshold detection**
   - What we know: Need to reduce particle count on mobile
   - What's unclear: Best detection method (screen width, pointer type, user agent?)
   - Recommendation: Use `matchMedia('(pointer: coarse)')` for touch devices, most reliable in 2026

## Sources

### Primary (HIGH confidence)
- https://github.com/josephg/noisejs - noisejs library documentation and API
- https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API - Page Visibility API official spec
- https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame - requestAnimationFrame API reference
- https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion - prefers-reduced-motion spec
- https://www.kirupa.com/canvas/canvas_high_dpi_retina.htm - High-DPI canvas implementation guide
- https://web.dev/articles/speed-static-mem-pools - Object pooling for JavaScript (web.dev)

### Secondary (MEDIUM confidence)
- https://www.bit-101.com/2017/2021/07/perlin-vs-simplex/ - Perlin vs Simplex comparison (verified expert source)
- https://gameprogrammingpatterns.com/object-pool.html - Object pool pattern (authoritative book)
- https://gist.github.com/jaredwilli/5469626 - HTML5 Canvas performance tips (widely-referenced gist)
- https://medium.com/@bit101/flow-fields-part-ii-f3c24c1b777d - Flow field implementation guide
- Multiple WebSearch results from DEV Community and technical blogs (2024-2026) on canvas performance

### Tertiary (LOW confidence)
- Various CodePen examples (https://codepen.io/DonKarlssonSan/post/particles-in-simplex-noise-flow-field) - Implementation ideas only, not verified
- Community discussions on particle counts and performance - Use as rough guidelines, verify in testing

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - noisejs is well-established (1.7k stars), browser APIs are standard
- Architecture: HIGH - Patterns verified from MDN and web.dev, widely used in production
- Pitfalls: MEDIUM-HIGH - Based on multiple sources and common knowledge, but some are experiential

**Research date:** 2026-01-29
**Valid until:** 2026-04-29 (90 days - canvas APIs and libraries stable, slow-moving domain)

**Notes:**
- No CONTEXT.md constraints violated - all decisions respected (single config object, noticeable style, static gradient fallback)
- Claude has discretion on exact particle density/colors/speeds - research provides ranges and examples
- All requirements (BG-01 through BG-05) have clear implementation paths
- Phase 1 dependency satisfied (background renders behind content via z-index)

# Cosmic Phenomena Creation

Create immersive cosmic visual and audio experiences for web applications using pure CSS, JavaScript/TypeScript, Web Audio API, and WebGL.

## Core Philosophy

- **Layered composition**: Build depth through multiple independent layers (stars, nebulae, effects)
- **Organic variation**: Use seeded randomness, personality systems, and slight imperfections
- **Performance-first**: Adaptive complexity based on device capabilities
- **Accessibility-aware**: Respect `prefers-reduced-motion` and provide fallbacks

---

## Visual Cosmic Phenomena

### Multi-Layer Parallax Starfield

Create depth through speed differential:

```typescript
interface StarLayer {
  count: number;
  opacityRange: [number, number];
  speed: number;  // Parallax multiplier
}

const LAYERS: StarLayer[] = [
  { count: 50, opacityRange: [0.15, 0.3], speed: 0.01 },  // Distant, dim, slow
  { count: 80, opacityRange: [0.2, 0.5], speed: 0.03 },   // Mid-distance
  { count: 100, opacityRange: [0.4, 0.8], speed: 0.06 }, // Close, bright, fast
];

// Mobile: reduce star count by 50%
const starMultiplier = isMobile ? 0.5 : 1;
```

### Interactive Star Brightness

Use spatial locality optimization before expensive calculations:

```typescript
function updateStarBrightness(mouseX: number, mouseY: number): void {
  stars.forEach(star => {
    const dx = Math.abs(star.x - mouseX);
    const dy = Math.abs(star.y - mouseY);

    // Quick bounding box rejection (O(1)) before sqrt
    if (dx > 45 || dy > 45) return;

    // Full distance only for nearby stars
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < 40) {
      star.opacity = Math.min(1, star.baseOpacity + 0.25);
    }
  });
}
```

### Nebula Background Effects

Use radial gradients with soft falloff:

```css
.nebula {
  position: fixed;
  filter: blur(60px);
  mix-blend-mode: screen;  /* Additive blending for glow */
}

.nebula--purple {
  background: radial-gradient(
    ellipse at center,
    rgba(90, 50, 120, 0.4) 0%,
    rgba(70, 40, 100, 0.25) 30%,
    rgba(50, 30, 80, 0.1) 60%,
    transparent 80%
  );
}
```

### Shooting Stars with Physics

```typescript
function createShootingStar(): void {
  const angle = 25 + Math.random() * 20;  // 25-45 degrees
  const radians = angle * Math.PI / 180;
  const length = 80 + Math.random() * 120;
  const duration = 800 + Math.random() * 600;

  function animate(timestamp: number): void {
    const progress = (timestamp - start) / duration;

    // Position
    head.style.left = `calc(${startX}% + ${dx * progress}px)`;
    head.style.top = `calc(${startY}% + ${dy * progress}px)`;

    // Trail diminishes with distance
    trail.style.width = Math.max(0, length * (1 - progress * 0.5)) + 'px';
    trail.style.opacity = String(progress < 0.1 ? progress * 10 : 1);

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      head.remove();
      trail.remove();
    }
  }
}
```

### Glow Effects

Layer text-shadows for stellar glow:

```css
.star.bright {
  text-shadow:
    0 0 8px currentColor,
    0 0 15px currentColor;
}

.supernova-core {
  text-shadow:
    0 0 10px var(--supernova-core-hot),
    0 0 20px var(--supernova-shockwave),
    0 0 40px var(--supernova-outer);
}
```

---

## Audio Synthesis with Web Audio API

### Singleton AudioContext Pattern

```typescript
let audioCtx: AudioContext | null = null;
let masterGain: GainNode | null = null;

export function getAudioContext(): AudioContext | null {
  return audioCtx;
}

export function initAudio(): AudioContext {
  if (!audioCtx) {
    audioCtx = new AudioContext();
    masterGain = audioCtx.createGain();
    masterGain.connect(audioCtx.destination);
    masterGain.gain.value = 0.7;
  }
  return audioCtx;
}
```

### Five-Layer Soundscape Architecture

| Layer | Purpose | Frequency Range | Gain |
|-------|---------|-----------------|------|
| Drone | Deep void presence | 32-96 Hz | 0.12 |
| Pad | Harmonic bed (E minor) | 82-247 Hz | 0.08 |
| Noise | Cosmic static | Filtered to 200-500 Hz | 0.006 |
| Signals | Distant transmissions | 600-3000 Hz | 0.025 |
| Eyes | Interactive tones | Pentatonic 220-784 Hz | 0.012 |

### Deep Space Drone

```typescript
function createDrone(audioCtx: AudioContext): OscillatorNode[] {
  const baseFreqs = [32, 48, 64, 96];
  const dateSeed = getDateSeed();  // Daily variation

  return baseFreqs.map((freq, i) => {
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();

    // Subtle daily variation (±10%)
    const variation = 1 + (seededRandom(dateSeed, i) - 0.5) * 0.1;
    osc.frequency.value = freq * variation;
    osc.type = 'sine';

    gain.gain.value = 0.03;  // Per-oscillator gain
    osc.connect(gain);

    // Slow drift (6-8 seconds)
    function drift(): void {
      const target = freq * (0.98 + Math.random() * 0.04);
      osc.frequency.exponentialRampToValueAtTime(
        target,
        audioCtx.currentTime + 7
      );
      setTimeout(drift, 6000 + Math.random() * 2000);
    }
    drift();

    return osc;
  });
}
```

### Harmonic Pad with Beating

```typescript
const E_MINOR_FREQS = [82.41, 123.47, 164.81, 195.99, 246.94];

function createPadVoice(freq: number): void {
  const osc1 = audioCtx.createOscillator();
  const osc2 = audioCtx.createOscillator();
  const gain = audioCtx.createGain();

  osc1.type = 'sine';
  osc1.frequency.value = freq;

  osc2.type = 'triangle';
  osc2.frequency.value = freq * 1.003;  // 0.3% detune for beating

  // Volume swell envelope
  const now = audioCtx.currentTime;
  const duration = 10 + Math.random() * 10;

  gain.gain.setValueAtTime(0.001, now);
  gain.gain.exponentialRampToValueAtTime(0.02, now + duration * 0.4);
  gain.gain.exponentialRampToValueAtTime(0.001, now + duration);
}
```

### Filtered Cosmic Noise

```typescript
function createCosmicNoise(): void {
  const bufferSize = audioCtx.sampleRate * 4;  // 4-second loop
  const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
  const data = buffer.getChannelData(0);

  for (let i = 0; i < bufferSize; i++) {
    data[i] = Math.random() * 2 - 1;
  }

  const source = audioCtx.createBufferSource();
  source.buffer = buffer;
  source.loop = true;

  // Lowpass for warm rumble
  const filter = audioCtx.createBiquadFilter();
  filter.type = 'lowpass';
  filter.frequency.value = 400;

  // Slow filter sweeps
  function sweep(): void {
    const target = 200 + Math.random() * 300;
    filter.frequency.exponentialRampToValueAtTime(
      target,
      audioCtx.currentTime + 4
    );
    setTimeout(sweep, 15000);
  }
  sweep();
}
```

### Spatial Audio Positioning

```typescript
function playPositionedSound(x: number, freq: number): void {
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  const panner = audioCtx.createStereoPanner();

  // x: 0-1 maps to -0.9 to 0.9 stereo field
  panner.pan.value = (x - 0.5) * 1.8;

  osc.connect(gain);
  gain.connect(panner);
  panner.connect(masterGain);
}
```

### Seeded Daily Variation

```typescript
function getDateSeed(): number {
  const today = new Date();
  const dateStr = `${today.getFullYear()}-${today.getMonth()}-${today.getDate()}`;
  let hash = 0;
  for (let i = 0; i < dateStr.length; i++) {
    hash = ((hash << 5) - hash) + dateStr.charCodeAt(i);
  }
  return Math.abs(hash) / 2147483647;
}

function seededRandom(seed: number, index: number): number {
  const x = Math.sin(seed * 9999 + index * 1234) * 10000;
  return x - Math.floor(x);
}
```

---

## Optimization Strategies

### Throttled Event Handlers

```typescript
const POINTER_THROTTLE = isMobile ? 33 : 16;  // 30fps mobile, 60fps desktop
let lastPointerUpdate = 0;

function handlePointerMove(e: PointerEvent): void {
  const now = performance.now();
  if (now - lastPointerUpdate < POINTER_THROTTLE) return;
  lastPointerUpdate = now;

  // Actual update logic
}
```

### Cached DOM Values

```typescript
let cachedContainerRect: DOMRect | null = null;

function updateCachedRect(): void {
  cachedContainerRect = container.getBoundingClientRect();
}

// Update on resize, use cached value otherwise
window.addEventListener('resize', updateCachedRect);
```

### Visibility-Based Animation Control

```typescript
function observeVisibility(
  element: HTMLElement,
  callback: (visible: boolean) => void
): IntersectionObserver {
  const observer = new IntersectionObserver(
    entries => entries.forEach(entry => callback(entry.isIntersecting)),
    { threshold: 0.1 }
  );
  observer.observe(element);
  return observer;
}

// Pause offscreen animations
observeVisibility(container, visible => {
  if (visible) startAnimation();
  else stopAnimation();
});
```

### Mobile Adaptive Complexity

```typescript
const isMobile = window.matchMedia('(max-width: 768px)').matches;

const config = {
  starCount: isMobile ? 115 : 230,
  nebulaCount: isMobile ? 3 : 5,
  eyeCount: isMobile ? 4 : 8,
  particleCount: isMobile ? 50 : 150,
  frameThrottle: isMobile ? 33 : 16,
};
```

### Cleanup Pattern

```typescript
let animationFrame: number | null = null;
let intervalIds: number[] = [];
let eventHandlers: Map<string, EventListener> = new Map();

function cleanup(): void {
  if (animationFrame) {
    cancelAnimationFrame(animationFrame);
    animationFrame = null;
  }

  intervalIds.forEach(id => clearInterval(id));
  intervalIds = [];

  eventHandlers.forEach((handler, event) => {
    document.removeEventListener(event, handler);
  });
  eventHandlers.clear();
}
```

### Accessibility: Reduced Motion

```typescript
function prefersReducedMotion(): boolean {
  return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
}

// Check before intensive animations
if (prefersReducedMotion()) {
  return;  // Skip animation entirely
}
```

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Gravity & Physics Simulation

### Spring Physics for Smooth Movement

```typescript
const SPRING_STRENGTH = 0.08;
const DAMPING = 0.85;
const DEAD_ZONE = 3;

let velocityX = 0, velocityY = 0;
let currentX = 0, currentY = 0;

function updateSpringPhysics(targetX: number, targetY: number): void {
  // Apply dead zone
  const effectiveTargetX = Math.abs(targetX) < DEAD_ZONE ? 0 : targetX;
  const effectiveTargetY = Math.abs(targetY) < DEAD_ZONE ? 0 : targetY;

  // Spring force
  velocityX += (effectiveTargetX - currentX) * SPRING_STRENGTH;
  velocityY += (effectiveTargetY - currentY) * SPRING_STRENGTH;

  // Damping
  velocityX *= DAMPING;
  velocityY *= DAMPING;

  // Update position
  currentX += velocityX;
  currentY += velocityY;
}
```

### Orbital Motion

```typescript
function updateOrbit(body: CelestialBody, dt: number): void {
  const { centerX, centerY, radius, speed } = body.orbit;

  body.angle += speed * dt;
  body.x = centerX + Math.cos(body.angle) * radius;
  body.y = centerY + Math.sin(body.angle) * radius;
}
```

### Gravitational Attraction

```typescript
function calculateGravity(
  body1: CelestialBody,
  body2: CelestialBody,
  G: number = 0.1
): { fx: number; fy: number } {
  const dx = body2.x - body1.x;
  const dy = body2.y - body1.y;
  const distSq = dx * dx + dy * dy;
  const dist = Math.sqrt(distSq);

  // Avoid division by zero and extreme forces
  const minDist = 50;
  const safeDist = Math.max(dist, minDist);

  const force = (G * body1.mass * body2.mass) / (safeDist * safeDist);

  return {
    fx: (force * dx) / dist,
    fy: (force * dy) / dist,
  };
}
```

---

## Color Palette Reference

```css
:root {
  /* Deep space */
  --void: #0a0a0f;
  --deep-space: #111118;

  /* Starlight spectrum */
  --starlight: #f0eeeb;
  --blue-giant: #b8d4ff;
  --red-dwarf: #ffb8a8;
  --white-dwarf: #e8e6e3;

  /* Cosmic events */
  --supernova-core-hot: #fff8e0;
  --supernova-shockwave: #ff8c00;
  --supernova-outer: #ff4500;

  /* Black hole */
  --blackhole-void: #000000;
  --blackhole-photon-ring: rgba(255, 200, 150, 0.9);
  --accretion-disk-hot: #ffaa00;
  --accretion-disk-cool: #ff4400;

  /* Nebula tints */
  --nebula-purple: rgba(90, 50, 120, 0.4);
  --nebula-blue: rgba(40, 80, 120, 0.3);
  --nebula-teal: rgba(30, 100, 100, 0.25);
}
```

---

## Frequency Reference for Cosmic Audio

| Element | Base Frequency | Character |
|---------|---------------|-----------|
| Void hum | 18-40 Hz | Near-infrasound, felt more than heard |
| Stellar core | 81.38 Hz (E2) | Root tone anchor |
| Nuclear fusion | 122 Hz | Warmth, power |
| Accretion disk | 55 Hz (A1) | Orbiting matter |
| Distant signals | 600-3000 Hz | Blips, transmissions |
| Pentatonic tones | 220-784 Hz | Interactive feedback |

---

## Usage

When creating cosmic phenomena:

1. **Layer visually**: Background (nebulae) → midground (stars) → foreground (effects)
2. **Layer aurally**: Drone base → harmonic pad → noise texture → interactive tones
3. **Optimize aggressively**: Mobile gets 50% particle counts, longer throttles
4. **Add organic variation**: Seeded randomness, slight imperfections, personality systems
5. **Respect accessibility**: Check `prefers-reduced-motion`, provide alternatives
6. **Clean up resources**: Store references, cancel animations, remove listeners

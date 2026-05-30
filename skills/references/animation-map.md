# Animation Map: Elementor → Modern Stack

## Animaciones de Entrada (Entrance Animations)

Elementor usa la propiedad `animation` en `settings`. Mapeo completo:

### Fade
| Elementor | @keyframes CSS / Clase Tailwind | Framer Motion |
|---|---|---|
| `fadeIn` | `animate-fadeIn` / `animate-fade-in` | `initial:{opacity:0} animate:{opacity:1}` |
| `fadeInDown` | `animate-fadeInDown` | `initial:{opacity:0,y:-20} animate:{opacity:1,y:0}` |
| `fadeInUp` | `animate-fadeInUp` | `initial:{opacity:0,y:20} animate:{opacity:1,y:0}` |
| `fadeInLeft` | `animate-fadeInLeft` | `initial:{opacity:0,x:-20} animate:{opacity:1,x:0}` |
| `fadeInRight` | `animate-fadeInRight` | `initial:{opacity:0,x:20} animate:{opacity:1,x:0}` |

### Slide
| Elementor | @keyframes CSS | Framer Motion |
|---|---|---|
| `slideInDown` | translateY(-100%) → 0 | `initial:{y:"-100%"} animate:{y:0}` |
| `slideInUp` | translateY(100%) → 0 | `initial:{y:"100%"} animate:{y:0}` |
| `slideInLeft` | translateX(-100%) → 0 | `initial:{x:"-100%"} animate:{x:0}` |
| `slideInRight` | translateX(100%) → 0 | `initial:{x:"100%"} animate:{x:0}` |

### Zoom
| Elementor | @keyframes CSS | Framer Motion |
|---|---|---|
| `zoomIn` | scale(0.5) → 1 | `initial:{scale:0.5,opacity:0} animate:{scale:1,opacity:1}` |
| `zoomInDown` | scale(0.5) + translateY(-60px) → 0 | combinar scale + y |
| `zoomOut` | scale(1) → 0 | `exit:{scale:0,opacity:0}` |

### Bounce / Attention
| Elementor | CSS equivalente |
|---|---|
| `bounce` | `animation: bounce 1s infinite` (Tailwind: `animate-bounce`) |
| `flash` | `animation: flash 1s` → opacidad 0/1 rápido |
| `pulse` | `animation: pulse 2s infinite` (Tailwind: `animate-pulse`) |
| `shake` | `animation: shake 0.5s` → translateX ±10px |
| `swing` | `animation: swing 1s` → rotate ±15deg |
| `tada` | `animation: tada 1s` → scale + rotate |
| `wobble` | `animation: wobble 1s` → translateX wave |
| `jello` | `animation: jello 1s` → skewX wave |
| `heartBeat` | `animation: heartBeat 1.3s` → scale 1 → 1.3 → 1 |

### Rotate / Flip
| Elementor | CSS equivalente |
|---|---|
| `rotateIn` | `rotate(-200deg) opacity(0) → rotate(0) opacity(1)` |
| `rotateInDownLeft` | origen bottom-left, rotate(-45deg) → 0 |
| `flipInX` | `perspective(400px) rotateX(90deg) → 0` |
| `flipInY` | `perspective(400px) rotateY(90deg) → 0` |
| `lightSpeedIn` | skewX(-30deg) + translateX(100%) → 0 |
| `rollIn` | translateX(-100%) + rotate(-120deg) → 0 |

---

## Template @keyframes CSS

```css
/* Usar con Intersection Observer — agregar clase .is-visible al entrar en viewport */

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes zoomIn {
  from {
    opacity: 0;
    transform: scale(0.85);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* Clase base para elementos animados */
[data-animate] {
  opacity: 0;
}

[data-animate].is-visible {
  animation-fill-mode: both;
  animation-duration: var(--animation-duration, 0.6s);
  animation-timing-function: var(--animation-easing, cubic-bezier(0.16, 1, 0.3, 1));
}

[data-animate="fadeInUp"].is-visible { animation-name: fadeInUp; }
[data-animate="zoomIn"].is-visible   { animation-name: zoomIn; }
/* ... agregar los que se usen */
```

---

## Template Intersection Observer (Vanilla / Astro)

```javascript
// animations.js — copiar tal cual, sin dependencias externas
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const delay = entry.target.dataset.animateDelay || '0ms';
        entry.target.style.animationDelay = delay;
        entry.target.classList.add('is-visible');
        observer.unobserve(entry.target); // animar solo una vez
      }
    });
  },
  {
    threshold: 0.15,
    rootMargin: '0px 0px -50px 0px',
  }
);

document.querySelectorAll('[data-animate]').forEach((el) => observer.observe(el));
```

**Uso en HTML:**
```html
<div data-animate="fadeInUp" data-animate-delay="200ms">
  Contenido que aparece al hacer scroll
</div>
```

---

## Propiedades de Timing de Elementor

| Propiedad Elementor | CSS equivalente |
|---|---|
| `animation_duration` (normal/slow/fast) | normal=0.6s, slow=1.2s, fast=0.3s |
| `animation_delay` | `animation-delay: [valor]ms` |
| `_animation_duration` (tablet/mobile) | En la media query correspondiente |

---

## Framer Motion — Setup para React

```tsx
// Para animaciones de scroll en Next.js / Vite + React
import { motion, useInView } from 'framer-motion';
import { useRef } from 'react';

const FadeInUp = ({ children, delay = 0 }) => {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true, margin: '-50px' });

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 30 }}
      animate={isInView ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.6, delay, ease: [0.16, 1, 0.3, 1] }}
    >
      {children}
    </motion.div>
  );
};
```

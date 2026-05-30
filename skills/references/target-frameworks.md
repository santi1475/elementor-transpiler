# Guías por Framework Destino

## Vanilla HTML5 / CSS3

### Estructura de archivos
```
proyecto/
├── index.html
├── css/
│   ├── reset.css        ← normalize o reset CSS moderno
│   ├── variables.css    ← :root con todos los tokens
│   ├── layout.css       ← grid, flex, secciones
│   ├── components.css   ← botones, cards, nav, etc.
│   ├── animations.css   ← @keyframes
│   └── responsive.css   ← media queries (o integradas)
└── js/
    └── animations.js    ← Intersection Observer
```

### Reset CSS Recomendado
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
img, video { max-width: 100%; display: block; }
body { min-height: 100vh; text-rendering: optimizeSpeed; }
p, h1, h2, h3, h4, h5, h6 { overflow-wrap: break-word; }
```

### Grid Layout
```css
.container {
  width: 100%;
  max-width: var(--container-width, 1200px);
  margin-inline: auto;
  padding-inline: var(--container-padding, 1.5rem);
}

.grid-2 { display: grid; grid-template-columns: repeat(2, 1fr); gap: var(--gap, 2rem); }
.grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: var(--gap, 2rem); }

@media (max-width: 768px) {
  .grid-2, .grid-3 { grid-template-columns: 1fr; }
}
```

---

## Astro

### Reglas específicas de Astro
- Cada sección de Elementor → un componente `.astro` separado
- Estilos dentro de `<style>` son scoped por defecto (no globales)
- Variables CSS globales van en `src/styles/global.css` importado en `Layout.astro`
- Imágenes → usar `<Image>` de `astro:assets` siempre para optimización
- Para animaciones: usar `client:visible` para cargar JS solo cuando el componente está en viewport

### Estructura de archivos
```
src/
├── pages/
│   └── index.astro
├── layouts/
│   └── Layout.astro        ← head, nav, footer, import global.css
├── components/
│   ├── Hero.astro
│   ├── Features.astro
│   ├── Testimonials.astro
│   └── CTA.astro
└── styles/
    └── global.css          ← :root variables + reset + base
```

### Template de componente Astro
```astro
---
// Hero.astro
interface Props {
  title: string;
  subtitle?: string;
  ctaText?: string;
  ctaHref?: string;
}
const { title, subtitle, ctaText = 'Get Started', ctaHref = '#' } = Astro.props;
---

<section class="hero" data-animate="fadeIn">
  <div class="container">
    <h1 class="hero__title">{title}</h1>
    {subtitle && <p class="hero__subtitle">{subtitle}</p>}
    <a href={ctaHref} class="btn btn--primary">{ctaText}</a>
  </div>
</section>

<style>
  .hero {
    padding-block: var(--spacing-section);
    background: var(--color-bg-dark);
  }
  /* estilos específicos del componente */
</style>
```

### View Transitions (Astro 3+)
Si la página de Elementor tenía efectos de transición entre secciones:
```astro
---
import { ViewTransitions } from 'astro:transitions';
---
<head>
  <ViewTransitions />
</head>
```

---

## Next.js + React + Tailwind

### Reglas específicas
- App Router (Next.js 13+): todo en `app/` directory
- `'use client'` solo cuando sea necesario (interactividad, hooks, estado)
- Server Components por defecto para contenido estático
- `<Image>` de `next/image` para todas las imágenes
- Fuentes: usar `next/font/google` en lugar de `<link>` en `<head>`

### Estructura de archivos
```
app/
├── page.tsx               ← Server Component (sin 'use client')
├── layout.tsx             ← root layout, fonts, metadata
├── globals.css            ← variables CSS custom + @layer base
└── components/
    ├── Hero/
    │   ├── Hero.tsx        ← puede ser Server Component
    │   └── HeroClient.tsx  ← 'use client' solo si tiene interactividad
    ├── Features/
    └── shared/
        ├── Button.tsx
        └── Container.tsx
tailwind.config.ts
```

### Setup de fuentes con next/font
```typescript
// app/layout.tsx
import { Poppins, Inter } from 'next/font/google';

const poppins = Poppins({
  subsets: ['latin'],
  weight: ['400', '600', '700'],
  variable: '--font-heading',
});

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-body',
});

export default function RootLayout({ children }) {
  return (
    <html lang="es" className={`${poppins.variable} ${inter.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

### Metadata
```typescript
// app/layout.tsx
export const metadata = {
  title: '[extraído del título de la página en Elementor]',
  description: '[extraído del meta description si está en el JSON]',
};
```

### Animaciones con Framer Motion
```typescript
// Solo en Client Components
'use client';
import { motion } from 'framer-motion';

// Variantes reutilizables
const fadeInUp = {
  hidden: { opacity: 0, y: 30 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.6, ease: [0.16, 1, 0.3, 1] } },
};

// Uso
<motion.section
  initial="hidden"
  whileInView="visible"
  viewport={{ once: true, margin: '-100px' }}
  variants={fadeInUp}
>
```

---

## Vite + React + Tailwind

### Diferencias vs Next.js
- Sin Server Components — todo es Client Side Rendering
- Sin `<Image>` optimizada — usar `<img>` con `loading="lazy"` y `decoding="async"`
- Sin file-based routing — definir rutas con React Router si es necesario
- Estructura similar pero sin `app/` directory: usar `src/`

### Estructura de archivos
```
src/
├── main.tsx
├── App.tsx
├── index.css           ← variables CSS + @layer base
├── components/
│   ├── Hero/
│   ├── Features/
│   └── shared/
└── hooks/
    └── useInView.ts    ← custom hook para animaciones
tailwind.config.ts
vite.config.ts
```

### Custom Hook para Animaciones (sin Framer Motion)
```typescript
// src/hooks/useInView.ts
import { useEffect, useRef, useState } from 'react';

export const useInView = (options = {}) => {
  const ref = useRef<HTMLElement>(null);
  const [isInView, setIsInView] = useState(false);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsInView(true);
        observer.unobserve(entry.target);
      }
    }, { threshold: 0.15, ...options });

    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, []);

  return { ref, isInView };
};
```

---

## Checklist de Accesibilidad (Todos los Targets)

Elementor no genera HTML accesible — asegurarse de incluir:

- [ ] `lang="es"` (o el idioma correcto) en `<html>`
- [ ] `<main>`, `<nav>`, `<header>`, `<footer>` en lugar de solo `<div>`
- [ ] Jerarquía de headings correcta (un solo `<h1>`, luego `<h2>`, etc.)
- [ ] `alt` en todas las imágenes (vacío `alt=""` si son decorativas)
- [ ] `aria-label` en botones que solo tienen iconos
- [ ] `role="tablist"` + `role="tab"` + `aria-selected` en tabs
- [ ] `aria-expanded` en acordeones
- [ ] `aria-label` en `<nav>` si hay múltiples navs
- [ ] Focus visible (no eliminar outline sin reemplazarlo)
- [ ] Ratio de contraste mínimo 4.5:1 para texto normal, 3:1 para texto grande

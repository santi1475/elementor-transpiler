---
name: elementor-transpiler
description: >
  Transpiler experto que convierte JSON de Elementor (WordPress page builder) en código
  frontend moderno y listo para producción. Úsalo SIEMPRE que el usuario mencione:
  "convertir Elementor", "JSON de Elementor", "exportar diseño de Elementor",
  "migrar de Elementor", "página de WordPress a código", o cuando pegue un bloque
  JSON con estructura de Elementor (nodos con `elType`, `widgetType`, `settings`).
  Soporta salida en HTML/CSS Vanilla, Astro, Next.js (React + Tailwind), y Vite.
  No necesitas que el usuario diga "usa el skill" — si hay un JSON de Elementor o
  intención de migración, activa este skill automáticamente.
---

# Elementor → Modern Stack Transpiler (v2.0)

Eres un arquitecto frontend y compilador experto. Tu única función es tomar la
estructura JSON de Elementor y convertirla en código frontend limpio, semántico y
listo para producción, adaptado al ecosistema del proyecto destino.

---

## FASE 0 — Validación del Input

Antes de cualquier otra cosa, verifica que el JSON de entrada sea válido:

**Señales de JSON de Elementor legítimo:**
- Nodos con campos `elType` (valores: `"widget"`, `"section"`, `"column"`, `"container"`)
- Widgets con `widgetType` (ej: `"heading"`, `"image"`, `"button"`, `"text-editor"`)
- Objeto `settings` en cada nodo con propiedades de estilo

**Si el JSON está incompleto o malformado:**
- Genera el código con lo que haya disponible
- Inserta comentarios `<!-- ELEMENTOR_DATA_MISSING: [campo] -->` donde falte información
- Nunca abortes — produce el mejor output posible

**Plugins de terceros (Essential Addons, JetElements, WooCommerce widgets):**
- Identifícalos por prefijos en `widgetType`: `eael-`, `jet-`, `woocommerce-`
- Genera un skeleton component estilizado + comentario:
  ```html
  <!-- THIRD_PARTY_WIDGET: [nombre del plugin] — reemplazar con implementación nativa -->
  ```

---

## FASE 1 — Detección del Ecosistema

Lee los archivos de configuración si el usuario los provee. Si no hay ninguno, usa **HTML5/CSS3 Vanilla**.

| Archivo detectado | Target |
|---|---|
| `astro.config.mjs` | Astro (Island Architecture) |
| `next.config.js/ts` + `tailwind.config` | Next.js + React + Tailwind |
| `vite.config.js` + `tailwind.config` | Vite + React/Vue + Tailwind |
| Solo `tailwind.config` | HTML + Tailwind CDN |
| Ninguno | HTML5 / CSS3 / JS Vanilla |

**Pregunta al usuario si no puedes deducirlo:**
> "¿Cuál es el framework destino: Vanilla HTML, Astro, Next.js o Vite + Tailwind?"

---

## FASE 2 — Extracción de Tokens de Diseño

Antes de generar código, extrae y centraliza todos los tokens de diseño del JSON.

### 2.1 Sistema de Color
Lee `references/color-system.md` para las reglas completas.

Resumen rápido:
- Extrae todos los hex codes únicos de los campos: `color`, `background_color`, `border_color`, `overlay_color`, `heading_color`
- Nómbralos semánticamente basándote en su uso más frecuente: `--color-primary`, `--color-accent`, `--color-bg`, `--color-text`
- Para Tailwind: extiende `theme.extend.colors` en `tailwind.config`

### 2.2 Tipografía
Lee `references/typography-map.md` para las reglas completas.

Resumen rápido:
- Campos a extraer: `typography_font_family`, `typography_font_size`, `typography_font_weight`, `typography_line_height`, `typography_letter_spacing`
- Mapear a variables CSS: `--font-heading`, `--font-body`, `--text-base`, `--text-lg`...
- Si hay Google Fonts: agregar el `<link>` de preconnect + stylesheet al `<head>`
- Si hay fuentes custom: generar `@font-face` con `font-display: swap`

### 2.3 Espaciado
- Extraer padding/margin de secciones principales para definir `--spacing-section`, `--spacing-block`
- Evitar hardcodear valores repetidos; usar variables CSS o la escala de Tailwind

---

## FASE 3 — Reglas de Traducción

### 3.1 Estructura HTML (Anti-Divitis)

❌ **Nunca hagas esto (patrón Elementor):**
```html
<div class="elementor-section">
  <div class="elementor-container">
    <div class="elementor-row">
      <div class="elementor-column">
        <div class="elementor-widget-wrap">
```

✅ **Haz esto:**
```html
<section class="hero">
  <div class="hero__content">
    <h1 class="hero__title">...</h1>
```

**Reglas de semántica:**
- Primera sección → `<header>` o `<section class="hero">`
- Secciones de contenido → `<section>`, `<article>` según el tipo
- Barra de navegación → `<nav>`
- Pie → `<footer>`
- Máximo 2 niveles de anidamiento por widget
- Nomenclatura de clases: BEM para Vanilla/Astro, utilidades para Tailwind

### 3.2 Mapeo de Widgets Core

| Widget Elementor | Output Vanilla/Astro | Output Tailwind/React |
|---|---|---|
| `heading` | `<h1>`–`<h6>` semántico | `<h1 className="...">` |
| `text-editor` | `<p>` o `<div class="prose">` | `<p className="prose">` |
| `image` | `<img loading="lazy" alt="">` | `<Image>` (Next.js) / `<img>` |
| `button` | `<a class="btn">` | `<button className="btn">` |
| `icon` | SVG inline o `<span>` con aria | SVG component |
| `spacer` | `margin` / `padding` en CSS | clases de margen Tailwind |
| `divider` | `<hr>` estilizado | `<hr className="...">` |
| `video` | `<figure><iframe>` con lazy | componente Video con suspense |
| `testimonial` | `<blockquote><cite>` | `<blockquote className="...">` |
| `icon-list` | `<ul>` con SVG icons | `<ul className="...">` |
| `counter` | `<span>` + Intersection Observer | componente Counter con hook |
| `progress-bar` | `<progress>` nativo | componente animado |
| `tabs` | `<div role="tablist">` ARIA | componente Tabs |
| `accordion` | `<details><summary>` | componente Accordion |
| `nav-menu` | `<nav><ul>` semántico | componente Nav |

### 3.3 Animaciones e Interactividad
Lee `references/animation-map.md` para la tabla completa de equivalencias.

**Regla de decisión por entorno:**

```
Animación de entrada (fade, slide, zoom):
├── Vanilla / Astro → @keyframes CSS puro + clase .animate-[nombre]
│   activada por Intersection Observer API (sin librerías)
├── Next.js / React → Framer Motion (<motion.div>) o Tailwind animate-*
└── Astro con interactividad → client:visible + transition:animate

Scroll effects / parallax:
├── Vanilla / Astro → Intersection Observer API con threshold multiple
├── React → useScroll + useTransform de Framer Motion
└── Efecto parallax complejo → GSAP ScrollTrigger (solo si es necesario)

Mouse tracking / hover effects:
├── Vanilla → CSS :hover + transform / CSS @starting-style
└── React → onMouseMove con useState para transform dinámico
```

### 3.4 Fondos y Efectos Visuales

**Gradientes:**
```css
/* Elementor: background_type = gradient */
background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-accent) 100%);
```

**Background overlay (color sobre imagen):**
```css
.section-with-overlay {
  position: relative;
}
.section-with-overlay::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.5); /* extraído del JSON */
  z-index: 1;
}
```

**Glassmorphism** (`background_type = glass` o `backdrop_filter` en settings):
```css
.glass-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}
/* Tailwind: backdrop-blur-md bg-white/10 border border-white/20 */
```

### 3.5 Responsividad

Elementor usa 3 breakpoints. Mapeo a equivalentes modernos:

| Elementor | CSS Media Query | Tailwind prefix |
|---|---|---|
| desktop (default) | base styles | base (sin prefijo) |
| tablet (`_tablet` suffix) | `@media (max-width: 1024px)` | `lg:` |
| mobile (`_mobile` suffix) | `@media (max-width: 767px)` | `md:` / `sm:` |

**Regla:** Si el valor mobile es igual al desktop, omite la media query (no duplicar).

### 3.6 Imágenes y Assets

- URLs absolutas de WordPress → reemplazar con placeholder comentado:
  ```html
  <img src="/images/[nombre-descriptivo].jpg" <!-- REEMPLAZAR con asset real -->
       alt="[alt extraído del JSON o vacío si no existe]"
       loading="lazy"
       width="[ancho del JSON]"
       height="[alto del JSON]">
  ```
- Para Next.js: usar `<Image>` de `next/image` siempre
- Para Astro: usar `<Image>` de `astro:assets` siempre
- SVG inline: si el JSON contiene SVG, extraerlo limpio sin atributos innecesarios

---

## FASE 4 — Organización de la Salida

### Vanilla HTML
```
index.html          ← estructura completa
css/
  variables.css     ← tokens de diseño (:root)
  layout.css        ← grid, flex, secciones
  components.css    ← botones, cards, etc.
  animations.css    ← @keyframes
js/
  animations.js     ← Intersection Observer
```

### Astro
```
src/
  pages/index.astro          ← página principal
  components/
    Hero.astro
    [Sección].astro
  styles/global.css          ← variables CSS + reset
```

### Next.js + Tailwind
```
app/
  page.tsx                   ← página principal
  components/
    [Sección]/
      index.tsx
      [Sección].module.css   ← solo si hay estilos no cubribles con Tailwind
  styles/globals.css         ← variables CSS custom
tailwind.config.ts           ← tokens extendidos
```

---

## FASE 5 — Optimización y Limpieza

Antes de entregar el output, verifica:

- [ ] Sin clases o IDs de Elementor en el output (`elementor-*`, `e-`)
- [ ] Sin estilos inline salvo valores dinámicos imposibles de predecir
- [ ] Todas las imágenes tienen `alt` (vacío si decorativo, descriptivo si informativo)
- [ ] Todos los componentes interactivos tienen atributos ARIA mínimos
- [ ] Las Google Fonts tienen `rel="preconnect"` antes del `<link>` de la fuente
- [ ] Los colores y fuentes están en variables, no hardcodeados
- [ ] El código está comentado en secciones principales (`/* === HERO SECTION === */`)

---

## Formato de Salida

1. Entrega **solo código**, sin explicaciones de lo que encontraste en el JSON
2. Comienza con el archivo más importante (generalmente `index.html` o `page.tsx`)
3. Separa archivos con un comentario de cabecera claro:
   ```
   /* ==============================
      ARCHIVO: css/variables.css
      ============================== */
   ```
4. Si hay widgets de terceros, agrúpalos al final en una sección `PENDIENTES`
5. Si el JSON es muy grande, pregunta si prefieres procesar sección por sección

---

## Referencias (leer según necesidad)

- `references/animation-map.md` — Tabla completa de 40+ animaciones de Elementor
- `references/color-system.md` — Extracción y naming semántico de colores
- `references/typography-map.md` — Mapeo de tipografía y Google Fonts
- `references/target-frameworks.md` — Guías específicas por framework

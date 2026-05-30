# Color System: Elementor → Modern Stack

## Dónde Viven los Colores en el JSON de Elementor

### Global Colors (Elementor Kit)
Si el JSON incluye datos del kit global de Elementor, los colores aparecen así:
```json
{
  "system_colors": [
    { "id": "primary", "color": "#6EC1E4", "title": "Primary" },
    { "id": "secondary", "color": "#54595F", "title": "Secondary" },
    { "id": "text", "color": "#7A7A7A", "title": "Text" },
    { "id": "accent", "color": "#61CE70", "title": "Accent" }
  ]
}
```
→ Estos se mapean directamente a variables CSS con sus nombres.

### Colores Inline (en settings de cada widget)
```json
{
  "color": "#FFFFFF",
  "background_color": "#1A1A1A",
  "border_color": "#EEEEEE",
  "heading_color": "#2C2C2C",
  "button_background_color": "#6EC1E4",
  "button_text_color": "#FFFFFF",
  "overlay_color": "rgba(0,0,0,0.5)"
}
```

---

## Algoritmo de Naming Semántico

Cuando los colores no tienen nombre en el JSON, inferir semánticamente:

1. **El color más oscuro** (luminosidad < 20%) = `--color-dark` o `--color-text`
2. **El color más claro** (luminosidad > 85%) = `--color-light` o `--color-bg`
3. **El color de mayor frecuencia** en buttons/CTAs = `--color-primary`
4. **El color de menor frecuencia** en CTAs = `--color-accent`
5. **Colores de fondo de sección** = `--color-bg-[section-name]`
6. **Grises de texto** = `--color-text-muted`, `--color-text-subtle`

**Si hay más de 6 colores únicos**, agrupar los muy similares (diferencia < 15 en hex) bajo el mismo nombre con variantes:
```css
--color-primary: #6EC1E4;
--color-primary-dark: #4DA8CB;
--color-primary-light: #8FD3EF;
```

---

## Output CSS Variables

```css
:root {
  /* Paleta Principal */
  --color-primary:     #6EC1E4;
  --color-secondary:   #54595F;
  --color-accent:      #61CE70;

  /* Textos */
  --color-heading:     #2C2C2C;
  --color-text:        #7A7A7A;
  --color-text-muted:  #AAAAAA;

  /* Fondos */
  --color-bg:          #FFFFFF;
  --color-bg-alt:      #F9F9F9;
  --color-bg-dark:     #1A1A1A;

  /* UI */
  --color-border:      #EEEEEE;
  --color-overlay:     rgba(0, 0, 0, 0.5);
}
```

---

## Output Tailwind Config

```typescript
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      primary: {
        DEFAULT: '#6EC1E4',
        dark: '#4DA8CB',
        light: '#8FD3EF',
      },
      secondary: '#54595F',
      accent: '#61CE70',
      background: {
        DEFAULT: '#FFFFFF',
        alt: '#F9F9F9',
        dark: '#1A1A1A',
      },
      text: {
        DEFAULT: '#7A7A7A',
        heading: '#2C2C2C',
        muted: '#AAAAAA',
      },
    },
  },
}
```

---

## Gradientes

### Gradiente lineal simple
```json
{ "background_color_b": "#FF6B6B", "gradient_angle": 135 }
```
→ CSS: `background: linear-gradient(135deg, var(--color-primary), #FF6B6B);`
→ Tailwind: `bg-gradient-to-br from-primary to-[#FF6B6B]`

### Gradiente con múltiples stops
```css
background: linear-gradient(
  180deg,
  var(--color-bg) 0%,
  var(--color-bg-alt) 50%,
  var(--color-primary) 100%
);
```

### Radial gradient
```css
background: radial-gradient(circle at center, var(--color-primary) 0%, transparent 70%);
```

---

## Overlays y Capas

Elementor permite imagen de fondo + overlay de color encima. Patrón CSS:

```css
.section-with-overlay {
  position: relative;
  background-image: url('/images/bg.jpg');
  background-size: cover;
  background-position: center;
}

.section-with-overlay::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--color-overlay); /* rgba extraída del JSON */
  z-index: 0;
}

.section-with-overlay > * {
  position: relative;
  z-index: 1; /* Contenido sobre el overlay */
}
```

---

## Glassmorphism

Detectar cuando `settings` tenga `background_color` con alpha bajo + presencia de blur:

```css
.glass {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}
```

Tailwind: `bg-white/10 backdrop-blur-md border border-white/20 shadow-lg`

---

## Box Shadow

| Elementor preset | CSS equivalente |
|---|---|
| None | `box-shadow: none` |
| Box Shadow (custom) | `box-shadow: [h] [v] [blur] [spread] [color]` |

Extraer de: `box_shadow_box_shadow_type`, `box_shadow_box_shadow`

```css
/* Ejemplo extraído */
box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
/* Tailwind: shadow-md o shadow-[0_4px_20px_rgba(0,0,0,0.08)] */
```

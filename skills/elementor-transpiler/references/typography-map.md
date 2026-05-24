# Typography Map: Elementor → Modern Stack

## Campos de Tipografía en el JSON de Elementor

Elementor almacena tipografía con este patrón en `settings`:

```json
{
  "typography_font_family": "Roboto",
  "typography_font_size": { "unit": "px", "size": 18 },
  "typography_font_weight": "600",
  "typography_line_height": { "unit": "em", "size": 1.6 },
  "typography_letter_spacing": { "unit": "px", "size": 0.5 },
  "typography_text_transform": "uppercase",
  "typography_font_style": "italic",
  "typography_text_decoration": "underline"
}
```

También pueden aparecer con prefijos de grupo:
- `title_typography_font_family` (en widget heading)
- `button_typography_font_size` (en widget button)
- `description_typography_*` (en varios widgets)

---

## Estrategia de Extracción

### Paso 1: Identificar fuentes únicas

Recorre todos los nodos y extrae todos los `*_font_family` únicos. Clasifícalos:

**Google Fonts** (lista parcial de las más comunes en Elementor):
Roboto, Open Sans, Lato, Montserrat, Poppins, Raleway, Oswald, Source Sans Pro,
Nunito, Playfair Display, Merriweather, Ubuntu, Rubik, Inter, DM Sans, Plus Jakarta Sans

**System fonts** (no requieren `<link>`):
Arial, Helvetica, Georgia, Times New Roman, Courier New, system-ui, -apple-system

**Si hay Google Fonts → generar en `<head>`:**
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&family=Inter:wght@400;500&display=swap" rel="stylesheet">
```
Agrupar todas las fuentes en **un solo `<link>`** — nunca múltiples requests.

### Paso 2: Crear escala tipográfica

Analiza los tamaños encontrados y crea una escala semántica:

```css
:root {
  /* Familias */
  --font-heading: 'Poppins', sans-serif;
  --font-body: 'Inter', sans-serif;
  --font-mono: 'Courier New', monospace; /* si aplica */

  /* Escala de tamaños */
  --text-xs:   0.75rem;   /* 12px */
  --text-sm:   0.875rem;  /* 14px */
  --text-base: 1rem;      /* 16px */
  --text-lg:   1.125rem;  /* 18px */
  --text-xl:   1.25rem;   /* 20px */
  --text-2xl:  1.5rem;    /* 24px */
  --text-3xl:  1.875rem;  /* 30px */
  --text-4xl:  2.25rem;   /* 36px */
  --text-5xl:  3rem;      /* 48px */
  --text-6xl:  3.75rem;   /* 60px */

  /* Pesos */
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;

  /* Line heights */
  --leading-tight:  1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;
}
```

Mapea los tamaños del JSON al valor de escala más cercano. Si el tamaño es muy específico (ej: 17px), usa el más próximo (--text-lg = 18px) y comenta el original.

---

## Mapeo a Tailwind (Next.js / Vite)

En `tailwind.config.ts`:

```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  theme: {
    extend: {
      fontFamily: {
        heading: ['Poppins', 'sans-serif'],
        body: ['Inter', 'sans-serif'],
      },
      fontSize: {
        // Solo agregar tamaños NO en la escala default de Tailwind
        '2xs': ['0.625rem', { lineHeight: '1rem' }],
        '7xl': ['5rem', { lineHeight: '1' }],
      },
    },
  },
};
```

---

## Estilos Base Recomendados

```css
/* En global.css o el CSS principal */
body {
  font-family: var(--font-body);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  color: var(--color-text);
}

h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
  font-weight: var(--font-bold);
  line-height: var(--leading-tight);
  color: var(--color-heading);
}

/* Tailwind: usar @layer base para lo mismo */
@layer base {
  body {
    @apply font-body text-base leading-normal text-gray-800;
  }
  h1, h2, h3, h4 {
    @apply font-heading font-bold leading-tight;
  }
}
```

---

## Conversión de Unidades

| Unidad Elementor | Conversión |
|---|---|
| `px` → `rem` | valor ÷ 16 (base 16px) |
| `em` | mantener como está |
| `vw` | mantener como está |
| `%` | mantener como está |

**Regla:** Convertir siempre px a rem para tipografía (accesibilidad). Para bordes, shadows y valores de 1-4px está bien mantener px.

---

## Responsive Typography

Elementor permite tamaños diferentes por breakpoint. Usar clamp() cuando sea posible:

```css
/* En vez de múltiples media queries para font-size: */
h1 {
  font-size: clamp(2rem, 5vw, 3.75rem);
  /* mobile: 2rem, desktop: 3.75rem, fluido entre ellos */
}
```

Para Tailwind: `text-2xl md:text-4xl lg:text-6xl`

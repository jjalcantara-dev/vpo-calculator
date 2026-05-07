# VPO Calculator — Contexto para Claude Code

## Qué es este proyecto

Calculadora interactiva de VPO (Vivienda de Protección Oficial) en régimen cooperativo en Andalucía, basada en documentación pública de la promoción "Galivivienda Fortalezas de Vélez" en Vélez-Málaga.

## Stack

- **HTML estático puro** — un solo archivo `index.html`, sin framework, sin dependencias npm
- **CSS con variables** — sistema de diseño oscuro con tokens en `:root`
- **JavaScript vanilla** — toda la lógica en un `<script>` al final del HTML
- **Google Fonts** — DM Serif Display (títulos), DM Mono (etiquetas/valores), Outfit (cuerpo)

## Estructura del proyecto

```
vpo-calc/
├── index.html     — toda la aplicación en un único archivo
├── vercel.json    — headers de caché para Vercel
├── README.md      — descripción del proyecto
└── CLAUDE.md      — este archivo
```

## Arquitectura del HTML

El archivo `index.html` tiene estas secciones en orden:

1. **`<head>`** — meta, Google Fonts, CSS completo con variables
2. **`.hero`** — título, badges, distribución de parcelas
3. **Sección intro** — explicación VPO cooperativa, guía de parámetros (fondo bg2)
4. **`.controls-bar`** — inputs editables (valor piso, alquiler) + botones de parámetros (IPC, descuento, comunidad, años)
5. **Barra de métricas** — 6 KPIs reactivos justo debajo de los controles
6. **`.main`** — contenido principal:
   - Caja de confirmados por contrato
   - Tabla de pagos detallada
   - Comparativa de escenarios (4 tarjetas)
   - Riesgos e incertidumbres (id="riesgos")
   - Timeline de propiedad (~2029 → ~2059)
   - Conclusión (pros y contras)
7. **`<script>`** — lógica JS completa

## Lógica JavaScript

### Estado global
```js
const params = { ipc: 2, pct: 50, com: 0, rentIpc: true, years: 20 };
```

### Funciones clave
- `setParam(k, v, el)` — actualiza params y llama render()
- `getInputs()` — lee los inputs editables (valor piso, alquiler base)
- `parseNum(str)` — parsea números con punto de miles español (196.760 → 196760)
- `fmtInput(el)` — formatea input con separador de miles al salir del campo
- `calcBuy(ipc, pct, rentIpc, years)` — calcula adjFinal, totalRent, buyGross, buyNet, rentLastYear
- `render()` — actualiza todos los elementos del DOM con los valores calculados
- `fmt(n)` — formatea número entero con separador de miles español + €
- `fmtDec(n)` — formatea número decimal con 2 decimales español + €

### Cálculo principal
```
adjFinal = adjBase × (1 + ipc/100)^years
totalRent = Σ rentBase × (1 + ipc/100)^(m/12)  [para m=0..months, si rentIpc=true]
discount = totalRent × (pct/100)
buyGross = adjFinal - discount
buyNet = buyGross - 39000   // los 39k de aportaciones se devuelven íntegramente al comprar
```

## Sistema de diseño

### Variables CSS (tema claro, estilo portfolio)
```css
--bg: #fff           /* blanco */
--bg2: #fafaf9       /* fondo tarjetas */
--bg3: #f4f3f1       /* fondo inputs/headers */
--border: rgba(0,0,0,0.09)
--border2: rgba(0,0,0,0.2)
--text: #111110
--muted: #78716c
--accent: #111110    /* negro */
--ok: #16a34a        /* verde */
--danger: #dc2626    /* rojo */
--warn: #b45309      /* ámbar */
--info: #2563eb      /* azul */
```

### Tipografía
- `Inter` — fuente principal (cuerpo, títulos, UI)
- `DM Mono` — exclusivamente para números y etiquetas de datos

### Componentes CSS principales
- `.kpi` — tarjeta de métrica
- `.btn-opt` — botón de opción (con `.active` para seleccionado)
- `.tag` + `.tag-ok/warn/danger/info` — etiquetas de estado
- `.risk-row` — fila de riesgo (grid 3 columnas)
- `.scenario-card` — tarjeta de escenario comparativo
- `.summary-box` — caja de conclusión (grid 2 columnas)
- `.confirmed-box` — caja de datos confirmados por contrato

## Datos base (valores por defecto)

- Valor piso 2025: **196.760 €** (Ático A, Edificio 1, 3 dorm, 81,70 m²)
- Alquiler mensual base: **819,84 €/mes**
- Aportaciones cooperativa: **39.000 €** desglosadas en:
  - 25.000 € al firmar la adhesión a la cooperativa
  - 208,33 €/mes × 24 meses desde licencia de obras = 5.000 €
  - 9.000 € en entrega de la vivienda e inicio del periodo de alquiler
- IBI: **~500–600 €/año** (tipo 0,65% sobre valor catastral en Vélez-Málaga; bonificación 50% VPO solo los 3 primeros años)
- Periodo mínimo alquiler: **20 años**
- Descuento mínimo contractual: **50%** (cláusula 16ª contrato arrendamiento)

## Contexto legal relevante

- Decreto 149/2006 — Reglamento VPO Andalucía
- Decreto 91/2020 — Plan Vive Andalucía 2020-2030
- Contrato de participación social Galivivienda (cláusula 4ª: aportaciones, cláusula 16ª: opción de compra)
- Contrato de arrendamiento Galivivienda (cláusula 5ª: gastos, cláusula 8ª: prohibición subarriendo)
- Calificación VPO: 30 años desde calificación definitiva

## Convenciones

- Nunca hardcodear colores — siempre usar variables CSS
- Nunca usar `innerHTML` para textos con datos del usuario
- Siempre verificar sintaxis JS con `node --check` antes de guardar
- Los números en español: punto como separador de miles, coma como decimal
- Todas las cadenas dinámicas con concatenación de strings, NO template literals anidados (causa errores de sintaxis)

## Deploy

```bash
vercel --prod
# Custom domain: vpo.jjalcantara.dev
# DNS: CNAME vpo → cname.vercel-dns.com
```
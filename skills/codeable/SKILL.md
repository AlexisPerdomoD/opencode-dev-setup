---
name: codeable-plantilla-docs
description: >
  Usa este skill para generar documentos Word (.docx) siguiendo la plantilla corporativa.
  Aplica siempre que el usuario pida crear un documento Word, un plan de trabajo, informe
  semanal, agenda de sesiones, entregable de proyecto, o cualquier documento profesional
  que deba seguir el formato y estilo visual de la organización. Triggered por frases como
  "crea un documento", "genera un plan de trabajo", "necesito un informe", "hazme un .docx",
  "documento con la plantilla", etc. Úsalo incluso si el usuario solo dice "haz un Word de X"
  sin mencionar explícitamente la plantilla.
---

# Plantilla Corporativa — Documentos Word

Este skill genera documentos `.docx` siguiendo el formato visual y estructural de la
plantilla corporativa.

## Configuración de logos

Los logos se configuran en `config.json` (mismo directorio que este SKILL.md). Cada logo tiene:
- `file`: ruta relativa al asset PNG
- `width` / `height`: dimensiones en pixeles
- `offsetX` / `offsetY`: posición flotante en DXA

Para usar otro cliente, reemplazar los PNGs en `assets/` y ajustar dimensiones en `config.json` si es necesario.

## Dependencias

```bash
npm install -g docx
```

Siempre lee `/mnt/skills/public/docx/SKILL.md` antes de escribir código.

---

## Referencia rápida de la plantilla

| Elemento | Valor |
|---|---|
| Fuente | **Inter** (fallback Arial) |
| Página | A4: 11909 × 16834 DXA |
| Márgenes | top=850, bottom=567, left=567, right=567, header=720, footer=720 |
| Color header sección (fondo) | `#000000` (negro puro) |
| Color texto header sección | `#FFFFFF` (blanco) |
| Color header tabla de datos (fondo) | `#F5F9FF` (azul muy claro) |
| Color borde azul (tablas datos) | `#3A83F7` |
| Ancho tabla sección | 10777 DXA |
| Ancho tabla datos | 10710 DXA |
| Texto filas de datos | sz=18 (9pt), Inter, sin bold |

---

## Tipografía por elemento

| Elemento | sz (half-pts) | pt | Bold | Color | Notas |
|---|---|---|---|---|---|
| Título principal | 40 | 20pt | Sí | default | |
| Subtítulo | 30 | 15pt | Sí | default | |
| Header de sección | 20 | 10pt | Sí | `FFFFFF` | spacing line=240 |
| Texto de contenido | sin especificar | hereda | No | default | jc=both, before=200 |
| Header de día (fecha) | 20 | 10pt | Sí (fecha) / No (objetivo) | default | texto mixto |
| Header tabla datos | 20 | 10pt | Sí | default | fondo F5F9FF, borde azul |
| Celda tabla datos | 18 | 9pt | No | default | jc=left, line=276 |
| Viñeta nivel 1 (○) | 20 | 10pt | No (o Sí si es título de sesión) | default | |
| Viñeta nivel 2 (■) | 20 | 10pt | No | default | |
| Separador | 10 | 5pt | No | default | spacing line=240 |

---

## Estructura correcta del documento

⚠️ **CRÍTICO**: Las tablas de sección tienen **SOLO 1 FILA** (el header oscuro).
El contenido (texto y viñetas) va **directamente en el cuerpo del documento**, NO dentro de tablas.
Excepción: las **tablas de datos** (`dataTable`) sí contienen múltiples filas (header + datos).

```
Documento body:
├── Párrafo: Título (sz=40, bold)
├── Párrafo: Subtítulo (sz=30, bold)
├── Párrafo: Separador (sz=10)
├── TABLA seccionHeader("1. OBJETIVO") ← solo 1 fila negra
├── Párrafo: contenido(texto objetivo) ← en body, sin tabla
├── Párrafo: Separador
├── TABLA seccionHeader("2. AGENDA")   ← solo 1 fila negra
├── Párrafo vacío (sz=20)
├── TABLA diaHeader("Lunes...", "Objetivo: ...")  ← 1 fila azul claro, bordes azules
├── Párrafo: bullet("Sesión X", level=1)         ← en body, sin tabla
├── Párrafo: bullet("sub-item", level=2)         ← en body, sin tabla
├── ...más bullets...
├── Párrafo vacío (sz=20)
├── TABLA diaHeader("Martes...", "Objetivo: ...")
├── ...bullets...
├── TABLA seccionHeader("3. RIESGOS")
├── ...bullets...
├── Párrafo: Separador
├── TABLA dataTable(headers, rows)  ← multi-fila: header azul + filas de datos
└── ...
```

---

## Tablas: seccionHeader

Una sola fila. Fondo negro `#000000`, texto blanco 10pt bold. Layout fijo.

```javascript
function seccionHeader(titulo) {
  return new Table({
    width: { size: TABLE_WIDTH, type: WidthType.DXA },
    columnWidths: [TABLE_WIDTH],
    layout: TableLayoutType.FIXED,
    borders: { top: borde("000000"), bottom: borde("000000"), left: borde("000000"),
               right: borde("000000"), insideH: borde("000000"), insideV: borde("000000") },
    rows: [new TableRow({ children: [new TableCell({
      width:   { size: TABLE_WIDTH, type: WidthType.DXA },
      shading: { fill: "000000", type: ShadingType.CLEAR },
      margins: { top: 100, bottom: 100, left: 100, right: 100 },
      verticalAlign: "top",
      children: [new Paragraph({
        spacing: { line: 240, lineRule: "auto" },
        children: [new TextRun({ text: titulo, font: "Inter", size: 20, bold: true, color: "FFFFFF" })]
      })]
    })]})],
  });
}
```

---

## Tablas: diaHeader

Una sola fila. Fondo `#F5F9FF`, bordes top/bottom azul `#3A83F7`, sin bordes laterales. Layout fijo.
Texto: **fecha** bold + `|` normal + ` objetivo` normal. Todo 10pt.

```javascript
function diaHeader(fecha, objetivo) {
  const azul = { style: BorderStyle.SINGLE, size: 8, color: "3A83F7" };
  const nil  = { style: BorderStyle.NIL,    size: 0, color: "000000" };
  return new Table({
    width: { size: TABLE_WIDTH, type: WidthType.DXA },
    columnWidths: [2190, 2862, 2862, 2863],
    layout: TableLayoutType.FIXED,
    borders: { top: nil, bottom: nil, left: nil, right: nil, insideH: nil, insideV: nil },
    rows: [new TableRow({
      height: { value: 500, rule: "atLeast" },
      children: [new TableCell({
        columnSpan: 4,
        width:   { size: TABLE_WIDTH, type: WidthType.DXA },
        borders: { top: azul, bottom: azul, left: nil, right: nil },
        shading: { fill: "F5F9FF", type: ShadingType.CLEAR },
        margins: { top: 100, bottom: 100, left: 100, right: 100 },
        verticalAlign: "top",
        children: [new Paragraph({ children: [
          new TextRun({ text: fecha + " ",    font: "Inter", size: 20, bold: true }),
          new TextRun({ text: "|",            font: "Inter", size: 20, bold: false }),
          new TextRun({ text: " " + objetivo, font: "Inter", size: 20, bold: false }),
        ]})]
      })]
    })],
  });
}
// Uso: diaHeader("Miércoles 11 de Febrero", "Objetivo: Entender la realidad del terreno...")
```

---

## Tablas: dataTable (tabla de datos con header + filas)

Tabla multi-fila con header azul claro y filas de datos. Layout fijo, ancho 10710 DXA.
Bordes de tabla todos NIL. Los bordes se controlan por celda.

**Header (fila 0)**: fondo `#F5F9FF`, bordes top/bottom azul `#3A83F7`, sin bordes laterales.
Texto sz=20 bold, font Inter.

**Filas de datos (fila 1..N-1)**: sin fondo, todos los bordes nil.
Texto sz=18 (9pt), font Inter, sin bold. Spacing: before=0, after=0, line=276. jc=left.

**Última fila de datos (fila N)**: igual que filas de datos PERO con borde bottom azul `#3A83F7`
para cerrar visualmente la tabla.

```javascript
const DATA_TABLE_WIDTH = 10710;

function dataTable(headers, rows) {
  // headers: [{ text: "Riesgo", width: 4485 }, { text: "Impacto", width: 1170 }, ...]
  // rows: [["Regresiones no detectadas", "Alto", "Alta", "Sin suite de testing"], ...]
  const azul = { style: BorderStyle.SINGLE, size: 8, color: "3A83F7" };
  const nil  = { style: BorderStyle.NIL,    size: 0, color: "000000" };

  // Header row
  const headerRow = new TableRow({
    height: { value: 500, rule: HeightRule.AT_LEAST },
    children: headers.map(h => new TableCell({
      width: { size: h.width, type: WidthType.DXA },
      shading: { fill: "F5F9FF", type: ShadingType.CLEAR },
      borders: { top: azul, bottom: azul, left: nil, right: nil },
      margins: { top: 100, bottom: 100, left: 100, right: 100 },
      verticalAlign: "top",
      children: [new Paragraph({ children: [
        new TextRun({ text: h.text, font: "Inter", size: 20, bold: true })
      ]})]
    }))
  });

  // Data rows
  const dataRows = rows.map((row, rowIdx) => {
    const isLast = rowIdx === rows.length - 1;
    return new TableRow({
      height: { value: 531, rule: HeightRule.AT_LEAST },
      children: row.map((cellText, colIdx) => new TableCell({
        width: { size: headers[colIdx].width, type: WidthType.DXA },
        borders: {
          top: nil,
          bottom: isLast ? azul : nil,  // última fila cierra con borde azul
          left: nil,
          right: nil,
        },
        margins: { top: 100, bottom: 100, left: 100, right: 100 },
        verticalAlign: "top",
        children: [new Paragraph({
          alignment: AlignmentType.LEFT,
          spacing: { before: 0, after: 0, line: 276, lineRule: "auto" },
          indent: { left: 0, right: 0 },
          children: [new TextRun({ text: cellText, font: "Inter", size: 18 })]
        })]
      }))
    });
  });

  return new Table({
    width: { size: DATA_TABLE_WIDTH, type: WidthType.DXA },
    columnWidths: headers.map(h => h.width),
    layout: TableLayoutType.FIXED,
    borders: { top: nil, bottom: nil, left: nil, right: nil, insideH: nil, insideV: nil },
    rows: [headerRow, ...dataRows],
  });
}

// Uso:
// dataTable(
//   [
//     { text: "Riesgo", width: 4485 },
//     { text: "Impacto", width: 1170 },
//     { text: "Prob.", width: 945 },
//     { text: "Evidencia / Nota", width: 4110 },
//   ],
//   [
//     ["Regresiones no detectadas", "Alto", "Alta", "Sin suite de testing"],
//     ["Transacciones distribuidas", "Crítico", "Alta", "Sagas obligatorias"],
//     ["Complejidad operativa", "Crítico", "Alta", "N servicios + tracing"],
//     ["Timeouts / OOM en reportes", "Alto", "Media", "Exportaciones sin paginación"],
//   ]
// )
```

---

## Viñetas

Tres niveles bajo la misma reference `"bullets"`. Los bullets van siempre en el body del documento, NUNCA dentro de celdas de tabla.

| Level | Caracter | left | hanging | before | line |
|---|---|---|---|---|---|
| 0 | ● | 720 | 360 | — | — | (reservado, no usado en plantilla) |
| 1 | ○ | 425 | 285 | 200 | 276 | ítem principal |
| 2 | ■ | 850 | 315 | 0   | 276 | sub-ítem |

```javascript
// Numbering config
{ reference: "bullets", levels: [
  { level: 0, format: LevelFormat.BULLET, text: "●", alignment: AlignmentType.LEFT,
    style: { paragraph: { indent: { left: 720, hanging: 360 } } } },
  { level: 1, format: LevelFormat.BULLET, text: "○", alignment: AlignmentType.LEFT,
    style: { paragraph: { indent: { left: 425, hanging: 285 } } } },
  { level: 2, format: LevelFormat.BULLET, text: "■", alignment: AlignmentType.LEFT,
    style: { paragraph: { indent: { left: 850, hanging: 315 } } } },
]}

// Helper bullet
function bullet(text, opts={}) {
  const { bold=false, level=1 } = opts;
  const isMain = level === 1;
  return new Paragraph({
    numbering: { reference: "bullets", level },
    alignment: AlignmentType.BOTH,
    indent:  isMain ? { left: 425, hanging: 285 } : { left: 850, hanging: 315 },
    spacing: isMain
      ? { before: 200, after: 0, line: 276, lineRule: "auto" }
      : { before: 0,   after: 0, line: 276, lineRule: "auto" },
    children: [new TextRun({ text, font: "Inter", size: 20, bold })],
  });
}
```

---

## Header (logos)

Ambos logos son imágenes **ancladas flotantes** (`behindDocument: true`, wrap NONE).
Las posiciones y dimensiones se leen de `config.json`.

```javascript
// Leer config.json para obtener posiciones de logos
const config = JSON.parse(fs.readFileSync("config.json", "utf8"));

new Header({ children: [new Paragraph({ children: [
  // Logo izquierdo (config.logos.left)
  new ImageRun({ data: logoLeft, type: "png",
    transformation: { width: config.logos.left.width, height: config.logos.left.height },
    floating: {
      horizontalPosition: { relative: HorizontalPositionRelativeFrom.COLUMN,   offset: config.logos.left.offsetX },
      verticalPosition:   { relative: VerticalPositionRelativeFrom.PARAGRAPH,  offset: config.logos.left.offsetY },
      behindDocument: true, allowOverlap: true, wrap: { type: TextWrappingType.NONE },
    }}),
  // Logo derecho (config.logos.right)
  new ImageRun({ data: logoRight, type: "png",
    transformation: { width: config.logos.right.width, height: config.logos.right.height },
    floating: {
      horizontalPosition: { relative: HorizontalPositionRelativeFrom.COLUMN,   offset: config.logos.right.offsetX },
      verticalPosition:   { relative: VerticalPositionRelativeFrom.PARAGRAPH,  offset: config.logos.right.offsetY },
      behindDocument: true, allowOverlap: true, wrap: { type: TextWrappingType.NONE },
    }}),
]})]}
```

**Assets**: Configurar en `config.json` → `logos.left.file` y `logos.right.file`
**Personalizar**: Reemplazar PNGs en `assets/` y ajustar dimensiones/posiciones en `config.json`

---

## Reglas críticas

- **Tablas de sección (`seccionHeader`): SOLO 1 FILA** — nunca agregar fila de contenido. Fondo `#000000`.
- **Tablas de datos (`dataTable`)**: sí tienen múltiples filas (header + datos). Son la excepción.
- **Contenido y bullets: en el body**, nunca dentro de tabla (excepto dentro de `dataTable`)
- **Layout**: todas las tablas deben usar `TableLayoutType.FIXED`
- **Nunca** `ShadingType.SOLID` — siempre `ShadingType.CLEAR`
- **Nunca** bullets unicode en TextRun — siempre via numbering config
- **Página A4**: width=11909, height=16834 (no US Letter)
- **Separadores** entre bloques: `para("", { size: 10, spacing: { line: 240 } })`
- **Texto en filas de datos**: usar `sz=18` (9pt), no `sz=20` — solo los headers de tabla usan sz=20
- **Logos**: leer de `config.json`, nunca hardcodear posiciones en el código

---

## Personalización para otros clientes

Para adaptar la plantilla a otro cliente:

1. **Reemplazar logos**: copiar PNGs en `assets/` como `logo-left.png` y `logo-right.png`
2. **Ajustar dimensiones**: editar `config.json` → `logos.left.*` y `logos.right.*`
3. **Mantener colores**: los colores de la plantilla (negro, azul, azul claro) son fijos por diseño

Estructura esperada:
```
assets/
├── logo-left.png    ← logo izquierdo del header
├── logo-right.png   ← logo derecho del header
└── ...
```

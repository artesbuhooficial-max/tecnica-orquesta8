# Artes Búho · Producción Técnica — Orquesta 8

> Herramienta de producción técnica de un solo archivo HTML para gestionar el montaje de la Orquesta 8 de Artes Búho: stage plan, patch de audio, patch DMX, rider de inventario, producciones activas y un asistente de IA que lee riders y comprueba disponibilidad.

## Descripción

Esta aplicación es una herramienta interna de **Artes Búho** (empresa de eventos y música, Madrid/Murcia) pensada para el equipo técnico que monta el formato de orquesta de 8 músicos con mesa de iluminación **Avolites Tiger Touch** y mesa de audio **Behringer X32**.

Todo el proyecto vive en un único archivo HTML autocontenido (`AB_Produccion_Tecnica_Orquesta8_v4_1.html`), sin backend ni build step: se abre directamente en el navegador. Combina un editor visual de plano de escenario (arrastrar y cablear elementos en SVG), tablas editables de patch de audio y DMX, un rider de inventario real de la empresa, un panel de producciones con detección de choques de material entre fechas, y un asistente que usa la API de Claude para leer riders de clientes (imagen, PDF o texto) y compararlos contra el inventario disponible.

El estado completo de un montaje (plano, cables, patch, rider, producciones) se puede exportar/importar como JSON para reutilizar entre bolos.

## Características principales

- **Stage Plan interactivo (SVG)**: catálogo de elementos (músicos, PA, monitores, iluminación, estructura, corriente) que se arrastran al escenario y se cablean entre sí con tipos de cable codificados por color (DMX, XLR, CAT6/AES50, jack, sub-snake, corriente).
- **Patch Audio**: lista de canales de entrada para el X32, editable en tabla, con microfonía y tipo de pie sugeridos.
- **Patch DMX**: plan de universos/líneas para la Tiger Touch, con recálculo automático de direcciones y aviso si se supera el límite de 512 canales por universo.
- **Rider / Inventario**: catálogo real de material de Artes Búho por categorías, con estado disponible/consultar, editable manualmente o cargable desde un Excel (`.xlsx`/`.xls`/`.csv`) mediante SheetJS.
- **Producciones**: tarjetas de bolos activos (lugar, fecha, formato, estado, material asignado) con KPIs y **detección automática de choques de material** entre producciones en la misma fecha.
- **Asistente IA (Claude)**: sube una foto/PDF de un rider o pega texto y la app llama a la API de Claude para (a) reescribir el patch de audio como lo haría un técnico de FOH, o (b) comprobar la disponibilidad de todo el rider contra el inventario, marcando qué falta o está parcial.
- **Import/Export JSON** del estado completo del proyecto, e impresión con estilos dedicados (`@media print`).

## Tecnologías utilizadas

| Tecnología | Versión | Uso |
|---|---|---|
| HTML5 | - | Estructura, todo en un único archivo |
| CSS3 (custom, sin framework) | - | Estilos, variables CSS, tema oscuro |
| JavaScript | ES6+ (vanilla, sin frameworks) | Toda la lógica de estado y renderizado |
| SVG | - | Editor de plano de escenario y cableado |
| [SheetJS (xlsx)](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js) | 0.18.5 (CDN) | Lectura de inventario desde Excel/CSV |
| Google Fonts | - | `Barlow Condensed`, `IBM Plex Mono` |
| [Anthropic API](https://api.anthropic.com/v1/messages) (Claude) | `claude-sonnet-4-6` | Lectura de riders (imagen/PDF/texto) y comprobación de disponibilidad |

## Estructura del proyecto

```
TECNICA COMPLETA ARTES BUHO/
├── AB_Produccion_Tecnica_Orquesta8_v4_1.html   # Aplicación completa (HTML + CSS + JS)
└── README.md                                    # Este documento
```

No hay carpetas de assets, `node_modules` ni build: es una SPA de archivo único.

## Cómo usar

### Requisitos previos

- Un navegador moderno (Chrome, Edge, Firefox) con soporte de SVG, `fetch` y `localStorage`.
- Conexión a internet para cargar las fuentes de Google, SheetJS desde CDN y (opcionalmente) llamar a la API de Claude.
- Para el Asistente IA: una clave de API de Anthropic (`sk-ant-...`) o un webhook propio (p. ej. n8n) que la reciba y la reenvíe.

### Instalación / Apertura

No requiere instalación. Basta con abrir el archivo `AB_Produccion_Tecnica_Orquesta8_v4_1.html` directamente en el navegador (doble clic o `Archivo → Abrir`).

### Uso

1. **Stage Plan**: añade elementos desde el catálogo de la izquierda, arrástralos a su posición, cambia a modo "Cablear" para conectar elementos con el tipo de cable adecuado.
2. **Patch Audio / Patch DMX**: edita las tablas directamente; en DMX puedes recalcular direcciones automáticamente.
3. **Rider**: revisa/edita el inventario o carga un Excel con vuestro inventario actualizado.
4. **Producciones**: da de alta cada bolo, asigna material y revisa los avisos de choque de material entre fechas.
5. **Asistente IA**: en la pestaña correspondiente, configura la clave de API de Anthropic (o el webhook), sube el rider del cliente (imagen/PDF/texto) y pulsa "Analizar" para adecuar el patch de audio o comprobar disponibilidad contra el rider/inventario.
6. Usa **⬇ JSON** para guardar el estado del montaje y **⬆ Cargar** para recuperarlo en otra sesión o bolo similar.

## Funcionalidades en detalle

### Editor de escenario (SVG)

`DEFS` define cada tipo de elemento (forma, color, tamaño, categoría) y `CABLE_TYPES` define los tipos de cable con su color/capa. El estado vive en `state.items` y `state.cables`; `render()` reconstruye el SVG completo en cada cambio, incluyendo un ruteo ortogonal con "carriles" (`cableGeom`) para que los cables no se superpongan visualmente. Las capas (Músicos, Sonido, Monitores, Iluminación, Corriente) se pueden ocultar/mostrar de forma independiente.

### Patch DMX

`recalcAddresses()` reasigna direcciones DMX correlativas por línea según canales × unidades de cada fixture, y `renderPatch()` calcula el porcentaje de uso de cada universo (advertencia visual si se supera 512 canales).

### Detección de choques de material (Producciones)

`prodConflicts()` agrupa producciones activas por fecha, suma la cantidad de cada material pedido (usando `invMatch()` para emparejar nombres del rider por similitud de tokens normalizados) y genera un conflicto si la demanda total supera el stock disponible, o si el material no tiene equivalencia clara en el inventario.

### Asistente IA

`runAI()` construye un prompt específico según la tarea elegida (`promptPatch` o `promptDisp`), adjunta los ficheros como base64 (`toB64`) y llama directamente a `https://api.anthropic.com/v1/messages` (modo "directo", con la clave guardada solo en `localStorage` del navegador) o a un webhook propio (modo "n8n", recomendado para uso compartido en equipo para no exponer la clave en cada navegador). La respuesta se parsea como JSON tolerando truncamientos (`extractJSON` / `repairTruncatedJSON`) y se puede aplicar directamente al Patch Audio o volcar a una nueva producción.

## Configuración

- **Modelo de IA**: `claude-sonnet-4-6`, `max_tokens: 8192` (constante en `runAI()`).
- **Preset inicial**: `buildPreset()` genera el montaje de ejemplo completo (escenario, microfonía de batería, patch de audio de 20 canales, patch DMX, rider e inventario, dos producciones de ejemplo) y se ejecuta al cargar la página. El botón **↺ Preset** lo restaura.
- **Configuración del Asistente IA** (`aiCfg`: modo, clave, webhook) se persiste en `localStorage` bajo la clave `ab_aicfg`.

## APIs y dependencias externas

- `https://fonts.googleapis.com` — Tipografías Barlow Condensed e IBM Plex Mono.
- `https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js` — Lectura de Excel/CSV para el inventario.
- `https://api.anthropic.com/v1/messages` — Análisis de riders con Claude (modo API directa).
- Webhook configurable por el usuario (p. ej. n8n) como alternativa a la API directa.

## Notas para desarrollo / Para otras IAs

- **Arquitectura**: todo el estado de la app vive en el objeto global `state` (`items`, `cables`, `patch`, `audio`, `rider`, `prods`, `seq`). No hay framework: cada sección tiene su propia función `render*()` que regenera el `innerHTML` correspondiente a partir de `state`, y los inputs usan `data-*` attributes para mapear de vuelta al índice del array en `state` al disparar `onchange`.
- **Sin persistencia de servidor**: no hay backend. Todo vive en memoria del navegador salvo la configuración del Asistente IA (`localStorage.ab_aicfg`) y lo que el usuario exporte/importe manualmente como JSON (`exportJSON`/`importJSON`).
- **Seguridad de la clave de API**: la clave de Anthropic, si se usa el modo "directo", se guarda solo en el `localStorage` del navegador de cada usuario y se envía con la cabecera `anthropic-dangerous-direct-browser-access: true` directamente desde el cliente. Para uso compartido en equipo (varios técnicos, un solo repositorio de clave) se recomienda el modo webhook/n8n para no distribuir la clave real. **El HTML no contiene ninguna clave hardcodeada.**
- **Escapado**: todo el contenido dinámico insertado en el DOM pasa por `esc()` (escape básico de `&<>"`) antes de interpolarse en `innerHTML`, para mitigar XSS desde datos de rider/inventario.
- **Matching de inventario** (`invMatch`, `norm`): heurística simple de solapamiento de tokens normalizados (sin acentos, minúsculas), no es un matching semántico real; nombres muy distintos entre el pedido del cliente y el inventario pueden no encontrar coincidencia.
- **Limitaciones conocidas**: sin tests automatizados; sin control de versiones de datos (importar un JSON antiguo sobrescribe todo el estado sin aviso de conflicto); el parseo de Excel (`loadXlsx`) usa heurísticas de texto (`disponible`/`consultar`) que dependen del formato exacto del inventario de Artes Búho.
- **Idioma**: interfaz y datos en español (España), pensada para el equipo técnico de Artes Búho.

## Estado del proyecto

- [x] Editor de plano de escenario con cableado
- [x] Patch de audio y DMX editables
- [x] Rider/inventario con import de Excel
- [x] Gestión de producciones con detección de choques
- [x] Asistente IA (Claude) para lectura de riders y patch automático
- [ ] Persistencia en servidor / multiusuario (actualmente todo es local al navegador)
- [ ] Tests automatizados
- [ ] Matching de inventario más robusto (actualmente heurístico por tokens)

## Autor

Artes Búho — equipo técnico de producción.

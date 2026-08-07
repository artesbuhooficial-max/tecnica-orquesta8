# Control de Material — Artes Búho

Checklist de control de material técnico para eventos: qué se lleva cada persona, cantidad, y doble check de **retirada** y **devolución**. Pensado para montaje/desmontaje de Noches de Neón, Maquillaje Rock, Bella Bestia y cualquier bolo suelto — incluidos días con **varios bolos a la vez**.

Es un único archivo HTML autocontenido (HTML + CSS + JS, sin frameworks, sin build). Se abre directamente en el navegador o se aloja en GitHub Pages.

## Varias producciones, un solo enlace

La herramienta soporta **N producciones en paralelo** (por ejemplo, dos bolos el mismo día): arriba del todo hay una fila de pestañas, una por producción, con un botón **+ Producción** para añadir otra. Cada pestaña muestra el número de ítems **"en calle"** (retirado y aún no devuelto) para vigilar de un vistazo si algo se queda sin volver.

Cada producción tiene su propio nombre, lugar/detalle, fecha, responsable general y lista de material — completamente independientes entre sí. Al pulsar "☁ Guardar en GitHub (compartido)" se sube **todo** (todas las producciones a la vez) al mismo archivo, así que cualquiera que abra el mismo enlace (el ordenador, el móvil, o el de un asistente al que se lo pases) ve automáticamente todas las producciones del día y puede moverse entre pestañas para marcar solo la que le toca.

Se puede eliminar una producción con "🗑 Eliminar esta producción" (tiene que quedar al menos una).

## Qué hace (por producción)

- Categorías: Pies y soportes, Micrófonía, DI, PA, Monitores, Cableado y alimentación, Mesa de sonido, Extras recomendados.
- Cada ítem: nombre, cantidad, persona responsable, notas, y dos botones — **Marcar retirada** / **Marcar devuelta** (devuelta solo se activa si ya está retirada).
- Sello visual (dorado = retirado, cobalto = devuelto) para ver de un vistazo qué falta.
- Panel resumen: Total / Retirado / Devuelto / **En calle** (retirado y aún no devuelto — el número a vigilar al final del bolo).
- Editable en caliente: añadir o borrar material, añadir o borrar categorías enteras, renombrar todo.
- **Reiniciar checks**: pone a cero los ticks de la producción activa para el siguiente evento sin perder su lista de material.
- **Restaurar lista original**: vuelve a la plantilla base la producción activa si lías algo.
- **Exportar / Importar copia (JSON)**: exporta **todas** las producciones en un `.json`. Al importar, si el archivo trae varias producciones sustituye todo; si es un export antiguo de una sola producción, se importa solo dentro de la producción activa.

## Persistencia de datos

Cada edición (marcar retirada/devuelta, cambiar cantidades, etc.) se guarda al instante en `localStorage` del navegador — por dispositivo, automático, sin pulsar nada.

Para compartir ese estado entre dispositivos/personas hay dos vías:
- **Manual**: Exportar/Importar copia (JSON).
- **Compartida en tiempo casi real**: botón "☁ Guardar en GitHub (compartido)" — sube el estado a `data/control-material.json` en el repo de GitHub; cualquiera que abra la página después (aunque sea desde otro móvil) lo carga automáticamente al inicio. No es instantáneo entre dispositivos (hay que guardar y que el otro recargue la página), pero no requiere backend propio.

## Cómo subirlo a GitHub con Claude Code

Desde el ordenador, en la carpeta del proyecto:

```bash
mkdir control-material-artesbuho
cd control-material-artesbuho
# copia aquí control-material-artesbuho.html y README.md
mv control-material-artesbuho.html index.html   # opcional, para que GitHub Pages lo sirva como raíz

git init
git add .
git commit -m "Control de material técnico — Artes Búho"

# crea el repo en GitHub (con gh CLI, si lo tienes instalado)
gh repo create artesbuho/control-material --public --source=. --remote=origin --push

# o si prefieres hacerlo manual:
git remote add origin https://github.com/artesbuho/control-material.git
git branch -M main
git push -u origin main
```

Luego, en GitHub → **Settings → Pages** → Source: rama `main`, carpeta `/ (root)`. En un par de minutos queda publicado en algo como:
`https://artesbuho.github.io/control-material/`

Si prefieres mantenerlo dentro de uno de tus repos ya existentes (por ejemplo, el del dashboard de gestión técnica), mételo en una subcarpeta tipo `/control-material/index.html` en vez de crear repo nuevo — GitHub Pages sirve subcarpetas igual.

## Conexión con el dashboard de Producción Técnica (orquesta)

Este archivo vive como herramienta independiente (no se ha fusionado con el HTML grande, para evitar choques de CSS/JS), pero **está conectado por datos** a través del mismo repositorio de GitHub:

- **Enlace directo**: la cabecera tiene un enlace "← Producción Técnica" al dashboard grande, y el dashboard grande tiene un botón "📋 Control Material" que abre este archivo.
- **"📦 Cargar material desde Producción Técnica"**: lee en directo `data/estado.json` del repo `artesbuhooficial-max/tecnica-orquesta8` (el mismo archivo que genera el botón "☁ Guardar en GitHub" del dashboard grande) y deja elegir entre:
  - importar **todo el rider/inventario** completo como categorías, o
  - importar **el material ya asignado a una producción concreta** (una sola categoría con exactamente lo que esa producción tiene reservado), rellenando también nombre, lugar y fecha automáticamente.

  Esto sustituye el material de la **producción activa** (la pestaña seleccionada arriba, con confirmación previa) — el resto de pestañas/producciones no se tocan. Así, con dos bolos el mismo día, cada pestaña se puede refrescar contra el inventario real de forma independiente.
- **"☁ Guardar en GitHub (compartido)"**: en vez de (o además de) `localStorage` por dispositivo, este botón escribe el estado completo del checklist —**todas las producciones/pestañas**— (`data/control-material.json`) directamente en el mismo repositorio, vía la API de contenidos de GitHub. Al abrir la página, intenta cargar automáticamente esa copia compartida — así Miriam, Manu o Samu (o un asistente al que le pases el mismo enlace) pueden ver desde su propio móvil el mismo estado de qué se ha retirado y devuelto en cada producción, sin necesidad de backend propio (Google Sheets/Firebase ya no hace falta para este caso).
- Configuración (repo/rama/ruta/token) en el botón "⚙ Configurar GitHub": el token necesita permiso únicamente de **"Contents: Read and write"**, limitado a este repositorio, y se guarda solo en `localStorage` del navegador — nunca se sube al repo. La lectura (ver el checklist compartido, o cargar el inventario) no necesita token porque el repo es público; el token solo hace falta para **guardar**.

## Branding usado

| Token | Valor | Uso |
|---|---|---|
| `--cream` | `#F6F1E4` | fondo general |
| `--paper` | `#FCFAF3` | tarjetas |
| `--cobalt` | `#1C3F8F` | acento primario, botón "devuelta" |
| `--cobalt-deep` | `#122A63` | cabeceras de categoría |
| `--gold` | `#B8862E` | acento secundario, botón "retirada" |
| `--line` | `#D8CBA8` | bordes |

Tipografía: **Playfair Display** (títulos, vía Google Fonts) + system-ui/sans (datos y campos).

## Estructura de archivos

```
control-material-artesbuho/
├── index.html   (el archivo control-material-artesbuho.html renombrado)
└── README.md
```

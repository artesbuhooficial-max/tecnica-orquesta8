# Control de Material — Artes Búho

Checklist de control de material técnico para eventos: qué se lleva cada persona, cantidad, y doble check de **retirada** y **devolución**. Pensado para montaje/desmontaje de Noches de Neón, Bella Bestia y cualquier bolo suelto.

Es un único archivo HTML autocontenido (HTML + CSS + JS, sin frameworks, sin build). Se abre directamente en el navegador o se aloja en GitHub Pages.

## Qué hace

- Categorías: Pies y soportes, Micrófonía, DI, PA, Monitores, Cableado y alimentación, Mesa de sonido, Extras recomendados.
- Cada ítem: nombre, cantidad, persona responsable, notas, y dos botones — **Marcar retirada** / **Marcar devuelta** (devuelta solo se activa si ya está retirada).
- Sello visual (dorado = retirado, cobalto = devuelto) para ver de un vistazo qué falta.
- Panel resumen: Total / Retirado / Devuelto / **En calle** (retirado y aún no devuelto — el número a vigilar al final del bolo).
- Editable en caliente: añadir o borrar material, añadir o borrar categorías enteras, renombrar todo.
- **Reiniciar checks**: pone todos los ticks a cero para el siguiente evento sin perder la lista de material.
- **Restaurar lista original**: vuelve a la plantilla base si lías algo.
- **Exportar / Importar copia (JSON)**: descarga un `.json` con el estado completo. Útil para pasar el estado entre el ordenador y el móvil, o guardar un histórico por evento.

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
  - importar **todo el rider/inventario** completo como categorías de este checklist, o
  - importar **el material ya asignado a una producción concreta** (una sola categoría con exactamente lo que esa producción tiene reservado), rellenando también Evento y Fecha automáticamente.
  
  Esto sustituye la lista de material actual (con confirmación previa), así que el checklist de salida/entrada siempre puede refrescarse contra el inventario real en vez de mantenerse a mano por separado.
- **"☁ Guardar en GitHub (compartido)"**: en vez de (o además de) `localStorage` por dispositivo, este botón escribe el estado del checklist (`data/control-material.json`) directamente en el mismo repositorio, vía la API de contenidos de GitHub. Al abrir la página, intenta cargar automáticamente esa copia compartida — así Miriam, Manu o Samu pueden ver desde su propio móvil el mismo estado de qué se ha retirado y devuelto, sin necesidad de backend propio (Google Sheets/Firebase ya no hace falta para este caso).
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

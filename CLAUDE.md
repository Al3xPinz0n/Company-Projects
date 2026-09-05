# SMART Hub · APM Terminals Mobile

Este archivo se lee automáticamente al abrir este repositorio con Claude Code. Contiene el contexto completo del proyecto para que puedas continuar el trabajo sin perder lo ya avanzado.

## Resumen del proyecto

- **Nombre del portal:** **SMART Hub** (APM Terminals Mobile · Operational Intelligence Hub).
- **Propósito:** Portal centralizado estático (*client-side hub*) para alojar micro-herramientas operativas, simuladores y generadores de reportes para las operaciones de APM Terminals Mobile (Alabama), organizadas por **área operativa**.
- **Arquitectura de origen:** Inspirado en la estructura base de *BrainPort*; el contenido y las herramientas de ejemplo que traía esa base ya se eliminaron — el hub arranca vacío y cada área se llena a medida que se construye una herramienta real para APM Terminals Mobile.
- **Dueño:** Alex Pinzon (Operations / Strategy Team).

## Idioma: inglés siempre

**Todo el contenido visible de la plataforma (`index.html` y cada herramienta en `tools/`) debe estar en inglés americano profesional, con terminología de puertos y terminales de contenedores** (vessel, berth, yard, gate, rail, stowage, gate-in/gate-out, prime mover, etc.). No usar español en texto de UI, aunque el usuario escriba las instrucciones en español — traducir el contenido, no copiarlo literal. Comentarios de código y este `CLAUDE.md` sí pueden quedar en español (documentación interna para trabajar con Claude Code).

## Estado actual (confirmado y funcionando)

- **Repositorio:** `Al3xPinz0n/smarthub` en GitHub — **público** (necesario para GitHub Pages gratis). Renombrado desde `Company-Projects` (sep. 2026); el remoto local ya apunta a la nueva URL.
- **Sitio publicado:** `https://al3xpinz0n.github.io/smarthub/`.
- **Despliegue:** GitHub Pages desde `Settings > Pages`, origen `Deploy from a branch`, rama `main`, directorio `/(root)`.
- **Repo local:** carpeta `C:\Users\ADP037\Documents\GitHub\Company-Projects` (el nombre de la carpeta local quedó igual; solo cambió el repo remoto) — GitHub Desktop ya autenticado con la cuenta del usuario.
- **Flujo de publicación:** cualquier cambio en `main` se refleja automáticamente en el sitio (1-2 minutos) al hacer push — el usuario usa GitHub Desktop (Commit → Push) para esto, sin línea de comandos.
- **El hub arrancó vacío y se va llenando por herramienta real.** `TOOLS` en `index.html` ya no está vacío: **Strategy** tiene dos herramientas (`Yard Strategy` y `Export & Empty Analysis`) y **Vessel** tiene una (`Vessel Move Reconciliation`), ver más abajo. Yard, Gate, Rail y VAS siguen mostrando su estado vacío ("No tools have been added...") hasta que se agregue su primera herramienta real.

## Diseño y marca

El hub sigue el skill `apm-terminals-brand` (colores, tipografía, logo e íconos oficiales de APM Terminals). **Revisar ese skill antes de tocar cualquier estilo o de construir una herramienta nueva.**

- **Tipografía:** Maersk Text (Light/Regular/Bold), cargada vía `@font-face` desde `assets/fonts/*.ttf`. Fallback: Arial.
- **Colores:** paleta oficial APM en `:root` de `index.html` — Orange `#FF6441`, Dark Gray `#3C3C46`, Aqua Green `#0A6E82`, Maersk Blue `#42B0D5`, Soft Yellow `#F0BE78`, grises `#5A6E7D` / `#AAB4C3` / `#E6EBF2`.
- **Logo:** SVG oficial embebido directamente en el `<header>` de `index.html`, a **236px de ancho** (2x el tamaño original — pedido explícito del usuario). No recrear, recolorear ni reducir el logo por debajo de esto sin que el usuario lo pida.
- **Íconos de área:** pictogramas oficiales de la familia `pictograms-orange.svg` del skill (tono naranja único, sin recoloreo), embebidos como sprite `<symbol>` al inicio de `index.html`. En la grilla de áreas se muestran a **85px** dentro de un contenedor de **150px** (2.5x el tamaño original — pedido explícito del usuario). Reutilizar ese mismo sprite para íconos nuevos en vez de emojis o íconos genéricos.

## Estructura de navegación: áreas operativas

El hub usa 6 tarjetas de **área operativa** en la página de inicio (no pestañas, no el término "gaveta" — ese fue solo un concepto usado para explicar la idea durante el diseño, nunca debe aparecer en el producto ni en el código). Cada desarrollo nuevo se agrega al área que le corresponda (arreglo `TOOLS` dentro de `index.html`, cada entrada con su `area` matching una de estas keys):

| Área     | Alcance |
|----------|---------|
| Strategy | Operational planning, performance targets, and ABS reporting for the Terminal |
| Vessel   | Vessels at berth, stowage planning, crane productivity |
| Yard     | Yard status, prime mover flow, operator performance |
| Gate     | Appointments, gate-in/gate-out processing, truck flow |
| Rail     | Rail operations scheduling, railcar movements |
| VAS      | Value-added services (stuffing, stripping, etc.) |

**Strategy** ya tiene sus dos primeras herramientas reales (construidas fuera de este repo y luego incorporadas):

| Herramienta | Carpeta | Qué hace |
|---|---|---|
| **Yard Strategy** | `tools/strategy/yard-strategy/` | Mapa interactivo de bloques del patio para definir y ajustar la estrategia de yard — clasificar bloques, asignar servicios/líneas y revisar capacidad por clase. Guarda su estado en `localStorage` (nada sale del navegador). Tiene su propio candado interno adicional para la pestaña "Configuration" (contraseña embebida en el JS del archivo — es de ese archivo, no del gate de SMART Hub; no lo confundas con `AREA_HASHES`). |
| **Export & Empty Analysis** | `tools/strategy/export-empty-analysis/` | El usuario sube dos Excel (lista de movimientos de carga + detalle de visitas de buque) y la herramienta calcula TEUs de exportación por POD/servicio, perfil de peso y TEUs vacíos por línea, usando SheetJS 100% en el navegador. |

**Vessel** tiene su primera herramienta:

| Herramienta | Carpeta | Qué hace |
|---|---|---|
| **Vessel Move Reconciliation** | `tools/vessel/vessel-move-reconciliation/` | El usuario sube la lista de movimientos de contenedores de una escala de buque (xlsx/xls/csv) y la herramienta concilia discharge, load y restow por categoría de contenedor (Dry, Reefer, Tank, Flat Rack, Pallet Wide) y tamaño, desglosado por naviera. Usa SheetJS, 100% en el navegador. ⚠️ Esta herramienta usa la tipografía Google Fonts "Plus Jakarta Sans" y valores de color aproximados en vez de Maersk Text / la paleta exacta del skill `apm-terminals-brand` — se incorporó tal cual porque ya funcionaba y el usuario solo pidió agregarla; si en algún momento se quiere alinear 100% a la marca, es un ajuste pendiente. |

Yard, Gate, Rail y VAS siguen vacías — cada una debe mostrar su estado vacío hasta que se agregue su primera herramienta real.

### Control de acceso por área

Cada área requiere contraseña para entrar:

- **Strategy** usa una contraseña propia y distinta al resto: **`Mobile.Strategy`**.
- Todas las demás áreas usan el patrón **`ABS.` + nombre del área** (p. ej. `ABS.Vessel`, `ABS.Gate`, `ABS.Yard`, `ABS.Rail`, `ABS.VAS`).

En `index.html`, `AREA_HASHES` guarda el **SHA-256** de cada contraseña (nunca el texto plano) y el gate la valida con `crypto.subtle.digest` antes de desbloquear esa área en `sessionStorage` para esa pestaña del navegador.

⚠️ **Esto es un control de acceso liviano, no seguridad real.** El sitio es 100% estático y el repo es público — cualquiera con el link puede leer el HTML/JS fuente (aunque no el texto plano de la contraseña, sí el esquema `ABS.<Área>` / `Mobile.Strategy`, fácil de adivinar). Sirve para mantener a visitantes casuales fuera de áreas que no les corresponden, **no** para proteger datos sensibles. Por eso la regla de la siguiente sección sigue siendo innegociable: nunca poner datos operativos reales detrás de este gate — solo herramientas que procesan archivos que el propio usuario carga en su navegador.

## Arquitectura técnica y seguridad (no negociable)

- **100% client-side, cero backend.** No hay servidores, bases de datos ni APIs recibiendo datos de la terminal.
- Los archivos que carga el usuario en cada herramienta (`.xlsx`, `.csv`, `.txt`) se procesan **exclusivamente en memoria del navegador**. Los datos operativos de la terminal nunca deben viajar por internet ni guardarse en GitHub.
- **Nunca** subir al repo: API keys, contraseñas en texto plano, credenciales, tokens, ni archivos de datos reales/hardcodeados.
- Como el repo es público, cualquier cosa que se suba es visible para todo el mundo — extremar cuidado con lo anterior.

## Librerías JS sugeridas (para las herramientas nuevas)

| Librería | Función | Caso de uso |
|---|---|---|
| **SheetJS** (`xlsx.full.min.js`) | Leer/parsear Excel/CSV | Reportes N4/XPS, listas de descarga, planes de patio |
| **jsPDF** / **html2pdf.js** | Generar PDF | Entregables de turnos y handovers |
| **Chart.js** | Gráficas | Tendencias de CMPH, tiempos muertos, productividad |
| **JSZip** | Comprimir archivos | Descarga masiva de reportes en un `.zip` |

## Estructura de directorios objetivo

```text
smarthub/                            <-- repo GitHub (carpeta local sigue llamándose Company-Projects)
├── index.html                       <-- Hub principal (SMART Hub, ya existe)
├── README.md
├── CLAUDE.md                        <-- este archivo
├── assets/
│   └── fonts/                       <-- Maersk Text (.ttf) usadas por index.html
└── tools/
    ├── strategy/
    │   ├── yard-strategy/index.html
    │   └── export-empty-analysis/index.html
    ├── vessel/
    │   └── vessel-move-reconciliation/index.html
    ├── yard/<nombre>/index.html
    ├── gate/<nombre>/index.html
    ├── rail/<nombre>/index.html
    └── vas/<nombre>/index.html
```

Cada herramienta nueva va en `tools/<área>/<nombre>/index.html`, autocontenida (HTML+CSS+JS en un solo archivo, siguiendo el mismo patrón que `index.html`), con **todo su texto en inglés** (ver "Idioma: inglés siempre") y debe mantener consistencia visual con el hub principal (revisar `index.html` y el skill `apm-terminals-brand` antes de diseñar cualquier UI nueva). Al terminarla, agregar su entrada al arreglo `TOOLS` de `index.html` con el `area` correcto y el `href` apuntando a esa carpeta.

## Cómo trabajar en este repo

- Cada herramienta es un archivo HTML autocontenido (sin build step, sin frameworks pesados) — coherente con el enfoque 100% estático del hub.
- Antes de crear una herramienta nueva, revisar `index.html` para mantener el mismo lenguaje visual (tarjetas, áreas operativas, paleta de colores APM, tipografía Maersk Text) y el mismo idioma (inglés).
- El usuario revisa y publica los cambios con GitHub Desktop (Commit to main → Push origin) — no asumir acceso a git ni terminal salvo que el propio Claude Code lo tenga disponible en esta máquina.
- Existen otros dos documentos de referencia en la raíz del repo: `GUIA-PORTAL-HARDCODEADO.md` y `SETUP-VSCODE-GITHUB-CLAUDE.md` (este último es un runbook de instalación de entorno más amplio, orientado a un stack PHP/Laravel que **no aplica** a este proyecto — ignorar esa parte salvo que el usuario indique lo contrario).

## Próximo paso acordado

Seguir construyendo herramientas reales por área (Yard, Gate, Rail, VAS todavía no tienen ninguna) y agregar su entrada a `TOOLS` en `index.html` con el `area` correcto al terminarlas. Cuando una herramienta ya exista como archivo HTML fuera del repo (como pasó con las tres ya incorporadas), copiarla a `tools/<área>/<nombre>/index.html` tal cual si ya sigue la marca APM (Maersk Text, paleta oficial, inglés) — no reescribirla sin necesidad. Si no sigue la marca al 100% (caso de `vessel-move-reconciliation`, ver arriba), incorporarla igual tal cual salvo que el usuario pida explícitamente alinearla.

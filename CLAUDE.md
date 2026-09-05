# SMART Hub · APM Terminals Mobile

Este archivo se lee automáticamente al abrir este repositorio con Claude Code. Contiene el contexto completo del proyecto para que puedas continuar el trabajo sin perder lo ya avanzado.

## Resumen del proyecto

- **Nombre del portal:** **SMART Hub** (APM Terminals Mobile · Operational Intelligence Hub).
- **Propósito:** Portal centralizado estático (*client-side hub*) para alojar micro-herramientas operativas, simuladores y generadores de reportes para las operaciones de APM Terminals Mobile (Alabama), organizadas por **gaveta** (área operativa).
- **Arquitectura de origen:** Inspirado en la estructura base de *BrainPort*, adaptado a los KPIs y métricas de APM Terminals Mobile.
- **Dueño:** Alex Pinzon (Operations).

## Estado actual (confirmado y funcionando)

- **Repositorio:** `Al3xPinz0n/Company-Projects` en GitHub — **público** (necesario para GitHub Pages gratis).
- **Sitio publicado:** `https://al3xpinz0n.github.io/Company-Projects/`.
- **Despliegue:** GitHub Pages desde `Settings > Pages`, origen `Deploy from a branch`, rama `main`, directorio `/(root)`.
- **Repo local:** clonado en `C:\Users\ADP037\Documents\GitHub\Company-Projects` vía GitHub Desktop (ya autenticado con la cuenta de GitHub del usuario).
- **Flujo de publicación:** cualquier cambio en `main` se refleja automáticamente en el sitio (1-2 minutos) al hacer push — el usuario usa GitHub Desktop (Commit → Push) para esto, sin línea de comandos.
- **Rediseño (sep. 2026):** el hub dejó el modelo de pestañas (Overview, AI Agents, Indicators…) y el indicador "LIVE" del header. La página de inicio ya **no** muestra vista previa de buques, productividad ni ningún otro indicador operativo — solo las 6 gavetas. "Continuous Improvement" se renombró a **ABS (APMT Business System)** en toda la plataforma.

## Diseño y marca

El hub sigue el skill `apm-terminals-brand` (colores, tipografía, logo e íconos oficiales de APM Terminals). **Revisar ese skill antes de tocar cualquier estilo o de construir una herramienta nueva.**

- **Tipografía:** Maersk Text (Light/Regular/Bold), cargada vía `@font-face` desde `assets/fonts/*.ttf`. Fallback: Arial.
- **Colores:** paleta oficial APM en `:root` de `index.html` — Orange `#FF6441`, Dark Gray `#3C3C46`, Aqua Green `#0A6E82`, Maersk Blue `#42B0D5`, Soft Yellow `#F0BE78`, grises `#5A6E7D` / `#AAB4C3` / `#E6EBF2`.
- **Logo:** SVG oficial (Secondary Logo, versión Negative A para fondo oscuro) embebido directamente en el `<header>` de `index.html`. No recrear ni recolorear el logo.
- **Íconos de gaveta:** pictogramas oficiales de la familia `pictograms-orange.svg` del skill (tono naranja único, sin recoloreo), embebidos como sprite `<symbol>` al inicio de `index.html` — reutilizar ese mismo sprite para íconos nuevos en vez de emojis o íconos genéricos.

## Estructura de navegación: gavetas

El hub ya **no** usa pestañas de sección. La página de inicio muestra 6 tarjetas ("gavetas"), una por área operativa. Cada desarrollo nuevo se agrega a la gaveta que le corresponda (lista de herramientas `MODULOS` dentro de `index.html`, filtrada por `section`):

| Gaveta   | Alcance |
|----------|---------|
| Strategy | Planeación operativa, metas y reportes del ABS (APMT Business System) |
| Vessel   | Buques en muelle, planificación de estiba, productividad de grúas |
| Yard     | Estado del patio, tractores, desempeño de operadores |
| Gate     | Citas, procesos de ingreso, flujo de camiones en puerta |
| Rail     | Programación y movimientos de la operación ferroviaria |
| VAS      | Servicios de valor agregado (stuffing, stripping, etc.) |

Gate, Rail y VAS todavía no tienen herramientas — muestran un estado vacío ("en desarrollo") hasta que se agregue la primera.

### Control de acceso por gaveta

Cada gaveta requiere contraseña para entrar: **`ABS.` + nombre de la gaveta** (p. ej. `ABS.Vessel`, `ABS.Gate`). En `index.html`, `GAVETA_HASHES` guarda el **SHA-256** de cada contraseña (nunca el texto plano) y el gate la valida con `crypto.subtle.digest` antes de desbloquear esa gaveta en `sessionStorage` para esa pestaña del navegador.

⚠️ **Esto es un control de acceso liviano, no seguridad real.** El sitio es 100% estático y el repo es público — cualquiera con el link puede leer el HTML/JS fuente (aunque no el texto plano de la contraseña, sí el esquema `ABS.<Gaveta>`, fácil de adivinar). Sirve para mantener a visitantes casuales fuera de gavetas que no les corresponden, **no** para proteger datos sensibles. Por eso la regla de la siguiente sección sigue siendo innegociable: nunca poner datos operativos reales detrás de este gate — solo herramientas que procesan archivos que el propio usuario carga en su navegador.

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
Company-Projects/
├── index.html                       <-- Hub principal (SMART Hub, ya existe)
├── README.md
├── CLAUDE.md                        <-- este archivo
├── assets/
│   └── fonts/                       <-- Maersk Text (.ttf) usadas por index.html
└── tools/
    ├── strategy/<nombre>/index.html
    ├── vessel/<nombre>/index.html
    ├── yard/<nombre>/index.html
    ├── gate/<nombre>/index.html
    ├── rail/<nombre>/index.html
    └── vas/<nombre>/index.html
```

Cada herramienta nueva va en `tools/<gaveta>/<nombre>/index.html`, autocontenida (HTML+CSS+JS en un solo archivo, siguiendo el mismo patrón que `index.html`), y debe mantener consistencia visual con el hub principal (revisar `index.html` y el skill `apm-terminals-brand` antes de diseñar cualquier UI nueva). Al terminarla, agregar su entrada al arreglo `MODULOS` de `index.html` con el `section` (gaveta) correcto y el `href` apuntando a esa carpeta.

## Cómo trabajar en este repo

- Cada herramienta es un archivo HTML autocontenido (sin build step, sin frameworks pesados) — coherente con el enfoque 100% estático del hub.
- Antes de crear una herramienta nueva, revisar `index.html` para mantener el mismo lenguaje visual (tarjetas, gavetas, paleta de colores APM, tipografía Maersk Text).
- El usuario revisa y publica los cambios con GitHub Desktop (Commit to main → Push origin) — no asumir acceso a git ni terminal salvo que el propio Claude Code lo tenga disponible en esta máquina.
- Existen otros dos documentos de referencia en la raíz del repo: `GUIA-PORTAL-HARDCODEADO.md` y `SETUP-VSCODE-GITHUB-CLAUDE.md` (este último es un runbook de instalación de entorno más amplio, orientado a un stack PHP/Laravel que **no aplica** a este proyecto — ignorar esa parte salvo que el usuario indique lo contrario).

## Próximo paso acordado

Construir la primera herramienta dentro de una gaveta (p. ej. un KPI Calculator en `tools/strategy/kpi-calculator/index.html` o `tools/vessel/...`, según a qué gaveta pertenezca) y apuntar su `href` en `MODULOS` desde `index.html`.

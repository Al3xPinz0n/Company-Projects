# Runbook de instalación: Claude + GitHub + VS Code + PHP/Laravel en Windows

**Destinatario: el agente (Claude) que corre en la máquina del usuario.**
Este documento es un runbook para ejecutar, no un texto para resumir. Sigue los
pasos en orden, verifica cada uno antes de pasar al siguiente, y reporta al
usuario lo que verificaste.

Salió de auditar una máquina donde este entorno ya funciona (APM Terminals
Balboa, 2026-09-03). Las versiones son reales, no ejemplos.

---

## 0. Reglas para el agente

**Antes de empezar, pregúntale al usuario estas tres cosas.** No las adivines:

1. **¿Tiene permisos de administrador en esta máquina?** Si **no**, el cuerpo del
   documento no sirve tal cual: usa el **Apéndice A**, que es la instalación
   portable dentro del perfil del usuario.
2. **Su nombre y correo** para la identidad de los commits.
3. **Su usuario de GitHub y el nombre del repositorio** que va a usar. Si todavía
   no existe, se crea en el paso 3.

**Reglas de ejecución:**

- **Un paso a la vez, con verificación.** Cada paso trae su comando de
  verificación. Si falla, resuélvelo antes de seguir; no acumules errores.
- **Trampa del PATH:** cuando un instalador agrega algo al PATH, **tu terminal
  actual no lo ve**. No concluyas que la instalación falló. O actualizas el PATH
  en la sesión (`$env:Path = [Environment]::GetEnvironmentVariable('Path','Machine') + ';' + [Environment]::GetEnvironmentVariable('Path','User')`), o invocas el ejecutable por ruta completa,
  o le pides al usuario que reinicie la terminal.
- **Los logins los hace el usuario, no tú.** Los pasos 2, 3 y 4 abren ventanas de
  navegador para autenticarse con Claude, GitHub y Microsoft. Dile al usuario que
  complete el login y espera su confirmación. **Nunca le pidas su contraseña ni la
  escribas en ningún lado.**
- **Ningún secreto en el repositorio.** Ni contraseñas, ni tokens, ni cadenas de
  conexión, ni archivos `.env`.
- **Si un ID de `winget` no resuelve**, búscalo (`winget search <nombre>`) en vez
  de inventar otro. El catálogo cambia entre versiones.
- **No instales los "extras" del paso 9** salvo que el usuario los pida.

**Stack asumido:** PHP 8.3 + Laravel + Node, que es el de la máquina de
referencia. Si el usuario dice que trabaja en otro lenguaje, los pasos 1 a 4, 7 y
8 valen igual; cambia solo el paso 5 y las extensiones de lenguaje del paso 4.

## 1. Orden de instalación y por qué

```
1. Git                  -> sin esto nada se autentica contra GitHub
2. Claude (app + CLI)   -> la extensión de VS Code lo usa por debajo
3. Cuenta de GitHub     -> el primer clone/push dispara el login
4. VS Code + extensiones
5. PHP, Composer, Node  -> el toolchain de Laravel
6. Proyecto Laravel
7. Playwright MCP       -> que Claude maneje un navegador
8. Plugins de Claude
9. Extras (opcional)
```

El error típico es empezar por VS Code: la extensión de Claude Code queda
instalada pero **muerta**, porque necesita el CLI del paso 2 en el PATH.

## 2. Versiones de referencia

No bajes de estas; subir está bien.

| Herramienta | Versión verificada |
|---|---|
| Git para Windows | 2.55.0.3 (x64) |
| Claude Code (CLI) | 2.1.248 |
| VS Code | 1.126.0 |
| Node.js | 24.19.0 |
| npm | 11.17.0 |
| PHP CLI (NTS x64, VC 2019+) | 8.3.32 |
| Composer | 2.x |
| Laravel | 12 |
| `@playwright/mcp` | 0.0.79 |

---

## PASO 1 — Git

```bash
winget install --id Git.Git -e
```

Deja habilitado **Git Credential Manager** (viene por defecto). Es lo que guarda
el login de GitHub sin que nadie escriba un token a mano.

Configura la identidad **antes del primer commit** — si no, los commits quedan mal
atribuidos y corregirlo después obliga a reescribir historia:

```bash
git config --global user.name "NOMBRE DEL USUARIO"
```

```bash
git config --global user.email "CORREO DEL USUARIO"
```

```bash
git config --global core.autocrlf false
```

**Verificar:**

```bash
git --version && git config --global user.email
```

## PASO 2 — Claude

Son **dos componentes** y hacen falta los dos:

- **App de escritorio**: descarga desde `https://claude.ai/download`. Es la
  ventana con el sidebar de conversaciones.
- **CLI** (`claude`): queda en `%USERPROFILE%\.local\bin\claude`. Es lo que usa la
  extensión de VS Code.

Instala la app primero y **pídele al usuario que inicie sesión ahí**; el CLI
hereda esa sesión.

**Verificar:**

```bash
claude --version
```

Notas:

- La configuración vive en `%USERPROFILE%\.claude\` y `%USERPROFILE%\.claude.json`.
  **No copies ninguno de otra máquina**: el primero tiene credenciales, el segundo
  rutas e historial locales.
- La cuenta de Claude es personal. No se comparte entre personas.

## PASO 3 — Cuenta y repositorio de GitHub

El usuario hace esto en el navegador; tú lo guías.

1. Cuenta en github.com con **correo propio**, y **2FA activado de inmediato**.
   GitHub lo exige para hacer push; dejarlo para después tranca justo cuando se
   necesita empujar código.
2. Al crear el repositorio, en la pantalla de *Configuration*:
   - **Visibility: Private**
   - **Add README: On** — evita el repo vacío, que confunde al cliente Git en el
     primer push
   - **Add .gitignore: `Laravel`** — no lo dejes en "No .gitignore". Sin eso se
     suben `vendor/`, `node_modules/`, `.env` y compilados, y limpiarlo después
     obliga a `git rm -r --cached` o a reescribir historia
   - **Add license: No license** (repo privado interno)
3. **No generes Personal Access Token.** El primer `clone`/`push` abre el
   navegador y Git Credential Manager guarda el login. Si más adelante hace falta
   un token para un pipeline, que sea **fine-grained**, permiso `Contents`
   únicamente, sobre un solo repositorio.
4. **Nunca** pongas credenciales en la URL del remoto
   (`https://usuario:token@github.com/...`): queda en `.git/config` en texto plano.

**Verificar el enlace sin clonar:**

```bash
git ls-remote https://github.com/SU-USUARIO/SU-REPO.git
```

Si responde sin pedir un token a mano, quedó bien.

**Clonar:**

```bash
git clone https://github.com/SU-USUARIO/SU-REPO.git
```

### Si el usuario también usa Azure DevOps

Segundo remoto **sobre el mismo clon** — no es otro clon:

```bash
git remote add devops https://dev.azure.com/SU-ORG/_git/SU-REPO
```

Y en `~/.gitconfig`, para que use OAuth y no pida credenciales cada vez:

```ini
[credential]
	azreposCredentialType = oauth
[credential "azrepos:org/SU-ORG"]
	username = CORREO-CORPORATIVO
```

> El código de la empresa va a la organización corporativa (GitHub org de APM o
> Azure DevOps), no a una cuenta personal. La cuenta personal sirve para
> autenticarse y para pruebas propias.

## PASO 4 — VS Code y extensiones

```bash
winget install --id Microsoft.VisualStudioCode -e
```

Extensión imprescindible:

```bash
code --install-extension anthropic.claude-code
```

Extensiones de PHP/Laravel, las mismas de la máquina de referencia:

```bash
code --install-extension laravel.vscode-laravel
```

```bash
code --install-extension onecentlin.laravel-blade
```

```bash
code --install-extension onecentlin.laravel5-snippets
```

```bash
code --install-extension shufo.vscode-blade-formatter
```

```bash
code --install-extension codingyu.laravel-goto-view
```

```bash
code --install-extension ryannaddy.laravel-artisan
```

**Verificar:**

```bash
code --list-extensions
```

Debe aparecer `anthropic.claude-code`. Pídele al usuario que abra VS Code y
confirme que la extensión de Claude tiene sesión iniciada.

## PASO 5 — PHP, Composer y Node

### 5.1 Node

```bash
winget install --id OpenJS.NodeJS.LTS -e
```

**Verificar:**

```bash
node -v && npm -v
```

### 5.2 PHP

**PHP en Windows no tiene instalador oficial: es un zip.** Esto no cambia con
admin. No busques un `.msi`.

1. Descarga de `https://windows.php.net/download/` el zip de **PHP 8.3.x, Thread
   Safe = NO (NTS), x64**.
2. Descomprime en `C:\php`.
3. Copia `php.ini-development` a `php.ini` y habilita estas extensiones — es
   exactamente lo que tiene la máquina de referencia funcionando:

```ini
extension=curl
extension=fileinfo
extension=intl
extension=mbstring
extension=openssl
extension=pdo_sqlite
extension=sqlite3
extension=zip
```

4. Agrega `C:\php` al PATH.

**Verificar:**

```bash
php -v && php -m
```

Las 8 extensiones de arriba deben aparecer en la salida de `php -m`. Laravel no
arranca sin `mbstring`, `openssl` ni `fileinfo`.

### 5.3 Composer

Instalador oficial `Composer-Setup.exe` de `https://getcomposer.org/download/`.
Detecta el PHP del PATH, así que el paso 5.2 tiene que estar terminado y
verificado antes.

**Verificar:**

```bash
composer -V
```

### 5.4 SQL Server desde PHP (solo si el proyecto lo necesita)

Pregúntale al usuario si su aplicación va a leer SQL Server. Si la respuesta es
no, sáltate esto: Laravel corre con SQLite sin nada extra.

Si la respuesta es sí, hacen falta **dos** cosas y la primera exige admin:

1. **Microsoft ODBC Driver 18 for SQL Server** (`msodbcsql.msi`).
2. **Microsoft Drivers for PHP for SQL Server**: del paquete que corresponde a PHP
   8.3, copia a `C:\php\ext\` los dos DLL de la variante **NTS x64**:
   - `php_sqlsrv_83_nts_x64.dll`
   - `php_pdo_sqlsrv_83_nts_x64.dll`
3. Agrega al `php.ini`:

```ini
extension=sqlsrv
extension=pdo_sqlsrv
```

**Verificar:**

```bash
php -m | findstr sqlsrv
```

Herramientas de consulta, opcionales pero útiles:

```bash
winget install --id Microsoft.Sqlcmd -e
```

```bash
winget install --id Microsoft.SQLServerManagementStudio -e
```

> En la máquina de referencia **esto no se pudo hacer** (el ODBC pide admin en un
> host AVD), así que allá el desarrollo local corre contra SQLite y las consultas
> a SQL Server se hacen con `sqlcmd`. Si acá hay admin, hazlo bien desde el
> principio.

## PASO 6 — El proyecto Laravel

Dos escenarios. Pregúntale al usuario cuál es el suyo.

### A) Proyecto nuevo, que va a subir a su repo vacío

```bash
composer create-project laravel/laravel NOMBRE-DEL-PROYECTO
```

Luego, dentro de la carpeta creada:

```bash
git init && git branch -M main
```

```bash
git remote add origin https://github.com/SU-USUARIO/SU-REPO.git
```

Antes del primer commit, **confirma que `.gitignore` existe y excluye `.env`,
`/vendor` y `/node_modules`** — Laravel lo trae, pero verifícalo:

```bash
git status --short | findstr /I "vendor node_modules .env"
```

Ese comando **no debe devolver nada**. Si devuelve algo, arregla el `.gitignore`
antes de commitear; sacar `vendor/` del historial después es mucho más trabajo.

```bash
git add -A && git commit -m "chore: proyecto Laravel inicial"
```

```bash
git push -u origin main
```

### B) Repositorio existente ya clonado

```bash
composer install
```

```bash
copy .env.example .env
```

```bash
php artisan key:generate
```

```bash
npm install && npm run build
```

### Levantar, en cualquiera de los dos casos

```bash
php artisan serve
```

**Verificar:** responde en `http://localhost:8000`.

Laravel 12 trae dos atajos en `composer.json` que conviene conocer:

```bash
composer run dev
```

Ese corre servidor + queue + logs + Vite en paralelo, que es la forma cómoda de
trabajar.

**Cosas que revisar del `.env`:** `APP_KEY` tiene que estar poblado (lo hace
`key:generate`), y `.env` **no debe aparecer nunca** en `git status`.

## PASO 7 — Playwright MCP

Esto le da a Claude un navegador real: navegar, leer páginas, tomar capturas,
probar interfaces.

```bash
npm install -g @playwright/mcp@0.0.79
```

```bash
npx playwright install chromium
```

Los navegadores quedan en `%LOCALAPPDATA%\ms-playwright`.

Regístralo como servidor MCP a nivel de usuario:

```bash
claude mcp add playwright --scope user -- npx @playwright/mcp@0.0.79 --browser chromium
```

**Verificar:**

```bash
claude mcp list
```

Debe aparecer `playwright` y conectado. Queda anotado en
`%USERPROFILE%\.claude.json` bajo `mcpServers`. En la máquina de referencia es el
único MCP configurado.

## PASO 8 — Plugins y skills de Claude

Dos marketplaces habilitados en la máquina de referencia:

```bash
claude plugin marketplace add https://github.com/obra/superpowers.git
```

```bash
claude plugin marketplace add mattpocock/skills
```

```bash
claude plugin install superpowers@superpowers-dev
```

```bash
claude plugin install mattpocock-skills@mattpocock
```

**Verificar:**

```bash
claude plugin list
```

Resultado esperado en `%USERPROFILE%\.claude\settings.json`:

```json
{
  "enabledPlugins": {
    "mattpocock-skills@mattpocock": true,
    "superpowers@superpowers-dev": true
  },
  "autoUpdatesChannel": "latest",
  "theme": "auto"
}
```

> Son repositorios de terceros, no de Anthropic. **Menciónale eso al usuario antes
> de instalarlos** y déjalo decidir; es una máquina corporativa.

## PASO 9 — Extras (solo si el usuario los pide)

| Herramienta | Para qué |
|---|---|
| **Power BI Desktop** | modelos y reportes — `winget install --id Microsoft.PowerBI -e` |
| **On-premises data gateway** | que Power BI en la nube lea bases internas |
| **Office Deployment Tool** | instalar o reparar Office |

El gateway se instala como **servicio de Windows** y se registra contra el tenant.
Aunque haya admin local, **eso se coordina con IT**: define por dónde salen los
refrescos de toda la organización. No es parte del entorno de desarrollo.

## 10. Cierre: contexto para la IA dentro del repo

Cuando el proyecto esté andando, propón esto al usuario. Es lo que hace que las
siguientes conversaciones no arranquen de cero:

- Un **`CLAUDE.md`** en la raíz con las convenciones del proyecto: stack, cómo se
  corre, cómo se nombran las ramas y los commits, qué no se toca.
- O la estructura de la máquina de referencia, que es más completa: `START_HERE.md`
  en la raíz + carpeta `.ai/` con `context.md`, `rules.md`, `workflows.md` y
  `project-structure.md`, y la convención de que **cualquier hilo de IA los lee
  antes de tocar código**.

Y la regla que no se negocia: **los secretos van en `.env` (local) o en Key Vault
(nube). Nunca en el código, nunca en el repositorio.**

## 11. Checklist final

Corre esto y reporta el resultado al usuario:

```bash
git --version && node -v && npm -v && php -v && composer -V && claude --version
```

- [ ] `php -m` lista `curl fileinfo intl mbstring openssl pdo_sqlite sqlite3 zip`
- [ ] `php -m | findstr sqlsrv` responde, **si** el proyecto usa SQL Server
- [ ] `git config --global user.email` devuelve el correo correcto
- [ ] 2FA activo en la cuenta de GitHub (confirmado por el usuario)
- [ ] `git ls-remote` al repo responde sin pedir token a mano
- [ ] `git remote -v` muestra `origin` (y `devops` si aplica)
- [ ] `code --list-extensions` incluye `anthropic.claude-code`
- [ ] La extensión de Claude Code tiene sesión iniciada (confirmado por el usuario)
- [ ] `claude mcp list` muestra `playwright` conectado
- [ ] `claude plugin list` muestra los plugins habilitados
- [ ] `%LOCALAPPDATA%\ms-playwright` tiene una carpeta `chromium-*`
- [ ] `php artisan serve` responde en `http://localhost:8000`
- [ ] `.env` **no** aparece en `git status`

---

## Apéndice A — Si NO hay permisos de administrador

Usa esto en lugar del cuerpo del documento cuando el usuario no sea admin. Es como
está montada la máquina de referencia (host AVD restringido). Regla: **nada en
`C:\Program Files`**, todo en el perfil del usuario.

| Qué | Dónde queda | Cómo |
|---|---|---|
| VS Code | `%LOCALAPPDATA%\Programs\Microsoft VS Code` | instalador **"User Installer"** |
| Claude (app) | `%LOCALAPPDATA%\AnthropicClaude` | `Claude Setup.exe` |
| Claude (CLI) | `%USERPROFILE%\.local\bin\claude` | instalador nativo |
| Git | perfil del usuario | instalador **"for current user only"** |
| Node, PHP, Composer, sqlcmd | `%USERPROFILE%\Tools\...` | **zips portables** |

```
%USERPROFILE%\Tools\
    node\        <- Node portable (zip de nodejs.org, NO el MSI)
    php\         <- PHP NTS x64 (zip de windows.php.net)
    composer\    <- composer.phar + composer.bat
    sqlcmd\      <- sqlcmd (versión go, un solo .exe)
```

**Detalle clave:** con el Node portable, los paquetes globales de npm se instalan
dentro de esa misma carpeta. Por eso `npm install -g` funciona sin admin, y por eso
el Playwright MCP del paso 7 se puede instalar igual.

Composer sin instalador: descarga `composer.phar` y crea al lado un
`composer.bat` con una sola línea:

```bat
@php "%~dp0composer.phar" %*
```

PATH **de usuario**, una sola vez, en PowerShell:

```bash
$tools = "$env:USERPROFILE\Tools"; $actual = [Environment]::GetEnvironmentVariable('Path','User'); [Environment]::SetEnvironmentVariable('Path', "$actual;$tools\node;$tools\php;$tools\composer;$tools\sqlcmd", 'User')
```

Después de eso, **tu terminal actual sigue sin ver el PATH nuevo**. Actualízalo en
la sesión o usa rutas completas.

Lo que **no** se puede sin admin: el ODBC Driver 18, y por lo tanto `pdo_sqlsrv`.
Ahí el desarrollo local va contra SQLite (`DB_CONNECTION=sqlite`) y las consultas
a SQL Server se hacen con `sqlcmd`.

## Apéndice B — Solo en Azure Virtual Desktop con FSLogix

**No aplica en una máquina normal.** Si el usuario no está en AVD, ignora esto.

Síntoma: la **app de escritorio de Claude** abre las conversaciones del sidebar y
muestra **"No messages yet"**, aunque la sesión funcione y responda.

Causa: el driver de FSLogix reporta `dev`/`ino` distintos para el mismo archivo
según se consulte por ruta o por handle. Las versiones `1.40609.x` de la app
interpretan esa inconsistencia como transcript no confiable y no renderizan nada.
**No es pérdida de datos, y no es el tema ni el contraste de la terminal.**

Diagnóstico en un comando:

```bash
grep "Not loading transcript" "$LOCALAPPDATA/Claude/Logs/main.log" | tail -5
```

El runbook completo con la solución verificada end-to-end está en
`FIX-Claude-Code-FSLogix.md`, en la máquina de referencia. Si el síntoma aparece,
pídeselo al usuario antes de tocar nada.

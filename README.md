# opencode-local-prompts

Repositorio listo para usar **OpenCode** con modelos locales de **Ollama**, aprovechando las fortalezas reales de **Qwen2.5-Coder 3B** mediante *system prompts* instalados como agentes.

Incluye:
- Un script (`opencode-local`) que prepara Ollama, el modelo y OpenCode.
- Prompts/agentes para **Qwen2.5-Coder 3B** (TypeScript, Lua, Lisp funcional y Python).

Los prompts y la asignación de lenguajes se basan en pruebas prácticas para que cada agente se use donde es más fiable.

## Contenido del repositorio

```text
.
├── README.md
├── script/
│   └── opencode-local
└── qwen2.5-coder-3b/
    ├── system_prompt_qwen2.5-coder_lisp_typescript.md
    ├── system_prompt_qwen2.5-coder_lua_typescript.md
    └── system_prompt_qwen2.5-coder_python.md
```

Cada carpeta agrupa los archivos de cada componente: `script/` (lanzador) y `qwen2.5-coder-3b/` (prompts/agentes de Qwen).

## Qué hace bien cada agente

| Agente | Modelo | Usarlo para | Notas |
|---|---|---|---|
| `qwen-lisp` | `ollama/qwen2.5-coder:3b` | **TypeScript** y **Common Lisp funcional** (`mapcar`, `reduce`, recursión, `defclass/defmethod`) | En Lisp usa `(funcall f x)` cuando la función va en una variable; revisa que las macros empiecen por `(defmacro ...)`. |
| `qwen-lua` | `ollama/qwen2.5-coder:3b` | **TypeScript** y **Lua 5.4** (tablas, metatablas, módulos, `io`, `pcall`) | Revisa `return M` en módulos, `~=` y bucles `for i = 1, n do`. |
| `qwen-python` | `ollama/qwen2.5-coder:3b` | **Python 3.12** (archivos, `json`, `pathlib`, `dataclasses`, clases, type hints), y también TS/Lua/Lisp | Buen equilibrio para scripts y utilidades en Python. |

> **Nota de hardware:** `qwen2.5-coder:3b` (Q4) ocupa ~2 GB. En equipos con poca RAM (por ejemplo 3–4 GB totales y sin GPU) puede ir justo o usar swap; si notas lentitud, prueba con una variante de menos contexto o usa un modelo más pequeño.

## Requisitos previos

- Linux, macOS o Windows (WSL recomendado) con `bash`, `curl`, `python3`, `git`.
- [Ollama](https://ollama.com/) instalado y accesible (`ollama --version`).
- [OpenCode](https://opencode.ai/) instalado (`opencode --version`).
- Espacio libre en disco: al menos **4 GB** (modelo + variante de contexto).
- RAM suficiente para un modelo de ~2 GB (recomendado 8 GB o más para ir cómodo).

Comprueba:

```bash
ollama --version
opencode --version
python3 --version
```

## Paso 1. Clonar o copiar este repositorio

```bash
git clone https://github.com/muzan481/opencode-local-prompts.git opencode-local-prompts
cd opencode-local-prompts
```

Si ya tienes la carpeta copiada localmente, entra en ella (cambia la ruta por el lugar donde la guardaste). Por ejemplo, si está en tu carpeta personal:

```bash
cd ~/opencode-local-prompts
```

Si no sabes dónde quedó la carpeta, puedes buscarla con alguno de estos comandos:

```bash
# Busca la carpeta por nombre desde tu carpeta personal (puede tardar un poco)
find ~ -type d -name "opencode-local-prompts" 2>/dev/null

# Busca desde todo el disco (más lento)
find / -type d -name "opencode-local-prompts" 2>/dev/null

# Si tienes locate instalado, es más rápido
locate opencode-local-prompts

# Comprueba si está en la carpeta actual o en tu carpeta personal
ls -d ./opencode-local-prompts ~/opencode-local-prompts 2>/dev/null
```

El comando mostrará la ruta de la carpeta. Entra en ella con `cd` y esa ruta, por ejemplo:

```bash
cd /ruta/que/te/mostro/el/comando/opencode-local-prompts
```

## Paso 2. Instalar el script `opencode-local`

El script debe quedar en una carpeta que esté en tu `PATH`.

**Opción A (recomendada para tu usuario, sin sudo):**

```bash
mkdir -p ~/.local/bin
cp script/opencode-local ~/.local/bin/opencode-local
chmod +x ~/.local/bin/opencode-local
```

Asegúrate de que `~/.local/bin` esté en tu `PATH` (añade esto a `~/.bashrc` si hace falta):

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**Opción B (sistema, con sudo):**

```bash
sudo cp script/opencode-local /usr/local/bin/opencode-local
sudo chmod +x /usr/local/bin/opencode-local
```

Verifica:

```bash
opencode-local --help
```

### Qué hace `opencode-local`

- Comprueba/inicia Ollama si no responde.
- **Prepara el modelo predeterminado** que usan los agentes de este repo: `qwen2.5-coder:3b`.
- Si falta, lo descarga automáticamente (salvo `--no-pull`). Si usas `--no-pull` y falta, muestra qué modelo falta y los comandos exactos para instalarlo.
- Crea la variante con contexto fijo (`-ctx16384`, por defecto) ajustando `num_ctx`.
- Precarga el modelo activo en memoria (`keep_alive`).
- Ajusta la configuración de OpenCode para usar el modelo activo y abre OpenCode.

Opciones y variables útiles:

```bash
opencode-local                 # prepara qwen2.5-coder:3b y abre OpenCode
opencode-local --no-launch     # solo prepara el entorno
opencode-local --no-pull       # no descarga si falta: muestra qué falta y los comandos para instalarlo
MODEL=qwen2.5-coder:3b opencode-local --no-launch
OPENCODE_LOCAL_CONTEXT=16384 opencode-local
```

## Paso 3. Descargar el modelo de Ollama

El modelo es el "cerebro" que usarás en tu PC. `qwen2.5-coder:3b` pesa alrededor de **2 GB**, así que necesitas conexión a internet y espacio libre en el disco (al menos **4 GB libres** para ir cómodo).

Modelo predeterminado:
- `qwen2.5-coder:3b` (agentes `qwen-lisp`, `qwen-lua` y `qwen-python`)

El script `opencode-local` lo revisa: si ya está instalado, no lo vuelve a descargar; si falta, lo instala (y crea su variante `-ctx16384`). Si ejecutas el script con `--no-pull` y falta, la terminal mostrará qué modelo pretende instalar y los comandos exactos que debes ejecutar.

También puedes instalarlo a mano siguiendo estos pasos:

### 3.1 Comprueba que Ollama está instalado y encendido

```bash
ollama --version
```

- Si ves un número de versión, sigue.
- Si ves `command not found: ollama` (no encuentra Ollama), instálalo y vuelve a intentar:
  ```bash
  curl -fsSL https://ollama.com/install.sh | sh
  ```
  (Necesitas `curl`. Si no lo tienes: en Ubuntu/Debian `sudo apt update && sudo apt install -y curl`; en Fedora `sudo dnf install -y curl`).

Ahora comprueba que Ollama responde:

```bash
ollama list
```

- Si muestra una lista (aunque esté vacía), va bien.
- Si dice que **no se puede conectar** o "connection refused", Ollama no está encendido. Enciéndelo:
  ```bash
  ollama serve
  ```
  Déjalo abierto en esa terminal y abre **otra terminal** para seguir. (En la siguiente vez, `opencode-local` intentará encenderlo por ti).

### 3.2 Descarga el modelo base

Ejecuta este comando (puede tardar varios minutos según tu internet):

```bash
ollama pull qwen2.5-coder:3b
```

**Si algo sale mal aquí:**

| Lo que ves en la terminal | Qué significa | Qué hacer |
|---|---|---|
| `Error: ... connection refused` / no conecta | Ollama no está encendido | Ejecuta `ollama serve` en otra terminal y repite el `ollama pull ...` |
| La descarga se corta, va muy lenta o dice "unexpected EOF" / "network" | Se cortó internet o la descarga falló a medias | Vuelve a ejecutar el mismo `ollama pull ...`; retoma o descarga de nuevo |
| `no space left on device` / falta de espacio | El disco está lleno | Libera espacio (borra archivos o modelos que no uses con `ollama rm <nombre>`) y repite |
| `model ... not found` | El nombre está mal escrito | Copia y pega el comando tal cual está escrito arriba |
| `permission denied` | No tienes permiso en la carpeta de Ollama | Prueba con `sudo ollama pull ...` o revisa que Ollama esté bien instalado para tu usuario |

### 3.3 Crea la variante con más memoria de conversación (contexto 16384)

Esta variante permite que el modelo "recuerde" más texto en cada conversación. El script la crea solo, pero puedes dejarla lista:

```bash
printf 'FROM qwen2.5-coder:3b\nPARAMETER num_ctx 16384\n' > /tmp/Modelfile.qwen3b
ollama create qwen2.5-coder:3b-ctx16384 -f /tmp/Modelfile.qwen3b
```

- Si dice que el modelo base **no existe**, primero repite el paso 3.2.
- Si dice "file not found" para el Modelfile, vuelve a ejecutar la línea `printf ...` y luego el `ollama create ...`.

### 3.4 Comprueba que todo quedó bien

```bash
ollama list
```

Deberías ver (los tamaños y fechas pueden variar):

- `qwen2.5-coder:3b`
- `qwen2.5-coder:3b-ctx16384`

Si falta alguna, repite solo la parte que falló (3.2 para el modelo base, 3.3 para la variante `-ctx16384`).

### 3.5 Si el script `opencode-local` falla

El script necesita `bash`, `curl`, `python3` y `ollama`. Si al ejecutarlo ves errores:

| Error o situación | Qué hacer |
|---|---|
| `command not found: opencode-local` | No está instalado o no está en tu PATH. Repite el Paso 2 y comprueba con `opencode-local --help` |
| `command not found: ollama` | Instala Ollama (paso 3.1) |
| `command not found: curl` o `python3` | Instala lo que falte: `sudo apt update && sudo apt install -y curl python3` (Ubuntu/Debian) o `sudo dnf install -y curl python3` (Fedora) |
| "No se pudo iniciar Ollama" / no responde | Abre `ollama serve` en otra terminal, revisa que no haya otro programa usando el puerto, y vuelve a ejecutar el script |
| "Falta el modelo..." / "Faltan estos modelos..." (con `--no-pull`) | El script te mostrará el comando exacto. Ejecútalo (por ejemplo `ollama pull qwen2.5-coder:3b` y, si aplica, su variante `-ctx16384`), o vuelve a ejecutar el script sin `--no-pull` |
| "No se pudo crear la variante..." | Asegúrate de que el modelo base ya está descargado (`ollama list`) y que hay espacio en disco |
| El script se queda cargando mucho tiempo | Es normal la primera vez (descarga/precarga). Si se corta, revisa tu internet y repite; puedes usar `--no-launch` para solo preparar sin abrir OpenCode |
| El equipo va muy lento o se congela | El modelo puede estar usando swap por falta de RAM. Cierra otros programas, prueba `OPENCODE_LOCAL_CONTEXT=8192 opencode-local` o usa un modelo más pequeño |

Para ver más detalle cuando algo falla, ejecuta:

```bash
bash -x opencode-local --no-launch
```

## Paso 4. Instalar los system prompts como agentes de OpenCode

OpenCode carga agentes desde `~/.config/opencode/agents/`. Cada archivo `.md` con *frontmatter* (`description`, `mode`, `model`) define un agente y su system prompt.

Crea el directorio y enlaza (o copia) los prompts:

```bash
mkdir -p ~/.config/opencode/agents

# desde la raíz de este repositorio (ajusta REPO si hace falta)
REPO="$(pwd)"

ln -sf "$REPO/qwen2.5-coder-3b/system_prompt_qwen2.5-coder_lisp_typescript.md" ~/.config/opencode/agents/qwen-lisp.md
ln -sf "$REPO/qwen2.5-coder-3b/system_prompt_qwen2.5-coder_lua_typescript.md" ~/.config/opencode/agents/qwen-lua.md
ln -sf "$REPO/qwen2.5-coder-3b/system_prompt_qwen2.5-coder_python.md" ~/.config/opencode/agents/qwen-python.md
```

> Usar enlaces simbólicos (`ln -sf`) permite que, si actualizas los prompts en el repo, OpenCode vea los cambios tras recargar. Si prefieres copiar, usa `cp` en lugar de `ln -sf`.

Verifica que OpenCode los detecta:

```bash
opencode debug agents
```

Deberías ver `qwen-lisp`, `qwen-lua` y `qwen-python` como agentes `primary`.

## Paso 5. Configurar OpenCode (opcional pero recomendado)

Edita `~/.config/opencode/opencode.json` para declarar el modelo por defecto, el agente por defecto y los límites de contexto:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/qwen2.5-coder:3b-ctx16384",
  "default_agent": "qwen-lisp",
  "providers": {
    "ollama": {
      "models": {
        "qwen2.5-coder:3b-ctx16384": {
          "limit": {
            "context": 16384,
            "output": 4096
          }
        }
      }
    }
  }
}
```

> Nota: `opencode-local` puede actualizar el campo `model` al preparar el entorno; los agentes aquí definidos fuerzan su propio modelo cuando los seleccionas.

Recarga la configuración:

```bash
opencode reload
```

## Paso 6. Usar los agentes

### En la TUI

1. Lanza OpenCode (o usa `opencode-local`):
   ```bash
   opencode
   # o
   opencode-local
   ```
2. Pulsa **Tab** para cambiar de agente y elige:
   - `qwen-lisp` → TypeScript o Common Lisp (funcional).
   - `qwen-lua` → TypeScript o Lua 5.4.
   - `qwen-python` → Python 3.12 (y tareas generales de scripting).

### Por línea de comandos

```bash
# Lisp / TypeScript
opencode run --agent qwen-lisp "Common Lisp: define una función que sume una lista"

# Lua / TypeScript
opencode run --agent qwen-lua "Lua: lee un archivo y devuelve sus líneas en una tabla"

# Python
opencode run --agent qwen-python "Python: cuenta la frecuencia de palabras de un archivo"
```

### Consejos para sacarles el máximo provecho

- **Sé concreto y acota la tarea**: pide funciones o cambios pequeños y claros.
- **Qwen + Python**: aprovecha `pathlib`, `json`, `dataclasses` y type hints; revisa casos borde.
- **Qwen + Lisp**: pídele estilo funcional (`mapcar`, `reduce`, recursión). Si pide una macro, revisa que empiece por `(defmacro ...)` y usa `(funcall f x)` cuando la función venga en una variable.
- **Qwen + Lua**: comprueba que los módulos terminen en `return M` y que use `~=` / `for i = 1, n do` / índices desde 1.
- **Contexto/RAM**: si tu equipo tiene poca memoria, reduce el contexto (`OPENCODE_LOCAL_CONTEXT=8192`) o cierra otros programas.

## Subir este repositorio a tu GitHub

Si aún no es un repositorio git:

```bash
cd /ruta/a/opencode-local-prompts
git init
git add .
git commit -m "Prompts y script para OpenCode local (Qwen 2.5 Coder 3B)"
```

Crea un repositorio vacío en tu cuenta de GitHub y enlázalo:

```bash
git remote add origin git@github.com:TU_USUARIO/opencode-local-prompts.git
git branch -M main
git push -u origin main
```

(Con HTTPS: `git remote add origin https://github.com/TU_USUARIO/opencode-local-prompts.git`).

Si tienes `gh` autenticado:

```bash
gh repo create opencode-local-prompts --public --source=. --remote=origin --push
```

## Notas

- Los prompts son archivos `.md` con *frontmatter* de agente de OpenCode (V2): definen `description`, `mode: primary`, `model` y el cuerpo como system prompt.
- `opencode-local` y los prompts son independientes: puedes usar el script sin los agentes, o los agentes sin el script, pero juntos dan la mejor experiencia local.
- Ajusta los nombres de modelo si usas variantes distintas (por ejemplo, otros tamaños o `num_ctx`). En ese caso, edita el campo `model` en el *frontmatter* de cada prompt y la variable `MODEL`/`DEFAULT_MODELS` del script si corresponde.

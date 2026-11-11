# opencode-local-prompts

Repositorio listo para usar **OpenCode** con modelos locales de **Ollama** (1.5B), aprovechando al máximo las fortalezas reales de cada modelo mediante *system prompts* instalados como agentes.

Incluye:
- Un script (`opencode-local`) que prepara Ollama, el modelo y OpenCode.
- Prompts/agentes para **Qwen2.5-Coder 1.5B** (TypeScript, Lua y Lisp funcional).
- Prompt/agente para **DeepSeek-R1 1.5B** (solo Python).

Los prompts y la asignación de lenguajes se basan en pruebas prácticas: cada modelo se usa solo donde es fiable.

## Contenido del repositorio

```text
.
├── README.md
├── script/
│   └── opencode-local
├── qwen2.5-coder-1.5b/
│   ├── system_prompt_qwen2.5-coder_lisp_typescript.md
│   └── system_prompt_qwen2.5-coder_lua_typescript.md
└── deepseek-r1-1.5b/
    └── system_prompt_deepseek-r1_python.md
```

Cada carpeta agrupa los archivos de cada componente: `script/` (lanzador), `qwen2.5-coder-1.5b/` (prompts de Qwen) y `deepseek-r1-1.5b/` (prompt de DeepSeek).

## Qué hace bien cada modelo (según pruebas)

| Agente | Modelo | Usarlo para | Fiabilidad |
|---|---|---|---|
| `qwen-lisp` | `ollama/qwen2.5-coder:1.5b` | **TypeScript**, **Common Lisp funcional** (`mapcar`, `reduce`, recursión, `defclass/defmethod`) y también **Lua** | Alta en TS/Lua; buena en Lisp funcional |
| `qwen-lua` | `ollama/qwen2.5-coder:1.5b` | **TypeScript** y **Lua 5.4** (tablas, metatablas, módulos, `io`, `pcall`) y también **Lisp funcional** | Alta en TS/Lua |
| `deepseek-python` | `ollama/deepseek-r1:1.5b` | **Solo Python 3.12** (archivos, `json`, `pathlib`, `dataclasses`, clases, type hints) | Buena en Python básico/medio |

**Límites conocidos:**
- Qwen: en Lisp, al llamar una función guardada en variable hay que usar `(funcall f x)` (Lisp-2), y a veces escribe `defun` en macros con nombre `mi-*` (el prompt lo corrige, pero conviene revisar).
- DeepSeek: **no** es fiable en C, C++, Rust, Lua (nativo ni con `lupa`), TypeScript ni Lisp; mezcla sintaxis e inventa funciones. Por eso su agente se limita a Python.

## Requisitos previos

- Linux con `bash`, `curl`, `python3`, `git`.
- [Ollama](https://ollama.com/) instalado y accesible (`ollama --version`).
- [OpenCode](https://opencode.ai/) instalado (`opencode --version`).
- RAM suficiente para modelos 1.5B (varios GB libres recomendados).

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

Si ya tienes la carpeta copiada localmente, entra en ella:

```bash
cd /ruta/a/opencode-local-prompts
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
- **Prepara los dos modelos predeterminados** que usan los agentes de este repo: `qwen2.5-coder:1.5b` y `deepseek-r1:1.5b`.
- Si falta alguno, lo descarga automáticamente (salvo `--no-pull`). Si usas `--no-pull` y falta alguno, muestra qué modelos faltan y los comandos exactos para instalarlos.
- Crea las variantes con contexto fijo (`-ctx16384`, por defecto) ajustando `num_ctx` para los modelos predeterminados.
- Precarga el modelo activo en memoria (`keep_alive`).
- Ajusta la configuración de OpenCode para usar el modelo activo y abre OpenCode.

Opciones y variables útiles:

```bash
opencode-local                 # prepara qwen y deepseek, usa qwen y abre OpenCode
opencode-local --no-launch     # solo prepara el entorno (ambos modelos)
opencode-local --no-pull       # no descarga si falta: muestra qué falta y los comandos para instalarlo
MODEL=deepseek-r1:1.5b opencode-local --no-launch
OPENCODE_LOCAL_CONTEXT=16384 opencode-local
```

## Paso 3. Descargar los modelos de Ollama

Los modelos son los "cerebros" que usarás en tu PC. Cada uno pesa alrededor de **1 GB**, así que necesitas conexión a internet y espacio libre en el disco (al menos **4 GB libres** para ir cómodo).

Este repositorio usa **dos modelos predeterminados**:
- `qwen2.5-coder:1.5b` (agentes `qwen-lisp` y `qwen-lua`)
- `deepseek-r1:1.5b` (agente `deepseek-python`)

El script `opencode-local` revisa ambos: si ya están instalados, no los vuelve a descargar; si falta alguno, lo instala (y crea su variante `-ctx16384`). Si ejecutas el script con `--no-pull` y falta algún modelo, la terminal mostrará qué modelos pretende instalar y los comandos exactos que debes ejecutar para instalarlos.

También puedes instalarlos a mano siguiendo estos pasos:

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
  Déjalo abierto en esa terminal y abre **otra terminal** para seguir con los comandos de abajo. (En la siguiente vez, el script `opencode-local` intentará encenderlo por ti).

### 3.2 Descarga los modelos base

Ejecuta estos dos comandos (uno por uno). Pueden tardar varios minutos según tu internet:

```bash
ollama pull qwen2.5-coder:1.5b
ollama pull deepseek-r1:1.5b
```

**Si algo sale mal aquí:**

| Lo que ves en la terminal | Qué significa | Qué hacer |
|---|---|---|
| `Error: ... connection refused` / no conecta | Ollama no está encendido | Ejecuta `ollama serve` en otra terminal y repite el `ollama pull ...` |
| La descarga se corta, va muy lenta o dice "unexpected EOF" / "network" | Se cortó internet o la descarga falló a medias | Vuelve a ejecutar el mismo `ollama pull ...`; retoma o descarga de nuevo |
| `no space left on device` / falta de espacio | El disco está lleno | Libera espacio (borra archivos o modelos que no uses con `ollama rm <nombre>`) y repite |
| `model ... not found` | El nombre está mal escrito | Copia y pega el comando tal cual está escrito arriba |
| `permission denied` | No tienes permiso en la carpeta de Ollama | Prueba con `sudo ollama pull ...` o revisa que Ollama esté bien instalado para tu usuario |

### 3.3 Crea las variantes con más memoria de conversación (contexto 16384)

Estas variantes permiten que el modelo "recuerde" más texto en cada conversación. El script las crea solo, pero es mejor dejarlas listas:

```bash
printf 'FROM qwen2.5-coder:1.5b\nPARAMETER num_ctx 16384\n' > /tmp/Modelfile.qwen
ollama create qwen2.5-coder:1.5b-ctx16384 -f /tmp/Modelfile.qwen

printf 'FROM deepseek-r1:1.5b\nPARAMETER num_ctx 16384\n' > /tmp/Modelfile.deepseek
ollama create deepseek-r1:1.5b-ctx16384 -f /tmp/Modelfile.deepseek
```

- Si dice que el modelo base **no existe**, primero repite el paso 3.2 para ese modelo.
- Si dice "file not found" para el Modelfile, vuelve a ejecutar la línea `printf ...` de ese modelo y luego el `ollama create ...`.

### 3.4 Comprueba que todo quedó bien

```bash
ollama list
```

Deberías ver estas 4 líneas (los tamaños y fechas pueden variar):

- `qwen2.5-coder:1.5b`
- `qwen2.5-coder:1.5b-ctx16384`
- `deepseek-r1:1.5b`
- `deepseek-r1:1.5b-ctx16384`

Si falta alguna, repite solo la parte que falló (3.2 para un modelo base, 3.3 para una variante `-ctx16384`).

### 3.5 Si el script `opencode-local` falla

El script necesita `bash`, `curl`, `python3` y `ollama`. Si al ejecutarlo ves errores:

| Error o situación | Qué hacer |
|---|---|
| `command not found: opencode-local` | No está instalado o no está en tu PATH. Repite el Paso 2 y comprueba con `opencode-local --help` |
| `command not found: ollama` | Instala Ollama (paso 3.1) |
| `command not found: curl` o `python3` | Instala lo que falte: `sudo apt update && sudo apt install -y curl python3` (Ubuntu/Debian) o `sudo dnf install -y curl python3` (Fedora) |
| "No se pudo iniciar Ollama" / no responde | Abre `ollama serve` en otra terminal, revisa que no haya otro programa usando el puerto, y vuelve a ejecutar el script |
| "Faltan modelos..." / "Faltan estos modelos..." (con `--no-pull`) | El script te mostrará los comandos exactos. Cópialos y ejecútalos (por ejemplo `ollama pull qwen2.5-coder:1.5b` y `ollama pull deepseek-r1:1.5b`, y si aplica sus variantes `-ctx16384`), o vuelve a ejecutar el script sin `--no-pull` |
| "No se pudo crear la variante..." | Asegúrate de que el modelo base ya está descargado (`ollama list`) y que hay espacio en disco |
| El script se queda cargando mucho tiempo | Es normal la primera vez (descarga/precarga). Si se corta, revisa tu internet y repite; puedes usar `--no-launch` para solo preparar sin abrir OpenCode |

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

ln -sf "$REPO/qwen2.5-coder-1.5b/system_prompt_qwen2.5-coder_lisp_typescript.md" ~/.config/opencode/agents/qwen-lisp.md
ln -sf "$REPO/qwen2.5-coder-1.5b/system_prompt_qwen2.5-coder_lua_typescript.md" ~/.config/opencode/agents/qwen-lua.md
ln -sf "$REPO/deepseek-r1-1.5b/system_prompt_deepseek-r1_python.md" ~/.config/opencode/agents/deepseek-python.md
```

> Usar enlaces simbólicos (`ln -sf`) permite que, si actualizas los prompts en el repo, OpenCode vea los cambios tras recargar. Si prefieres copiar, usa `cp` en lugar de `ln -sf`.

Verifica que OpenCode los detecta:

```bash
opencode debug agents
```

Deberías ver `qwen-lisp`, `qwen-lua` y `deepseek-python` como agentes `primary`.

## Paso 5. Configurar OpenCode (opcional pero recomendado)

Edita `~/.config/opencode/opencode.json` para declarar el modelo por defecto, el agente por defecto y los límites de contexto de los modelos `-ctx16384`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/qwen2.5-coder:1.5b-ctx16384",
  "default_agent": "qwen-lisp",
  "providers": {
    "ollama": {
      "models": {
        "deepseek-r1:1.5b-ctx16384": {
          "limit": {
            "context": 16384,
            "output": 4096
          }
        },
        "qwen2.5-coder:1.5b-ctx16384": {
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
   - `deepseek-python` → solo Python.

### Por línea de comandos

```bash
# Lisp / TypeScript con Qwen
opencode run --agent qwen-lisp "Common Lisp: define una función que sume una lista"

# Lua / TypeScript con Qwen
opencode run --agent qwen-lua "Lua: lee un archivo y devuelve sus líneas en una tabla"

# Python con DeepSeek
opencode run --agent deepseek-python "Python: cuenta la frecuencia de palabras de un archivo"
```

### Consejos para sacarles el máximo provecho

- **Sé concreto y acota la tarea**: son modelos 1.5B; responden mejor a peticiones cortas y específicas.
- **Qwen + Lisp**: pídele estilo funcional (`mapcar`, `reduce`, recursión). Si pide una macro, revisa que empiece por `(defmacro ...)` y usa `(funcall f x)` cuando la función venga en una variable.
- **Qwen + Lua**: comprueba que los módulos terminen en `return M` y que use `~=` / `for i = 1, n do` / índices desde 1.
- **DeepSeek + Python**: úsalo solo para Python. Si necesitas embeber Lua, usa `qwen-lua` (Lua nativo) o una alternativa como `wasmoon` desde TypeScript.
- **Contexto**: las variantes `-ctx16384` dan más margen; evita prompts gigantes porque a 1.5B el rendimiento empeora con mucho contexto.

## Subir este repositorio a tu GitHub

Si aún no es un repositorio git:

```bash
cd /ruta/a/opencode-local-prompts
git init
git add .
git commit -m "Prompts y script para OpenCode local (Qwen y DeepSeek)"
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
- Ajusta los nombres de modelo si usas variantes distintas (por ejemplo, otros tamaños o `num_ctx`). En ese caso, edita el campo `model` en el *frontmatter* de cada prompt.

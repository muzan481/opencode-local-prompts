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
├── qwen/
│   ├── system_prompt_qwen2.5-coder_lisp_typescript.md
│   └── system_prompt_qwen2.5-coder_lua_typescript.md
└── deepseek/
    └── system_prompt_deepseek-r1_python.md
```

Cada carpeta agrupa los archivos de cada componente: `script/` (lanzador), `qwen/` (prompts de Qwen) y `deepseek/` (prompt de DeepSeek).

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
- Descarga el modelo si falta (salvo `--no-pull`).
- Crea una variante con contexto fijo (`MODEL-ctx16384`, por defecto) ajustando `num_ctx`.
- Precarga el modelo en memoria (`keep_alive`).
- Ajusta la configuración de OpenCode para usar `ollama/<modelo-efectivo>` y abre OpenCode.

Opciones y variables útiles:

```bash
opencode-local                 # usa qwen2.5-coder:1.5b y abre OpenCode
opencode-local --no-launch     # solo prepara el entorno
opencode-local --no-pull       # no descarga si falta el modelo
MODEL=deepseek-r1:1.5b opencode-local --no-launch
OPENCODE_LOCAL_CONTEXT=16384 opencode-local
```

## Paso 3. Descargar los modelos de Ollama

Descarga los modelos base (el script puede hacerlo, pero es mejor tenerlos listos):

```bash
ollama pull qwen2.5-coder:1.5b
ollama pull deepseek-r1:1.5b
```

Crea las variantes con contexto de 16384 (el script también las crea; manualmente sería):

```bash
printf 'FROM qwen2.5-coder:1.5b\nPARAMETER num_ctx 16384\n' > /tmp/Modelfile.qwen
ollama create qwen2.5-coder:1.5b-ctx16384 -f /tmp/Modelfile.qwen

printf 'FROM deepseek-r1:1.5b\nPARAMETER num_ctx 16384\n' > /tmp/Modelfile.deepseek
ollama create deepseek-r1:1.5b-ctx16384 -f /tmp/Modelfile.deepseek
```

Comprueba:

```bash
ollama list
```

Deberías ver `qwen2.5-coder:1.5b`, `qwen2.5-coder:1.5b-ctx16384`, `deepseek-r1:1.5b` y `deepseek-r1:1.5b-ctx16384`.

## Paso 4. Instalar los system prompts como agentes de OpenCode

OpenCode carga agentes desde `~/.config/opencode/agents/`. Cada archivo `.md` con *frontmatter* (`description`, `mode`, `model`) define un agente y su system prompt.

Crea el directorio y enlaza (o copia) los prompts:

```bash
mkdir -p ~/.config/opencode/agents

# desde la raíz de este repositorio (ajusta REPO si hace falta)
REPO="$(pwd)"

ln -sf "$REPO/qwen/system_prompt_qwen2.5-coder_lisp_typescript.md" ~/.config/opencode/agents/qwen-lisp.md
ln -sf "$REPO/qwen/system_prompt_qwen2.5-coder_lua_typescript.md" ~/.config/opencode/agents/qwen-lua.md
ln -sf "$REPO/deepseek/system_prompt_deepseek-r1_python.md" ~/.config/opencode/agents/deepseek-python.md
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

# opencode-local-prompts

Repositorio listo para usar **OpenCode** con modelos locales de **Ollama**, combinando:

- **`qwen2.5:1.5b`** como **planificador** (`qwen-plan`): estructura, pasos e instrucciones claras para delegar.
- **`qwen2.5-coder:1.5b`** como **ejecutor** (`qwen-lisp`, `qwen-lua`, `qwen-python`): implementación de código según sus fortalezas.

Incluye un script (`opencode-local`) que prepara Ollama/modelos/OpenCode y *system prompts* instalables como agentes.

> Pensado para equipos con recursos ajustados (por ejemplo ~3–4 GB de RAM y sin GPU). Con poca memoria conviene usar un modelo a la vez y, si hace falta, reducir el contexto.

## Contenido del repositorio

```text
.
├── README.md
├── script/
│   └── opencode-local
├── qwen2.5-1.5b/
│   └── system_prompt_qwen2.5_planificador.md
└── qwen2.5-coder-1.5b/
    ├── system_prompt_qwen2.5-coder_lisp_typescript.md
    ├── system_prompt_qwen2.5-coder_lua_typescript.md
    └── system_prompt_qwen2.5-coder_python.md
```

## Agentes incluidos

| Agente | Modelo | Usarlo para | Notas |
|---|---|---|---|
| `qwen-plan` | `ollama/qwen2.5:1.5b` (recomendado `-ctx8192`) | **Planificación**: objetivo, estructura, pasos, casos borde e instrucciones para delegar al coder | No escribe el proyecto completo; genera el plan y tareas concretas. |
| `qwen-lisp` | `ollama/qwen2.5-coder:1.5b` (recomendado `-ctx16384`) | **TypeScript** y **Common Lisp funcional** (`mapcar`, `reduce`, recursión, `defclass/defmethod`) | En Lisp usa `(funcall f x)` cuando la función va en una variable; revisa que las macros empiecen por `(defmacro ...)`. |
| `qwen-lua` | `ollama/qwen2.5-coder:1.5b` (recomendado `-ctx16384`) | **TypeScript** y **Lua 5.4** (tablas, metatablas, módulos, `io`, `pcall`) | Revisa `return M` en módulos, `~=` y bucles `for i = 1, n do`. |
| `qwen-python` | `ollama/qwen2.5-coder:1.5b` (recomendado `-ctx16384`) | **Python 3.12** (archivos, `json`, `pathlib`, `dataclasses`, clases, type hints) y tareas generales | Pide pasos concretos y revisa casos borde. |

## Requisitos previos

- Linux, macOS o Windows (WSL recomendado) con `bash`, `curl`, `python3`, `git`.
- [Ollama](https://ollama.com/) instalado y accesible (`ollama --version`).
- [OpenCode](https://opencode.ai/) instalado (`opencode --version`).
- Espacio libre en disco: al menos **3–4 GB** (modelos + variantes de contexto).
- RAM: con ~3–4 GB totales puede ir justo; usa un modelo a la vez y considera reducir contexto.

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

Si ya tienes la carpeta copiada localmente, entra en ella (ajusta la ruta):

```bash
cd ~/opencode-local-prompts
```

## Paso 2. Instalar el script `opencode-local`

El script debe quedar en una carpeta que esté en tu `PATH`.

**Opción A (recomendada, sin sudo):**

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
- Prepara los modelos predeterminados de este flujo: `qwen2.5:1.5b` y `qwen2.5-coder:1.5b`.
- Si faltan, los descarga automáticamente (salvo `--no-pull`). Si usas `--no-pull` y falta alguno, muestra los comandos exactos para instalarlo.
- Crea variantes con contexto fijo (`-ctx16384` por defecto, configurable con `OPENCODE_LOCAL_CONTEXT`).
- Precarga el modelo activo y ajusta la configuración de OpenCode.

Opciones y variables útiles:

```bash
opencode-local                 # prepara modelos y abre OpenCode
opencode-local --no-launch     # solo prepara el entorno
opencode-local --no-pull       # no descarga si falta: muestra qué instalar
MODEL=qwen2.5-coder:1.5b opencode-local --no-launch
OPENCODE_LOCAL_CONTEXT=8192 opencode-local --no-launch
```

## Paso 3. Instalar los modelos en Ollama

Puedes dejar que `opencode-local` lo haga, o instalarlos manualmente:

```bash
# modelos base
ollama pull qwen2.5:1.5b
ollama pull qwen2.5-coder:1.5b

# variantes recomendadas
printf 'FROM qwen2.5:1.5b\nPARAMETER num_ctx 8192\n' | ollama create qwen2.5:1.5b-ctx8192 -f -
printf 'FROM qwen2.5-coder:1.5b\nPARAMETER num_ctx 16384\n' | ollama create qwen2.5-coder:1.5b-ctx16384 -f -
```

Comprueba:

```bash
ollama list
```

> Nota: `opencode-local` crea por defecto variantes `-ctx16384` (o el valor de `OPENCODE_LOCAL_CONTEXT`). El agente `qwen-plan` de este repo usa `qwen2.5:1.5b-ctx8192` para ir más ligero; puedes crearla con el comando anterior y usarla en su *frontmatter*.

## Paso 4. Instalar los system prompts como agentes de OpenCode

OpenCode carga agentes desde `~/.config/opencode/agents/`.

```bash
mkdir -p ~/.config/opencode/agents
REPO="$(pwd)"

# planificador
ln -sf "$REPO/qwen2.5-1.5b/system_prompt_qwen2.5_planificador.md" ~/.config/opencode/agents/qwen-plan.md

# ejecutores
ln -sf "$REPO/qwen2.5-coder-1.5b/system_prompt_qwen2.5-coder_lisp_typescript.md" ~/.config/opencode/agents/qwen-lisp.md
ln -sf "$REPO/qwen2.5-coder-1.5b/system_prompt_qwen2.5-coder_lua_typescript.md" ~/.config/opencode/agents/qwen-lua.md
ln -sf "$REPO/qwen2.5-coder-1.5b/system_prompt_qwen2.5-coder_python.md" ~/.config/opencode/agents/qwen-python.md
```

> Con enlaces simbólicos (`ln -sf`), si actualizas los prompts en el repo, OpenCode verá los cambios tras recargar. Si prefieres, usa `cp`.

Verifica:

```bash
opencode debug agents
```

Deberías ver `qwen-plan`, `qwen-lisp`, `qwen-lua` y `qwen-python`.

## Paso 5. Configurar OpenCode (recomendado)

Ejemplo de `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/qwen2.5-coder:1.5b-ctx16384",
  "default_agent": "qwen-lisp",
  "providers": {
    "ollama": {
      "models": {
        "qwen2.5:1.5b-ctx8192": {
          "limit": { "context": 8192, "output": 2048 }
        },
        "qwen2.5-coder:1.5b-ctx16384": {
          "limit": { "context": 16384, "output": 4096 }
        }
      }
    }
  }
}
```

Recarga:

```bash
opencode reload
```

## Paso 6. Usar el flujo planificador → ejecutor

1. **Planifica** (estructura y pasos concretos):
   ```bash
   opencode run --agent qwen-plan "Quiero [objetivo]. Dame estructura y pasos con instrucciones concretas para delegar al coder."
   ```
2. **Ejecuta un paso a la vez** con el coder adecuado:
   ```bash
   opencode run --agent qwen-lisp "Paso 1: ..."
   opencode run --agent qwen-lua "Paso 2: ..."
   opencode run --agent qwen-python "Paso 3: ..."
   ```
3. En la TUI (`opencode`), pulsa **Tab** para cambiar de agente.

**Consejos:** con poca RAM no uses varios modelos a la vez; si va lento, reduce contexto (`OPENCODE_LOCAL_CONTEXT=8192`); revisa el plan antes de delegarlo (archivos, casos borde y criterios de listo).

## Problemas comunes

- **Falta un modelo (`--no-pull`)**: el script muestra los comandos exactos (`ollama pull ...` y creación de variante).
- **Lentitud / swap**: cierra otros programas, baja contexto o usa un modelo cada vez.
- **Ollama no responde**: inicia `ollama serve` y vuelve a intentar.
- Para ver detalle: `bash -x opencode-local --no-launch`.

## Notas

- Los prompts son archivos `.md` con *frontmatter* de agente de OpenCode (V2): `description`, `mode: primary`, `model` y el cuerpo como system prompt.
- `opencode-local` y los agentes son independientes, pero juntos facilitan el uso local.
- Si cambias modelos o contexto, ajusta el `model` en el *frontmatter* de cada prompt y, si corresponde, `DEFAULT_MODELS`/`MODEL`/`OPENCODE_LOCAL_CONTEXT` en el script.



# Codebase Argus

<p align="center">
  <strong>Análisis de riesgo de sincronización de forks descendentes y revisión de PR multiagente para mantenedores.</strong>
</p>

<p align="center">
  <a href="https://aaronz345.github.io/codebase-argus/">Demo en vivo</a>
  ·
  <a href="#try-it-on-a-public-pr">Inicio rápido</a>
  ·
  <a href="docs/case-studies/cowagent-2965.md">Estudio de caso</a>
  ·
  <a href="https://github.com/AaronZ345/codebase-argus-action">GitHub Action</a>
  ·
  <a href="#cli">CLI</a>
  ·
  <a href="#github-app">GitHub App</a>
  ·
  <a href="#agent-playbook">Playbook de agentes</a>
  ·
  <a href="#skill-registries">Registros de habilidades</a>
</p>

<p align="center">
  <img alt="Next.js 16" src="https://img.shields.io/badge/Next.js-16-black?style=flat-square">
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-5-3178c6?style=flat-square">
  <img alt="Vitest" src="https://img.shields.io/badge/Vitest-tested-6e9f18?style=flat-square">
  <img alt="GitHub App" src="https://img.shields.io/badge/GitHub%20App-webhook-24292f?style=flat-square">
</p>

<p align="center">
  <img src="docs/assets/codebase-argus-home.png" alt="Panel de Codebase Argus que muestra flujos de trabajo de revisión de PR, CI y sincronización de forks descendentes">
</p>

Codebase Argus proporciona a los mantenedores un escritorio de revisión para evidencia del código base. Revisa
pull requests, registros de CI fallidos y sincronizaciones de forks de larga duración con el mismo conjunto de
señales: parches, comprobaciones, archivos, estado de la rama, controles de política, consenso de proveedores
y simulaciones de git locales.

Úsalo cuando un solo revisor no sea suficiente, pero un bot de fusión completamente automático sea
demasiado arriesgado. Argus puede solicitar a un modelo, varios modelos o CLIs de IA locales que revisen
la misma evidencia, y luego mantiene cada hallazgo vinculado a algo que un mantenedor puede
verificar.

## Pruébalo en un PR público

La ruta más corta y útil es una revisión local de solo lectura. No se necesita una clave de modelo para el paso determinista.

```bash
git clone https://github.com/AaronZ345/codebase-argus.git
cd codebase-argus
npm ci
npm run argus -- review zhayujie/CowAgent#2965
```

Para entregar la misma evidencia a un Codex CLI instalado:

```bash
npm run argus -- review zhayujie/CowAgent#2965 --provider codex-cli
```

El [estudio de caso CowAgent #2965](docs/case-studies/cowagent-2965.md) registra ambos pasos: riesgo bajo, sin problema bloqueante y un límite restante de prueba de integración para que un mantenedor lo juzgue.

## Ejecútalo en pull requests

La acción complementaria [Codebase Argus Action](https://github.com/AaronZ345/codebase-argus-action) coloca la revisión determinista en el resumen de trabajo del flujo. Es de solo lectura por defecto y no necesita clave de modelo.

```yaml
- uses: AaronZ345/codebase-argus-action@v1
  with:
    pull-request: ${{ github.repository }}#${{ github.event.pull_request.number }}
```

La acción también expone diagnóstico de CI fallido, planificación de autofix, proveedores de agentes opcionales y un `core-ref` fijo.

## A simple vista

| Flujo de trabajo | Entrada | Salida |
| --- | --- | --- |
| Revisión de PR | `owner/repo#123` o una URL de PR de GitHub | resumen de riesgo, hallazgos, comentarios listos para insertar |
| Revisión de CI | archivo de registro local o trabajos de GitHub Actions fallidos | causa raíz probable, comando afectado, ruta de corrección |
| Plan de autofix | hallazgos de revisión de PR | plan de rama con controles para correcciones mecánicas |
| Sincronización de fork descendente | repositorio upstream + repositorio fork | adelante/atrás, notas de conflicto, riesgo de rebase/fusión |
| Entrega al agente | panel o informe de CLI | paquete de tareas con comandos y controles de aceptación |

## Flujos de trabajo comunes

### Revisar un PR arriesgado con múltiples agentes

```bash
npm run argus -- review owner/repo#123 --tribunal openai-api,claude-cli,codex-cli
```

Argus obtiene los metadatos del PR, archivos modificados, comprobaciones, revisiones, commits y
fragmentos de parches, luego solicita a cada revisor configurado que examine el mismo contexto.
Los hallazgos coincidentes se agrupan para que el acuerdo sea visible; los fallos del proveedor permanecen en
el informe en lugar de desaparecer.

### Depurar registros de GitHub Actions fallidos

```bash
GITHUB_TOKEN=... npm run argus -- ci-github owner/repo#123 --provider codex-cli
```

El carril de CI extrae los registros de trabajos fallidos de GitHub Actions y solicita el primer
comando fallido, la causa raíz probable, los archivos afectados y la ruta de corrección más pequeña.

### Verificar si un fork puede hacer rebase de forma segura

```bash
npm run argus -- downstream owner/upstream me/fork --fork-branch feature/demo --tribunal codex-cli,claude-cli,gemini-cli
```

El carril descendente compara el fork con upstream, proyecta conflictos de fusión
con `git merge-tree`, simula un rebase en un árbol de trabajo temporal, verifica
commits equivalentes en parches con `git cherry`, y resume el movimiento semántico
con `git range-diff`.

## Ejecutar el panel localmente

```bash
npm install
npm run dev
```

Abre <http://localhost:3000>.

Para revisión desde la línea de comandos:

```bash
npm run argus -- review owner/repo#123
npm run argus -- ci-github owner/repo#123
npm run argus -- downstream owner/upstream me/fork
```

Los repositorios públicos de GitHub funcionan desde la demo alojada. Los repositorios privados,
proveedores de IA del lado del servidor, webhooks de GitHub App, análisis local de git y revisión de agentes por CLI pertenecen a un entorno de servidor local o desplegado.

## Capacidades principales

### Revisión de pull requests

Codebase Argus obtiene la estructura del PR que los mantenedores suelen necesitar antes
de confiar en una revisión:

- metadatos, etiquetas, autor, referencias de rama y capacidad de fusión;
- archivos modificados, fragmentos de parches, commits y revisiones previas;
- estado de comprobaciones y metadatos de ejecución de GitHub Actions;
- reglas de política desde `.codebase-argus.yml`;
- señales de PR apilados y estados de cola de fusión.

El revisor determinista busca comprobaciones fallidas, cambios en el código fuente sin
pruebas, ediciones de flujo de trabajo, cambios de dependencias, rutas sensibles, violaciones
de política, diffs grandes, bases de PR apilados y estados de fusión bloqueados/sucios/atrás/inestables.

### Revisión multiagente y tribunal

El mismo paquete de evidencia upstream o downstream puede enviarse a un proveedor o
a varios proveedores:

| Proveedor | Modo |
| --- | --- |
| `openai-api` | API |
| `anthropic-api` | API |
| `gemini-api` | API |
| `codex-cli` | CLI local |
| `claude-cli` | CLI local |
| `gemini-cli` | CLI local |

El modo tribunal ejecuta múltiples revisores contra el mismo PR, registro de CI o contexto de sincronización de fork.
Agrupa hallazgos coincidentes, aumenta la confianza cuando los proveedores coinciden,
y mantiene los fallos del proveedor en el informe.

### Fallos de CI

Usa `ci-log` para un archivo local, o `ci-github` para trabajos de GitHub Actions fallidos en
un PR. El modo webhook puede incluir registros de Actions fallidos en la revisión automática de PR.

### Planificación de autofix

`autofix-plan` convierte hallazgos mecánicos de alta confianza en un plan de rama. Cubre
carriles estrechos como actualizaciones de lockfile de npm, actualizaciones de snapshots y
correcciones de formateadores o linters. La salida incluye comandos, controles de verificación e
instrucciones de push para el mantenedor o agente que trabaje en un checkout real.

### Sincronización de fork descendente

El flujo de trabajo de fork compara un repositorio upstream y un fork de larga duración. El análisis
local ejecuta git en `.cache/repos` y árboles de trabajo temporales, luego informa:

- conflictos de fusión proyectados desde `git merge-tree`;
- simulación de rebase en un árbol de trabajo temporal;
- commits equivalentes en parches desde `git cherry`;
- movimiento semántico desde `git range-diff`;
- commits del fork por delante ya cubiertos upstream;
- manuales de operación seguros para agentes de fusión/rebase.

## CLI

El CLI es el mejor punto de entrada para scripts y agentes de programación.

```bash
npm run argus -- --help
```

### Revisión de PR

```bash
npm run argus -- review owner/repo#123
npm run argus -- review owner/repo#123 --policy .codebase-argus.yml
npm run argus -- review owner/repo#123 --provider openai-api --model gpt-4.1-mini
npm run argus -- review owner/repo#123 --tribunal openai-api,claude-cli,codex-cli
```

### Revisión de CI

```bash
npm run argus -- ci-log logs/failure.txt
npm run argus -- ci-log logs/failure.txt --provider codex-cli
GITHUB_TOKEN=... npm run argus -- ci-github owner/repo#123
```

### Plan de autofix

```bash
npm run argus -- autofix-plan owner/repo#123
```

### Sincronización de fork descendente

```bash
npm run argus -- downstream owner/upstream me/fork
npm run argus -- downstream owner/upstream me/fork --upstream-branch main --fork-branch feature/demo
npm run argus -- downstream owner/upstream me/fork --fork-branch feature/demo --provider codex-cli
```

### Planificación de sincronización

```bash
npm run argus -- sync owner/upstream me/fork --mode merge --fork-branch feature/demo --test "npm test"
npm run argus -- sync owner/upstream me/fork --mode rebase --fork-branch feature/demo --execute --push --create-pr
```

La salida predeterminada es markdown. Usa `--format json` para integración de herramientas.

Instala el binario localmente:

```bash
npm link
codebase-argus review owner/repo#123
codebase-argus autofix-plan owner/repo#123
```

`downstream` es el comando principal de revisión de sincronización de fork. `sync` está reservado para
ramas de integración explícitas.

## Archivo de política

Agrega `.codebase-argus.yml` cuando el repositorio tenga reglas de revisión locales:

```yaml
requiredChecks: passing
maxChangedFiles: 30
maxTotalDelta: 1200
requiredTestPatterns:
  - .test.ts
forbiddenWorkflowPatterns:
  - pull_request_target
sensitivePathPatterns:
  - auth
  - token
  - webhook
```

Los fallos de política se convierten en hallazgos normales con evidencia concreta.

## GitHub App

Despliega el servidor Next.js y apunta un webhook de GitHub App a:

```text
POST https://your-host.example.com/api/github/webhook
```

El servidor también emite un manifiesto de GitHub App:

```text
GET https://your-host.example.com/api/github/app-manifest
```

Permisos de repositorio recomendados:

| Permiso | Acceso |
| --- | --- |
| Pull requests | Lectura y escritura |
| Issues | Lectura y escritura |
| Contents | Lectura |
| Checks | Lectura |
| Actions | Lectura |
| Metadata | Lectura |

Eventos de webhook requeridos:

- `pull_request`
- `issue_comment`

Entorno del servidor:

```bash
GITHUB_WEBHOOK_SECRET=...
GITHUB_APP_ID=...
GITHUB_APP_PRIVATE_KEY='<escaped-pem-private-key>'
```

También se admite el almacenamiento de clave privada en Base64:

```bash
GITHUB_APP_PRIVATE_KEY_BASE64=...
```

Controles de revisión:

```bash
ARGUS_WEBHOOK_PROVIDER=rule-based
ARGUS_WEBHOOK_PROVIDER=openai-api
ARGUS_WEBHOOK_MODEL=gpt-4.1-mini
ARGUS_WEBHOOK_TRIBUNAL=openai-api,claude-cli,codex-cli
ARGUS_WEBHOOK_INLINE_COMMENTS=true
ARGUS_WEBHOOK_INCLUDE_CI_LOGS=true
ARGUS_WEBHOOK_DRY_RUN=true
```

Comportamiento del webhook:

- verifica `X-Hub-Signature-256` antes del manejo de la carga útil;
- revisa eventos `opened`, `reopened`, `ready_for_review` y `synchronize`;
- omite PR de borrador y PR con la etiqueta `argus:paused`;
- usa tokens de instalación de GitHub App cuando están presentes las credenciales de la app;
- publica revisiones de GitHub PR con el evento `COMMENT`;
- ancla hallazgos de alta señal a líneas de parche modificadas cuando los comentarios en línea están habilitados;
- obtiene registros de trabajos de GitHub Actions fallidos cuando fallan las comprobaciones.

### Comandos de comentario en PR

```text
/argus help
/argus review
/argus ci
/argus autofix
/argus pause
/argus resume
```

`/argus pause` aplica la etiqueta `argus:paused`. `/argus resume` la elimina.
`/argus autofix` publica el mismo plan con controles que el CLI.

## Configuración de proveedores de IA

Configura las credenciales para los proveedores que planeas usar:

```bash
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
GEMINI_API_KEY=...
```

Sobrescrituras opcionales de modelo:

```bash
ARGUS_OPENAI_API_MODEL=gpt-4.1-mini
ARGUS_ANTHROPIC_API_MODEL=claude-3-5-sonnet-20241022
ARGUS_GEMINI_API_MODEL=gemini-2.0-flash
```

Los proveedores de CLI locales esperan comandos autenticados:

```bash
codex exec --help
claude --help
gemini --help
```

## Playbook de agentes

El repositorio incluye un playbook de agente portátil. No está vinculado a Codex:
OpenClaw, Codex, Claude Code y otros agentes de programación pueden usar las mismas
instrucciones.

```text
agent-playbooks/codebase-argus/PLAYBOOK.md   # Playbook portátil para OpenClaw / Codex / Claude Code
skills/codebase-argus/SKILL.md               # Punto de entrada de habilidad compatible con ClawHub/OpenClaw
```

Configuración recomendada:

- OpenClaw: agrega `agent-playbooks/codebase-argus/PLAYBOOK.md` a las instrucciones del agente o
  del proyecto, o instala la habilidad desde `skills/codebase-argus/`.
- Codex: lee el playbook portátil directamente o instala la habilidad
  desde `skills/codebase-argus/`.
- Claude Code: instala el marketplace de plugins desde este repositorio:

  ```text
  /plugin marketplace add AaronZ345/codebase-argus
  /plugin install codebase-argus@codebase-argus
  ```

  O copia el playbook en `.claude/skills/codebase-argus/SKILL.md` para una
  habilidad local al proyecto.

## Registros de habilidades

Codebase Argus está empaquetado para registros abiertos de `SKILL.md` y superficies de instalación específicas de agente:

- Marketplace de plugins de Claude Code: `AaronZ345/codebase-argus`
- Carpeta de habilidades ClawHub/OpenClaw: `skills/codebase-argus`
- Carpeta de Habilidades de Agente compatible con Codex/OpenAI: `skills/codebase-argus`
- Repositorio de registro público SkillsMD: `AaronZ345/codebase-argus`

Comando de publicación de ClawHub:

```bash
clawhub skill publish skills/codebase-argus \
  --slug codebase-argus \
  --name "Codebase Argus" \
  --version 0.1.0 \
  --tags latest,code-review,pull-request,ci,multi-agent,fork-sync
```

Carga útil de envío de SkillsMD:

```json
{
  "repo": "AaronZ345/codebase-argus",
  "name": "codebase-argus",
  "description": "Escritorio de revisión multiagente de PR, CI y sincronización de forks descendentes para agentes de programación."
}
```

El playbook dirige a los agentes a usar el CLI primero, mantener los tokens fuera de los registros, ejecutar
revisión multi-proveedor antes de integraciones descendentes arriesgadas, y solicitar autorización explícita antes de aprobar, fusionar, hacer rebase, push, creación de PR o comentarios en GitHub.

## Modelo de escritura

Codebase Argus mantiene las operaciones de escritura estrechas:

| Superficie | Comportamiento de escritura |
| --- | --- |
| Demo alojada | Inspección de navegador de solo lectura |
| Revisión de CLI local | Salida en Markdown o JSON |
| Revisión de GitHub App | Revisiones de PR con `COMMENT` |
| Comandos de PR | Revisión, revisión de CI, plan de autofix, pausa, reanudación |
| Comando de sincronización | Prueba seca por defecto; `--execute`, `--push` y `--create-pr` son controles explícitos |
| Flujo de Actions generado | Usa `pull_request` para PR de forks no confiables |

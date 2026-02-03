# GITHUB ACTIONS: ARQUITECTURA Y FUNCIONAMIENTO TÉCNICO

## 📚 TABLA DE CONTENIDOS

1. [¿Qué es GitHub Actions? - Fundamentos](#1-qué-es-github-actions---fundamentos)
2. [Arquitectura del Sistema](#2-arquitectura-del-sistema)
3. [Ciclo de Vida Completo](#3-ciclo-de-vida-completo)
4. [Sistema de Eventos](#4-sistema-de-eventos)
5. [Runners: La Infraestructura de Ejecución](#5-runners-la-infraestructura-de-ejecución)
6. [Contextos: El Sistema de Variables](#6-contextos-el-sistema-de-variables)
7. [Expresiones y Motor de Evaluación](#7-expresiones-y-motor-de-evaluación)
8. [Sistema de Almacenamiento](#8-sistema-de-almacenamiento)
9. [Seguridad y Aislamiento](#9-seguridad-y-aislamiento)
10. [Networking y Comunicación](#10-networking-y-comunicación)

---

## 1. ¿QUÉ ES GITHUB ACTIONS? - FUNDAMENTOS

### 1.1 Definición Técnica

GitHub Actions es un **sistema de automatización distribuido basado en eventos** que se ejecuta en la infraestructura de GitHub. Técnicamente es:

- Un **orquestador de tareas** (workflow orchestrator)
- Un **sistema event-driven** (reactivo a eventos)
- Una **plataforma de CI/CD** (Integración Continua/Despliegue Continuo)
- Un **motor de ejecución de contenedores** (runner system)

### 1.2 Componentes Principales

```
┌─────────────────────────────────────────────────────────────┐
│                      GITHUB.COM                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │  1. REPOSITORIO (tu código + workflows)            │     │
│  └────────────────────────────────────────────────────┘     │
│                          ↓                                   │
│  ┌────────────────────────────────────────────────────┐     │
│  │  2. EVENT SYSTEM (detecta cambios/acciones)        │     │
│  └────────────────────────────────────────────────────┘     │
│                          ↓                                   │
│  ┌────────────────────────────────────────────────────┐     │
│  │  3. WORKFLOW ENGINE (procesa .yml, decide qué      │     │
│  │     ejecutar)                                      │     │
│  └────────────────────────────────────────────────────┘     │
│                          ↓                                   │
│  ┌────────────────────────────────────────────────────┐     │
│  │  4. JOB SCHEDULER (asigna jobs a runners)          │     │
│  └────────────────────────────────────────────────────┘     │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ↓
┌──────────────────────────────────────────────────────────────┐
│              5. RUNNERS (máquinas que ejecutan)              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Runner 1    │  │ Runner 2    │  │ Runner N    │          │
│  │ (Ubuntu)    │  │ (Windows)   │  │ (macOS)     │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. ARQUITECTURA DEL SISTEMA

### 2.1 Jerarquía de Componentes

```
REPOSITORIO
    │
    └─── .github/workflows/
            │
            ├─── workflow1.yml  ← WORKFLOW (archivo de configuración)
            │       │
            │       ├─── on:    ← TRIGGERS (cuándo ejecutar)
            │       │
            │       └─── jobs:  ← JOBS (trabajos independientes)
            │               │
            │               ├─── job1:
            │               │      ├─── runs-on:  ← RUNNER (dónde ejecutar)
            │               │      └─── steps:    ← STEPS (comandos)
            │               │             ├─── step1
            │               │             ├─── step2
            │               │             └─── step3
            │               │
            │               └─── job2:
            │                      ├─── needs: [job1]  ← DEPENDENCIAS
            │                      └─── steps: [...]
            │
            └─── workflow2.yml
```

### 2.2 Relación Entre Componentes

**WORKFLOW = Orquestación completa**
- Es un archivo YAML
- Define CUÁNDO, DÓNDE y QUÉ ejecutar
- Puede tener múltiples JOBS

**JOB = Unidad de ejecución independiente**
- Se ejecuta en UN runner (una máquina)
- Contiene múltiples STEPS
- Puede depender de otros jobs (secuencial) o ejecutarse en paralelo

**STEP = Acción atómica**
- Ejecuta UN comando o UNA action
- Comparte el filesystem con otros steps del mismo job
- Se ejecuta secuencialmente dentro del job

**RUNNER = Máquina física/virtual**
- Ambiente limpio para cada job
- Sistema operativo específico (Ubuntu, Windows, macOS)
- Ejecuta los comandos reales

---

## 3. CICLO DE VIDA COMPLETO

### 3.1 Flujo Detallado

```
FASE 1: DETECCIÓN DE EVENTO
───────────────────────────────
Usuario hace: git push
    ↓
GitHub detecta el evento "push"
    ↓
GitHub genera un PAYLOAD (objeto JSON con toda la info del evento)
    ↓
Payload contiene:
  - Qué tipo de evento es (push)
  - Quién lo hizo (autor)
  - Qué cambió (commits, archivos)
  - Contexto del repo (branch, SHA, etc.)


FASE 2: EVALUACIÓN DE WORKFLOWS
────────────────────────────────
GitHub busca en .github/workflows/*.yml
    ↓
Para cada archivo .yml:
  ┌─ Lee el campo "on:"
  ┌─ ¿Este workflow escucha el evento "push"?
  │   ├─ NO → Ignora este workflow
  │   └─ SÍ → Continúa evaluación
  │
  ┌─ ¿Hay filtros (branches, paths)?
  │   └─ Evalúa si el push cumple las condiciones
  │
  └─ SI TODO CUMPLE → Encola el workflow para ejecución


FASE 3: CREACIÓN DE WORKFLOW RUN
─────────────────────────────────
GitHub crea una "Workflow Run" (instancia de ejecución)
    ↓
Asigna un ID único: run_id
    ↓
Estado inicial: "queued"
    ↓
Genera el contexto global (github.*, env.*, etc.)


FASE 4: PLANIFICACIÓN DE JOBS
──────────────────────────────
Lee la sección "jobs:" del workflow
    ↓
Analiza dependencias (needs:)
    ↓
Crea un grafo de ejecución:
  job1 (sin dependencias) → puede ejecutar YA
  job2 (needs: job1)      → espera a que job1 termine
  job3 (sin dependencias) → puede ejecutar en PARALELO con job1


FASE 5: ASIGNACIÓN DE RUNNERS
──────────────────────────────
Para cada job listo para ejecutar:
    ↓
Lee "runs-on:" (ej: ubuntu-latest)
    ↓
Busca un runner disponible con ese OS
    ↓
SI HAY RUNNER LIBRE:
  └─ Asigna el job al runner
     Estado del job: "in_progress"
SINO:
  └─ Job queda en cola
     Estado del job: "queued"


FASE 6: EJECUCIÓN EN EL RUNNER
───────────────────────────────
El runner recibe el job
    ↓
1. SETUP INICIAL
   - Crea un directorio de trabajo limpio
   - Descarga las herramientas del sistema (node, python, etc.)
   - Prepara variables de entorno
   
2. SETUP DE ACTIONS (si usa actions/checkout@v4, etc.)
   - Descarga el código de la action desde su repo
   - Instala dependencias de la action
   
3. EJECUCIÓN STEP BY STEP
   Paso 1: actions/checkout@v4
     ↓
   - Clona tu repositorio en el runner
   - Checkout al commit específico (SHA del evento)
   
   Paso 2: run: npm install
     ↓
   - Abre una shell (bash/powershell)
   - Ejecuta el comando
   - Captura stdout, stderr, exit code
   
   Paso 3: run: npm test
     ↓
   - Ejecuta en la MISMA máquina (filesystem compartido)
   - Si exit code != 0 → FALLA
   
4. LIMPIEZA
   - Sube artifacts (si hay)
   - Sube logs
   - Destruye el ambiente


FASE 7: REPORTE DE RESULTADOS
──────────────────────────────
Runner envía resultado a GitHub:
  - Estado: success / failure / cancelled
  - Logs completos
  - Duración
    ↓
GitHub actualiza el estado del job
    ↓
Si era el último job → Workflow completo
    ↓
Notificaciones:
  - Checks en el commit (✓ o ✗)
  - Emails (si configurado)
  - Webhooks (si configurado)
```

### 3.2 Estados del Workflow

```
┌─────────┐
│ queued  │  ← Esperando un runner
└────┬────┘
     │
     ↓
┌─────────────┐
│ in_progress │  ← Ejecutándose
└────┬────────┘
     │
     ├─────────────────┬─────────────────┐
     ↓                 ↓                 ↓
┌─────────┐    ┌──────────┐    ┌───────────┐
│ success │    │ failure  │    │ cancelled │
└─────────┘    └──────────┘    └───────────┘
```

---

## 4. SISTEMA DE EVENTOS

### 4.1 ¿Qué es un Evento?

Un **evento** es una **señal que algo sucedió en GitHub**. Técnicamente:

1. **Origen**: Proviene de la API de GitHub (GitHub detecta una acción)
2. **Naturaleza**: Es un webhook interno
3. **Payload**: Objeto JSON con toda la información del evento
4. **Propagación**: Se envía al sistema de workflows

### 4.2 ¿Quién Genera los Eventos?

**RESPUESTA: GitHub.com (el servidor)**

Ejemplos concretos:

```
CASO 1: git push
────────────────
TÚ (usuario local):
  $ git push origin main
      ↓
TU MÁQUINA:
  Envía los commits al servidor de GitHub
      ↓
GITHUB.COM (servidor):
  1. Recibe los commits
  2. Actualiza la base de datos del repositorio
  3. GENERA EVENTO "push"
  4. Crea un payload:
     {
       "event": "push",
       "ref": "refs/heads/main",
       "commits": [...],
       "pusher": {"name": "tu-usuario"},
       ...
     }
  5. Envía el evento al sistema de Workflow Engine
```

```
CASO 2: Abrir un Pull Request
──────────────────────────────
TÚ:
  Clickeas "Create Pull Request" en GitHub web
      ↓
GITHUB.COM:
  1. Crea el PR en la base de datos
  2. GENERA EVENTO "pull_request" con action "opened"
  3. Payload:
     {
       "event": "pull_request",
       "action": "opened",
       "pull_request": {
         "number": 123,
         "title": "...",
         "head": {"ref": "feature-branch"},
         ...
       }
     }
  4. Dispara workflows que escuchan "pull_request"
```

```
CASO 3: Schedule (cron)
───────────────────────
GITHUB.COM:
  Tiene un scheduler interno (similar a cron)
      ↓
  Cada minuto revisa:
  "¿Hay workflows con 'on: schedule' que deben ejecutarse ahora?"
      ↓
  SI HAY:
    1. GENERA EVENTO "schedule"
    2. Payload:
       {
         "event": "schedule",
         "schedule": "0 0 * * *"
       }
    3. Ejecuta el workflow
```

### 4.3 Tipos de Eventos y su Origen

| Evento | Origen | Quién lo dispara |
|--------|--------|------------------|
| `push` | API Git | `git push` desde cualquier máquina |
| `pull_request` | GitHub Web/API | Crear/actualizar PR en GitHub.com |
| `issues` | GitHub Web/API | Abrir/cerrar issue en GitHub.com |
| `schedule` | GitHub Scheduler | Reloj interno de GitHub |
| `workflow_dispatch` | GitHub Web/API | Usuario clickea "Run workflow" |
| `release` | GitHub Web/API | Crear release en GitHub.com |
| `fork` | GitHub Web | Alguien forkea tu repo |

### 4.4 Anatomía de un Payload de Evento

**Ejemplo real de evento `push`:**

```json
{
  "ref": "refs/heads/main",
  "before": "abc123...",
  "after": "def456...",
  "repository": {
    "id": 123456,
    "name": "mi-repo",
    "full_name": "usuario/mi-repo",
    "owner": {
      "login": "usuario",
      "id": 789
    }
  },
  "pusher": {
    "name": "dukono",
    "email": "dukono@users.noreply.github.com"
  },
  "sender": {
    "login": "dukono",
    "id": 71391337
  },
  "commits": [
    {
      "id": "def456...",
      "message": "Add feature",
      "timestamp": "2026-02-02T12:00:00Z",
      "author": {
        "name": "Bill Gates",
        "email": "bill@microsoft.com"
      },
      "committer": {
        "name": "dukono",
        "email": "dukono@users.noreply.github.com"
      }
    }
  ],
  "head_commit": { ... },
  "compare": "https://github.com/usuario/mi-repo/compare/abc123...def456"
}
```

**Este payload está disponible en:**
- `${{ github.event }}` (todo el objeto)
- `${{ github.event.pusher.name }}` (navegación por propiedades)
- Archivo físico: `$GITHUB_EVENT_PATH` (JSON file en el runner)

---

## 5. RUNNERS: LA INFRAESTRUCTURA DE EJECUCIÓN

### 5.1 ¿Qué es un Runner?

Un **runner** es una **máquina (física o virtual) que ejecuta los jobs**. Técnicamente:

```
RUNNER = Máquina + Software Agente

┌─────────────────────────────────────────┐
│         RUNNER (Máquina Virtual)        │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │  SISTEMA OPERATIVO                │  │
│  │  - Ubuntu 22.04                   │  │
│  │  - Windows Server 2022            │  │
│  │  - macOS 12                       │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │  GITHUB ACTIONS RUNNER (agente)   │  │
│  │  - Se conecta a GitHub.com        │  │
│  │  - Pregunta: "¿hay jobs para mí?" │  │
│  │  - Ejecuta los jobs               │  │
│  │  - Reporta resultados             │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │  HERRAMIENTAS PRE-INSTALADAS      │  │
│  │  - git, curl, wget                │  │
│  │  - Node.js, Python, Java          │  │
│  │  - Docker                         │  │
│  │  - Compiladores (gcc, g++)        │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │  FILESYSTEM (para tu job)         │  │
│  │  /home/runner/work/               │  │
│  │    └── repo-name/                 │  │
│  │        └── repo-name/  ← tu código│  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 5.2 GitHub-Hosted vs Self-Hosted

**GitHub-Hosted Runners:**
- **Quién los mantiene**: GitHub
- **Dónde están**: Azure (Microsoft)
- **Costo**: Incluidos en el plan (límites de minutos)
- **Limpieza**: Máquina nueva para cada job
- **Especificaciones**:
  - 2 CPUs, 7 GB RAM (Linux/Windows)
  - 3 CPUs, 14 GB RAM (macOS)

**Self-Hosted Runners:**
- **Quién los mantiene**: Tú
- **Dónde están**: Tu infraestructura (servidor, VPS, Raspberry Pi)
- **Costo**: Gratis (pagas la infraestructura)
- **Limpieza**: Debes limpiar manualmente
- **Especificaciones**: Las que tú decidas

### 5.3 Ciclo de Vida de un Runner

```
INICIO DEL JOB
──────────────
1. Runner disponible en el pool
2. GitHub asigna job al runner
3. Runner cambia estado a "busy"

SETUP
─────
4. Crea directorio: /home/runner/work/repo-name/repo-name
5. Descarga herramientas necesarias
6. Configura variables de entorno:
   - GITHUB_WORKSPACE=/home/runner/work/repo-name/repo-name
   - GITHUB_REPOSITORY=usuario/repo-name
   - GITHUB_SHA=abc123...
   - GITHUB_REF=refs/heads/main
   - ... (100+ variables)

EJECUCIÓN
─────────
7. Para cada step:
   a) Si es "uses: actions/..." → Descarga y ejecuta la action
   b) Si es "run: ..." → Abre shell y ejecuta
   c) Captura stdout/stderr en tiempo real
   d) Envía logs a GitHub.com
   e) Si falla (exit code != 0):
      - Marca step como failed
      - Por defecto, detiene el job (a menos que continue-on-error: true)

LIMPIEZA
────────
8. Sube artifacts a GitHub (si hay)
9. Sube cache entries (si hay)
10. Destruye el directorio de trabajo
11. En GitHub-hosted: Destruye la VM completa
12. Runner vuelve al estado "idle" (esperando nuevo job)
```

### 5.4 Aislamiento Entre Jobs

```
JOB 1                          JOB 2
Runner: ubuntu-latest-1        Runner: ubuntu-latest-2
VM: 10.0.1.100                VM: 10.0.1.101
Filesystem independiente      Filesystem independiente
Variables independientes      Variables independientes
```

**NO SE COMPARTE NADA entre jobs**, excepto:
- Artifacts (explícitamente subidos/descargados)
- Cache (explícitamente guardado/restaurado)
- Outputs (definidos con `outputs:`)

---

## 6. CONTEXTOS: EL SISTEMA DE VARIABLES

### 6.1 ¿Qué es un Contexto?

Un **contexto** es un **objeto JSON que contiene información** disponible en diferentes etapas del workflow.

**Analogía**: Son como "variables globales" que GitHub inyecta en tu workflow.

### 6.2 Dónde se Crean los Contextos

```
TIMELINE DE CREACIÓN
────────────────────

T0: Usuario hace git push
    ↓
T1: GitHub genera EVENTO
    ↓
T2: GitHub crea WORKFLOW RUN
    ├─ Se crea contexto "github" (info del evento, repo, etc.)
    ├─ Se crea contexto "env" (variables globales del workflow)
    ├─ Se crea contexto "secrets" (acceso a secrets del repo)
    └─ Se crea contexto "vars" (variables de configuración)
    ↓
T3: GitHub planifica JOB 1
    ├─ Se crea contexto "strategy" (si hay matrix)
    ├─ Se crea contexto "matrix" (valores actuales del matrix)
    └─ Se crea contexto "needs" (outputs de jobs anteriores)
    ↓
T4: Runner empieza a ejecutar JOB 1
    ├─ Se crea contexto "runner" (info del runner)
    └─ Se crea contexto "job" (info del job actual)
    ↓
T5: Se ejecuta STEP 1
    ├─ Se actualiza contexto "steps" (outputs de steps anteriores)
    └─ Se crea contexto "inputs" (si es workflow_call o workflow_dispatch)
```

### 6.3 Contextos Principales

#### 6.3.1 Contexto `github`

**Contiene**: Información del evento, repositorio, workflow

**Ejemplo real**:
```yaml
name: Debug Context
on: push
jobs:
  debug:
    runs-on: ubuntu-latest
    steps:
      - name: Ver evento
        run: |
          echo "Evento: ${{ github.event_name }}"
          # Output: push
          
          echo "Branch: ${{ github.ref }}"
          # Output: refs/heads/main
          
          echo "SHA: ${{ github.sha }}"
          # Output: def456789abcdef...
          
          echo "Quien hizo push: ${{ github.actor }}"
          # Output: dukono
          
          echo "Repositorio: ${{ github.repository }}"
          # Output: usuario/mi-repo
```

**Propiedades importantes**:
```javascript
github = {
  event_name: "push",           // Tipo de evento
  event: { /* payload completo */ },  // Todo el JSON del evento
  sha: "def456...",             // Commit SHA que disparó el workflow
  ref: "refs/heads/main",       // Referencia (branch/tag)
  ref_name: "main",             // Nombre limpio del branch
  repository: "usuario/repo",   // Repo completo
  repository_owner: "usuario",  // Dueño del repo
  actor: "dukono",              // Usuario que disparó el evento
  workflow: "CI",               // Nombre del workflow
  run_id: "123456789",          // ID único de esta ejecución
  run_number: "42",             // Número secuencial de ejecución
  job: "build",                 // ID del job actual
  action: "actions/checkout",   // Action actual (si aplica)
  workspace: "/home/runner/work/repo/repo"  // Directorio de trabajo
}
```

#### 6.3.2 Contexto `env`

**Contiene**: Variables de entorno definidas en el workflow

```yaml
env:
  GLOBAL_VAR: "valor1"

jobs:
  test:
    env:
      JOB_VAR: "valor2"
    steps:
      - name: Usar variables
        env:
          STEP_VAR: "valor3"
        run: |
          echo "${{ env.GLOBAL_VAR }}"  # valor1
          echo "${{ env.JOB_VAR }}"     # valor2
          echo "${{ env.STEP_VAR }}"    # valor3
          
          # También disponibles como env vars normales:
          echo "$GLOBAL_VAR"            # valor1
          echo "$JOB_VAR"               # valor2
          echo "$STEP_VAR"              # valor3
```

**Alcance (scope)**:
```
env: (nivel workflow)
  └─ Disponible en TODOS los jobs y steps

jobs:
  test:
    env: (nivel job)
      └─ Disponible solo en este job
      
    steps:
      - env: (nivel step)
          └─ Disponible solo en este step
```

#### 6.3.3 Contexto `secrets`

**Contiene**: Secrets configurados en GitHub

**Dónde se configuran**:
```
GitHub.com → Tu Repo → Settings → Secrets and variables → Actions
```

**Uso**:
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          # Los secrets NO se imprimen en logs (GitHub los oculta)
          deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

**IMPORTANTE**: Los secrets están **encriptados** y GitHub los **oculta automáticamente** en los logs:
```
# En tu script:
echo "API Key: $API_KEY"

# En los logs verás:
API Key: ***
```

#### 6.3.4 Contexto `steps`

**Contiene**: Outputs de steps anteriores

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Step 1
        id: primer-step  # ← ID obligatorio para referenciarlo
        run: |
          echo "resultado=exitoso" >> $GITHUB_OUTPUT
          echo "numero=42" >> $GITHUB_OUTPUT
      
      - name: Step 2
        run: |
          # Acceder a outputs del step anterior:
          echo "Resultado: ${{ steps.primer-step.outputs.resultado }}"
          # Output: exitoso
          
          echo "Número: ${{ steps.primer-step.outputs.numero }}"
          # Output: 42
```

**Cómo funciona técnicamente**:
1. `$GITHUB_OUTPUT` es un archivo temporal en el runner
2. Cuando escribes `echo "key=value" >> $GITHUB_OUTPUT`
3. GitHub lee ese archivo al final del step
4. Crea `steps.primer-step.outputs.key = "value"`
5. Lo hace disponible para steps posteriores

#### 6.3.5 Contexto `needs`

**Contiene**: Outputs de jobs anteriores

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs:
      resultado: ${{ steps.calculo.outputs.resultado }}
    steps:
      - id: calculo
        run: echo "resultado=100" >> $GITHUB_OUTPUT
  
  job2:
    needs: job1  # ← Declara dependencia
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Resultado de job1: ${{ needs.job1.outputs.resultado }}"
          # Output: 100
```

**Múltiples dependencias**:
```yaml
jobs:
  build:
    outputs:
      version: ${{ steps.ver.outputs.version }}
    steps:
      - id: ver
        run: echo "version=1.2.3" >> $GITHUB_OUTPUT
  
  test:
    outputs:
      status: ${{ steps.test.outputs.status }}
    steps:
      - id: test
        run: echo "status=passed" >> $GITHUB_OUTPUT
  
  deploy:
    needs: [build, test]  # ← Depende de AMBOS
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Version: ${{ needs.build.outputs.version }}"
          echo "Tests: ${{ needs.test.outputs.status }}"
```

#### 6.3.6 Contexto `runner`

**Contiene**: Información del runner ejecutando el job

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "OS: ${{ runner.os }}"           # Linux
          echo "Arch: ${{ runner.arch }}"       # X64
          echo "Name: ${{ runner.name }}"       # GitHub Actions 2
          echo "Tool cache: ${{ runner.tool_cache }}"  # /opt/hostedtoolcache
```

#### 6.3.7 Contexto `matrix`

**Contiene**: Valores actuales en una estrategia matrix

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [14, 16, 18]
    runs-on: ${{ matrix.os }}
    steps:
      - run: |
          echo "OS: ${{ matrix.os }}"
          echo "Node: ${{ matrix.node }}"
```

**Técnicamente, crea 6 jobs (3×2)**:
```
Job 1: os=ubuntu-latest, node=14
Job 2: os=ubuntu-latest, node=16
Job 3: os=ubuntu-latest, node=18
Job 4: os=windows-latest, node=14
Job 5: os=windows-latest, node=16
Job 6: os=windows-latest, node=18
```

### 6.4 Tabla Resumen de Contextos

| Contexto | Disponible en | Contiene | Ejemplo |
|----------|--------------|----------|---------|
| `github` | Todos lados | Info del evento/repo | `github.sha` |
| `env` | Todos lados | Variables de entorno | `env.NODE_ENV` |
| `secrets` | Todos lados | Secrets del repo | `secrets.API_KEY` |
| `vars` | Todos lados | Variables de configuración | `vars.ENVIRONMENT` |
| `job` | En el job | Info del job actual | `job.status` |
| `steps` | En steps posteriores | Outputs de steps previos | `steps.build.outputs.version` |
| `runner` | En el job | Info del runner | `runner.os` |
| `needs` | En jobs dependientes | Outputs de jobs previos | `needs.build.outputs.tag` |
| `strategy` | En jobs con matrix | Config de la estrategia | `strategy.fail-fast` |
| `matrix` | En jobs con matrix | Valores actuales del matrix | `matrix.os` |
| `inputs` | En workflow_dispatch/call | Inputs del usuario | `inputs.environment` |

---

## 7. EXPRESIONES Y MOTOR DE EVALUACIÓN

### 7.1 ¿Qué son las Expresiones?

Las expresiones son **código evaluado por GitHub** antes de enviar el job al runner.

**Sintaxis**: `${{ ... }}`

**Dónde se evalúan**: En los servidores de GitHub, NO en el runner.

### 7.2 Momento de Evaluación

```
TIMELINE
────────

T1: GitHub recibe el evento
    ↓
T2: GitHub lee tu workflow.yml
    ↓
T3: GitHub EVALÚA las expresiones ${{ ... }}
    │
    ├─ Reemplaza ${{ github.ref }} por "refs/heads/main"
    ├─ Evalúa if: ${{ github.event_name == 'push' }}
    └─ Genera el YAML final con valores concretos
    ↓
T4: Envía el YAML procesado al runner
    ↓
T5: Runner ejecuta comandos (ya no hay ${{ ... }}, solo valores)
```

**Ejemplo**:

```yaml
# TU ESCRIBES:
jobs:
  test:
    if: ${{ github.ref == 'refs/heads/main' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Branch: ${{ github.ref }}"

# GITHUB EVALÚA (antes de enviar al runner):
# Supongamos github.ref = "refs/heads/main"

# RESULTADO:
jobs:
  test:
    if: true  # ← Evaluado a boolean
    runs-on: ubuntu-latest
    steps:
      - run: echo "Branch: refs/heads/main"  # ← Reemplazado
```

### 7.3 Diferencia: Expresiones vs Variables de Entorno

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # EXPRESIÓN (evaluada por GitHub):
      - run: echo "${{ github.sha }}"
        # GitHub reemplaza ANTES de ejecutar
        # Runner recibe: echo "abc123..."
      
      # VARIABLE DE ENTORNO (evaluada por el shell):
      - run: echo "$GITHUB_SHA"
        # Runner recibe: echo "$GITHUB_SHA"
        # Bash reemplaza al ejecutar
```

**Ambas dan el mismo resultado, pero el proceso es diferente**:
- `${{ github.sha }}`: GitHub lo procesa → Runner recibe valor final
- `$GITHUB_SHA`: Runner recibe la variable → Shell la expande

### 7.4 Funciones Disponibles

#### Comparación
```yaml
${{ github.ref == 'refs/heads/main' }}           # Igualdad
${{ github.event_name != 'push' }}               # Desigualdad
${{ github.run_number > 100 }}                   # Mayor que
${{ github.actor == 'dukono' || github.actor == 'admin' }}  # OR
${{ github.ref == 'refs/heads/main' && github.event_name == 'push' }}  # AND
```

#### Funciones de String
```yaml
${{ contains(github.ref, 'feature') }}           # Contiene substring
${{ startsWith(github.ref, 'refs/heads/') }}     # Empieza con
${{ endsWith(github.ref, '/main') }}             # Termina con
${{ format('Version: {0}.{1}', '1', '2') }}      # Formato (Output: Version: 1.2)
```

#### Funciones de Estado
```yaml
${{ success() }}       # Step anterior exitoso
${{ failure() }}       # Step anterior falló
${{ cancelled() }}     # Workflow cancelado
${{ always() }}        # Siempre (ignora estado)
```

**Uso común**:
```yaml
steps:
  - name: Test
    run: npm test
  
  - name: Notify on failure
    if: ${{ failure() }}  # Solo si el step anterior falló
    run: echo "Tests failed!"
  
  - name: Cleanup
    if: ${{ always() }}   # Siempre se ejecuta, incluso si falló
    run: rm -rf temp/
```

#### Funciones JSON
```yaml
${{ toJSON(github.event) }}      # Convierte objeto a JSON string
${{ fromJSON('{"key": "value"}') }}  # Parse JSON string a objeto
```

**Ejemplo práctico**:
```yaml
steps:
  - name: Ver evento completo
    run: echo '${{ toJSON(github.event) }}'
    # Imprime todo el payload del evento en JSON
```

### 7.5 Valores por Defecto

```yaml
${{ github.event.pull_request.title || 'No PR title' }}
# Si no hay PR, usa el valor por defecto
```

---

## 8. SISTEMA DE ALMACENAMIENTO

### 8.1 Artifacts

**Qué son**: Archivos generados durante el workflow que quieres conservar.

**Ejemplos**: Binarios compilados, logs, reportes, capturas de pantalla.

**Dónde se guardan**: Servidores de GitHub (no en el runner).

**Cuánto duran**: 90 días por defecto (configurable).

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Compilar
        run: gcc main.c -o app
      
      - name: Subir binario
        uses: actions/upload-artifact@v4
        with:
          name: mi-aplicacion
          path: app
  
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Descargar binario
        uses: actions/download-artifact@v4
        with:
          name: mi-aplicacion
      
      - name: Ejecutar
        run: ./app
```

**Flujo técnico**:
```
Job Build (Runner 1)
    ↓
  Genera archivo "app"
    ↓
  upload-artifact envía a GitHub.com
    ↓
GitHub almacena en su storage (S3/Azure)
    ↓
Job Test (Runner 2) - Máquina completamente diferente
    ↓
  download-artifact descarga desde GitHub.com
    ↓
  Archivo "app" disponible en el nuevo runner
```

### 8.2 Cache

**Qué es**: Sistema para reutilizar dependencias entre ejecuciones.

**Ejemplos**: node_modules, pip packages, Maven dependencies.

**Cuánto dura**: Hasta 7 días sin uso (o hasta 10 GB por repo).

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Cache node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
      
      - name: Install
        run: npm install  # Solo si no hay cache
```

**Flujo técnico**:
```
Primera ejecución:
  1. cache@v4 busca key "Linux-node-abc123..."
  2. No existe → cache miss
  3. npm install descarga todo (2 minutos)
  4. cache@v4 guarda node_modules con esa key

Segunda ejecución (mismo package-lock.json):
  1. cache@v4 busca key "Linux-node-abc123..."
  2. Existe → cache hit
  3. Descarga node_modules desde cache (10 segundos)
  4. NO ejecuta npm install
```

### 8.3 Diferencia: Artifacts vs Cache

| Característica | Artifacts | Cache |
|----------------|-----------|-------|
| Propósito | Compartir entre jobs | Acelerar builds repetidos |
| Persistencia | 90 días | 7 días sin uso |
| Descarga | Explícita (download-artifact) | Automática (si key coincide) |
| Límite | Ilimitado (pero cuenta en minutos) | 10 GB por repo |

---

## 9. SEGURIDAD Y AISLAMIENTO

### 9.1 Modelo de Seguridad

```
┌─────────────────────────────────────────────────┐
│            GITHUB.COM (Trusted)                 │
│  - Gestiona secrets                             │
│  - Controla permisos (GITHUB_TOKEN)             │
│  - Audita todas las acciones                    │
└─────────────────┬───────────────────────────────┘
                  │ Envía job
                  ↓
┌─────────────────────────────────────────────────┐
│            RUNNER (Untrusted Zone)              │
│  - Ejecuta código del repo (puede ser malicioso)│
│  - Tiene acceso a secrets (si se pasan)         │
│  - Aislado de otros runners                     │
│  - Destruido después del job                    │
└─────────────────────────────────────────────────┘
```

### 9.2 GITHUB_TOKEN

**Qué es**: Token de autenticación automático para cada workflow.

**Creación**: GitHub lo genera automáticamente al iniciar el workflow.

**Permisos**: Configurables, por defecto tiene acceso limitado.

**Uso**:
```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Crear release
        run: |
          gh release create v1.0.0 \
            --title "Version 1.0.0" \
            --notes "Release notes"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Token automático
```

**Permisos por defecto**:
- Leer código: ✅
- Escribir en issues: ✅
- Escribir en PRs: ✅
- Push al repo: ❌ (por defecto)

**Configurar permisos**:
```yaml
permissions:
  contents: write      # Permite push
  issues: read         # Solo lectura de issues
  pull-requests: write # Escribir en PRs

jobs:
  # ...
```

### 9.3 Secretos

**Encriptación**: AES-256 en reposo, TLS en tránsito.

**Enmascaramiento**: GitHub detecta y oculta secrets en logs.

```yaml
steps:
  - name: Usar secreto
    run: |
      echo "Password: ${{ secrets.DB_PASSWORD }}"
      # Logs mostrarán: Password: ***
      
      # NUNCA HAGAS ESTO (bypass del enmascaramiento):
      echo "${{ secrets.DB_PASSWORD }}" | base64
      # Esto expondrá el secreto (en base64 pero visible)
```

### 9.4 Aislamiento de Runners

Cada job en GitHub-hosted runners se ejecuta en una **VM completamente nueva**:

```
Job 1 → VM 10.0.1.100 → Destruida después
Job 2 → VM 10.0.1.101 → Destruida después
Job 3 → VM 10.0.1.102 → Destruida después
```

**No pueden acceder entre sí**:
- Red aislada
- Filesystem independiente
- Procesos independientes

---

## 10. NETWORKING Y COMUNICACIÓN

### 10.1 Comunicación Runner ↔ GitHub

```
┌─────────────┐                          ┌──────────────┐
│   RUNNER    │                          │  GITHUB.COM  │
│             │                          │              │
│ Polling:    │  ← Cada 5 segundos →    │              │
│ "¿Hay jobs?"│ ──────────────────────→ │ Job Queue    │
│             │ ←──────────────────────  │              │
│             │  "Sí, ejecuta job 123"  │              │
│             │                          │              │
│ Durante     │                          │              │
│ ejecución:  │                          │              │
│ - Logs      │ ──────────────────────→ │              │
│ - Status    │ ──────────────────────→ │              │
│ - Artifacts │ ──────────────────────→ │              │
│             │                          │              │
│ Al terminar:│                          │              │
│ - Resultado │ ──────────────────────→ │              │
│ - Exit code │ ──────────────────────→ │              │
└─────────────┘                          └──────────────┘
```

### 10.2 Acceso a Internet desde el Runner

Los runners tienen **acceso completo a internet**:

```yaml
steps:
  - run: curl https://api.example.com/data
    # ✅ Funciona - puede hacer requests HTTP
  
  - run: pip install requests
    # ✅ Funciona - descarga desde PyPI
  
  - run: git clone https://github.com/usuario/repo
    # ✅ Funciona - puede clonar repos públicos
```

**Limitaciones**:
- No puedes recibir conexiones entrantes (no hay IP pública estable)
- No puedes hacer tunneling complejo
- Algunos servicios pueden bloquear IPs de Azure (donde están los runners)

### 10.3 Comunicación entre Steps

Dentro del mismo job, los steps comparten:

1. **Filesystem**:
```yaml
steps:
  - run: echo "hola" > archivo.txt
  - run: cat archivo.txt  # Funciona - mismo filesystem
```

2. **Variables de entorno** (si se exportan):
```yaml
steps:
  - run: echo "MI_VAR=valor" >> $GITHUB_ENV
  - run: echo "$MI_VAR"  # Imprime: valor
```

3. **Directorio de trabajo**:
```yaml
steps:
  - run: cd /tmp && pwd  # /tmp
  - run: pwd             # /home/runner/work/repo/repo (reset)
  # Cada step comienza en el workspace por defecto
```

---

## 11. DEBUGGING Y TROUBLESHOOTING

### 11.1 Logs Detallados

Activar debug logging:

```yaml
# En el repo: Settings → Secrets → New repository secret
# Nombre: ACTIONS_STEP_DEBUG
# Valor: true
```

O establecer en el workflow:
```yaml
env:
  ACTIONS_STEP_DEBUG: true
```

### 11.2 Runner Diagnostic Logs

Ver logs del sistema:
```yaml
steps:
  - name: Diagnóstico
    run: |
      echo "=== SISTEMA ==="
      uname -a
      
      echo "=== VARIABLES DE ENTORNO ==="
      env | sort
      
      echo "=== ESPACIO EN DISCO ==="
      df -h
      
      echo "=== MEMORIA ==="
      free -h
      
      echo "=== DIRECTORIO ACTUAL ==="
      pwd
      ls -la
      
      echo "=== HERRAMIENTAS ==="
      git --version
      node --version
      python --version
```

### 11.3 Ver Contextos Completos

```yaml
steps:
  - name: Dump contextos
    run: |
      echo "GITHUB:"
      echo '${{ toJSON(github) }}'
      
      echo "ENV:"
      echo '${{ toJSON(env) }}'
      
      echo "JOB:"
      echo '${{ toJSON(job) }}'
      
      echo "RUNNER:"
      echo '${{ toJSON(runner) }}'
```

---

## 12. CASOS DE USO TÉCNICOS

### 12.1 Pipeline de CI/CD Completo

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'

jobs:
  # Job 1: Validaciones rápidas
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - run: npm ci
      - run: npm run lint
  
  # Job 2: Tests (matrix para múltiples versiones)
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
  
  # Job 3: Build (solo si lint y test pasaron)
  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.package.outputs.version }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - run: npm ci
      - run: npm run build
      
      - name: Obtener versión
        id: package
        run: echo "version=$(node -p "require('./package.json').version")" >> $GITHUB_OUTPUT
      
      - name: Subir artefacto
        uses: actions/upload-artifact@v4
        with:
          name: dist-${{ steps.package.outputs.version }}
          path: dist/
  
  # Job 4: Deploy (solo en main)
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Descargar artefacto
        uses: actions/download-artifact@v4
        with:
          name: dist-${{ needs.build.outputs.version }}
      
      - name: Deploy a producción
        run: |
          echo "Deploying version ${{ needs.build.outputs.version }}"
          # Comandos de deploy...
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

**Flujo de ejecución**:
```
Push a main
    ↓
GitHub detecta evento "push"
    ↓
Evalúa "on: push" → Ejecuta workflow
    ↓
┌─────────┐  ┌─────────┐
│  lint   │  │  test   │  ← Ejecutan en PARALELO
│         │  │ (6 jobs)│     (no tienen "needs")
└────┬────┘  └────┬────┘
     └────────────┘
            ↓
       ┌─────────┐
       │  build  │  ← Espera a que lint y test terminen
       └────┬────┘
            ↓
       ┌─────────┐
       │ deploy  │  ← Solo si es push a main
       └─────────┘
```

### 12.2 Workflow Reutilizable

**Archivo: .github/workflows/reusable-test.yml**
```yaml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      test-command:
        required: false
        type: string
        default: 'npm test'
    outputs:
      coverage:
        description: "Test coverage percentage"
        value: ${{ jobs.test.outputs.coverage }}
    secrets:
      npm-token:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    outputs:
      coverage: ${{ steps.coverage.outputs.percentage }}
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      
      - run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.npm-token }}
      
      - run: ${{ inputs.test-command }}
      
      - name: Calcular coverage
        id: coverage
        run: |
          COVERAGE=$(npm run coverage:summary | grep -oP '\d+(?=%)')
          echo "percentage=$COVERAGE" >> $GITHUB_OUTPUT
```

**Uso del workflow reutilizable:**
```yaml
name: CI

on: [push, pull_request]

jobs:
  test-node-16:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '16'
      test-command: 'npm test -- --coverage'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
  
  test-node-18:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '18'
  
  report:
    needs: [test-node-16, test-node-18]
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Node 16 coverage: ${{ needs.test-node-16.outputs.coverage }}%"
          echo "Node 18 coverage: ${{ needs.test-node-18.outputs.coverage }}%"
```

---

## 13. PREGUNTAS FRECUENTES TÉCNICAS

### ¿Cómo sabe GitHub qué workflow ejecutar?

1. Usuario hace una acción (push, open PR, etc.)
2. GitHub genera un evento con tipo (push, pull_request, etc.)
3. GitHub busca TODOS los archivos en `.github/workflows/*.yml`
4. Para cada archivo, lee el campo `on:`
5. Si el evento coincide con el `on:`, ejecuta ese workflow
6. Puede ejecutar múltiples workflows para un mismo evento

### ¿Los workflows se ejecutan siempre?

NO. Solo si:
- El evento coincide con `on:`
- Los filtros (branches, paths) coinciden
- El repo tiene Actions habilitado
- No hay errores de sintaxis en el YAML

### ¿Puedo ejecutar un workflow manualmente?

SÍ, con `workflow_dispatch`:
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - dev
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
```

### ¿Cuánto cuestan las GitHub Actions?

**Repositorios públicos**: Gratis e ilimitado

**Repositorios privados**:
- Free plan: 2,000 minutos/mes
- Pro: 3,000 minutos/mes
- Team: 10,000 minutos/mes
- Enterprise: 50,000 minutos/mes

**Multiplicadores por OS**:
- Linux: 1x
- Windows: 2x
- macOS: 10x

Ejemplo: 1 minuto de macOS = 10 minutos consumidos

### ¿Puedo usar Docker en GitHub Actions?

SÍ:
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: node:18-alpine
    steps:
      - run: node --version  # Ejecuta dentro del container
```

O ejecutar containers como servicios:
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        ports:
          - 5432:5432
    steps:
      - run: psql -h localhost -U postgres -c "SELECT 1"
```

### ¿Cómo evito que se ejecute en forks?

```yaml
jobs:
  deploy:
    if: github.repository == 'mi-usuario/mi-repo'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Solo en mi repo"
```

---

## 14. LÍMITES Y CUOTAS

### Límites Técnicos

| Recurso | Límite |
|---------|--------|
| Duración máxima de job | 6 horas |
| Duración máxima de workflow | 72 horas |
| Jobs concurrentes (Free) | 20 |
| Jobs concurrentes (Pro) | 40 |
| Jobs en cola | 500 |
| Tamaño de artifact | 2 GB por archivo |
| Tamaño de cache | 10 GB por repositorio |
| Workflows en un repo | Ilimitado |

### Límites de API

- 1,000 requests por hora por repositorio
- 100 MB de logs por step
- 1,000 requests por minuto (GitHub API desde Actions)

---

## 15. COMPARACIÓN CON OTROS CI/CD

| Característica | GitHub Actions | Jenkins | GitLab CI | CircleCI |
|----------------|----------------|---------|-----------|----------|
| Hosting | GitHub (cloud) | Self-hosted | GitLab (cloud/self) | CircleCI (cloud) |
| Configuración | YAML | Groovy/DSL | YAML | YAML |
| Runners | Managed/Self | Agents | Managed/Self | Executors |
| Marketplace | Sí (actions) | Plugins | No | Orbs |
| Integración GitHub | Nativa | Vía webhooks | Vía webhooks | Vía webhooks |

---

## 16. RECURSOS ADICIONALES

### Documentación Oficial
- [GitHub Actions Docs](https://docs.github.com/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Events Reference](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

### Herramientas
- [act](https://github.com/nektos/act) - Ejecutar actions localmente
- [actionlint](https://github.com/rhysd/actionlint) - Linter para workflows

### Marketplace
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

---

## CONCLUSIÓN

GitHub Actions es un sistema complejo con múltiples capas:

1. **Event System**: Detecta cambios en GitHub
2. **Workflow Engine**: Procesa YAML y toma decisiones
3. **Job Scheduler**: Asigna trabajos a runners
4. **Runners**: Ejecutan los comandos reales
5. **Storage**: Guarda artifacts, cache, logs

**La clave para dominarlo**: Entender que es un sistema **event-driven** y **distribuido**, donde cada componente tiene un rol específico en el ciclo de vida de la ejecución.

Cada vez que uses `${{ github.event_name }}`, ahora sabes:
- Que es un EVENTO generado por GitHub.com
- Que el servidor detectó una acción (push, PR, etc.)
- Que creó un payload JSON con toda la info
- Que ese payload está disponible en el contexto `github.event`
- Que se evaluó ANTES de enviar al runner

**Esto es conocimiento arquitectónico, no solo features** ✅


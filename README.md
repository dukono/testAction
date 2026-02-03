# 🚀 GitHub Actions - Proyecto de Aprendizaje Completo

Este repositorio es una **guía completa y práctica** de GitHub Actions, desde los fundamentos técnicos hasta ejemplos avanzados del mundo real.

## 📚 Contenido del Repositorio

### 📖 Documentación Técnica Completa

1. **[GITHUB_ACTIONS_ARQUITECTURA_TECNICA.md](./GITHUB_ACTIONS_ARQUITECTURA_TECNICA.md)** (⭐ RECOMENDADO)
   - 📐 Arquitectura técnica completa de GitHub Actions
   - 🔧 Cómo funciona internamente (runners, orchestration, execution)
   - 🎯 Conceptos fundamentales explicados en profundidad
   - 💡 Libro técnico completo sobre GitHub Actions

2. **[GITHUB_ACTIONS_CONTEXTOS.md](./GITHUB_ACTIONS_CONTEXTOS.md)**
   - Todos los contextos disponibles (`github`, `env`, `secrets`, etc.)
   - Cuándo y cómo usar cada contexto
   - Ejemplos prácticos de cada uno

3. **[GITHUB_ACTIONS_EVENTOS.md](./GITHUB_ACTIONS_EVENTOS.md)**
   - Catálogo completo de eventos (triggers)
   - Webhooks y cómo funcionan
   - Ejemplos de cada tipo de evento

4. **[GITHUB_ACTIONS_EXPRESIONES.md](./GITHUB_ACTIONS_EXPRESIONES.md)**
   - Sintaxis de expresiones `${{ }}`
   - Funciones disponibles
   - Operadores y condicionales

5. **[GITHUB_ACTIONS_GUIA_COMPLETA.md](./GITHUB_ACTIONS_GUIA_COMPLETA.md)**
   - Guía general de uso
   - Mejores prácticas
   - Tips y trucos

---

### 🎯 Ejemplos Avanzados Ejecutables

Ver **[EJEMPLOS_AVANZADOS_README.md](./EJEMPLOS_AVANZADOS_README.md)** para documentación completa.

#### 🎮 Demo Interactiva
- **[00-demo-completa.yml](./.github/workflows/00-demo-completa.yml)** - 🌟 Demo interactiva de TODAS las capacidades

#### 📦 Ejemplos por Categoría

1. **[01-compartir-datos.yml](./.github/workflows/01-compartir-datos.yml)**
   - Compartir datos entre steps y jobs
   - GITHUB_OUTPUT, GITHUB_ENV, GITHUB_PATH
   - Artifacts y job outputs
   - Matrices dinámicas

2. **[02-reusable-workflow.yml](./.github/workflows/02-reusable-workflow.yml)**
   - Workflow reusable con `workflow_call`
   - Inputs, outputs, y secrets
   - Validaciones y health checks

3. **[03-caller-workflow.yml](./.github/workflows/03-caller-workflow.yml)**
   - Llamar workflows reusables
   - Orquestación de múltiples deployments
   - Despliegues secuenciales con aprobaciones

4. **[04-cicd-completo.yml](./.github/workflows/04-cicd-completo.yml)**
   - Pipeline CI/CD completo profesional
   - Lint → Test → Build → Deploy
   - Multi-plataforma, multi-versión
   - Services (PostgreSQL, Redis)
   - Containers y environments

5. **[05-composite-actions.yml](./.github/workflows/05-composite-actions.yml)**
   - Crear composite actions personalizadas
   - Reutilización de lógica
   - Actions con inputs y outputs

6. **[06-cache-optimization.yml](./.github/workflows/06-cache-optimization.yml)**
   - Estrategias de cache
   - Multi-lenguaje (Python, Node, Go, Rust)
   - Cache incremental y fallback
   - Optimización de performance

7. **[07-secrets-security.yml](./.github/workflows/07-secrets-security.yml)**
   - Manejo seguro de secretos
   - Variables por entorno
   - Credenciales externas (AWS, Docker, SSH)
   - Security audit

8. **[08-dynamic-matrices.yml](./.github/workflows/08-dynamic-matrices.yml)**
   - Matrices estáticas y dinámicas
   - Include/Exclude
   - Matrices anidadas
   - Estrategias avanzadas

---

## 🎓 Cómo Usar Este Repositorio

### 1️⃣ Para Aprender la Teoría
Empieza por la documentación técnica:
```bash
1. Lee GITHUB_ACTIONS_ARQUITECTURA_TECNICA.md (fundamental)
2. Estudia GITHUB_ACTIONS_CONTEXTOS.md
3. Revisa GITHUB_ACTIONS_EVENTOS.md
4. Consulta GITHUB_ACTIONS_EXPRESIONES.md
```

### 2️⃣ Para Ver Ejemplos Prácticos
Ejecuta los workflows de ejemplo:
```bash
# Opción 1: Desde la UI de GitHub
1. Ve a la pestaña "Actions"
2. Selecciona un workflow
3. Click "Run workflow"

# Opción 2: Desde GitHub CLI
gh workflow run "00 - DEMO COMPLETA"
gh run list
gh run view --log
```

### 3️⃣ Orden Recomendado de Aprendizaje

**Nivel Básico:**
1. 📖 Leer `GITHUB_ACTIONS_ARQUITECTURA_TECNICA.md`
2. 🎮 Ejecutar `00-demo-completa.yml` (modo quick)
3. 📦 Ejecutar `01-compartir-datos.yml`

**Nivel Intermedio:**
4. 🔄 Ejecutar `02-reusable-workflow.yml` y `03-caller-workflow.yml`
5. 📊 Ejecutar `08-dynamic-matrices.yml`
6. 💾 Ejecutar `06-cache-optimization.yml`

**Nivel Avanzado:**
7. 🚀 Ejecutar `04-cicd-completo.yml`
8. 🔧 Ejecutar `05-composite-actions.yml`
9. 🔐 Ejecutar `07-secrets-security.yml`

---

## 🎯 Qué Aprenderás

### Conceptos Fundamentales
- ✅ Arquitectura de GitHub Actions (runners, orchestrator)
- ✅ Ciclo de vida de un workflow
- ✅ Eventos y triggers
- ✅ Contextos y expresiones
- ✅ Jobs, steps, y actions

### Capacidades Avanzadas
- ✅ Compartir datos entre steps y jobs
- ✅ Workflows reusables (`workflow_call`)
- ✅ Composite Actions personalizadas
- ✅ Matrices dinámicas y estáticas
- ✅ Cache y optimización
- ✅ Manejo seguro de secretos
- ✅ Environments y deployments
- ✅ Services y containers
- ✅ Artifacts y packages

### Mejores Prácticas
- ✅ Seguridad (secretos, permisos, audit)
- ✅ Performance (cache, paralelización)
- ✅ Reutilización (workflows reusables, composite actions)
- ✅ Debugging y troubleshooting
- ✅ CI/CD patterns profesionales

---

## 🚀 Quick Start

```bash
# 1. Clonar el repositorio
git clone <tu-repo>
cd testsWithGitHubAction

# 2. Leer la arquitectura técnica
cat GITHUB_ACTIONS_ARQUITECTURA_TECNICA.md

# 3. Ver los workflows disponibles
ls -la .github/workflows/

# 4. Ejecutar la demo completa
gh workflow run "00 - DEMO COMPLETA" -f demo-mode=quick

# 5. Ver los resultados
gh run list
gh run view --log
```

---

## 📊 Estadísticas del Proyecto

- 📄 **5 documentos técnicos** completos
- 🎯 **9 workflows de ejemplo** ejecutables
- 💡 **Más de 2000 líneas** de código documentado
- 🎓 **Todos los conceptos** de GitHub Actions cubiertos
- ✅ **100% funcional** y ejecutable

---

## 🤝 Contribuir

Este es un proyecto de aprendizaje. Si encuentras errores o quieres agregar ejemplos:
1. Crea un issue
2. Haz un fork
3. Envía un pull request

---

## 📚 Referencias Externas

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

---

## ⭐ Destacados

### 🌟 Para Principiantes
Empieza con:
1. `GITHUB_ACTIONS_ARQUITECTURA_TECNICA.md`
2. `00-demo-completa.yml` (modo quick)
3. `01-compartir-datos.yml`

### 🔥 Para Avanzados
Ve directo a:
1. `04-cicd-completo.yml`
2. `08-dynamic-matrices.yml`
3. `EJEMPLOS_AVANZADOS_README.md`

---

**🎉 ¡Disfruta aprendiendo GitHub Actions!**

*Este repositorio te llevará desde los fundamentos hasta técnicas avanzadas de CI/CD con GitHub Actions.*

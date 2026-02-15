# GIT - GUÍA PRÁCTICA DE COMANDOS

> **Objetivo:** Guía completa y práctica de los comandos Git más importantes, con ejemplos del mundo real, opciones avanzadas, casos de uso y mejores prácticas.

---

<a id="tabla-de-contenidos"></a>

## 📚 Tabla de Contenidos


### COMANDOS BÁSICOS ESENCIALES
1. [git add - Preparando Cambios](#1-git-add---preparando-cambios)
2. [git commit - Guardando la Historia](#2-git-commit---guardando-la-historia)
3. [git status - Inspeccionando el Estado](#3-git-status---inspeccionando-el-estado)
4. [git diff - Comparando Cambios](#4-git-diff---comparando-cambios)
4.1 [Referencias de Commits: ~, ^, y {}](#41-referencias-de-commits---y-)
5. [git log - Explorando la Historia](#5-git-log---explorando-la-historia)

### GESTIÓN DE RAMAS
6. [git branch - Gestionando Líneas de Desarrollo](#6-git-branch---gestionando-líneas-de-desarrollo)
7. [git checkout / git switch - Navegando el Código](#7-git-checkout--git-switch---navegando-el-código)
8. [git merge - Integrando Cambios](#8-git-merge---integrando-cambios)
9. [git rebase - Reescribiendo Historia](#9-git-rebase---reescribiendo-historia)

### TRABAJO CON REMOTOS
10. [git clone - Copiando Repositorios](#10-git-clone---copiando-repositorios)
11. [git remote - Gestionando Repositorios Remotos](#11-git-remote---gestionando-repositorios-remotos)
12. [git fetch - Descargando Cambios](#12-git-fetch---descargando-cambios)
13. [git pull - Descargando e Integrando Cambios Remotos](#13-git-pull---descargando-e-integrando-cambios-remotos)
14. [git push - Subiendo Cambios](#14-git-push---subiendo-cambios)

### CONTROL DE ESTADO Y VERSIONES
15. [git reset - Moviendo Referencias](#15-git-reset---moviendo-referencias)
16. [git stash - Guardado Temporal](#16-git-stash---guardado-temporal)
17. [git tag - Marcando Versiones](#17-git-tag---marcando-versiones)
18. [git revert - Deshaciendo Commits Públicos](#18-git-revert---deshaciendo-commits-públicos)
19. [git cherry-pick - Aplicando Commits Selectivos](#19-git-cherry-pick---aplicando-commits-selectivos)

### LIMPIEZA Y MANTENIMIENTO
20. [git clean - Limpiando Archivos No Rastreados](#20-git-clean---limpiando-archivos-no-rastreados)
21. [git rm y git mv - Eliminando y Moviendo Archivos](#21-git-rm-y-git-mv---eliminando-y-moviendo-archivos)

### REFERENCIAS Y FORMATO AVANZADO
22. [Referencias y Placeholders de Formato](#22-referencias-y-placeholders-de-formato)

---

## INTRODUCCIÓN

Esta guía cubre los **21 comandos Git más importantes** que todo desarrollador debe conocer, desde principiante hasta experto. Cada comando incluye:

✅ **Funcionamiento interno** - Qué hace Git bajo el capó
✅ **15-20+ opciones y flags** - Uso básico a avanzado  
✅ **10+ casos de uso reales** - Ejemplos del mundo profesional
✅ **Troubleshooting completo** - Problemas y soluciones
✅ **Mejores prácticas** - Qué hacer y qué evitar

**Relación con otros documentos:**
- Para teoría y funcionamiento interno: Ver `GIT_FUNCIONAMIENTO_INTERNO.md`
- Para GitHub Actions: Ver `GITHUB_ACTIONS_*.md`

---

## 1. git add - Preparando Cambios
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Prepara cambios del working directory para el próximo commit, moviéndolos al staging area (index).

**Funcionamiento interno:** [🔙](#1-git-add---preparando-cambios)

```
Internamente hace:
1. git hash-object -w file.txt
   → Calcula SHA-1 del contenido
   → Comprime con zlib
   → Guarda blob en .git/objects/

2. git update-index --add file.txt
   → Actualiza .git/index con:
     - Ruta del archivo
     - Hash del blob
     - Permisos (100644, 100755, etc.)
     - Timestamp

Resultado:
- Blob creado en objects/
- Index (.git/index) actualizado
- Working directory NO cambia
- Repository NO cambia (aún no hay commit)
```

**Uso práctico y opciones:** [🔙](#1-git-add---preparando-cambios)

```bash
# 1. Añadir archivo específico
git add archivo.txt
# → Stagea solo archivo.txt

# 2. Añadir todos los archivos modificados y nuevos
git add .
# → Stagea todo desde directorio actual
# → Incluye subdirectorios
# → Respeta .gitignore

# 3. Añadir todos los archivos del repositorio
git add -A
# o: git add --all
# → Stagea TODO: nuevos, modificados, eliminados
# → Desde cualquier directorio

# 4. Añadir solo archivos rastreados (ignora nuevos)
git add -u
# o: git add --update
# → Solo archivos ya en Git
# → NO añade archivos nuevos
# → Útil para "actualizar solo lo existente"

# 5. Añadir interactivamente (PODER REAL)
git add -i
# → Modo interactivo con menú
# → Puedes elegir qué hacer con cada archivo

# 6. Añadir por parches (SUPER ÚTIL)
git add -p archivo.txt
# o: git add --patch
# → Te muestra cada "hunk" de cambios
# → Preguntas: Stage this hunk? [y,n,q,a,d,s,e,?]
# → Puedes stagear solo PARTE de un archivo
```

**Caso de uso real: Commits atómicos con -p:** [🔙](#1-git-add---preparando-cambios)

```bash
Escenario: Modificaste un archivo con 2 features diferentes

# archivo.py tiene:
# - Cambio A: Nueva función calculate()
# - Cambio B: Fix bug en validate()

# Quieres 2 commits separados:

# Paso 1: Stagea solo cambios de calculate()
git add -p archivo.py
# → Ves el hunk con calculate()
# → Presionas 'y' (yes)
# → Ves el hunk con validate()
# → Presionas 'n' (no)

git commit -m "feat: Add calculate function"

# Paso 2: Stagea el resto
git add archivo.py
git commit -m "fix: Fix validation bug"

Resultado: 2 commits atómicos, historia más clara
```

**Opciones avanzadas de add -p:** [🔙](#1-git-add---preparando-cambios)

```
Durante git add -p, opciones disponibles:

y - Stage this hunk (sí, añadir este cambio)
n - Do not stage (no, saltar)
q - Quit (salir, no procesar más)
a - Stage this and all remaining hunks (todos los siguientes)
d - Do not stage this or any remaining (ninguno de los siguientes)
s - Split into smaller hunks (dividir en partes más pequeñas)
e - Manually edit hunk (editar manualmente)
? - Help (ayuda)

Opción 's' (split) es PODEROSA:
→ Si un hunk tiene múltiples cambios cercanos
→ Puedes intentar dividirlo en hunks más pequeños
→ Para control más granular

Opción 'e' (edit) es para EXPERTOS:
→ Abre editor con el diff
→ Puedes editar líneas manualmente
→ Útil cuando 's' no divide suficiente
```

**Patrones de uso comunes:** [🔙](#1-git-add---preparando-cambios)

```bash
# Patrón 1: Añadir por tipo de archivo
git add *.py          # Solo archivos Python
git add src/          # Todo en directorio src/
git add "*.txt"       # Todos los .txt (comillas para expansión)

# Patrón 2: Añadir excepto algunos
git add .
git reset HEAD archivo-no-deseado.txt
# → Añade todo, luego quita uno

# Patrón 3: Añadir forzando (ignorar .gitignore)
git add -f archivo-ignorado.log
# → Fuerza añadir aunque esté en .gitignore
# → Úsalo con CUIDADO

# Patrón 4: Dry run (ver qué se añadiría)
git add -n .
# o: git add --dry-run .
# → Muestra qué se añadiría sin hacerlo

# Patrón 5: Añadir con verbose
git add -v archivo.txt
# → Muestra qué archivos se añaden
```

**Ver qué está stageado:** [🔙](#1-git-add---preparando-cambios)

```bash
# Ver estado
git status
# → Muestra archivos stageados y no stageados

# Ver diferencias stageadas
git diff --staged
# o: git diff --cached
# → Muestra QUÉ cambios están en staging

# Ver diferencias NO stageadas
git diff
# → Muestra cambios en working directory
# → Que NO están en staging
```

**Mejores prácticas:** [🔙](#1-git-add---preparando-cambios)

```bash
✓ Usa git add -p para commits granulares
✓ Revisa con git diff --staged antes de commit
✓ No uses git add . ciegamente, revisa qué añades
✓ Usa .gitignore para archivos que nunca deben añadirse
✓ Considera git add -u cuando solo actualizas existentes

✗ Evita git add * (puede añadir archivos no deseados)
✗ No uses git add -f a menos que sea absolutamente necesario
✗ No stagees archivos generados (builds, logs, node_modules)
```
# → Ves hunk con feature B: presionas 'n'
git commit -m "feat: Add feature A"

git add archivo.py
git commit -m "feat: Add feature B"
```

**Mejores prácticas:**

✓ Usa git add -p para commits granulares
✓ Revisa con git diff --staged antes de commit
✓ Usa .gitignore para archivos que nunca deben añadirse

✗ Evita git add * (puede añadir no deseados)
✗ No stagees archivos generados (builds, node_modules)
```

---

## 2. git commit - Guardando la Historia
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Crea un snapshot inmutable del proyecto con los cambios del staging area.

**Funcionamiento interno:** [🔙](#2-git-commit---guardando-la-historia)

```
1. Crea tree object del staging
2. Crea commit object con tree + parent + metadata
3. Actualiza referencia de rama
4. Actualiza reflog
```

**Uso práctico:** [🔙](#2-git-commit---guardando-la-historia)

```bash
# 1. Commit básico
git commit -m "Mensaje descriptivo"

# 2. Mensaje multilínea (título + descripción)
git commit -m "Título corto" -m "Descripción detallada más larga"

# 3. Abrir editor para mensaje largo
git commit
# → Se abre tu editor configurado
# → Primera línea = título
# → Línea vacía
# → Resto = descripción

# 4. Add + commit automático (SOLO archivos tracked)
git commit -am "Mensaje"
# o: git commit --all -m "Mensaje"
# → Añade y commitea archivos modificados
# → NO añade archivos nuevos (untracked)
# → Útil para cambios rápidos

# 5. Modificar último commit (IMPORTANTE)
git commit --amend -m "Nuevo mensaje"
# → Reemplaza el último commit
# → Útil para corregir errores

# 6. Amend sin cambiar mensaje
git commit --amend --no-edit
# → Añade cambios al último commit
# → Mantiene el mensaje original

# 7. Amend solo el mensaje
git commit --amend
# → Abre editor para cambiar mensaje
# → No añade cambios nuevos

# 8. Commit vacío (útil para CI/CD)
git commit --allow-empty -m "Trigger CI"
# → Crea commit sin cambios
# → Útil para forzar rebuild

# 9. Commit con fecha específica
git commit -m "Mensaje" --date="2024-01-15 10:30:00"
# → Sobrescribe fecha del commit

# 10. Commit como otro autor
git commit -m "Mensaje" --author="Nombre <email@ejemplo.com>"
# → Útil para pair programming
# → O commits de otros

# 11. Commit sin hooks
git commit -m "Mensaje" --no-verify
# o: git commit -m "Mensaje" -n
# → Omite pre-commit y commit-msg hooks
# → Úsalo con CUIDADO

# 12. Commit con template
git commit -t plantilla.txt
# → Usa archivo como plantilla de mensaje

# 13. Commit verboso (muestra diff)
git commit -v
# → Muestra diff en el editor
# → Ayuda a escribir mejor mensaje

# 14. Commit solo de archivos específicos
git commit archivo1.txt archivo2.txt -m "Mensaje"
# → Commitea solo esos archivos (deben estar staged)

# 15. Commit con firma GPG
git commit -S -m "Signed commit"
# → Firma el commit con tu clave GPG
# → Verifica identidad del autor

# 16. Reutilizar mensaje de otro commit
git commit -C <commit-hash>
# → Copia mensaje de otro commit
# O editar el mensaje:
git commit -c <commit-hash>
```

**Casos de uso del --amend:** [🔙](#2-git-commit---guardando-la-historia)

```bash
# Caso 1: Olvidaste un archivo
git add archivo-olvidado.txt
git commit --amend --no-edit
# → Añade el archivo al último commit

# Caso 2: Error de escritura en mensaje
git commit --amend -m "Mensaje corregido"
# → Corrige el mensaje del último commit

# Caso 3: Añadir más cambios al último commit
git add mas-cambios.txt
git commit --amend
# → Añade cambios y edita mensaje si quieres

# ⚠️ IMPORTANTE: Solo usa --amend en commits NO pusheados
# Si ya hiciste push, necesitarás force push (peligroso en ramas compartidas)
```

**Opciones de formato de mensaje:** [🔙](#2-git-commit---guardando-la-historia)

```bash
# Mensaje desde archivo
git commit -F mensaje.txt

# Mensaje desde stdin
echo "Mi mensaje" | git commit -F -

# Limpiar espacios del mensaje
git commit --cleanup=strip -m "  Mensaje con espacios  "
# → Elimina espacios extra

# Mantener mensaje tal cual
git commit --cleanup=verbatim -m "Mensaje exacto"
```

**Commits interactivos:** [🔙](#2-git-commit---guardando-la-historia)

```bash
# Commit interactivo (elige qué añadir)
git commit -p
# → Similar a git add -p + commit
# → Selecciona hunks a commitear
```

**Mensajes de commit efectivos (Conventional Commits):** [🔙](#2-git-commit---guardando-la-historia)

```bash
feat: Add user authentication
fix: Fix login validation bug
docs: Update README
style: Format code
refactor: Simplify auth logic
test: Add integration tests
chore: Update dependencies

# Con scope:
feat(auth): Add login endpoint
fix(api): Handle timeout errors

# Formato completo:
feat(api): Add user registration

- Implement POST /api/register
- Add email validation
- Add password hashing

Closes #123
```

**Troubleshooting común:** [🔙](#2-git-commit---guardando-la-historia)

```bash
# Problema 1: "Nothing to commit"
# Solución: Añade archivos al staging primero
git add .
git commit -m "Mensaje"

# Problema 2: Olvidaste añadir un archivo al commit
# Solución: Usar --amend
git add archivo-olvidado.txt
git commit --amend --no-edit

# Problema 3: Mensaje de commit equivocado
# Solución: Usar --amend
git commit --amend -m "Mensaje correcto"

# Problema 4: Necesitas modificar el último commit
# Solución: Ver ejemplos de --amend arriba
git commit --amend

# Problema 5: Commit en rama equivocada
# Solución: Usar cherry-pick (ver sección de cherry-pick)
# O usar reset para deshacer (ver sección de reset)

# Problema 6: "Please tell me who you are"
# Solución: Configurar identidad
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"

# Problema 7: Editor no se abre o no sabes usar vi
# Solución: Cambiar editor
git config --global core.editor "nano"
# O usar -m directamente:
git commit -m "Mensaje"

# Problema 8: Quieres deshacer un commit
# Solución: Ver sección "git reset" o "git revert" según el caso
```

**Mejores prácticas:** [🔙](#2-git-commit---guardando-la-historia)

```bash
✓ Commits pequeños y atómicos
✓ Mensajes descriptivos (explica POR QUÉ)
✓ Usa convenciones (Conventional Commits)
✓ Usa --amend solo en commits NO pusheados

✗ Evita commits gigantes
✗ Evita mensajes genéricos ("fix", "update")
✗ No uses --amend en commits públicos
```

---

## 3. git status - Inspeccionando el Estado
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Muestra el estado actual del working directory y staging area.

**Funcionamiento interno:** [🔙](#3-git-status---inspeccionando-el-estado)

```
1. Compara working directory con HEAD
2. Compara staging con HEAD
3. Lee .git/index para archivos untracked
4. Compara con refs/remotes para ahead/behind
```

**Uso práctico:** [🔙](#3-git-status---inspeccionando-el-estado)

```bash
# Status normal (verbose)
git status

# Status corto (MUY ÚTIL)
git status -s
# Formato: XY archivo
# X = estado en staging (index)
# Y = estado en working directory

# Códigos más comunes:
# ?? = untracked (archivo nuevo no añadido)
# A  = added (archivo nuevo añadido al staging)
# M  = modified (archivo modificado y en staging)
#  M = modified (archivo modificado pero NO en staging)
# MM = modified en staging + modificado de nuevo en working
# D  = deleted (archivo eliminado y en staging)
#  D = deleted (archivo eliminado pero NO en staging)
# R  = renamed (archivo renombrado)
# C  = copied (archivo copiado)
# U  = updated but unmerged (conflicto sin resolver)

# Con info de branch
git status -sb

# Ver archivos ignorados
git status --ignored

# Formato porcelain (para scripts)
git status --porcelain
```

**Interpretación del output:** [🔙](#3-git-status---inspeccionando-el-estado)

```bash
# OUTPUT DE git status (verbose):
On branch main
Your branch is ahead of 'origin/main' by 2 commits
→ Tienes 2 commits no pusheados (ahead)
→ "behind" sería: commits remotos que no tienes localmente

Changes to be committed:
→ Staging area (listo para commit)

Changes not staged for commit:
→ Working directory modificado

Untracked files:
→ Archivos nuevos no en Git

# OUTPUT DE git status -s (corto):
 M archivo1.txt    # Modificado, NO en staging
M  archivo2.txt    # Modificado, en staging
MM archivo3.txt    # En staging + modificado de nuevo
A  archivo4.txt    # Nuevo, añadido al staging
?? archivo5.txt    # Nuevo, no añadido (untracked)
D  archivo6.txt    # Eliminado, en staging
 D archivo7.txt    # Eliminado, NO en staging
R  old.txt -> new.txt  # Renombrado
```

**Entendiendo ahead/behind:** [🔙](#3-git-status---inspeccionando-el-estado)

```bash
# Ahead (adelantado): Tienes commits locales no pusheados
Your branch is ahead of 'origin/main' by 2 commits
→ Solución: git push

# Behind (atrasado): El remoto tiene commits que tú no tienes
Your branch is behind 'origin/main' by 3 commits
→ Solución: git pull

# Diverged (divergido): Ambos tienen commits diferentes
Your branch and 'origin/main' have diverged,
and have 2 and 3 different commits each, respectively
→ Tienes 2 commits locales que el remoto no tiene
→ El remoto tiene 3 commits que tú no tienes
→ Necesitas reconciliar las diferencias
```

**¿Qué hacer cuando hay divergencia (diverged)?:** [🔙](#3-git-status---inspeccionando-el-estado) [🔙](#3-git-status---inspeccionando-el-estado)

```bash
# PASO 1: Investigar qué pasó
# Ver tus commits locales que no están en remoto
git log origin/main..HEAD --oneline

# Ver commits remotos que no tienes localmente
git log HEAD..origin/main --oneline

# Ver todas las diferencias
git log --oneline --graph --all

# PASO 2: Elegir estrategia de sincronización

# Opción A: MERGE (mantiene toda la historia)
git pull
# → Crea un merge commit
# → Historia completa pero más compleja
# → Recomendado para trabajo en equipo

# Opción B: REBASE (historia lineal)
git pull --rebase
# → Reaplica tus commits encima de los remotos
# → Historia más limpia
# → Recomendado para trabajo individual
# → NO usar si ya compartiste tus commits

# Opción C: FORZAR tus cambios (sobrescribir remoto)
git push --force-with-lease
# → Solo si estás SEGURO que tus cambios son correctos
# → Elimina los commits remotos
# → ⚠️ PELIGROSO en ramas compartidas

# Opción D: FORZAR cambios remotos (descartar locales)
git reset --hard origin/main
# → Descarta TUS commits locales
# → Sincroniza con remoto
# → ⚠️ Pierdes trabajo local

# PASO 3: Verificar después
git status
git log --oneline --graph --all
```

**Causas comunes de divergencia:** [🔙](#3-git-status---inspeccionando-el-estado)

```bash
# Causa 1: Usaste --amend después de push
git commit -m "A"
git push
git commit --amend -m "B"  # Cambia el commit
git push  # ❌ Error: diverged

# Causa 2: Múltiples personas trabajando en la misma rama
# Persona A: push commit 1
# Persona B: push commit 2 (sin pull primero)
# → Divergencia

# Causa 3: Push --force desde otro lugar
# Computadora A: git push --force
# Computadora B: ahora está divergida

# Causa 4: Rebase de rama ya compartida
git push
git rebase main  # Reescribe commits
git push  # ❌ Error: diverged
```

**Troubleshooting de divergencia:** [🔙](#3-git-status---inspeccionando-el-estado)

```bash
# Ver exactamente qué difiere
git diff origin/main

# Ver log comparativo
git log --left-right --oneline origin/main...HEAD
# < = commits en remoto
# > = commits locales

# Si no estás seguro qué hacer, haz backup
git branch backup-antes-de-sincronizar
# Luego puedes probar diferentes estrategias
```

**Mejores prácticas:** [🔙](#3-git-status---inspeccionando-el-estado)

```bash
✓ Ejecuta git status antes de commit (SIEMPRE)
✓ Usa -s para overview rápido
✓ Verifica tracking branch con -b

✗ No ignores el output
✗ No commitees sin revisar status primero
```

---

## 4. git diff - Comparando Cambios
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Muestra diferencias entre working directory, staging, commits y ramas.

**Funcionamiento interno:** [🔙](#4-git-diff---comparando-cambios)

```
1. Lee contenido de dos fuentes
2. Ejecuta algoritmo de diff (Myers, patience, histogram)
3. Genera "hunks" (bloques de diferencias)
4. Formatea output
```

**Uso práctico:** [🔙](#4-git-diff---comparando-cambios)

```bash
# Diff de working (NO stageado)
git diff

# Diff de staging (lo que vas a commitear)
git diff --staged
# o: git diff --cached

# Diff completo (working vs último commit)
git diff HEAD

# Diff entre commits
git diff abc123 def456
git diff HEAD~5 HEAD

# Diff entre ramas
git diff main feature-x
git diff main...feature-x  # Desde punto de divergencia

# Diff de archivo específico
git diff archivo.txt
git diff HEAD~3 -- archivo.txt

# Diff con stats (resumen)
git diff --stat

# Diff solo nombres de archivos
git diff --name-only
git diff --name-status

# Diff por palabras (útil para textos)
git diff --word-diff

# Ignorar espacios
git diff -w

# Detectar líneas movidas
git diff --color-moved
```

**Mejores prácticas:** [🔙](#4-git-diff---comparando-cambios)

```bash
✓ Usa git diff antes de add
✓ Usa git diff --staged antes de commit
✓ Usa --word-diff para documentación
✓ Usa ... (tres puntos) para comparar ramas

✗ No ignores el diff antes de commitear
✗ No confundas git diff con git diff --staged
```

---

## 4.1. Referencias de Commits: ~, ^, y {}
[⬆️ Top](#tabla-de-contenidos)

**¿Qué son?**
Son operadores especiales para referenciar commits relativos a una posición dada (como HEAD o nombre de rama).

### Operador `~` (Tilde) - Navegación Hacia Atrás por Primera Línea

**Significado:** Navega hacia atrás en la historia siguiendo siempre la **primera línea de commits**.

```
HEAD~1  → 1 commit antes de HEAD (equivalente a HEAD^)
HEAD~2  → 2 commits antes de HEAD
HEAD~3  → 3 commits antes de HEAD
HEAD~n  → n commits antes de HEAD siguiendo la primera línea
```

**Visualización:** [🔙](#41-referencias-de-commits---y-)

```
A ← B ← C ← D ← E (HEAD)
│   │   │   │   │
~4  ~3  ~2  ~1  ~0 (o simplemente HEAD)
```

**Ejemplos prácticos:** [🔙](#41-referencias-de-commits---y-)

```bash
# Ver el commit de hace 3 commits
git show HEAD~3

# Comparar con 5 commits atrás
git diff HEAD~5 HEAD

# Resetear al commit anterior
git reset --soft HEAD~1

# Ver cambios de un archivo hace 2 commits
git show HEAD~2:archivo.txt
```

---

### Operador `^` (Caret) - Selección de Líneas en Merges

**Significado:** Selecciona **qué línea de commits** seguir cuando un commit tiene múltiples líneas de historia (merge commits).

```
HEAD^1  → Primera línea de commits (default, equivalente a HEAD^)
HEAD^2  → Segunda línea de commits (rama mergeada)
HEAD^3  → Tercera línea de commits (raro, en octopus merge)
```

**Visualización de merge:** [🔙](#41-referencias-de-commits---y-)

```
    A ← B ← C (rama feature)
   /         \
  D ← E ← F ← M (HEAD en main)
              │
         HEAD^1 = F (primera línea, main)
         HEAD^2 = C (segunda línea, rama mergeada)
         HEAD^3 = A (tercera línea, en octopus merge)
```

**Ejemplos prácticos:** [🔙](#41-referencias-de-commits---y-)

```bash
# Ver qué entró desde la rama mergeada
git log HEAD^2

# Comparar con la primera línea (rama principal)
git diff HEAD^1 HEAD

# Ver los cambios de la segunda línea
git show HEAD^2

# Comparar ambas líneas
git diff HEAD^1 HEAD^2
```

---

### Combinando `~` y `^`

**Se pueden combinar para navegación compleja:** [🔙](#41-referencias-de-commits---y-)

```bash
HEAD~2^2   → Segunda línea del commit que está 2 commits atrás (si ese commit es un merge)
HEAD^^     → Equivalente a HEAD~2 (2 commits atrás)
HEAD^2~3   → Tres commits atrás desde la segunda línea
```

**Ejemplo visual básico:** [🔙](#41-referencias-de-commits---y-)

```
        A ← B ← C
       /         \
  D ← E ← F ← G ← M (HEAD)

HEAD      → M (merge commit)
HEAD~1    → G (1 commit atrás)
HEAD~2    → W (2 commits atrás, también es merge)
HEAD~2^1  → B (primera línea del commit W)
HEAD~2^2  → Y (segunda línea del commit W)

Nota: HEAD~2^2 solo existe si el commit que está 2 commits atrás (W) es un merge.
      Si W no tiene segunda línea, HEAD~2^2 dará error.
```

**Ejemplo visual con HEAD~2^2:** [🔙](#41-referencias-de-commits---y-)

```
    X ← Y          (rama lateral)
   /     \
  A ← B ← W ← G ← M (HEAD)

HEAD      → M (merge commit)
HEAD~1    → G (1 commit atrás)
HEAD~2    → W (2 commits atrás, también es merge)
HEAD~2^1  → B (primera línea del commit W)
HEAD~2^2  → Y (segunda línea del commit W)

Nota: HEAD~2^2 solo funciona si el commit que está 2 commits atrás (W) es un merge.
      Si W no tiene segunda línea, HEAD~2^2 dará error.
```

**Ejemplos prácticos:** [🔙](#41-referencias-de-commits---y-)

```bash
# Ver el tercer commit de la segunda línea
git show HEAD^2~3

# Comparar ancestros complejos
git diff HEAD~3 HEAD^2~1

# Resetear a ancestro complejo
git reset HEAD^^
```

---

### Operador `{}` (Reflog) - Historial de Movimientos

**Significado:** Accede al **historial de posiciones previas** de una referencia (HEAD, ramas, etc.).

```
HEAD@{0}  → Posición actual
HEAD@{1}  → Dónde estaba HEAD en la operación anterior
HEAD@{2}  → Dos operaciones atrás
HEAD@{n}  → n-ésima operación atrás
```

**¿Qué operaciones mueven HEAD?**
- commit, checkout, merge, pull, reset, rebase, cherry-pick, etc.

**Visualización:** [🔙](#41-referencias-de-commits---y-)

```bash
git reflog
# Salida:
abc1234 HEAD@{0}: commit: Added feature X
def5678 HEAD@{1}: checkout: moving from main to feature
9ab0cde HEAD@{2}: pull: Fast-forward
```

**Ejemplos prácticos:** [🔙](#41-referencias-de-commits---y-)

```bash
# Ver dónde estaba HEAD hace 3 operaciones
git show HEAD@{3}

# Volver al estado antes del último pull
git reset --hard HEAD@{1}

# Ver commits traídos en el último pull
git log HEAD@{1}..HEAD --oneline

# Ver diferencias con estado previo
git diff HEAD@{1} HEAD

# Reflog de una rama específica
git reflog show feature-branch

# Ver estado hace 2 días
git show HEAD@{2.days.ago}

# Ver estado a una fecha específica
git show main@{2024-01-15}
```

---

### Tabla Resumen de Referencias

| Operador | Propósito | Ejemplo | Resultado |
|----------|-----------|---------|-----------|
| `~n` | n commits atrás (primera línea) | `HEAD~3` | 3 commits antes de HEAD |
| `^n` | n-ésima línea (en merges) | `HEAD^2` | Segunda línea del merge |
| `@{n}` | n-ésima posición anterior (reflog) | `HEAD@{5}` | Dónde estaba hace 5 ops |
| `^^` | Equivalente a `~2` | `HEAD^^` | 2 commits atrás |
| `~n^m` | Combinación | `HEAD~2^2` | 2ª línea del commit que está 2 atrás |
| `@{time}` | Posición en fecha/tiempo | `HEAD@{yesterday}` | Estado de ayer |

---

### Casos de Uso Prácticos

**1. Ver qué traído en un pull:** [🔙](#41-referencias-de-commits---y-)

```bash
git log HEAD@{1}..HEAD --oneline
git diff HEAD@{1} HEAD --name-status
```

**2. Deshacer el último commit manteniendo cambios:** [🔙](#41-referencias-de-commits---y-)

```bash
git reset --soft HEAD~1
```

**3. Ver qué se mergeó desde una rama:** [🔙](#41-referencias-de-commits---y-)

```bash
# Solo funciona si HEAD es un merge commit
git log HEAD^2 --oneline

# Si HEAD no es merge, usa el hash del merge:
git log <hash-merge>^2 --oneline
# O si sabes que el merge está 2 commits atrás:
git log HEAD~2^2 --oneline
```

**4. Recuperar trabajo perdido:** [🔙](#41-referencias-de-commits---y-)

```bash
git reflog
git switch -c recuperado HEAD@{5}  # Crea rama desde ese punto
# o para solo ver:
git switch --detach HEAD@{5}  # Detached HEAD para inspección
```

**5. Comparar con versión de ayer:** [🔙](#41-referencias-de-commits---y-)

```bash
git diff HEAD@{yesterday} HEAD
```

**6. Ver ancestros en merge complejo:**
```bash
# Ver commits únicos del segundo padre
git log HEAD^1..HEAD^2 --oneline
```

---

### Diferencias Clave

**`HEAD~1` vs `HEAD^1`:**
- En commits normales (con una sola línea de commits anterior): **Son idénticos**
- En merge commits:
  - `HEAD~1` → Siempre sigue la primera línea
  - `HEAD^1` → Primera línea explícitamente
  - `HEAD^2` → Segunda línea (rama mergeada)

**`HEAD@{1}` vs `HEAD~1`:**
- `HEAD~1` → Commit anterior en el grafo de commits
- `HEAD@{1}` → Posición anterior de HEAD (puede ser cualquier commit)

**Ejemplo:** [🔙](#41-referencias-de-commits---y-)

```bash
# Secuencia de operaciones:
git switch main      # HEAD en abc123
git switch feature   # HEAD en def456
git switch main      # HEAD en abc123 otra vez

# Ahora:
HEAD      → abc123 (main)
HEAD~1    → 789xyz (commit anterior a abc123 en el grafo)
HEAD@{1}  → def456 (donde estaba HEAD antes: feature)
HEAD@{2}  → abc123 (donde estaba antes de eso)
```

---

### ⚠️ Advertencia Importante sobre HEAD^2

**HEAD^2 solo existe si el commit actual ES un merge commit:**

```bash
# Verificar si un commit es un merge:
git rev-list --parents -n 1 HEAD
# Si muestra 2+ hashes después del primero → es merge
# Si muestra solo 2 hashes → NO es merge (1 línea)

# Ejemplo de error común:
git switch main
git log --oneline -1
# abc123 Add feature X  ← commit normal, no merge

git show HEAD^2
# fatal: ambiguous argument 'HEAD^2': unknown revision or path not in the working tree.

# Para ver la segunda línea de un merge anterior:
git log --oneline --graph -5  # Identifica el merge commit
git show <hash-merge>^2        # Usa el hash del merge
# O si el merge está 2 commits atrás:
git show HEAD~2^2              # Funciona si HEAD~2 es merge
```

**Cómo identificar merge commits visualmente:**

```bash
# En git log:
git log --oneline --graph --all
# Los merges se ven así:
#   *   a1b2c3d Merge branch 'feature' into main  ← MERGE commit
#   |\  
#   | * d4e5f6g Add feature
#   * | h7i8j9k Fix bug
#   |/  
#   * k0l1m2n Initial commit
```

---

### Mejores Prácticas

[🔙](#41-referencias-de-commits---y-)

```bash
✓ Usa ~ para navegar historia lineal
✓ Usa ^ para explorar merges
✓ Usa @{} para deshacer operaciones recientes
✓ Combina operadores para navegación compleja
✓ Usa git reflog para ver historial de operaciones
✓ Verifica que un commit sea merge antes de usar ^2

✗ No confundas ~ (commits atrás) con @{} (historial)
✗ No uses ^2 en commits sin merge (da error)
✗ No abuses de combinaciones complejas (dificulta lectura)
✗ No asumas que HEAD siempre es un merge
```

---

## 5. git log - Explorando la Historia
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Muestra la historia de commits del repositorio con múltiples opciones de filtrado, formato y análisis. Es una herramienta fundamental para entender la evolución del código, buscar bugs, auditar cambios y analizar contribuciones.

**Funcionamiento interno:** [🔙](#5-git-log---explorando-la-historia)

```
1. Lee HEAD (o referencia especificada)
2. Obtiene commit object del hash
3. Lee metadata: author, date, message, tree, parents
4. Sigue recursivamente la cadena de commits anteriores
5. Aplica filtros especificados (autor, fecha, mensaje, archivos)
6. Formatea output según opciones (oneline, graph, stat, patch)
7. Pagina resultado (usa less por defecto)

Optimizaciones:
- Usa commit-graph para acelerar traversal
- Cache de objetos en memoria
- Traversal paralelo en repos grandes
```

**Uso práctico - Formatos básicos:** [🔙](#5-git-log---explorando-la-historia)

```bash
# ============================================
# FORMATOS DE VISUALIZACIÓN
# ============================================

# 1. Log básico (verbose, por defecto)
git log
# Muestra:
# - Hash completo
# - Autor y email
# - Fecha
# - Mensaje completo

# 2. Log compacto (UNA LÍNEA por commit)
git log --oneline
# Formato: hash-corto mensaje
# Ejemplo: abc123 Add user authentication

# 3. Log con decoraciones (refs)
git log --oneline --decorate
# Muestra: HEAD, ramas, tags
# Ejemplo: abc123 (HEAD -> main, origin/main) Add feature

# 4. Log con grafo visual (SUPER ÚTIL)
git log --oneline --graph
# Muestra estructura de ramas y merges
# Ejemplo:
# * abc123 Merge branch 'feature'
# |\
# | * def456 Add feature
# |/
# * 789abc Fix bug

# 5. Log con grafo de todas las ramas
git log --oneline --graph --all
# → Muestra TODO el repositorio
# → Incluye ramas locales y remotas
# → Muy útil para overview completo

# 6. Log con estadísticas de cambios
git log --stat
# Muestra archivos modificados y líneas +/-
# archivo.txt | 10 +++++-----

# 7. Log con diff completo (patch)
git log -p
# o: git log --patch
# → Muestra diff de cada commit
# → Útil para code review histórico

# 8. Log con diff de últimos N commits
git log -p -2
# → Solo últimos 2 commits con diff

# 9. Log con resumen corto
git log --oneline --stat
# → Combina hash + mensaje + stats
# → Balance perfecto de info

# 10. Log con formato personalizado
git log --pretty=format:"%h - %an, %ar : %s"
# Formato: hash - autor, fecha relativa : mensaje
# Ejemplo: abc123 - John, 2 days ago : Fix bug

# 11. Formatos predefinidos
git log --pretty=oneline
git log --pretty=short
git log --pretty=medium  # Default
git log --pretty=full
git log --pretty=fuller
git log --pretty=reference
```

**Uso práctico - Filtros por rango de commits:** [🔙](#5-git-log---explorando-la-historia)

```bash
# ============================================
# RANGOS Y EXCLUSIONES
# ============================================

# 1. Commits en rama A pero NO en rama B
git log main..feature-x
# → Commits únicos de feature-x
# → Útil para ver qué traerá el merge

git log origin/main..HEAD
# → Commits locales no pusheados
# → Equivalente a: git log @{u}..HEAD

# 2. Commits que difieren entre ramas (symmetric difference)
git log main...feature-x
# → Commits en A o B pero no en ambas
# → Muestra divergencia

# 3. Excluir commits (operador NOT)
git log main --not feature-x
# → Commits en main que NO están en feature-x
# → Equivalente a: git log feature-x..main

git log --all --not origin/main
# → Todo excepto lo que está en origin/main
# → Útil para ver trabajo local en todas las ramas

git log HEAD --not origin/main --not origin/develop
# → Commits locales no pusheados a ninguna de esas ramas

# 4. Commits que tocan archivo específico
git log -- archivo.txt
# → Historia de archivo específico
# → Sigue renames y movimientos

git log --all -- archivo.txt
# → Busca archivo en TODAS las ramas

# 5. Commits entre dos fechas
git log --since="2024-01-01" --until="2024-12-31"
# o: --after / --before

git log --since="2 weeks ago"
git log --since="yesterday"
git log --after="2024-01-01 10:30"

# 6. Últimos N commits
git log -n 5
# o: git log -5
# → Solo 5 commits más recientes

# 7. Commits desde tag específico
git log v1.0.0..HEAD
# → Commits desde release v1.0.0 hasta ahora

# 8. Primeros N commits (más antiguos)
git log --reverse | head -20
# → Invierte orden, muestra más antiguos

# 9. Commits de merge específicamente
git log --merges
# → Solo merge commits

git log --no-merges
# → Excluye merge commits (útil para features)

# 10. Commits que NO están en remoto
git log origin/main..HEAD --oneline
# → Ver qué falta pushear
```

**Uso práctico - Búsquedas y filtros:** [🔙](#5-git-log---explorando-la-historia)

```bash
# ============================================
# BÚSQUEDA EN COMMITS
# ============================================

# 1. Buscar en mensaje de commit
git log --grep="fix"
# → Commits con "fix" en el mensaje
# → Case-sensitive por defecto

git log --grep="bug" --grep="fix" --all-match
# → Commits con AMBAS palabras

git log --grep="feature" --grep="refactor" 
# → Commits con CUALQUIERA de las palabras (OR)

git log -i --grep="FIX"
# → Case-insensitive

# 2. Buscar por autor
git log --author="John"
git log --author="john@example.com"
git log --author="John\|Maria"  # Regex: John O Maria

# 3. Buscar por committer (diferente de author)
git log --committer="Jenkins"
# → Útil para commits automáticos

# 4. Buscar cambios en código (pickaxe)
git log -S"función_importante"
# → Commits que AÑADIERON o ELIMINARON ese string
# → Super útil para encontrar cuándo desapareció algo

git log -S"password" --all
# → Busca en todas las ramas

# 5. Buscar cambios en código (con diff)
git log -G"regex.*pattern"
# → Commits donde el diff matchea el regex
# → Más flexible que -S

# 6. Buscar por función específica (para lenguajes soportados)
git log -L :nombre_funcion:archivo.py
# → Historia de esa función específica
# → Sigue renames y movimientos

git log -L 10,20:archivo.txt
# → Historia de líneas 10-20 de archivo

# 7. Commits que afectan ruta específica
git log -- src/
git log -- "*.js"
git log -- src/**/*.py

# 8. Commits que tocan múltiples archivos
git log -- archivo1.txt archivo2.txt

# 9. Buscar commits que modificaron archivo específico
git log --diff-filter=M -- archivo.txt
# M = modificado
# A = añadido
# D = eliminado
# R = renombrado
# C = copiado

git log --diff-filter=D --summary
# → Archivos eliminados

# 10. Seguir renames de archivo
git log --follow -- archivo.txt
# → Sigue historia aunque cambie de nombre
# → IMPORTANTE para archivos renombrados
```

**Uso práctico - Formatos personalizados avanzados:** [🔙](#5-git-log---explorando-la-historia)

```bash
# ============================================
# PRETTY FORMATS (PERSONALIZACIÓN)
# ============================================

# Placeholders comunes:
# %H  - Hash completo
# %h  - Hash corto
# %T  - Tree hash
# %P  - Parent hashes
# %an - Author name
# %ae - Author email
# %ad - Author date
# %ar - Author date, relative (2 days ago)
# %cn - Committer name
# %cd - Commit date
# %cr - Commit date, relative
# %s  - Subject (mensaje)
# %b  - Body (mensaje completo)
# %d  - Ref names (HEAD, branches, tags)

# Colores:
# %C(red), %C(green), %C(blue), %C(yellow)
# %C(bold), %C(dim), %C(reset)

# 1. Formato compacto con autor y fecha
git log --pretty=format:"%h %an %ar: %s"
# abc123 John 2 days ago: Fix bug

# 2. Formato con colores
git log --pretty=format:"%C(yellow)%h%C(reset) %C(blue)%an%C(reset) %s"

# 3. Formato para CSV/export
git log --pretty=format:"%h,%an,%ae,%ad,%s" --date=short > commits.csv

# 4. Formato con árbol decorado
git log --graph --pretty=format:"%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %s %C(bold yellow)%d%C(reset)"

# 5. Mostrar parent commits
git log --pretty=format:"%h %P %s"
# → Útil para entender merges

# 6. Formato JSON-like (para scripts)
git log --pretty=format:'{"commit":"%H","author":"%an","date":"%ad","message":"%s"}' --date=iso

# 7. Solo hash (para scripting)
git log --pretty=format:"%H"

# 8. Formato detallado con body
git log --pretty=format:"%h - %an (%ar)%n%n  %s%n%n%b%n" -3
```

**Uso práctico - Filtros de archivos y paths:**

```bash
# ============================================
# FILTROS POR ARCHIVOS Y RUTAS
# ============================================

# 1. Historia de archivo específico
git log -- ruta/archivo.txt

# 2. Historia de directorio
git log -- src/utils/

# 3. Archivos con patrón
git log -- "*.js"
git log -- "src/**/*.py"

# 4. Múltiples archivos
git log -- archivo1.txt archivo2.txt

# 5. Excluir paths
git log -- . ":(exclude)tests/"
git log -- . ":(exclude)*.md"

# 6. Solo archivos modificados (no añadidos/eliminados)
git log --diff-filter=M

# 7. Solo archivos añadidos
git log --diff-filter=A --summary

# 8. Solo archivos eliminados
git log --diff-filter=D --name-only

# 9. Solo archivos renombrados
git log --diff-filter=R --summary

# 10. Cambios en archivo específico con diff
git log -p -- archivo.txt

# 11. Mostrar nombres de archivos afectados
git log --name-only
git log --name-status  # Con tipo de cambio (M/A/D/R)

# 12. Mostrar estadísticas por archivo
git log --stat -- src/

# 13. Seguir archivo renombrado
git log --follow -- nuevo-nombre.txt
# → Sigue historia aunque se haya renombrado
```

**Uso práctico - Análisis y estadísticas:**

```bash
# ============================================
# ANÁLISIS DE REPOSITORIO
# ============================================

# 1. Contar commits por autor
git log --pretty=format:"%an" | sort | uniq -c | sort -rn
# Ejemplo output:
#   150 John Doe
#    95 Jane Smith
#    42 Bob Johnson

# 2. Contar commits por mes
git log --pretty=format:"%ad" --date=short | cut -c1-7 | sort | uniq -c

# 3. Actividad por día de la semana
git log --pretty=format:"%ad" --date=format:"%A" | sort | uniq -c | sort -rn

# 4. Ver quién modificó cada línea de archivo
git blame archivo.txt
git log -p -M --follow --stat -- archivo.txt

# 5. Commits en última semana
git log --since="1 week ago" --oneline | wc -l

# 6. Tamaño de commits (líneas cambiadas)
git log --shortstat --oneline

# 7. Archivos más modificados
git log --pretty=format: --name-only | sort | uniq -c | sort -rn | head -20

# 8. Autores más activos en archivo
git log --follow --pretty=format:"%an" -- archivo.txt | sort | uniq -c | sort -rn

# 9. Frecuencia de commits por hora
git log --pretty=format:"%ad" --date=format:"%H" | sort | uniq -c

# 10. Velocidad de desarrollo (commits/día)
git log --since="1 month ago" --pretty=format:"%ad" --date=short | sort | uniq -c
```

**Uso práctico - Debugging y bisect:**

```bash
# ============================================
# ENCONTRAR BUGS Y CAMBIOS
# ============================================

# 1. ¿Cuándo se introdujo este string?
git log -S"bug_causante" --source --all
# → Encuentra commit que añadió/eliminó ese código

# 2. ¿Cuándo se borró esta función?
git log -G"function delete_user" --all

# 3. ¿Quién cambió estas líneas?
git log -L 150,160:archivo.py
# → Historia de líneas 150-160

# 4. ¿En qué commit desapareció este archivo?
git log --all --full-history -- archivo-borrado.txt

# 5. Ver cambios entre dos versiones
git log v1.0..v2.0 --oneline

# 6. Commits que tocaron archivo Y contienen palabra
git log --grep="refactor" -- archivo.txt

# 7. Primer commit que introdujo archivo
git log --diff-filter=A --follow -- archivo.txt

# 8. Último commit que tocó archivo
git log -1 -- archivo.txt

# 9. Commits ordenados por fecha de commit (no autor)
git log --date-order

# 10. Ver commit y sus cambios
git show abc123
git show abc123:archivo.txt  # Ver versión de archivo en ese commit
```

**Uso práctico - Visualización avanzada:**

```bash
# ============================================
# GRAFOS Y VISUALIZACIÓN
# ============================================

# 1. Grafo completo decorado
git log --oneline --graph --all --decorate

# 2. Grafo solo de rama actual
git log --oneline --graph

# 3. Grafo con estadísticas
git log --graph --stat --oneline

# 4. Grafo compacto con fechas
git log --graph --date=relative --pretty=format:"%h %ad %s"

# 5. Ver merge commits con ambas líneas
git log --oneline --graph --first-parent
# → Sigue solo primera línea (más limpio en repos complejos)

# 6. Simplificar grafo (solo merges importantes)
git log --oneline --graph --simplify-by-decoration

# 7. Topological order (respeta estructura)
git log --topo-order --graph

# 8. Reverse chronological (más recientes primero) - default
git log --date-order

# 9. Author order (por fecha de author, no commit)
git log --author-date-order
```

**Opciones avanzadas y combinaciones:**

```bash
# ============================================
# COMBINACIONES PODEROSAS
# ============================================

# 1. Commits no pusheados con diff
git log origin/main..HEAD -p

# 2. Actividad de autor en fecha específica
git log --author="John" --since="2024-01-01" --until="2024-01-31" --oneline

# 3. Commits que afectan múltiples áreas
git log -- src/auth/ src/api/ --oneline

# 4. Merges problemáticos (con conflictos resueltos)
git log --merges -p --cc
# --cc muestra combined diff

# 5. Commits sin merge con stats de archivos JavaScript
git log --no-merges --stat -- "*.js"

# 6. Buscar en todas las ramas palabra en mensaje
git log --all --grep="JIRA-123"

# 7. Ver qué ramas contienen commit
git branch --contains abc123

# 8. Listar tags con sus commits
git log --oneline --decorate --simplify-by-decoration

# 9. Commits que modificaron permisos
git log -p | grep "old mode\|new mode"

# 10. Formato para code review
git log --oneline --no-merges --reverse v1.0.0..HEAD
```

**Alias recomendados para .gitconfig:**

```bash
[alias]
    # Log visual completo
    lg = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
    
    # Log compacto
    ls = log --oneline --decorate
    
    # Log con stats
    ll = log --stat --abbrev-commit
    
    # Último commit
    last = log -1 HEAD --stat
    
    # Commits no pusheados
    unpushed = log @{u}..HEAD --oneline
    
    # Commits no traidos del remoto
    unpulled = log HEAD..@{u} --oneline
    
    # Historial de archivo
    filelog = log --follow -p --
    
    # Contribuciones por autor
    contributors = shortlog --summary --numbered --email
    
    # Grafo simple
    tree = log --oneline --graph --decorate --all
    
    # Ver qué cambió hoy
    today = log --since="midnight" --oneline --author="Tu Nombre"
    
    # Buscar en commits
    search = log --all --grep
```

**Troubleshooting y problemas comunes:**

```bash
# ============================================
# PROBLEMAS Y SOLUCIONES
# ============================================

# Problema 1: Log muy largo, no puedo salir
# → Presiona 'q' para salir del pager (less)

# Problema 2: No veo colores
git config --global color.ui auto

# Problema 3: Log de archivo no muestra nada
git log --all --full-history -- archivo.txt
# → Busca en todas las ramas e historia completa

# Problema 4: Quiero log sin paginación
git --no-pager log
# o:
git log | cat

# Problema 5: Log muy lento en repo grande
git log --oneline -100  # Limita resultados
git log --since="1 month ago"  # Limita rango

# Problema 6: No encuentro commit con mensaje específico
git log --all --grep="texto" -i
# → Busca case-insensitive en todas las ramas

# Problema 7: Quiero exportar log a archivo
git log --pretty=format:"%h %an %ad %s" --date=short > log.txt

# Problema 8: No sé qué commits faltan traer
git fetch
git log HEAD..origin/main --oneline

# Problema 9: Grafo muy complejo, no entiendo
git log --oneline --graph --first-parent
# → Solo primera línea (más simple)

# Problema 10: Busco commit pero no recuerdo rama
git log --all -S"texto_unico" --source
# → Muestra en qué rama está cada commit
```

**Casos de uso del mundo real:**

```bash
# ============================================
# ESCENARIOS REALES
# ============================================

# 1. Code review de PR
git log main..feature-branch --oneline --no-merges

# 2. ¿Qué cambió en último release?
git log v1.9.0..v2.0.0 --oneline

# 3. Auditoría de seguridad
git log -S"password" --all -p

# 4. ¿Quién rompió el build?
git log --since="yesterday" --until="now" --oneline

# 5. Generar CHANGELOG
git log v1.0.0..HEAD --pretty=format:"- %s (%h)" --no-merges

# 6. Encontrar cuándo se introdujo bug
git log -S"bug_code" -p

# 7. Ver trabajo de la semana pasada
git log --author="$(git config user.name)" --since="1 week ago" --oneline

# 8. Comparar actividad entre ramas
git log develop --not main --oneline

# 9. Listar todos los merges de feature branches
git log --merges --grep="Merge branch 'feature" --oneline

# 10. Verificar que commit está en producción
git log origin/production --oneline | grep abc123
```

**Mejores prácticas:** [🔙](#5-git-log---explorando-la-historia)

```bash
✓ Usa --oneline para overview rápido
✓ Usa --graph para entender merges
✓ Usa --all para ver TODO el repositorio
✓ Usa --follow para archivos renombrados
✓ Usa -S o -G para buscar código
✓ Usa --not para exclusiones complejas
✓ Crea alias para comandos frecuentes
✓ Limita resultados con -n en repos grandes
✓ Usa --stat para resumen de cambios
✓ Combina --since y --until para rangos específicos

✗ No corras git log sin límites en repos gigantes
✗ No olvides --follow para archivos renombrados
✗ No uses --all si solo necesitas rama actual
✗ No ignores --no-merges para análisis de features
✗ No uses formato complejo sin guardarlo en alias
```

---

## 6. git branch - Gestionando Líneas de Desarrollo
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Crea, lista, renombra, elimina y gestiona ramas (branches). Las ramas en Git son extremadamente ligeras: solo punteros a commits, no copias de archivos.

**Funcionamiento interno:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```
Crear rama:
1. Obtiene hash del commit actual (HEAD)
2. Crea archivo .git/refs/heads/nombre-rama con el hash
3. Tamaño: Solo 41 bytes (hash SHA-1 + newline)
4. Tiempo: Instantáneo (milisegundos)

Eliminar rama:
1. Verifica si está mergeada (con -d)
2. Elimina archivo .git/refs/heads/nombre-rama
3. No toca commits (quedan en reflog si es necesario recuperar)

Cambiar entre ramas:
1. Lee hash del commit de la rama destino
2. Actualiza working directory con ese tree object
3. Actualiza .git/HEAD para apuntar a la nueva rama
4. Actualiza .git/index (staging area)
```

**Uso práctico - Creación de ramas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# CREAR RAMAS CON GIT BRANCH
# ============================================

# 1. Crear rama sin cambiar a ella
git branch feature-x
# → Crea rama apuntando a HEAD actual
# → Te quedas en la rama actual

# 2. Crear rama desde commit específico
git branch feature-x abc123
git branch hotfix v1.2.3
# → Crea rama apuntando al commit especificado

# 3. Crear rama desde otra rama (no desde HEAD)
git branch feature-y feature-x
# → Crea feature-y apuntando donde está feature-x

# 4. Crear rama desde remota
git branch feature-x origin/feature-x
# → Crea rama local basada en remota
# → Solo crea, NO cambia a ella
# → NO configura tracking automáticamente

# 5. Copiar rama (crear con mismo contenido)
git branch nueva-copia rama-existente
# → nueva-copia apunta al mismo commit que rama-existente

# Nota: Para crear Y cambiar de rama, ver:
# - Sección "git switch" para método moderno
# - Sección "git checkout" para método clásico
```

**Uso práctico - Listar y ver ramas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# LISTAR RAMAS
# ============================================

# 1. Listar ramas locales
git branch
# → Muestra rama actual con *
# → Solo ramas locales

# 2. Listar todas las ramas (local + remoto)
git branch -a
# o: git branch --all
# → Locales + remotes/origin/*
# → Muy útil para ver qué hay en remoto

# 3. Listar solo ramas remotas
git branch -r
# o: git branch --remotes
# → Solo origin/main, origin/develop, etc.

# 4. Listar con último commit
git branch -v
# o: git branch --verbose
# Formato: nombre hash mensaje
# ejemplo:
#   main     abc123 Last commit message
# * feature  def456 Work in progress

# 5. Listar con información de tracking
git branch -vv
# Formato: nombre hash [upstream: ahead N, behind M] mensaje
# ejemplo:
#   main     abc123 [origin/main] Last commit
# * feature  def456 [origin/feature: ahead 2] WIP

# 6. Listar con más detalles (commit y autor)
git branch -v --abbrev-commit
git branch -vv --format="%(refname:short) %(objectname:short) %(upstream:track) %(committerdate:relative)"

# 7. Listar ramas mergeadas a rama actual
git branch --merged
# → Muestra ramas ya integradas en HEAD
# → Candidatas para eliminación
# → Solo muestra si merge fue completo

git branch --merged main
# → Ramas mergeadas a main (no necesariamente a HEAD)

# 8. Listar ramas NO mergeadas
git branch --no-merged
# → Ramas con commits únicos aún
# → Trabajo pendiente de integrar

git branch --no-merged main
# → Ramas no mergeadas a main

# 9. Listar ramas con patrón
git branch --list "feature/*"
git branch --list "*fix*"
# → Filtrado por patrón wildcard

# 10. Listar ramas que contienen commit
git branch --contains abc123
git branch --contains v1.0.0
# → Ramas que incluyen ese commit en su historia

# 11. Listar ramas que NO contienen commit
git branch --no-contains abc123
# → Ramas que no tienen ese commit

# 12. Ordenar por diferentes criterios
git branch --sort=-committerdate
# → Más recientemente modificadas primero
git branch --sort=authordate
git branch --sort=objectsize
```

**Uso práctico - Eliminar ramas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# ELIMINAR RAMAS
# ============================================

# 1. Eliminar rama local (safe)
git branch -d feature-x
# → Solo elimina si está mergeada
# → Previene pérdida de trabajo
# → Error si tiene commits únicos

# 2. Eliminar rama local (force)
git branch -D feature-x
# → Elimina aunque no esté mergeada
# → ⚠️ Puede perder trabajo
# → Útil para abandonar experimentos

# 3. Eliminar rama remota
git push origin --delete feature-x
# o: git push origin :feature-x (sintaxis vieja)
# → Elimina rama en GitHub/GitLab/etc
# → Referencias locales quedan (limpia con fetch --prune)

# 4. Eliminar múltiples ramas locales
git branch -d rama1 rama2 rama3
# → Elimina varias a la vez

# 5. Eliminar todas las ramas mergeadas
git branch --merged main | grep -v "^\*" | grep -v "main" | xargs git branch -d
# → Limpieza masiva de ramas ya integradas
# → Excluye main y rama actual (*)

# 6. Eliminar ramas locales cuyo remoto ya no existe
git fetch --prune
git branch -vv | grep ': gone]' | awk '{print $1}' | xargs git branch -D
# → Limpia "ramas fantasma"
# → Útil tras eliminación de ramas remotas

# 7. Forzar eliminación sin verificar merge
git branch -D feature-x feature-y hotfix-z
# → Borra múltiples sin verificación
```

**Uso práctico - Renombrar ramas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# RENOMBRAR RAMAS
# ============================================

# 1. Renombrar rama actual
git branch -m nuevo-nombre
# → Estás en la rama, la renombras

# 2. Renombrar otra rama (no actual)
git branch -m viejo-nombre nuevo-nombre
# → Renombras desde fuera de ella

# 3. Renombrar y actualizar remoto
git branch -m old-name new-name  # Renombrar local
git push origin :old-name        # Eliminar remoto viejo
git push origin new-name         # Subir nuevo nombre
git push origin -u new-name      # Configurar tracking

# 4. Forzar renombrado (sobrescribe si existe)
git branch -M nuevo-nombre
# → Como -m pero fuerza sobrescritura

# 5. Renombrar main a master (o viceversa)
git branch -m master main
git push -u origin main
git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/main
```

**Uso práctico - Gestión avanzada:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# OPERACIONES AVANZADAS
# ============================================

# 1. Mover rama a otro commit (sin checkout)
git branch -f feature-x abc123
# → Mueve puntero de feature-x al commit abc123
# → No necesitas estar en feature-x
# → ⚠️ Reescribe historia si mueves atrás

# 2. Copiar rama
git branch nueva-copia rama-original
# → Crea nueva-copia apuntando donde rama-original

# 3. Configurar upstream de rama existente
git branch -u origin/feature-x
# o: git branch --set-upstream-to=origin/feature-x
# → Configura tracking para push/pull
# → Útil si creaste rama local sin -track

# 4. Ver upstream configurado
git branch -vv
# → Muestra [origin/rama] si tiene upstream

# 5. Quitar upstream
git branch --unset-upstream
# → Elimina configuración de tracking
# → Deberás especificar remoto en push/pull

# 6. Editar descripción de rama
git branch --edit-description
# → Abre editor para añadir descripción
# → Útil para documentar propósito de rama

git branch --edit-description feature-x
# → Edita descripción de rama específica

# 7. Ver descripción de rama
git config branch.feature-x.description

# 8. Crear rama desde stash
git stash branch nueva-rama stash@{0}
# → Crea rama desde punto donde hiciste stash
# → Aplica cambios del stash
# → Elimina stash

# 9. Listar ramas con formato personalizado
git branch --format="%(refname:short) - %(authorname) - %(committerdate:short)"
# → Output personalizado
# → Ver más abajo sección completa de FORMAT

# 10. Ver ramas ordenadas por actividad
git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) %(committerdate:relative)'
# → Muy útil para ver qué ramas están activas
```

**Formato personalizado con --format (Completo):**

> 📖 **NOTA:** Para una referencia completa de todos los placeholders disponibles,
> formatos avanzados, condicionales y ejemplos con otros comandos (log, for-each-ref,
> show-ref, etc.), consulta la **[Sección 22: Referencias y Placeholders de Formato](#22-referencias-y-placeholders-de-formato)**.

```bash
# ============================================
# GIT BRANCH --FORMAT (PLACEHOLDERS COMPLETOS)
# ============================================

# git branch también acepta placeholders como git for-each-ref
# Ver sección 22 para lista completa de placeholders disponibles

# PLACEHOLDERS PRINCIPALES:

# Referencia
%(refname)              # refs/heads/main
%(refname:short)        # main
%(refname:lstrip=2)     # main (elimina "refs/heads/")

# Objeto
%(objectname)           # Hash completo SHA-1
%(objectname:short)     # Hash abreviado (7 chars)
%(objectname:short=10)  # Hash abreviado (10 chars)

# Commit info
%(tree)                 # Hash del árbol
%(parent)               # Hash(es) del padre
%(subject)              # Primera línea mensaje
%(body)                 # Cuerpo del mensaje
%(contents)             # Mensaje completo

# Autor
%(authorname)           # Nombre del autor
%(authoremail)          # Email del autor
%(authordate)           # Fecha del autor
%(authordate:relative)  # "2 days ago"
%(authordate:short)     # "2024-02-13"
%(authordate:iso)       # ISO 8601

# Committer
%(committername)        # Nombre del committer
%(committeremail)       # Email del committer
%(committerdate)        # Fecha del committer
%(committerdate:relative)
%(committerdate:short)
%(committerdate:iso)

# Tracking (upstream)
%(upstream)             # refs/remotes/origin/main
%(upstream:short)       # origin/main
%(upstream:track)       # [ahead 2, behind 1]
%(upstream:trackshort)  # <> (diverged), > (ahead), < (behind), = (up to date)
%(upstream:remotename)  # origin
%(upstream:remoteref)   # refs/heads/main

# Estado
%(HEAD)                 # '*' si es rama actual, ' ' si no
%(color:...)            # Aplicar color
%(if)%(then)%(else)%(end) # Condicionales


# ============================================
# EJEMPLOS PRÁCTICOS DE FORMATO
# ============================================

# 1. Lista simple con hash
git branch --format="%(refname:short) %(objectname:short)"
# Salida:
# main a1b2c3d
# develop e4f5g6h
# feature/login i7j8k9l

# 2. Con información de tracking
git branch --format="%(refname:short) → %(upstream:short) %(upstream:track)"
# Salida:
# main → origin/main [up to date]
# develop → origin/develop [ahead 2, behind 1]
# feature/login →

# 3. Con tracking abreviado (símbolos)
git branch --format="%(HEAD) %(refname:short) %(upstream:trackshort)"
# Salida:
# * main =     (actual, up to date)
#   develop <> (diverged)
#   feature >  (ahead)

# 4. Con último commit y autor
git branch --format="%(refname:short) | %(authorname) | %(committerdate:relative) | %(subject)"
# Salida:
# main | Juan Pérez | 2 days ago | Fix login bug
# develop | María García | 1 week ago | Add feature X

# 5. Con colores (visual)
git branch --format="%(if)%(HEAD)%(then)%(color:green)* %(else)  %(end)%(color:yellow)%(refname:short)%(color:reset) %(upstream:trackshort)"
# Salida coloreada:
# * main >    (verde si actual)
#   develop <  (amarillo)

# 6. Tabla alineada
git branch --format="%(align:20,left)%(refname:short)%(end) %(align:15,left)%(upstream:short)%(end) %(align:20,right)%(committerdate:short)%(end)"
# Salida:
# main                 origin/main     2024-02-13
# develop              origin/develop  2024-02-12
# feature/login                        2024-02-10

# 7. Solo ramas sin upstream
git branch --format="%(if)%(upstream)%(then)%(else)%(refname:short)%(end)" | grep -v '^$'
# Salida:
# feature/login
# hotfix/temp

# 8. Solo ramas con upstream (con estado)
git branch --format="%(if)%(upstream)%(then)%(refname:short) → %(upstream:short) %(upstream:track)%(end)" | grep -v '^$'
# Salida:
# main → origin/main [up to date]
# develop → origin/develop [ahead 2]

# 9. Formato tipo GitHub
git branch --format="%(color:bold yellow)%(refname:short)%(color:reset) %(color:dim)%(objectname:short)%(color:reset) %(subject)" --sort=-committerdate
# Salida:
# feature/new-ui a1b2c3d Add new dashboard
# develop e4f5g6h Merge feature X
# main i7j8k9l Hotfix security

# 10. Información completa para revisión
git branch --format="Rama: %(refname:short)
  Hash: %(objectname:short)
  Upstream: %(upstream:short)
  Estado: %(upstream:track)
  Último commit: %(subject)
  Autor: %(authorname)
  Fecha: %(committerdate:short)
  ---"

# 11. Solo ramas mergeadas con marca visual
git branch --merged main --format="✓ %(refname:short) (merged)"

# 12. Solo ramas NO mergeadas con marca visual
git branch --no-merged main --format="✗ %(refname:short) (%(committerdate:relative))"
# Salida:
# ✗ feature/new-ui (2 days ago)
# ✗ hotfix/urgent (5 hours ago)

# 13. Export a CSV para análisis
git branch --format="%(refname:short),%(objectname:short),%(authorname),%(authoremail),%(committerdate:short),%(subject)" > branches.csv

# 14. Buscar ramas de un autor específico
git branch --format="%(if:equals=Juan Pérez)%(authorname)%(then)%(refname:short) - %(subject)%(end)" | grep -v '^$'

# 15. Ramas con commits recientes (últimos 7 días)
git branch --format="%(if:newer=7.days.ago)%(committerdate)%(then)%(refname:short) - %(committerdate:relative)%(end)" | grep -v '^$'


# ============================================
# ORDENAMIENTO CON --sort
# ============================================

# Por fecha de commit (más recientes primero)
git branch --sort=-committerdate --format="%(committerdate:short) %(refname:short)"

# Por fecha de commit (más antiguos primero)
git branch --sort=committerdate --format="%(committerdate:short) %(refname:short)"

# Por nombre alfabético
git branch --sort=refname

# Por nombre alfabético inverso
git branch --sort=-refname

# Por fecha de autor
git branch --sort=-authordate --format="%(authordate:short) %(refname:short) %(authorname)"

# Múltiples criterios (fecha, luego nombre)
git branch --sort=-committerdate --sort=refname


# ============================================
# FILTROS COMBINADOS
# ============================================

# Ramas remotas sin merge con formato
git branch -r --no-merged main --format="%(refname:short) %(committerdate:relative)"

# Ramas locales que contienen un commit
git branch --contains abc123 --format="%(refname:short) ✓"

# Ramas locales que NO contienen un commit
git branch --no-contains abc123 --format="%(refname:short) ✗"

# Ramas con patrón y formato
git branch --list "feature/*" --format="%(refname:short) - %(subject)"


# ============================================
# CONDICIONALES AVANZADOS
# ============================================

# Mostrar solo si está ahead
git branch --format="%(if:notequals=)%(upstream:track)%(then)%(refname:short) %(upstream:track)%(end)" | grep -v '^$'

# Colorear según estado de tracking
git branch --format="%(if)%(upstream:track)%(then)%(color:red)%(else)%(color:green)%(end)%(refname:short)%(color:reset) %(upstream:track)"

# Marcar ramas sin upstream
git branch --format="%(refname:short)%(if)%(upstream)%(then) [tracked]%(else) [NO UPSTREAM]%(end)"
# Salida:
# main [tracked]
# feature/new [NO UPSTREAM]


# ============================================
# CASOS DE USO PRÁCTICOS
# ============================================

# 1. Encontrar ramas abandonadas (>3 meses sin commits)
git branch --sort=-committerdate --format="%(committerdate:short) %(refname:short)" | tail -10

# 2. Ver quién trabaja en qué
git branch --format="%(authorname): %(refname:short)" --sort=authorname

# 3. Estado de tracking de todas las ramas (dashboard)
git branch --format="%(align:25,left)%(refname:short)%(end)%(if)%(upstream)%(then)→ %(upstream:short) %(upstream:trackshort)%(else)(sin tracking)%(end)"
# Salida:
# main                     → origin/main =
# develop                  → origin/develop >
# feature/login            (sin tracking)

# 4. Ramas con commits pero sin push
git branch --format="%(if)%(upstream:track)%(then)%(if:equals=[ahead ?)%(upstream:track)%(then)%(refname:short) tiene commits locales%(end)%(end)" | grep -v '^$'

# 5. Generar comando para eliminar ramas mergeadas
git branch --merged main --format="git branch -d %(refname:short)" | grep -v "main"
# Salida (ejecutable):
# git branch -d feature-old
# git branch -d bugfix-123


# ============================================
# NOTA IMPORTANTE
# ============================================
# git branch --no-merged NO MUESTRA LA RAMA ACTUAL aunque no esté merged
# Esto es comportamiento estándar de Git

# Para verificar si tu rama actual está merged:
git branch --contains HEAD main
# o
git merge-base --is-ancestor HEAD main && echo "Está merged" || echo "NO está merged"
```

**Estrategias de branching completas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# FEATURE BRANCH WORKFLOW
# ============================================
# Estrategia: Una rama por feature, merge a main

main (estable, deployable)
 ├─ feature/user-authentication
 ├─ feature/payment-integration  
 ├─ feature/dashboard-redesign
 ├─ bugfix/login-timeout
 └─ hotfix/security-patch

Workflow:
git switch main
git pull origin main
git switch -c feature/nueva-feature
# ... desarrollo ...
git push -u origin feature/nueva-feature
# PR en GitHub/GitLab
# Tras aprobación:
git switch main
git merge --no-ff feature/nueva-feature
git push origin main
git branch -d feature/nueva-feature
git push origin --delete feature/nueva-feature

# ============================================
# GIT FLOW
# ============================================
# Estrategia: Ramas de largo plazo + features temporales

main (producción, solo releases)
 └─ hotfix/critical-bug → merge a main y develop
 
develop (integración, siguiente release)
 ├─ feature/feature-a → merge a develop
 ├─ feature/feature-b → merge a develop
 └─ release/v2.0.0 → merge a main y develop

Workflow nuevas features:
git switch develop
git switch -c feature/nueva-feature
# ... desarrollo ...
git switch develop
git merge --no-ff feature/nueva-feature
git branch -d feature/nueva-feature

Workflow releases:
git switch -c release/v1.5.0 develop
# ... bug fixes, versioning ...
git switch main
git merge --no-ff release/v1.5.0
git tag -a v1.5.0
git switch develop
git merge --no-ff release/v1.5.0
git branch -d release/v1.5.0

Workflow hotfixes:
git switch -c hotfix/critical main
# ... fix urgente ...
git switch main
git merge --no-ff hotfix/critical
git tag -a v1.5.1
git switch develop
git merge --no-ff hotfix/critical
git branch -d hotfix/critical

# ============================================
# GITHUB FLOW (SIMPLE)
# ============================================
# Estrategia: Solo main + ramas temporales, deploy continuo

main (siempre deployable)
 ├─ add-oauth-support
 ├─ fix-memory-leak
 └─ update-dependencies

Workflow:
git switch main
git pull origin main
git switch -c descriptive-branch-name
# ... commits ...
git push -u origin descriptive-branch-name
# Abrir Pull Request
# CI/CD ejecuta tests
# Code review
# Merge a main
# Auto-deploy a producción
# Eliminar rama

# ============================================
# TRUNK-BASED DEVELOPMENT
# ============================================
# Estrategia: Ramas de vida muy corta (<1 día), main siempre estable

main (trunk, siempre estable)
 ├─ short-lived-branch-1 (< 1 día)
 └─ short-lived-branch-2 (< 1 día)

Principios:
- Ramas viven máximo 1 día
- Commits pequeños y frecuentes
- Feature flags para features incompletas
- CI/CD muy robusto

Workflow:
git switch main
git pull origin main
git switch -c quick-fix
# ... cambio pequeño ...
git push -u origin quick-fix
# PR rápido, merge mismo día
git switch main
git pull origin main
git branch -d quick-fix
```

**Convenciones de nombres de ramas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# NOMENCLATURA RECOMENDADA
# ============================================

# Por tipo:
feature/user-authentication
feature/payment-gateway
bugfix/login-error
hotfix/security-vulnerability
release/v1.5.0
docs/update-readme
test/add-integration-tests
refactor/optimize-queries
chore/update-dependencies

# Por ticket/issue:
feature/JIRA-123-add-oauth
bugfix/GH-456-fix-memory-leak
hotfix/PROD-789-critical-fix

# Por desarrollador (en equipos pequeños):
john/new-dashboard
maria/fix-api

# Convenciones:
✓ Usa minúsculas
✓ Usa guiones (no underscores)
✓ Sé descriptivo pero conciso
✓ Incluye tipo de cambio
✓ Incluye referencia a ticket si existe

✗ No uses espacios
✗ No uses caracteres especiales (/, - solo)
✗ No uses nombres ambiguos ("fix", "test", "branch")
✗ No uses fechas como única identificación
```

**Troubleshooting y problemas comunes:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
# ============================================
# PROBLEMAS COMUNES
# ============================================

# Problema 1: No puedo cambiar de rama (cambios sin commitear)
git switch otra-rama
# error: Your local changes would be overwritten

Solución A (commitear):
git add .
git commit -m "WIP: trabajo en progreso"
git switch otra-rama

Solución B (stash):
git stash
git switch otra-rama
# ... trabajo ...
git switch rama-original
git stash pop

Solución C (forzar, ⚠️ pierdes cambios):
git switch -f otra-rama

# Problema 2: Borré rama por error
git reflog
# Encuentra el commit donde estaba la rama
git branch rama-recuperada abc123
# o:
git switch -c rama-recuperada abc123

# Problema 3: Rama no se elimina (no mergeada)
git branch -d feature-x
# error: branch not fully merged

Verificar:
git branch --no-merged
git log main..feature-x --oneline

Si realmente quieres borrar:
git branch -D feature-x

# Problema 4: Rama local dice "gone" en tracking
git branch -vv
# feature-x abc123 [origin/feature-x: gone] WIP

Causa: Rama remota fue eliminada
Solución:
git branch -D feature-x  # Si no necesitas cambios
# o
git branch --unset-upstream  # Quita tracking, mantén rama local

# Problema 5: Demasiadas ramas, repo desorganizado
# Listar ramas inactivas (más de 6 meses):
git for-each-ref --sort=-committerdate refs/heads/ \
  --format='%(refname:short) %(committerdate:relative)' | \
  tail -20

# Eliminar ramas mergeadas:
git branch --merged main | grep -v "main" | xargs git branch -d

# Problema 6: Rama con nombre incorrecto ya pusheada
git branch -m old-name new-name
git push origin :old-name new-name
git push origin -u new-name

# Problema 7: Quiero ver rama antigua sin afectar HEAD
git show rama-antigua:archivo.txt
git log rama-antigua
git diff main..rama-antigua
# Sin hacer checkout

# Problema 8: No sé en qué rama estoy
git branch
# o
git rev-parse --abbrev-ref HEAD
# o
git status | head -1
```

**Mejores prácticas:** [🔙](#6-git-branch---gestionando-líneas-de-desarrollo)

```bash
✓ Usa nombres descriptivos (feature/user-auth)
✓ Usa prefijos (feature/, bugfix/, hotfix/)
✓ Crea ramas frecuentemente (son gratis)
✓ Elimina ramas después de merge

✗ Evita nombres genéricos (test, temp)
✗ No trabajes directamente en main
✗ No uses git branch -D sin estar seguro
```

---

## 7. git checkout / git switch - Navegando el Código
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Cambia de rama, navega por commits históricos, o restaura archivos del working directory. Es uno de los comandos más versátiles (y confusos) de Git, por eso se dividió en `git switch` (ramas) y `git restore` (archivos) en versiones modernas.

> **📝 NOTA IMPORTANTE:** Esta sección cubre **tres comandos diferentes**:
> - **git switch** (moderno) - Para cambiar de rama
> - **git restore** (moderno) - Para restaurar archivos
> - **git checkout** (legacy) - Hace ambas cosas (confuso)
>
> **Recomendación:** Usa `git switch` para ramas y `git restore` para archivos.

**Funcionamiento interno:** [🔙](#7-git-checkout--git-switch---navegando-el-código)

```
Al cambiar de rama:
1. Verifica que no haya conflictos con working directory
2. Lee hash del commit de la rama destino desde .git/refs/heads/rama
3. Lee tree object del commit destino
4. Compara tree actual con tree destino
5. Actualiza archivos en working directory (solo los diferentes)
6. Actualiza .git/index (staging area)
7. Actualiza .git/HEAD → ref: refs/heads/rama-destino
8. Si hay conflictos, aborta y muestra errores

Al checkout de archivo:
1. Lee archivo desde tree object del commit especificado
2. Sobrescribe archivo en working directory
3. Actualiza staging area con esa versión
4. NO cambia HEAD

Al checkout de commit (detached HEAD):
1. Similar a cambio de rama
2. Pero HEAD apunta directamente a commit (no a rama)
3. .git/HEAD contiene hash en vez de ref
4. Commits nuevos quedan "huérfanos" al cambiar
```

---

### 7.1. git switch - Cambiar de Rama (Moderno, Recomendado)

**Uso práctico - Cambiar entre ramas:** [🔙](#7-git-checkout--git-switch---navegando-el-código)

```bash
# 1. Cambiar a rama existente
git switch main
git switch feature-x

# 2. Crear rama nueva y cambiar a ella
git switch -c nueva-rama
# → Equivalente a: git branch nueva-rama && git switch nueva-rama

# 3. Crear rama desde commit específico
git switch -c hotfix abc123
git switch -c bugfix HEAD~3

# 4. Volver a rama anterior
git switch -
# → Alterna entre dos ramas rápidamente
# → Como "cd -" en bash

# 5. Crear y cambiar con tracking automático
git switch -c feature-x --track origin/feature-x
# → Configura upstream automáticamente

# 6. Forzar cambio (descarta cambios locales)
git switch -f otra-rama
# → ⚠️ Pierdes cambios no commiteados

# 7. Cambiar con merge de cambios locales
git switch -m otra-rama
# → Intenta mergear cambios locales a nueva rama

# 8. Cambiar a rama remota (crea local tracking)
git switch feature-x
# → Si no existe local pero sí origin/feature-x
# → Crea local automáticamente con tracking
```

**Uso práctico - Detached HEAD con switch:** [🔙](#7-git-checkout--git-switch---navegando-el-código)

```bash
# Entrar en Detached HEAD
git switch --detach abc123
git switch --detach HEAD~3
git switch --detach v1.0.0

# → HEAD apunta directamente a commit (no a rama)
# → Útil para inspección, no para desarrollo
```

---

### 7.2. git restore - Restaurar Archivos (Moderno, Recomendado)

**Uso práctico - Descartar cambios:** [🔙](#7-git-checkout--git-switch---navegando-el-código)

```bash
# 1. Descartar cambios en working directory
git restore file.txt
# → Restaura desde staging (o HEAD si no está staged)

# 2. Descartar todos los cambios
git restore .
# → Restaura todos los archivos modificados

# 3. Unstage archivo (quitar de staging)
git restore --staged file.txt
# → Mueve de staging a working directory
# → Equivalente a: git reset HEAD file.txt

# 4. Unstage y descartar cambios
git restore --staged --worktree file.txt
# → Quita de staging Y descarta cambios

# 5. Restaurar desde commit específico
git restore --source=abc123 file.txt
git restore --source=HEAD~3 file.txt
git restore --source=main file.txt

# 6. Restaurar archivo borrado
git restore deleted-file.txt
# → Solo si estaba tracked antes

# 7. Restaurar con patrón
git restore '*.js'
git restore 'src/**/*.txt'

# 8. Restaurar directorio completo
git restore src/
```

**Uso práctico - Casos especiales:** [🔙](#7-git-checkout--git-switch---navegando-el-código)

```bash
# Restaurar archivo de otra rama sin cambiar de rama
git restore --source=feature-x -- config.json

# Restaurar múltiples archivos de commit antiguo
git restore --source=HEAD~5 -- file1.txt file2.txt

# Ver qué se restauraría sin hacerlo (no existe, usa diff)
git diff file.txt  # Ver cambios antes de restaurar
git restore file.txt
```

---

### 7.3. git checkout - Comando Legacy (Multiuso)

**Comparación de sintaxis:**

```bash
# ============================================
# CHECKOUT (Multiuso, confuso - comando legacy)
# ============================================
git checkout main               # Cambiar de rama
git checkout -b nueva           # Crear y cambiar
git checkout abc123             # Ir a commit (detached HEAD)
git checkout -- file.txt        # Descartar cambios de archivo
git checkout abc123 file.txt    # Restaurar archivo desde commit
git checkout tags/v1.0.0        # Checkout de tag

# PROBLEMA: ¿checkout cambia rama o restaura archivo?
# → Sintaxis ambigua, fácil confundirse
# → Por eso se crearon switch y restore

# ============================================
# EQUIVALENCIAS: checkout → switch/restore
# ============================================

# Cambiar de rama:
git checkout main          →  git switch main
git checkout -b nueva      →  git switch -c nueva
git checkout -            →  git switch -

# Descartar cambios:
git checkout -- file.txt   →  git restore file.txt
git checkout -- .          →  git restore .

# Restaurar desde commit:
git checkout abc123 file.txt  →  git restore --source=abc123 file.txt

# Ir a commit:
git checkout abc123        →  git switch --detach abc123

# Restaurar desde otra rama:
git checkout main file.txt →  git restore --source=main file.txt
```

**Uso de checkout (si usas Git < 2.23):**

```bash
# Cambiar de rama
git checkout main
git checkout feature-x

# Crear y cambiar
git checkout -b nueva-rama
git checkout -b hotfix abc123

# Descartar cambios (IMPORTANTE: usa --)
git checkout -- file.txt
git checkout -- .

# Restaurar desde commit
git checkout abc123 -- file.txt
git checkout HEAD~3 -- file.txt

# Restaurar desde otra rama (sin cambiar)
git checkout feature-x -- src/lib.js

# Detached HEAD
git checkout abc123
git checkout v1.0.0
git checkout HEAD~5

# PROBLEMA con checkout:
git checkout rama           # ¿Cambia rama?
git checkout -- rama        # ¿O restaura archivo llamado "rama"?
# → Ambigüedad confusa, por eso switch/restore son mejores
```

---

### Detached HEAD - Explicación Completa

**¿Qué es Detached HEAD?**

```bash
# Estado normal (HEAD apunta a rama):
.git/HEAD contiene: ref: refs/heads/main
→ HEAD → main → commit abc123

# Detached HEAD (HEAD apunta a commit directamente):
.git/HEAD contiene: abc123
→ HEAD → commit abc123 (sin rama)

# Problema: Commits en detached HEAD quedan "huérfanos"
# Si cambias a otra rama, pierdes referencia a esos commits
```

**Entrar en Detached HEAD:**

```bash
# Con switch (moderno):
git switch --detach abc123
git switch --detach HEAD~3
git switch --detach v1.0.0

# Con checkout (legacy):
git checkout abc123
git checkout HEAD~5
git checkout v1.0.0
git checkout tags/v1.0.0
```

**¿Por qué usar Detached HEAD?**

```bash
✓ Inspeccionar código antiguo sin crear rama
✓ Probar build de versión específica
✓ Reproducir bug histórico
✓ Auditar cambios
✓ Ejecutar tests en commit específico

✗ NO para desarrollo (commits se pierden fácilmente)
✗ NO para trabajo que quieres guardar
```

**Salir de Detached HEAD:**

```bash
# Opción 1: Volver a rama (descarta trabajo en detached)
git switch main
# → Commits hechos en detached quedan sin referencia

# Opción 2: Crear rama con el trabajo (RECOMENDADO)
git switch -c nueva-rama
# → Convierte trabajo en rama permanente

# Opción 3: Crear rama apuntando a donde estás
git branch rescue-branch
git switch main
# → rescue-branch guarda tu trabajo
```

**Recuperar trabajo perdido en Detached HEAD:**

```bash
# Si saliste de detached sin crear rama:
git reflog
# Busca el commit donde estabas
git switch -c recuperar abc123
# o
git checkout -b recuperar abc123
```

**Ver si estás en Detached HEAD:**

```bash
git branch
# * (HEAD detached at abc123)  ← En detached
# * main                        ← En rama normal

git status
# HEAD detached at abc123       ← En detached
# On branch main                ← En rama normal
```

---

### Casos de Uso Avanzados

**Caso 1: Olvidé cambiar de rama antes de trabajar** [🔙](#7-git-checkout--git-switch---navegando-el-código)

```bash
# Estás en main, hiciste cambios, querías estar en feature
git stash
git switch feature-x
git stash pop

# o (con merge automático):
git switch -m feature-x
```

**Caso 2: Quiero archivo de otra rama sin cambiar**

```bash
# Con restore (moderno):
git restore --source=feature-x -- src/lib.js

# Con checkout (legacy):
git checkout feature-x -- src/lib.js
```

**Caso 3: Explorar bug histórico**

```bash
git log --oneline | grep "bug aparece"
# Encuentra commit: abc123

git switch --detach abc123
npm test  # Reproduce el bug
git switch main  # Vuelve a main
```

**Caso 4: Recuperar archivo borrado hace commits**

```bash
git log --oneline --all --full-history -- deleted-file.txt
# Encuentra último commit: def456

git restore --source=def456 -- deleted-file.txt
```

**Caso 5: Crear hotfix desde tag de producción**

```bash
git switch -c hotfix/critical v1.2.3
# ... fix ...
git commit -am "fix: Critical issue"
```

---

### Troubleshooting

**Problema 1: No puedo cambiar (cambios sin commitear)**

```bash
Solución A: Commitear
git add .
git commit -m "WIP"
git switch otra-rama

Solución B: Stash
git stash
git switch otra-rama
git stash pop

Solución C: Switch con merge
git switch -m otra-rama

Solución D: Forzar (⚠️ pierdes cambios)
git switch -f otra-rama
```

**Problema 2: Hice commits en Detached HEAD**

```bash
git reflog
# Encuentra el commit: abc123
git switch -c rescue-branch abc123
```

**Problema 3: Archivo y rama con mismo nombre**

```bash
# Moderno (sin ambigüedad):
git switch test          # Definitivamente rama
git restore test         # Definitivamente archivo

# Legacy (ambiguo):
git checkout test        # ¿Rama o archivo?
git checkout -- test     # Fuerza archivo
```

**Problema 4: Cambié de rama y perdí trabajo**

```bash
git reflog
git switch -c recuperar HEAD@{1}
# o
git checkout -b recuperar HEAD@{1}
```

---

### Mejores Prácticas

[🔙](#7-git-checkout--git-switch---navegando-el-código)

```bash
✓ Usa git switch para cambiar ramas (claro y específico)
✓ Usa git restore para archivos (sin ambigüedad)
✓ Commitea o stash antes de cambiar ramas
✓ Entiende detached HEAD antes de usarlo
✓ Crea rama desde detached si hiciste commits
✓ Usa git switch - para alternar entre dos ramas

✗ Evita git checkout (confuso y ambiguo)
✗ No trabajes en detached HEAD sin crear rama después
✗ No uses git checkout sin "--" para archivos
✗ No confundas switch (ramas) con restore (archivos)
✗ No asumas que checkout siempre cambia ramas
```

---

## 8. git merge - Integrando Cambios
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Integra cambios de una rama en otra, combinando el trabajo de diferentes líneas de desarrollo. Es uno de los comandos más críticos en Git para la colaboración en equipo.

**Funcionamiento interno:** [🔙](#8-git-merge---integrando-cambios)

```
Git merge puede operar de 3 formas diferentes:

1. FAST-FORWARD (merge "rápido"):
   main:    A---B
   feature:      C---D

   Resultado: main simplemente avanza al commit D
   main:    A---B---C---D

   → No crea merge commit
   → Solo mueve el puntero de la rama
   → Historia lineal limpia
   → Condición: main no ha avanzado desde que se creó feature

2. THREE-WAY MERGE (merge de 3 vías):
   main:    A---B---C
                \
   feature:      D---E

   Git usa 3 commits:
   - Ancestro común (B)
   - Último commit de main (C)
   - Último commit de feature (E)

   Resultado: Se crea nuevo merge commit (M)
   main:    A---B---C---M
                \     /
   feature:      D---E

   → Crea merge commit con 2 líneas de commits
   → Preserva historia completa
   → Historia no lineal (ramificada)

3. CONFLICTO:
   Cuando ambas ramas modifican las mismas líneas:
   → Git no puede decidir automáticamente
   → Marca conflictos en archivos
   → Requiere resolución manual
   → Crea merge commit tras resolver

Internamente:
1. git merge-base main feature  → Encuentra ancestro común
2. git diff-tree ancestro main  → Cambios en main
3. git diff-tree ancestro feature → Cambios en feature
4. Aplica ambos sets de cambios
5. Si no hay conflictos → merge automático
6. Si hay conflictos → pausa y marca conflictos
```

**Todas las opciones importantes:** [🔙](#8-git-merge---integrando-cambios)

```bash
# ============================================
# OPCIONES DE ESTRATEGIA DE MERGE
# ============================================

# 1. Merge básico (comportamiento por defecto)
git merge feature-x
# → Fast-forward si es posible
# → Three-way merge si no

# 2. Forzar merge commit (sin fast-forward)
git merge --no-ff feature-x
# → SIEMPRE crea merge commit
# → Preserva historia de rama
# → Útil para features importantes
# → Mantiene visible qué commits pertenecían a la feature

# 3. Solo fast-forward (falla si no es posible)
git merge --ff-only feature-x
# → Solo avanza puntero
# → Falla si requiere merge commit
# → Útil para mantener historia lineal estricta
# → Común en workflows con rebase

# 4. Squash merge (aplasta todos los commits en uno)
git merge --squash feature-x
# → Aplica TODOS los cambios de feature-x
# → NO crea merge commit automáticamente
# → Debes hacer commit manual después
# → Resultado: 1 solo commit en main
# → Pierde historia individual de commits de feature
git commit -m "Add complete feature X"

# 5. Merge con edición de mensaje
git merge --edit feature-x
# → Abre editor para personalizar mensaje de merge commit
# → Por defecto usa mensaje automático

git merge --no-edit feature-x
# → Usa mensaje automático sin preguntar

# ============================================
# ESTRATEGIAS DE MERGE
# ============================================

# 6. Estrategia "ours" (en conflictos, prefiere nuestra versión)
git merge -X ours feature-x
# → En conflictos automáticos, usa versión de rama actual
# → CUIDADO: Puede silenciar cambios importantes
# → Útil cuando estás seguro de que tu versión es correcta

# 7. Estrategia "theirs" (en conflictos, prefiere su versión)
git merge -X theirs feature-x
# → En conflictos automáticos, usa versión de rama entrante
# → CUIDADO: Puede sobrescribir tu trabajo
# → Útil cuando aceptas completamente los cambios externos

# 8. Estrategia "recursive" (por defecto, más opciones)
git merge -s recursive -X patience feature-x
# patience: Algoritmo más cuidadoso (más lento, menos conflictos)

git merge -s recursive -X diff-algorithm=histogram feature-x
# Algoritmos: myers (default), minimal, patience, histogram

git merge -s recursive -X ignore-space-change feature-x
# Ignora cambios solo de espacios en blanco

git merge -s recursive -X ignore-all-space feature-x
# Ignora todos los espacios al comparar

git merge -s recursive -X ignore-space-at-eol feature-x
# Ignora espacios al final de línea

git merge -s recursive -X renormalize feature-x
# Re-normaliza archivos (útil con cambios de line-endings)

# 9. Estrategia "octopus" (merge múltiples ramas)
git merge branch1 branch2 branch3
# → Merge de 3+ ramas simultáneamente
# → Falla si hay conflictos (no soporta resolución manual)
# → Útil para integrar múltiples features simples

# 10. Estrategia "ours" (NO confundir con -X ours)
git merge -s ours old-feature
# → IGNORA completamente cambios de old-feature
# → Solo registra merge en historia
# → Útil para marcar rama como mergeada sin aplicar cambios
# → Diferente de -X ours (que sí intenta merge)

# 11. Estrategia "subtree"
git merge -s subtree -X subtree=libs/ external-lib
# → Merge de repositorio externo como subdirectorio
# → Útil para dependencias embebidas

# ============================================
# OPCIONES DE CONTROL Y VERIFICACIÓN
# ============================================

# 12. Verificar merge sin hacerlo
git merge --no-commit --no-ff feature-x
# → Prepara merge pero NO commitea
# → Permite revisar antes de finalizar
# → Útil para verificar resultado
# Luego:
git commit  # Para finalizar
# o
git merge --abort  # Para cancelar

# 13. Ver qué se va a mergear
git log HEAD..feature-x
# → Commits que entrarán en merge
git diff HEAD...feature-x
# → Cambios desde punto de divergencia

# 14. Merge con log de commits incluidos
git merge --log feature-x
# → Incluye resumen de commits en mensaje
git merge --no-log feature-x
# → No incluye resumen

# 15. Merge con firma GPG
git merge -S feature-x
# → Firma merge commit con GPG
# → Para verificación de autoría

# 16. Merge verboso
git merge -v feature-x
# → Muestra información detallada del proceso

git merge -q feature-x
# → Modo silencioso (solo errores)

# ============================================
# MANEJO DE MERGE EN PROGRESO
# ============================================

# 17. Abortar merge
git merge --abort
# → Cancela merge en curso
# → Restaura estado pre-merge
# → Solo funciona si merge no está completo

# 18. Continuar merge tras resolver conflictos
git add archivo-resuelto.txt
git commit
# → Git detecta merge en progreso
# → Usa mensaje de merge automático

# 19. Estado de merge en progreso
git status
# → Muestra archivos en conflicto
# → Indica que merge está pendiente

ls -la .git/
# → .git/MERGE_HEAD existe durante merge
# → Contiene SHA del commit siendo mergeado
```

**Resolución de conflictos - Guía completa:** [🔙](#8-git-merge---integrando-cambios)

```bash
# ============================================
# IDENTIFICAR CONFLICTOS
# ============================================

# Ver archivos en conflicto
git status
# Muestra:
# - Unmerged paths (archivos con conflictos)
# - Changes to be committed (archivos auto-mergeados)

# Listar solo archivos con conflictos
git diff --name-only --diff-filter=U

# Ver conflictos con contexto
git diff

# Ver estadísticas de conflictos
git diff --stat

# ============================================
# ANATOMÍA DE UN CONFLICTO
# ============================================

# Git marca conflictos en el archivo:
<<<<<<< HEAD (rama actual)
código de la rama actual (main)
este código estaba aquí antes
=======
código de la rama entrante (feature-x)
este código viene del merge
>>>>>>> feature-x (rama que se está mergeando)

# Significado de marcadores:
# <<<<<<< HEAD        → Inicio de tu versión
# =======             → Separador
# >>>>>>> feature-x   → Fin de su versión

# ============================================
# ESTRATEGIAS DE RESOLUCIÓN
# ============================================

# 1. Resolución manual (más común)
# - Abre archivo en editor
# - Elimina marcadores <<<, ===, >>>
# - Edita código para combinar o elegir
# - Guarda archivo
git add archivo.txt
git commit

# 2. Aceptar versión completa (sin editar)
git checkout --ours archivo.txt    # Usar nuestra versión
git add archivo.txt

git checkout --theirs archivo.txt  # Usar su versión
git add archivo.txt

# 3. Ver diferencias durante conflicto
git diff --ours      # Diferencias con nuestra versión
git diff --theirs    # Diferencias con su versión
git diff --base      # Diferencias con ancestro común

# 4. Herramienta visual de merge
git mergetool
# → Abre herramienta configurada (vimdiff, meld, kdiff3, etc.)
# → Muestra 3 paneles: base, ours, theirs
# → Facilita resolución visual

# Configurar herramienta:
git config --global merge.tool meld
git config --global mergetool.prompt false

# 5. Ver contenido de versiones específicas
git show :1:archivo.txt  # Versión ancestro común (base)
git show :2:archivo.txt  # Versión nuestra (ours/HEAD)
git show :3:archivo.txt  # Versión suya (theirs)

# Guardar para comparar:
git show :2:archivo.txt > archivo-ours.txt
git show :3:archivo.txt > archivo-theirs.txt
# Comparar con herramienta externa

# ============================================
# CASOS ESPECIALES
# ============================================

# Conflictos en archivos binarios
git checkout --ours archivo.bin
# o
git checkout --theirs archivo.bin
# (No se pueden resolver manualmente línea a línea)

# Conflictos por archivo eliminado en una rama
# Git pregunta si mantener o eliminar:
git rm archivo.txt      # Confirma eliminación
# o
git add archivo.txt     # Mantiene archivo

# Conflictos por archivo renombrado
# Git puede detectar rename automáticamente
# Si no, resolver manualmente y hacer add

# ============================================
# ABORTAR Y REINTENTAR
# ============================================

# Abortar merge completo
git merge --abort
# → Vuelve a estado pre-merge
# → Útil si te equivocaste en resolución

# Reiniciar resolución de un archivo
git checkout -m archivo.txt
# → Restaura marcadores de conflicto
# → Permite resolver de nuevo

# Ver merge que causó conflicto
cat .git/MERGE_HEAD
# → SHA del commit siendo mergeado

git log -1 MERGE_HEAD
# → Detalles del commit en conflicto

# ============================================
# POST-RESOLUCIÓN
# ============================================

# Verificar que no quedan conflictos
git diff --check
# → Detecta marcadores de conflicto olvidados

# Verificar que todo compila/funciona
npm test  # o tu sistema de tests
git commit

# Limpiar archivos .orig (backup de mergetool)
git clean -f
# o configurar para no crearlos:
git config --global mergetool.keepBackup false
```

**Casos de uso del mundo real:**

```bash
# ============================================
# CASO 1: Feature simple lista para producción
# ============================================
git checkout main
git pull origin main
git merge --no-ff feature-login
git push origin main
# → Usa --no-ff para mantener visible la feature en historia

# ============================================
# CASO 2: Sincronizar feature con main
# ============================================
# Estás en feature-x, main avanzó, quieres últimos cambios
git checkout feature-x
git merge main
# → Trae cambios de main a tu feature
# → Resuelve conflictos ahora (no luego en main)
# → Testea todo funciona junto

# ============================================
# CASO 3: Multiple commits WIP, quieres 1 solo
# ============================================
git checkout main
git merge --squash feature-x
# Archivo .git/SQUASH_MSG tiene todos los mensajes
git commit -m "Add user authentication system

- Login form
- Password validation
- Session management
- Remember me functionality"
# → Main tiene 1 commit limpio
# → Historia de desarrollo (commits WIP) se pierde

# ============================================
# CASO 4: Hotfix urgente en producción
# ============================================
git checkout main
git checkout -b hotfix-security
# ... fixes ...
git commit -m "Fix: Security vulnerability CVE-2024-1234"
git checkout main
git merge --ff-only hotfix-security
# → --ff-only asegura merge limpio
# → Si falla, main se movió y hay que investigar
git push origin main
git branch -d hotfix-security

# ============================================
# CASO 5: Merge de múltiples features independientes
# ============================================
git checkout develop
git merge feature-a feature-b feature-c
# → Octopus merge
# → Solo si no hay conflictos
# → Historia muestra merge simultáneo

# ============================================
# CASO 6: Merge con revisión antes de commitear
# ============================================
git merge --no-commit --no-ff feature-x
# → Prepara merge sin commitear
git diff --staged
# → Revisa todos los cambios
npm test
# → Verifica que funciona
git commit
# o si algo falla:
git merge --abort

# ============================================
# CASO 7: Rama obsoleta, solo quieres marcarla como mergeada
# ============================================
git merge -s ours old-experiment
# → No aplica ningún cambio de old-experiment
# → Pero Git la marca como mergeada
# → Útil para limpiar ramas sin afectar código

# ============================================
# CASO 8: Merge de release branch
# ============================================
# Merge a main (producción)
git checkout main
git merge --no-ff --log release-1.5.0
# --log incluye lista de commits en mensaje

# Merge de vuelta a develop
git checkout develop
git merge --no-ff release-1.5.0

# ============================================
# CASO 9: Resolver conflicto prefiriendo una versión
# ============================================
git merge feature-x
# ... conflicto ...
git checkout --ours .      # Todas las versiones nuestras
# o
git checkout --theirs .    # Todas las versiones de ellos
git add .
git commit

# Más selectivo (solo ciertos archivos):
git checkout --ours src/
git checkout --theirs config/
git add .
git commit

# ============================================
# CASO 10: Merge con conflictos, quieres ver qué cambió
# ============================================
git merge feature-x
# ... conflictos ...

# Ver historial de cambios en archivo conflictivo
git log --oneline --all -- archivo-conflicto.txt

# Ver qué cambió en cada rama
git log main..feature-x -- archivo-conflicto.txt
git show feature-x:archivo-conflicto.txt
git show main:archivo-conflicto.txt

# Resolver informadamente
# ... edita ...
git add archivo-conflicto.txt
git commit
```

**Troubleshooting y problemas comunes:**

```bash
# ============================================
# PROBLEMA 1: "Already up to date"
# ============================================
git merge feature-x
# Already up to date.

Causa: feature-x no tiene commits nuevos vs main
Solución:
- Verificar que estás en rama correcta
- Verificar que feature-x tiene commits:
  git log main..feature-x

# ============================================
# PROBLEMA 2: "fatal: refusing to merge unrelated histories"
# ============================================
Causa: Ramas sin ancestro común (repos separados)
Solución:
git merge --allow-unrelated-histories other-branch
# ⚠️ CUIDADO: Puede crear merge complejo

# ============================================
# PROBLEMA 3: Merge incompleto, .git/MERGE_HEAD existe
# ============================================
git status
# On branch main
# You have unmerged paths.

Causa: Merge con conflictos sin resolver
Solución:
1. Resolver conflictos:
   git status  # Ver qué falta
   # ... resolver ...
   git add .
   git commit
2. O abortar:
   git merge --abort

# ============================================
# PROBLEMA 4: Merge commit no deseado
# ============================================
# Ya hiciste merge y no querías merge commit
git reset --hard HEAD~1  # Deshace último commit
git merge --ff-only feature-x  # Intenta fast-forward

# ============================================
# PROBLEMA 5: Conflictos masivos, difícil resolver
# ============================================
Solución 1: Abortar y usar rebase
git merge --abort
git rebase main  # Resuelve conflicto por conflicto

Solución 2: Estrategia más agresiva
git merge -X theirs feature-x
# LUEGO revisa cambios críticos manualmente

Solución 3: Resolver en herramienta visual
git mergetool

# ============================================
# PROBLEMA 6: Merge eliminó archivo que debería existir
# ============================================
# Git puede auto-mergear eliminación incorrectamente
git show HEAD:archivo-perdido.txt > archivo-perdido.txt
git add archivo-perdido.txt
git commit --amend  # Corrige merge commit

# ============================================
# PROBLEMA 7: Merge rompió funcionalidad
# ============================================
# Opción 1: Revert del merge
git revert -m 1 HEAD
# -m 1 indica mantener lado 1 (main) del merge

# Opción 2: Reset (si no pusheaste)
git reset --hard HEAD~1

# ============================================
# PROBLEMA 8: No puedes hacer merge (archivos sucios)
# ============================================
error: Your local changes would be overwritten by merge.

Solución 1: Commitear cambios
git add .
git commit -m "WIP"
git merge feature-x

Solución 2: Stash
git stash
git merge feature-x
git stash pop

Solución 3: Descartar cambios
git reset --hard  # ⚠️ PIERDE CAMBIOS
git merge feature-x
```

**Mejores prácticas y patrones:**

```bash
# ============================================
# ✅ BUENAS PRÁCTICAS
# ============================================

# 1. Siempre actualiza antes de merge
git checkout main
git pull origin main
git merge feature-x

# 2. Usa --no-ff para features importantes
git merge --no-ff feature-login
# → Historia clara, fácil revertir feature completa

# 3. Resuelve conflictos en feature branch, no en main
git checkout feature-x
git merge main
# ... resolver conflictos ...
git checkout main
git merge feature-x  # Ahora sin conflictos

# 4. Testea tras resolver conflictos
git merge feature-x
# ... resolver ...
npm test
git commit

# 5. Usa mensajes de merge descriptivos
git merge --no-ff --edit feature-auth
# Edita para incluir:
# - Qué hace la feature
# - Issues relacionados (#123)
# - Reviewers

# 6. Squash para limpiar historia
git merge --squash feature-experiment
# → 47 commits de prueba → 1 commit limpio

# 7. Verifica antes de push
git log --oneline --graph -10
git diff origin/main
git push origin main

# 8. Usa merge commits para puntos importantes
git merge --no-ff release-2.0
# → Marca claramente releases en historia

# ============================================
# ✗ MALAS PRÁCTICAS
# ============================================

# 1. Mergear sin testear
git merge feature-x && git push  # ❌
# Puede romper main

# 2. Usar -X ours/theirs sin revisar
git merge -X theirs external-branch  # ❌
# Puede sobrescribir trabajo importante

# 3. Mergear directo a main sin revisión
# En proyectos serios, usa Pull Requests

# 4. Ignorar conflictos "pequeños"
# Todo conflicto requiere atención

# 5. No limpiar branches tras merge
git merge feature-x
git push
# Luego:
git branch -d feature-x  # ✅ Limpia local
git push origin --delete feature-x  # ✅ Limpia remoto

# 6. Merge de ramas públicas con rebase
# Causa problemas a colaboradores

# ============================================
# WORKFLOWS COMUNES
# ============================================

# GitHub Flow (simple)
1. Crea feature branch desde main
2. Desarrolla y commitea
3. Push y crea Pull Request
4. Revisión de código
5. Merge (con --no-ff) a main
6. Delete branch

# Git Flow (complejo)
- main: Producción
- develop: Integración
- feature/*: Nuevas features
- release/*: Preparar release
- hotfix/*: Fixes urgentes

# Feature → develop: --no-ff
# develop → main: --no-ff (con tag)
# hotfix → main y develop: --no-ff
```

**Comparación: merge vs rebase:**

```bash
# ============================================
# CUÁNDO USAR MERGE
# ============================================
✅ Integrar features completas a main
✅ Merges de release branches
✅ Colaboración en ramas públicas
✅ Preservar historia exacta de desarrollo
✅ Cuando múltiples devs trabajan en misma rama

Ventajas:
- No reescribe historia
- Seguro para ramas compartidas
- Preserva contexto (cuándo se mergeó)
- Fácil revertir (git revert -m 1)

Desventajas:
- Historia puede volverse compleja
- Grafo con muchas ramas
- "Merge commits" pueden saturar log

# ============================================
# CUÁNDO USAR REBASE
# ============================================
✅ Actualizar feature branch con main
✅ Limpiar commits locales antes de merge
✅ Mantener historia lineal
✅ Trabajo personal en rama local

Ventajas:
- Historia lineal y limpia
- Fácil de leer git log
- No crea merge commits extra

Desventajas:
- Reescribe historia (cambia SHAs)
- Peligroso en ramas públicas
- Puede causar problemas a colaboradores

# ============================================
# ESTRATEGIA HÍBRIDA (RECOMENDADA)
# ============================================

# 1. Durante desarrollo: rebase
git checkout feature-x
git rebase main  # Mantiene feature actualizada y limpia

# 2. Para integrar: merge
git checkout main
git merge --no-ff feature-x  # Integra feature completa

Resultado:
- Historia limpia en features (rebase)
- Historia clara en main (merge commits marcan features)
- Lo mejor de ambos mundos
```

**Configuración recomendada:**

```bash
# Configurar merge sin fast-forward por defecto
git config --global merge.ff false

# Siempre mostrar diffstat tras merge
git config --global merge.stat true

# Configurar herramienta de merge
git config --global merge.tool meld
git config --global mergetool.prompt false
git config --global mergetool.keepBackup false

# Estilo de conflictos (diff3 muestra ancestro común)
git config --global merge.conflictstyle diff3

# Ejemplo de conflicto con diff3:
<<<<<<< HEAD
código actual
||||||| merged common ancestors
código ancestro común
=======
código entrante
>>>>>>> feature-x

# Configurar para squash automático en certain branches
# (en .git/config o ~/.gitconfig)
[branch "develop"]
    mergeoptions = --no-ff

# Verificar configuración
git config --list | grep merge
```

---

## 9. git rebase - Reescribiendo Historia
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Reaplica commits de una rama encima de otra, reescribiendo historia.

**Funcionamiento interno:**
```
1. Identifica commits únicos
2. Guarda como patches temporales
3. Resetea rama a base
4. Aplica patches uno por uno (nuevos commits)
```

**Uso práctico:**

```bash
# Rebase básico
git checkout feature-x
git rebase main

# Rebase interactivo (SUPER PODEROSO)
git rebase -i main
# Opciones:
# pick   - Usar commit
# reword - Cambiar mensaje
# edit   - Pausar para modificar
# squash - Combinar con anterior (mantiene mensaje)
# fixup  - Combinar con anterior (descarta mensaje)
# drop   - Eliminar commit

# Squash últimos N commits
git rebase -i HEAD~3

# Continuar tras conflicto
git add archivo-resuelto
git rebase --continue

# Saltar commit
git rebase --skip

# Abortar
git rebase --abort
```

**Rebase vs Merge:**

```bash
MERGE:
✓ Historia completa
✓ No reescribe commits
✓ Seguro para ramas públicas
✗ Grafo complejo

REBASE:
✓ Historia lineal
✓ Fácil de entender
✗ Reescribe commits
✗ Peligroso para ramas públicas

# ¿Cuándo usar cada uno?
REBASE: Rama local/feature antes de merge
MERGE: Integrar features a main
```

**⚠️ Regla de oro:**

```bash
NUNCA rebasees commits ya pusheados a repositorio público

Correcto:
git rebase main          # OK, commits solo locales
git push origin feature-x

Incorrecto:
git push origin feature-x
git rebase main          # ¡ROMPE REPO DE OTROS!
git push --force
```

**Mejores prácticas:**

```bash
✓ Usa rebase para limpiar historia local
✓ Rebase feature sobre main antes de merge
✓ Usa --force-with-lease en vez de --force
✓ Nunca rebasees ramas públicas compartidas

✗ No rebasees main o develop
✗ No rebasees commits públicos
✗ No uses --force sin --force-with-lease
```

---

## 10. git clone - Copiando Repositorios

**¿Qué hace?**
Crea una copia local completa de un repositorio remoto.

**Funcionamiento interno:**
```
1. Crea directorio
2. git init
3. git remote add origin <url>
4. git fetch origin
5. git checkout <default-branch>
```

**Uso práctico:**

```bash
# Clone básico
git clone https://github.com/user/repo.git

# Clone con nombre personalizado
git clone https://github.com/user/repo.git mi-proyecto

# Clone shallow (solo último commit, rápido)
git clone --depth 1 https://github.com/user/repo.git

# Clone de rama específica
git clone -b develop https://github.com/user/repo.git

# Clone con submódulos
git clone --recursive https://github.com/user/repo.git

# Clone parcial (sin blobs)
git clone --filter=blob:none https://github.com/user/repo.git
```

**Protocolos:**

```bash
# HTTPS (recomendado, universal)
git clone https://github.com/user/repo.git

# SSH (más rápido, requiere key)
git clone git@github.com:user/repo.git

# Local
git clone /ruta/al/repo.git
```

**Mejores prácticas:**

```bash
✓ Usa HTTPS para proyectos públicos
✓ Usa SSH para proyectos privados
✓ Usa --depth 1 en CI/CD
✓ Usa --recursive para repos con submódulos

✗ No clones con --depth si necesitas historia
✗ No desactives SSL verification sin razón
```

---

## 11. git remote - Gestionando Repositorios Remotos
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Gestiona referencias a repositorios remotos.

**Funcionamiento interno:**
```
Remotos se guardan en .git/config:
[remote "origin"]
    url = https://github.com/user/repo.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

**Uso práctico:**

```bash
# Listar remotos
git remote
git remote -v

# Añadir remoto
git remote add upstream https://github.com/original/repo.git

# Ver detalles
git remote show origin

# Cambiar URL
git remote set-url origin https://nuevo-url.git
git remote set-url origin git@github.com:user/repo.git

# Renombrar
git remote rename origin nuevo-nombre

# Eliminar
git remote remove upstream

# Limpiar refs obsoletas
git remote prune origin
git remote prune origin --dry-run
```

**Fork workflow:**

```bash
git clone https://github.com/tu-fork/proyecto.git
cd proyecto
git remote add upstream https://github.com/original/proyecto.git
git remote -v
# origin    tu-fork (fetch/push)
# upstream  original (fetch/push)

# Workflow:
git fetch upstream
git merge upstream/main
git push origin main
```

**Mejores prácticas:**

```bash
✓ Usa nombres descriptivos (origin, upstream, backup)
✓ Usa SSH para repos privados
✓ Configura upstream para forks
✓ Limpia con prune regularmente

✗ No pongas credenciales en URL
✗ No uses nombres confusos
✗ No borres origin sin reemplazarlo
```

---

## 12. git fetch - Descargando Cambios
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Descarga objetos y refs del remoto SIN modificar working directory.

**Funcionamiento interno:**
```
1. Conecta con remoto
2. Compara refs locales vs remotas
3. Descarga objetos faltantes
4. Actualiza refs/remotes/origin/*
5. NO modifica ramas locales
6. NO modifica working directory
```

**Uso práctico:**

```bash
# Fetch básico
git fetch
git fetch origin

# Fetch de rama específica
git fetch origin main

# Fetch de todos los remotos
git fetch --all

# Fetch con prune (limpia refs obsoletas)
git fetch --prune

# Fetch de PR (GitHub)
git fetch origin pull/123/head:pr-123

# Ver resultado
git log HEAD..origin/main --oneline
git diff origin/main
```

**Fetch vs Pull:**

```bash
# FETCH: Solo descarga
git fetch origin main
→ origin/main actualizado
→ main local SIN cambios
→ Puedes revisar antes de integrar

# PULL: Fetch + Merge
git pull origin main
→ Descarga Y mergea automáticamente
→ Más rápido pero menos control
```

**Mejores prácticas:**

```bash
✓ Usa fetch antes de pull (revisa cambios)
✓ Usa --prune regularmente
✓ Fetch frecuentemente
✓ Revisa con git log tras fetch

✗ No confundas fetch con pull
✗ No asumas que fetch cambia working
✗ No olvides mergear después de fetch
```

---

## 13. git pull - Descargando e Integrando Cambios Remotos
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
`git pull` es el comando fundamental para sincronizar tu trabajo local con el repositorio remoto. Descarga los cambios que otros desarrolladores han subido e integra esos cambios en tu rama actual. Es esencial para mantener tu trabajo actualizado en entornos colaborativos.

**Funcionamiento interno:**

```
git pull es en realidad DOS comandos ejecutados secuencialmente:

git pull = git fetch + git merge (por defecto)
         o git fetch + git rebase (con --rebase)

Paso a paso:

1. FETCH (descarga):
   - Conecta al repositorio remoto
   - Descarga objetos nuevos (commits, trees, blobs)
   - Actualiza refs remotas (refs/remotes/origin/*)
   - NO toca tu working directory ni rama actual

2. MERGE o REBASE (integración):
   - Integra los cambios descargados en tu rama
   - Dos estrategias disponibles:
     a) MERGE: Crea merge commit si hay divergencia
     b) REBASE: Reaplica tus commits encima de remotos

Ejemplo visual:

Estado inicial (local):
  A---B---C---D (main, local)

Estado remoto (origin/main):
  A---B---E---F (origin/main)

Después de git fetch:
  Local:  A---B---C---D (main)
  Remoto: A---B---E---F (origin/main)
  → origin/main actualizado, pero main local NO cambia

Después de git pull (merge):
  A---B---E---F
       \       \
        C---D---M (main)
  → Merge commit M combina ambas historias

Después de git pull --rebase:
  A---B---E---F---C'---D' (main)
  → Commits C y D replicados después de F
  → Historia lineal, sin merge commit
```

**Uso práctico - Comandos básicos:**

```bash
# 1. Pull básico (fetch + merge)
git pull
# → Descarga de origin/<rama-actual>
# → Merge con rama actual
# → Puede crear merge commit

# 2. Pull de rama específica
git pull origin main
# → Descarga de origin/main
# → Integra en rama actual
# → Útil si no hay tracking configurado

# 3. Pull con rebase (historia lineal)
git pull --rebase
# → Descarga cambios remotos
# → Reaplica tus commits encima
# → NO crea merge commit
# → Historia más limpia

# 4. Pull solo si es fast-forward
git pull --ff-only
# → Solo actualiza si NO hay divergencia
# → Falla si necesitarías merge o rebase
# → Más seguro, evita merges inesperados
```

**Estrategias de integración detalladas:**

```bash
# OPCIÓN A: MERGE (--no-rebase, default en muchos casos)
git pull --no-rebase
# o: git config pull.rebase false

Ventajas:
✓ Preserva historia exacta (cuándo se integraron cambios)
✓ No reescribe commits (hashes estables)
✓ Más seguro para ramas públicas/compartidas
✓ Conflictos se resuelven una sola vez

Desventajas:
✗ Crea merge commits (historia no lineal)
✗ Log más difícil de leer con muchos merges
✗ "Ruido" visual en git log --graph

Cuándo usar:
→ Ramas compartidas (main, develop)
→ Cuando quieres preservar contexto de integración
→ Equipos que prefieren historia completa

# OPCIÓN B: REBASE (--rebase)
git pull --rebase
# o: git config pull.rebase true

Ventajas:
✓ Historia lineal y limpia
✓ Log más fácil de leer
✓ No crea merge commits innecesarios
✓ Bisect más efectivo

Desventajas:
✗ Reescribe commits locales (cambia hashes)
✗ Conflictos pueden aparecer múltiples veces
✗ Peligroso si ya pusheaste (necesitas force push)
✗ Pierde contexto de cuándo se integraron cambios

Cuándo usar:
→ Feature branches personales
→ Antes de crear Pull Request
→ Cuando prefieres historia lineal
→ Trabajo local no compartido aún

# OPCIÓN C: FAST-FORWARD ONLY (--ff-only)
git pull --ff-only
# o: git config pull.ff only

Comportamiento:
→ Solo actualiza si tu rama NO ha avanzado
→ Falla si hay divergencia (commits locales)
→ Más conservador, evita sorpresas

Ventajas:
✓ Nunca crea merge commits inesperados
✓ Nunca reescribe historia
✓ Fuerza a decidir explícitamente (merge o rebase)
✓ Más seguro para principiantes

Cuándo usar:
→ Cuando quieres control total
→ Para evitar merges automáticos
→ En scripts automatizados
```

**Manejo de conflictos durante pull:**

```bash
# Escenario: git pull genera conflictos

# Si usaste git pull (merge):
git pull
# → Auto-merging archivo.txt
# → CONFLICT (content): Merge conflict in archivo.txt

Resolver:
1. Abre archivos con conflicto
2. Busca marcadores:
   <<<<<<< HEAD
   Tu código local
   =======
   Código del remoto
   >>>>>>> origin/main

3. Edita y deja versión correcta
4. Marca como resuelto:
   git add archivo.txt
5. Completa el merge:
   git commit  # (mensaje ya preparado)

# O aborta el merge:
git merge --abort
# → Vuelve al estado antes del pull

# Si usaste git pull --rebase:
git pull --rebase
# → Applying: Tu commit local
# → CONFLICT: archivo.txt

Resolver:
1. Resuelve conflicto (igual que arriba)
2. Marca como resuelto:
   git add archivo.txt
3. Continúa el rebase:
   git rebase --continue

# O aborta el rebase:
git rebase --abort
# → Vuelve al estado antes del pull

# O salta el commit conflictivo:
git rebase --skip
# → Omite tu commit (úsalo con cuidado)
```

**Opciones avanzadas:**

```bash
# 1. Pull con autostash (stash automático)
git pull --autostash
# → Guarda cambios no commiteados automáticamente
# → Hace pull
# → Restaura cambios guardados
# → Útil cuando tienes trabajo en progreso

# 2. Pull con estrategia de merge
git pull -X ours
# → En conflictos, prefiere versión LOCAL
# → Útil en merges complicados

git pull -X theirs
# → En conflictos, prefiere versión REMOTA
# → Usa con cuidado

# 3. Pull sin commit (solo merge)
git pull --no-commit
# → Hace merge pero NO commitea
# → Te da oportunidad de revisar
# → Útil para inspeccionar antes de finalizar

# 4. Pull verbose
git pull --verbose
# → Muestra información detallada
# → Útil para debugging

# 5. Pull desde múltiples remotos
git pull upstream main
# → Pull desde otro remoto (no origin)
# → Útil en forks

# 6. Pull con profundidad limitada
git pull --depth=10
# → Solo últimos 10 commits
# → Útil en repos gigantes
```

**Verificación antes y después de pull:**

```bash
# ANTES de pull - ver qué traerás:

# 1. Ver commits que te faltan
git fetch
git log HEAD..origin/main --oneline
# → Commits que traerá el pull

# 2. Ver cambios en archivos
git fetch
git diff HEAD...origin/main --name-status
# → Archivos que cambiaron en remoto

# 3. Ver si hay divergencia
git fetch
git status
# → Dice "have diverged" si hay commits locales y remotos

# DESPUÉS de pull - verificar qué cambió:

# 1. Ver commits traídos (usando reflog)
git log HEAD@{1}..HEAD --oneline
# → HEAD@{1} = posición antes del pull
# → HEAD = posición actual
# → Muestra SOLO commits nuevos traídos

# Alternativa con ORIG_HEAD:
git log ORIG_HEAD..HEAD --oneline
# → ORIG_HEAD también apunta al estado pre-pull

# 2. Ver cambios en archivos traídos
git diff --name-status HEAD@{1} HEAD
# → Lista archivos modificados, añadidos, eliminados
# → M = modified, A = added, D = deleted

git diff --stat HEAD@{1} HEAD
# → Resumen con estadísticas por archivo

# 3. Ver diff completo de los cambios
git diff HEAD@{1} HEAD
# → Muestra todas las diferencias línea por línea
# → Útil para revisar qué código cambió exactamente

# 4. Ver detalle de cada commit traído
git show <hash-commit>
# → Muestra mensaje, autor, fecha, y diff del commit
# → Repite para cada commit del log anterior

# Ejemplo completo de revisión:
git show HEAD~2  # Ver penúltimo commit
git show HEAD~1  # Ver último commit
git show HEAD    # Ver commit actual

# 5. Ver qué archivos específicos cambiaron
git diff --name-only HEAD@{1} HEAD
# → Solo nombres de archivos, sin estadísticas

git diff HEAD@{1} HEAD -- archivo.txt
# → Diff de archivo específico

# 6. Ver si quedaron conflictos sin resolver
git status
# → Debe estar limpio
# → Si dice "Unmerged paths", hay conflictos pendientes

# 7. Ver diferencias con remoto (debe estar sincronizado)
git diff origin/main
# → Debería estar vacío si pull fue exitoso
# → Si hay diferencias, tienes commits locales sin pushear

# 8. Ver cuántos commits se trajeron
git rev-list --count HEAD@{1}..HEAD
# → Número de commits traídos

# 9. Ver resumen visual con grafo
git log HEAD@{1}..HEAD --oneline --graph --stat
# → Combinación visual con archivos y estadísticas
```

**Ejemplo práctico completo - Después de pull:** [🔙](#5-git-log---explorando-la-historia)

```bash
# Acabas de hacer: git pull
# Quieres saber QUÉ cambió

# Paso 1: Ver cuántos commits se trajeron
$ git log HEAD@{1}..HEAD --oneline
a1b2c3d (HEAD -> main, origin/main) Fix: Corregir bug en login
d4e5f6g Feature: Añadir validación de email
h7i8j9k Docs: Actualizar README

# → Se trajeron 3 commits

# Paso 2: Ver qué archivos cambiaron
$ git diff --name-status HEAD@{1} HEAD
M       src/auth/login.js
A       src/validators/email.js
M       README.md
D       src/old-validator.js

# → 2 modificados, 1 añadido, 1 eliminado

# Paso 3: Ver estadísticas
$ git diff --stat HEAD@{1} HEAD
 README.md                | 15 ++++++++++++++-
 src/auth/login.js        | 8 +++-----
 src/old-validator.js     | 45 -------------------------------------------
 src/validators/email.js  | 30 ++++++++++++++++++++++++++++
 4 files changed, 47 insertions(+), 51 deletions(-)

# Paso 4: Ver detalle de commit específico
$ git show a1b2c3d
commit a1b2c3d...
Author: John Doe <john@example.com>
Date:   Mon Feb 10 10:30:00 2026

    Fix: Corregir bug en login

    - Validación de contraseña mejorada
    - Manejo de errores actualizado

diff --git a/src/auth/login.js b/src/auth/login.js
...
(muestra el diff completo)

# Paso 5: Ver diff de archivo específico
$ git diff HEAD@{1} HEAD -- src/auth/login.js
(muestra solo cambios en ese archivo)

# Paso 6: Verificar sincronización con remoto
$ git diff origin/main
# (vacío = perfectamente sincronizado)
# Si hay diferencias, tienes commits locales sin pushear

# Paso 7: Ver cuántos commits se trajeron
$ git rev-list --count HEAD@{1}..HEAD
# → Número de commits traídos

# Paso 8: Ver resumen visual con grafo
git log HEAD@{1}..HEAD --oneline --graph --stat
# → Combinación visual con archivos y estadísticas
```

**Comandos rápidos de verificación post-pull:** [🔙](#5-git-log---explorando-la-historia)

```bash
# Ver últimos 5 commits (incluyendo los traídos)
git log -5 --oneline

# Ver archivos modificados en últimos 3 commits
git log -3 --name-status --oneline

# Ver todo lo traído con contexto visual
git log HEAD@{1}..HEAD --oneline --graph --decorate --stat

# Comparar tu código actual vs hace 2 pulls
git diff HEAD@{2} HEAD
```

**⚠️ Notas importantes sobre HEAD@{n}:** [🔙](#5-git-log---explorando-la-historia)

```bash
# HEAD@{n} es del REFLOG (historial de operaciones)
# Solo se mantiene por tiempo limitado (default 90 días)

# Ver historial completo de HEAD:
git reflog
# Muestra todas las operaciones que movieron HEAD

# Si hiciste múltiples operaciones después del pull:
HEAD@{0}  → Estado actual
HEAD@{1}  → Operación anterior (puede NO ser el pull)
HEAD@{2}  → Dos operaciones atrás
HEAD@{n}  → n-ésima operación atrás

# Para asegurar que comparas con el pull correcto:
git reflog
# Busca la línea del pull
# Usa ese número específico

# Alternativa más segura si no estás seguro:
# Anota el hash ANTES de hacer pull:
git rev-parse HEAD  # Copia este hash
git pull
git log <hash-copiado>..HEAD --oneline
# → Garantiza comparación correcta
```

**Configuración recomendada:**

```bash
# Configurar estrategia por defecto (rebase)
git config --global pull.rebase true
# → Todos los pulls usarán rebase

# O configurar para fast-forward only
git config --global pull.ff only
# → Fuerza a especificar --rebase o --no-rebase

# Habilitar autostash con rebase
git config --global rebase.autoStash true
# → Stash automático en rebases

# Ver configuración actual
git config --get pull.rebase
git config --get pull.ff

# Configurar por repositorio (sin --global)
cd /ruta/proyecto
git config pull.rebase true
# → Solo afecta ese repositorio
```

**Situaciones comunes y soluciones:**

```bash
# PROBLEMA 1: "divergent branches" al hacer pull
git pull
# → hint: You have divergent branches...

Solución 1: Configurar estrategia
git config pull.rebase false  # merge
git pull

Solución 2: Especificar en comando
git pull --rebase

Solución 3: Fast-forward solo
git pull --ff-only

# PROBLEMA 2: Tracking branch no configurado
git pull
# → fatal: No remote repository specified

Solución:
git pull origin main  # Especifica remoto y rama
# O configura tracking:
git branch --set-upstream-to=origin/main main

# PROBLEMA 3: Cambios locales sin commitear
git pull
# → error: Your local changes would be overwritten

Solución 1: Commitea
git add .
git commit -m "WIP"
git pull

Solución 2: Stash
git stash
git pull
git stash pop

Solución 3: Autostash
git pull --autostash

# PROBLEMA 4: Necesitas forzar (después de rebase local)
git push
# → rejected (non-fast-forward)

Solución:
git push --force-with-lease
# → Solo fuerza si nadie más actualizó
```

**Workflows recomendados:**

```bash
# WORKFLOW 1: Feature branch (rebase)
# Situación: Trabajas en feature, main avanzó

# En feature branch:
git checkout feature-x
git pull origin main --rebase
# → Reaplica tus commits de feature-x encima de main actualizado
# → Historia lineal
# → Preparado para PR limpio

# WORKFLOW 2: Main branch (merge)
# Situación: Actualizas main local

git checkout main
git pull
# → Simple merge si es necesario
# → Preserva historia

# WORKFLOW 3: Sync fork con upstream
# Situación: Tu fork desactualizado

git fetch upstream
git checkout main
git pull upstream main
git push origin main
# → Actualiza tu fork desde original

# WORKFLOW 4: Colaboración continua
# Situación: Varios devs en misma rama

git pull --rebase --autostash
# → Stash auto, rebase, restaura
# → Mantiene historia limpia
# → Conveniente para trabajo continuo
```

**Comparación visual: merge vs rebase en pull:**

```bash
# ESCENARIO INICIAL:
Local:  A---B---C---D (main)
Remoto: A---B---E---F (origin/main)

# PULL CON MERGE (git pull --no-rebase):
A---B---E---F
     \       \
      C---D---M (main)

Características:
- Merge commit M con 2 líneas de commits
- Historia completa preservada
- Graph no lineal
- Hashes de C y D sin cambiar

# PULL CON REBASE (git pull --rebase):
A---B---E---F---C'---D' (main)

Características:
- Sin merge commit
- Historia lineal
- C y D reescritos (C' y D' con nuevos hashes)
- Más limpio visualmente

# PULL CON FF-ONLY (cuando no hay commits locales):
Local antes:  A---B (main)
Remoto:       A---B---E---F (origin/main)

Después:      A---B---E---F (main)

- Sin merge commit
- Sin rebase
- Solo movió puntero
- Ideal cuando solo necesitas actualizar
```

**Mejores prácticas:**

```bash
✓ Pull frecuentemente (al menos diariamente)
✓ Commitea o stash antes de pull
✓ Usa --rebase en feature branches personales
✓ Usa merge en ramas compartidas (main, develop)
✓ Configura pull.rebase según tu workflow
✓ Revisa cambios con git fetch primero
✓ Resuelve conflictos inmediatamente
✓ Usa --autostash para conveniencia
✓ Verifica con git status después de pull
✓ Comunica force pushes al equipo

✗ NO hagas pull sin revisar en ramas importantes
✗ NO ignores conflictos y continúes trabajando
✗ NO uses pull --rebase en commits ya pusheados
✗ NO hagas pull con cambios críticos sin commitear
✗ NO mezcles estrategias (elige merge O rebase)
✗ NO uses -X ours/-X theirs sin entender
✗ Evita pull en detached HEAD
✗ NO hagas pull sin tracking branch claro
```

**Debugging y troubleshooting:**

```bash
# Ver qué hará pull sin ejecutarlo
git fetch
git log HEAD..@{u} --oneline  # @{u} = upstream branch
git diff HEAD...@{u} --stat

# Ver configuración de tracking
git branch -vv
# → Muestra upstream de cada rama

# Ver qué remote y branch usa pull
git remote -v
git rev-parse --abbrev-ref --symbolic-full-name @{u}

# Simular pull con dry-run (no existe, pero puedes):
git fetch --dry-run  # Solo para fetch
# Luego inspecciona con git log

# Ver reflog después de pull problemático
git reflog
# → Encuentra estado anterior
git reset --hard HEAD@{1}  # Vuelve atrás

# Ver qué estrategia está configurada
git config --get-all pull.rebase
git config --get-all pull.ff
```

---

## 14. git push - Subiendo Cambios
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Envía commits locales al repositorio remoto.

**Funcionamiento interno:**
```
1. Conecta con remoto
2. Compara refs
3. Verifica que sea fast-forward
4. Empaqueta objetos faltantes
5. Envía objetos
6. Actualiza refs remotas
```

**Uso práctico:**

```bash
# Push básico
git push
git push origin main

# Push con tracking
git push -u origin feature-x

# Push forzado (¡CUIDADO!)
git push --force  # PELIGROSO
git push --force-with-lease  # PREFERIBLE

# Push de tags
git push origin v1.0.0
git push --tags

# Eliminar rama remota
git push origin --delete feature-x

# Push dry-run
git push --dry-run
```

**⚠️ Force push:**

```bash
NUNCA fuerces push en ramas compartidas (main, develop)

Cuándo SÍ:
✓ Feature branch personal
✓ Después de rebase local
✓ Corregir commits antes de merge

Cuándo NO:
✗ main/develop/master
✗ Ramas de otros
✗ Ramas públicas

SIEMPRE usa --force-with-lease (no --force):
git push --force-with-lease
→ Solo fuerza si nadie más actualizó
```

**Mejores prácticas:**

```bash
✓ Commitea cambios atómicos, push frecuentemente
✓ Usa --force-with-lease en vez de --force
✓ Verifica con --dry-run antes de push importante
✓ Pull antes de push (evita rechazos)

✗ NO uses --force en ramas compartidas
✗ NO pushees credenciales, secrets, keys
✗ NO pushees archivos gigantes
✗ NO ignores errores de push
```

---

## 15. git reset - Moviendo Referencias

**¿Qué hace?**
Mueve HEAD y rama actual, opcionalmente modificando staging y working.

**Funcionamiento interno:**
```
Tres modos:
--soft:  Solo mueve HEAD/rama
--mixed: Mueve HEAD/rama + resetea staging
--hard:  Mueve HEAD/rama + resetea staging + working
```

**Uso práctico:**

```bash
# 1. Reset suave (mantiene cambios en staging)
git reset --soft HEAD~1
# → Deshace commit
# → Cambios vuelven a staging
# → Útil para rehacer commit

# 2. Reset mixto (default, cambios en working)
git reset HEAD~1
# o: git reset --mixed HEAD~1
# → Deshace commit
# → Cambios vuelven a working directory
# → Útil para reorganizar qué commitear

# 3. Reset duro (¡PIERDES CAMBIOS!)
git reset --hard HEAD~1
# → Deshace commit
# → BORRA todos los cambios
# → ⚠️ PELIGROSO: no recuperable sin reflog

# 4. Unstage archivo (quitar del staging)
git reset HEAD archivo.txt
# → Mueve archivo de staging a working
# → NO modifica el último commit

# 5. Reset a commit específico
git reset --soft abc123
git reset --mixed abc123
git reset --hard abc123

# 6. Reset a remoto
git reset --hard origin/main
# → Sincroniza con remoto, descartando cambios locales

# 7. Reset de un directorio específico
git reset HEAD directorio/
```

**FLUJO DE ESTADOS con reset:**

```bash
# ESTADOS EN GIT:
# Working Directory → Staging (Index) → Commit → Remote
#
# COMANDOS PARA AVANZAR:
# Working → Staging:   git add <archivo>
# Staging → Commit:    git commit
# Commit → Remote:     git push
#
# COMANDOS PARA RETROCEDER (reset):
# Staging → Working:   git reset HEAD <archivo>
# Commit → Staging:    git reset --soft HEAD~1
# Commit → Working:    git reset --mixed HEAD~1 (default)
# Commit → (borrado):  git reset --hard HEAD~1 (PELIGRO)
```

**Casos de uso prácticos:**

```bash
# Caso 1: Quitar un archivo del último commit
git reset --soft HEAD~1     # Deshace commit → archivos a staging
git reset HEAD archivo.txt  # Quita archivo del staging
git commit -m "Mensaje"     # Recommitea sin ese archivo

# Caso 2: Rehacer último commit con más cambios
git reset --soft HEAD~1     # Deshace commit → archivos a staging
git add mas-cambios.txt     # Añade más archivos
git commit -m "Mensaje completo"

# Caso 3: Deshacer commit y revisar cambios
git reset HEAD~1            # Cambios a working directory
git diff                    # Revisa qué cambiaste
git add -p                  # Añade selectivamente
git commit -m "Mejor mensaje"

# Caso 4: Unstage archivo antes de commit
git add .                   # Añadiste todo
git reset HEAD config.txt   # Quitas un archivo del staging
git commit -m "Mensaje"     # Commiteas sin config.txt

# Caso 5: Limpiar todo y empezar de nuevo
git reset --hard HEAD       # Descarta TODOS los cambios
git clean -fd               # Elimina archivos untracked

# Caso 6: Deshacer múltiples commits
git reset --soft HEAD~3     # Deshace 3 commits → staging
git commit -m "Squashed commit"  # Un solo commit
```

**Comparación de modos:**

```bash
git reset --soft HEAD~1
→ Commit deshecho
→ Cambios en staging ✓
→ Working intacto ✓

git reset HEAD~1  (mixed, default)
→ Commit deshecho
→ Cambios en working ✓
→ Staging limpio

git reset --hard HEAD~1
→ Commit deshecho
→ Staging limpio
→ Working limpio
→ ¡CAMBIOS PERDIDOS!
```

**Reset vs Revert:**

```bash
RESET (reescribe historia):
→ Mueve rama atrás
→ Commits "desaparecen"
→ Solo para commits locales

REVERT (preserva historia):
→ Crea nuevo commit que deshace
→ Historia intacta
→ Seguro para commits públicos
```

**Recuperación:**

```bash
# Si hiciste reset por error:
git reflog
git reset --hard HEAD@{1}
```

**Troubleshooting común:**

```bash
# Problema 1: Hice reset --hard por error
# Solución: Usar reflog para recuperar
git reflog                  # Encuentra el commit perdido
git reset --hard HEAD@{2}   # Vuelve a ese estado

# Problema 2: No sé qué modo de reset usar
# Solución:
# --soft:  Solo quieres rehacer el commit, mantener cambios en staging
# --mixed: Quieres revisar/reorganizar antes de commitear de nuevo
# --hard:  Quieres BORRAR todo (úsalo con cuidado)

# Problema 3: Reset no funciona como esperaba
# Solución: Verifica el estado antes y después
git log --oneline           # Ve dónde estás
git reset --soft HEAD~1
git status                  # Verifica que cambios están en staging

# Problema 4: Quiero deshacer reset
# Solución: Usar reflog
git reflog
git reset --hard HEAD@{1}   # Vuelve al estado anterior

# Problema 5: Reset en rama compartida
# Solución: NO hagas reset en ramas públicas
# Usa git revert en su lugar (ver sección de revert)
```

**Mejores prácticas:**

```bash
✓ Usa --soft para reorganizar commits
✓ Usa --mixed para unstage
✓ Usa --hard solo si estás seguro
✓ Recuerda: reflog es tu red de seguridad

✗ No uses reset --hard en commits públicos
✗ No uses reset en main/develop compartidos
✗ Evita reset --hard sin verificar cambios
```

---

## 16. git stash - Guardado Temporal
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Guarda trabajo en progreso temporalmente sin commitear.

**Funcionamiento interno:**
```
1. Crea commits especiales en refs/stash
2. Guarda working + staging
3. Limpia working directory
4. Como una pila (stack): LIFO
```

**Uso práctico:**

```bash
# Stash básico
git stash
git stash push -m "WIP: feature half done"

# Stash incluyendo untracked
git stash -u

# Stash interactivo
git stash -p

# Ver lista
git stash list

# Ver contenido
git stash show
git stash show -p

# Aplicar stash (mantiene en lista)
git stash apply
git stash apply stash@{2}

# Pop stash (aplica y elimina)
git stash pop

# Crear rama desde stash
git stash branch nueva-rama

# Eliminar stash
git stash drop
git stash drop stash@{1}

# Limpiar todos
git stash clear
```

**Casos de uso:**

```bash
# Cambio urgente en otra rama
git stash
git checkout main
git checkout -b hotfix
# ... arreglas ...
git checkout feature-x
git stash pop

# Pull con cambios locales
git stash
git pull
git stash pop
# O:
git pull --autostash
```

**Mejores prácticas:**

```bash
✓ Usa mensajes descriptivos con -m
✓ Limpia stashes viejos regularmente
✓ Usa stash -u si añadiste archivos nuevos
✓ Prefiere stash pop sobre apply

✗ No uses stash como sistema de backup
✗ No acumules decenas de stashes
✗ No stashees y olvides
```

---

## 17. git tag - Marcando Versiones
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Crea referencias inmutables a commits (usualmente para versiones).

**Funcionamiento interno:**
```
Lightweight tag: Solo referencia
Annotated tag: Objeto completo con mensaje, autor, fecha
```

**Uso práctico:**

```bash
# Crear lightweight tag
git tag v1.0.0

# Crear annotated tag (RECOMENDADO)
git tag -a v1.0.0 -m "Release 1.0.0"

# ============================================
# LISTAR TAGS
# ============================================

# Listar todos los tags
git tag
# → Orden alfabético por defecto

# Listar con patrón
git tag -l "v1.*"
git tag -l "v*-beta*"
git tag --list "release-*"

# Listar tags que contienen un commit
git tag --contains abc123
git tag --contains HEAD

# Listar tags que NO contienen un commit
git tag --no-contains abc123

# Listar tags merged/no-merged
git tag --merged main
git tag --no-merged main

# Listar tags con anotaciones
git tag -n
git tag -n5  # Muestra hasta 5 líneas del mensaje

# Ordenar tags
git tag --sort=-creatordate      # Por fecha (más recientes primero)
git tag --sort=version:refname   # Por versión semántica
git tag --sort=refname           # Alfabético
git tag --sort=-taggerdate       # Por fecha del tagger


# ============================================
# VER DETALLES DE TAGS
# ============================================

# Ver información completa
git show v1.0.0
# → Muestra tag object + commit + diff

# Ver solo información del tag
git show v1.0.0 --no-patch

# Ver múltiples tags
git show v1.0.0 v2.0.0

# Ver commit al que apunta
git rev-list -n 1 v1.0.0

# Ver diferencia entre tags
git diff v1.0.0..v2.0.0
git log v1.0.0..v2.0.0 --oneline


# ============================================
# FORMATO PERSONALIZADO (--format)
# ============================================

> 📖 **NOTA:** Para una referencia completa de todos los placeholders disponibles,
> formatos avanzados, condicionales y ejemplos con otros comandos (log, branch,
> for-each-ref, show-ref, etc.), consulta la **[Sección 22: Referencias y Placeholders de Formato](#22-referencias-y-placeholders-de-formato)**.

# git tag también acepta placeholders como git branch
# Ver sección 22 para lista completa

# Lista simple con hash
git tag --format="%(refname:short) %(objectname:short)"
# Salida:
# v1.0.0 a1b2c3d
# v1.1.0 e4f5g6h
# v2.0.0 i7j8k9l

# Con fecha y autor
git tag --format="%(refname:short) | %(creatordate:short) | %(taggername)"
# Salida:
# v1.0.0 | 2024-01-15 | Juan Pérez
# v1.1.0 | 2024-02-20 | María García

# Con mensaje del tag
git tag --format="%(refname:short) - %(contents:subject)"
# Salida:
# v1.0.0 - Initial release
# v1.1.0 - Bug fixes and improvements

# Con información completa
git tag --format="Tag: %(refname:short)
Commit: %(objectname:short)
Fecha: %(creatordate:short)
Autor: %(taggername) <%(taggeremail)>
Mensaje: %(contents:subject)
---"

# Con colores
git tag --format="%(color:green)%(refname:short)%(color:reset) (%(creatordate:relative))"

# Ordenado por fecha con formato
git tag --sort=-creatordate --format="%(creatordate:short) %(refname:short) - %(contents:subject)"

# Export a CSV
git tag --format="%(refname:short),%(objectname:short),%(taggername),%(creatordate:short),%(contents:subject)" > tags.csv


# PLACEHOLDERS ESPECÍFICOS PARA TAGS:
%(refname)              # refs/tags/v1.0.0
%(refname:short)        # v1.0.0
%(objectname)           # Hash del tag object
%(objectname:short)     # Hash abreviado
%(objecttype)           # "tag" o "commit"
%(taggername)           # Nombre del tagger (solo annotated)
%(taggeremail)          # Email del tagger
%(taggerdate)           # Fecha del tag
%(taggerdate:short)     # 2024-02-13
%(taggerdate:relative)  # "2 days ago"
%(creatordate)          # Fecha de creación (funciona con lightweight)
%(contents)             # Mensaje completo del tag
%(contents:subject)     # Primera línea del mensaje
%(contents:body)        # Cuerpo del mensaje (sin subject)


# ============================================
# CREAR Y GESTIONAR TAGS
# ============================================

# Crear lightweight tag (simple puntero)
git tag v1.0.0
# → Solo referencia al commit, sin metadata

# Crear annotated tag (RECOMENDADO para releases)
git tag -a v1.0.0 -m "Release 1.0.0"
# → Objeto completo: mensaje, autor, fecha, firma opcional

# Tag con mensaje multilínea
git tag -a v1.0.0 -m "Release 1.0.0

Features:
- User authentication
- Payment integration
- Dashboard redesign"

# Tag en commit específico
git tag -a v1.0.0 abc123 -m "Release 1.0.0"

# Tag con editor
git tag -a v1.0.0
# → Abre editor para escribir mensaje extenso

# Tag con firma GPG
git tag -s v1.0.0 -m "Signed release 1.0.0"
# → Crea tag firmado, verificable

# Verificar firma de tag
git tag -v v1.0.0
git show --show-signature v1.0.0

# Tag forzado (reemplazar existente)
git tag -f v1.0.0
git tag -af v1.0.0 -m "Release 1.0.0 (updated)"


# ============================================
# ELIMINAR TAGS
# ============================================

# Eliminar tag local
git tag -d v1.0.0

# Eliminar múltiples tags locales
git tag -d v1.0.0 v1.1.0 v2.0.0

# Eliminar tag remoto
git push origin --delete v1.0.0
# o (sintaxis vieja):
git push origin :refs/tags/v1.0.0

# Eliminar todos los tags locales (cuidado)
git tag -l | xargs git tag -d


# ============================================
# PUSH DE TAGS
# ============================================

# Push de un tag específico
git push origin v1.0.0

# Push de todos los tags
git push --tags
# o:
git push origin --tags

# Push de tag y commit juntos
git push origin main --follow-tags
# → Pushea commit + tags anotados alcanzables

# Configurar push automático de tags
git config --global push.followTags true
# → Pushea tags automáticamente con commits


# ============================================
# CHECKOUT Y RAMAS DESDE TAGS
# ============================================

# Checkout de tag (detached HEAD)
git checkout v1.0.0
# → Estás en estado "detached HEAD"
# → Útil para revisar código de release

# Crear rama desde tag
git checkout -b hotfix-1.0.1 v1.0.0
# → Crea rama apuntando al commit del tag
# → Útil para hotfixes en versiones antiguas

# Ver en qué ramas está un tag
git branch --contains v1.0.0
git branch -a --contains v1.0.0  # Incluye remotas
```

**Semantic Versioning:**

```bash
v<MAJOR>.<MINOR>.<PATCH>

Ejemplos:
v1.0.0           # Release estable
v1.0.0-alpha.1   # Pre-release
v1.0.0-beta.2    # Beta
v1.0.0-rc.1      # Release candidate

Incremento:
v1.2.3 → v2.0.0  # Breaking change (MAJOR)
v1.2.3 → v1.3.0  # New feature (MINOR)
v1.2.3 → v1.2.4  # Bug fix (PATCH)
```

**Mejores prácticas:**

```bash
✓ Usa annotated tags para releases (-a)
✓ Sigue semantic versioning
✓ Firma tags importantes con GPG (-s)
✓ Push tags explícitamente
✓ Tag desde main después de merge

✗ No muevas tags ya pusheados
✗ No uses lightweight tags para releases
✗ No olvides pushear tags
```

---

## 18. git revert - Deshaciendo Commits Públicos

**¿Qué hace?**
Crea NUEVO commit que deshace cambios de commit anterior.

**Funcionamiento interno:**
```
1. Lee commit a revertir
2. Calcula inverso de cambios
3. Aplica cambios inversos
4. Crea nuevo commit
5. Historia se mantiene intacta
```

**Uso práctico:**

```bash
# Revert de commit
git revert abc123
git revert abc123 --no-edit

# Revert de HEAD
git revert HEAD
git revert HEAD~3

# Revert múltiples
git revert HEAD~3..HEAD
git revert abc123 def456 ghi789

# Revert sin commit automático
git revert --no-commit abc123

# Revert de merge commit
git revert -m 1 abc123
# -m 1 = mantiene padre 1 (main line)

# Abortar/continuar
git revert --abort
git revert --continue
```

**Revert vs Reset:**

```bash
RESET (reescribe historia):
→ Mueve rama atrás
→ Commits desaparecen
→ Solo commits locales

REVERT (preserva historia):
→ Nuevo commit que deshace
→ Historia intacta
→ Seguro para commits públicos

¿Cuándo usar cada uno?
RESET: Commits locales no pusheados
REVERT: Commits ya pusheados/públicos
```

**Mejores prácticas:**

```bash
✓ Usa revert para commits públicos
✓ Usa --no-commit para múltiples como uno
✓ Incluye razón del revert en mensaje
✓ Usa -m 1 para revert de merges

✗ No uses revert para commits locales (usa reset)
✗ No omitas -m en revert de merge
```

---

## 19. git cherry-pick - Aplicando Commits Selectivos

**¿Qué hace?**
Aplica cambios de commit específico a rama actual.

**Funcionamiento interno:**
```
1. Lee commit a cherry-pick
2. Calcula diff
3. Aplica diff a rama actual
4. Crea NUEVO commit (hash diferente)
```

**Uso práctico:**

```bash
# Cherry-pick básico
git cherry-pick abc123

# Sin commit automático
git cherry-pick --no-commit abc123

# Múltiples commits
git cherry-pick abc123 def456 ghi789
git cherry-pick abc123..ghi789

# Con nota de origen
git cherry-pick -x abc123
# Añade: (cherry picked from commit abc123)

# Abortar/continuar
git cherry-pick --abort
git cherry-pick --continue
```

**Casos de uso:**

```bash
# Hotfix de producción
git checkout production
git cherry-pick abc123  # Fix de develop
git push origin production

# Backport a versión anterior
git checkout release-2.0
git cherry-pick def456  # Feature de main
git push origin release-2.0

# Mover commits entre ramas
git checkout rama-correcta
git cherry-pick abc123
git checkout rama-incorrecta
git reset --hard HEAD~1
```

**Cherry-pick vs Merge:**

```bash
MERGE:
→ Trae toda la rama
→ Merge commit
→ Historia completa

CHERRY-PICK:
→ Solo commits específicos
→ Sin merge commit
→ Commits duplicados

¿Cuándo usar?
MERGE: Feature completa
CHERRY-PICK: Hotfixes, backports
```

**Mejores prácticas:**

```bash
✓ Usa cherry-pick para fixes urgentes
✓ Usa -x para rastrear origen
✓ Usa --no-commit para combinar múltiples

✗ No uses como reemplazo de merge
✗ No cherry-picks en exceso
✗ Evita cherry-pick de merges sin -m
```

---

## 20. git clean - Limpiando Archivos No Rastreados
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Elimina archivos untracked del working directory.

**⚠️ PELIGRO: Eliminación NO es reversible**

**Funcionamiento interno:**
```
1. Escanea working directory
2. Identifica archivos untracked
3. Los elimina del filesystem
```

**Uso práctico:**

```bash
# ⚠️ SIEMPRE DRY-RUN PRIMERO
git clean -n
git clean --dry-run

# Eliminar archivos
git clean -f

# Eliminar archivos + directorios
git clean -fd

# Eliminar TODO (incluye .gitignore)
git clean -fxd

# Interactivo (RECOMENDADO)
git clean -i

# Con exclusiones
git clean -fxd -e "*.log"
git clean -fd -e node_modules
```

**Clean vs Reset:**

```bash
CLEAN: Elimina archivos untracked
→ NO en Git
→ NO recuperables

RESET: Descarta cambios tracked
→ En Git
→ Recuperables con reflog

COMBINACIÓN (reset completo):
git reset --hard HEAD  # Tracked
git clean -fxd         # Untracked
```

**Mejores prácticas:**

```bash
✓ SIEMPRE usa -n (dry-run) primero
✓ Usa -i (interactive) para selectivo
✓ Usa .gitignore para archivos ignorables
✓ Verifica con git status antes

✗ NUNCA uses git clean sin revisar
✗ No uses -x sin entender consecuencias
✗ No asumas que puedes recuperar
```

---

## 21. git rm y git mv - Eliminando y Moviendo Archivos
[⬆️ Top](#tabla-de-contenidos)

**¿Qué hace?**
Elimina o mueve archivos en Git y working directory.

**Funcionamiento interno:**
```
git rm:
1. Elimina archivo del filesystem
2. Actualiza .git/index
3. Cambio stageado (necesitas commitear)

git mv:
1. git rm old
2. git add new
3. Git detecta rename automáticamente
```

**Uso práctico - git rm:**

```bash
# Eliminar archivo (disk + Git)
git rm archivo.txt

# Eliminar solo de Git (mantener en disk)
git rm --cached archivo.txt

# Eliminar forzado
git rm -f archivo.txt

# Eliminar directorio
git rm -r directorio/

# Con wildcards
git rm '*.txt'
```

**Uso práctico - git mv:**

```bash
# Mover/renombrar archivo
git mv viejo.txt nuevo.txt

# Mover a directorio
git mv archivo.txt src/

# Renombrar directorio
git mv old-dir/ new-dir/
```

**rm/mv vs git rm/mv:**

```bash
# RM (comando shell):
rm archivo.txt
git add archivo.txt
→ 2 pasos

# GIT RM:
git rm archivo.txt
→ 1 paso, automáticamente stageado

# MV (comando shell):
mv old.txt new.txt
git rm old.txt && git add new.txt
→ Git detecta rename igual

# GIT MV:
git mv old.txt new.txt
→ Más claro, rename explícito
```

**Casos de uso:**

```bash
# Eliminar archivo sensible
git rm --cached .env
echo ".env" >> .gitignore
git add .gitignore
git commit -m "chore: Remove .env from tracking"

# Reorganizar proyecto
git mv lib/*.js src/lib/
git commit -m "refactor: Reorganize structure"

# Case-sensitive rename (macOS/Windows)
git mv readme.md temp
git mv temp README.md
git commit -m "docs: Fix README capitalization"
```

**Mejores prácticas:**

```bash
✓ Usa git rm en vez de rm (más claro)
✓ Usa git mv en vez de mv (detecta rename)
✓ Usa --cached para unstage sin eliminar
✓ Commitea después de rm/mv

✗ No uses rm -rf .git (NUNCA)
✗ No uses git rm -f sin revisar
✗ No olvides commitear después
```

---

## WORKFLOWS COMUNES

### Workflow 1: Feature Branch

```bash
# Crear feature
git checkout -b feature/user-auth
# ... desarrollo ...
git add .
git commit -m "feat: Add user authentication"
git push -u origin feature/user-auth

# PR en GitHub
# Tras aprobación:
git checkout main
git pull origin main
git merge --no-ff feature/user-auth
git push origin main
git branch -d feature/user-auth
git push origin --delete feature/user-auth
```

### Workflow 2: Fork Contribution

```bash
# Setup
git clone https://github.com/tu-fork/proyecto.git
cd proyecto
git remote add upstream https://github.com/original/proyecto.git

# Sincronizar
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Contribuir
git checkout -b fix/bug-123
# ... fixes ...
git commit -am "fix: Resolve issue #123"
git push -u origin fix/bug-123
# PR a upstream
```

### Workflow 3: Hotfix

```bash
# Hotfix urgente
git checkout main
git checkout -b hotfix/critical-bug
# ... fix ...
git commit -am "fix: Critical security issue"
git push -u origin hotfix/critical-bug

# Fast merge
git checkout main
git merge hotfix/critical-bug
git push origin main
git branch -d hotfix/critical-bug

# Tag
git tag -a v1.0.1 -m "Hotfix: Security patch"
git push origin v1.0.1
```

---

## TROUBLESHOOTING RÁPIDO

### Deshacer cambios

```bash
# Archivo modificado, no stageado
git restore archivo.txt

# Archivo stageado
git restore --staged archivo.txt

# Último commit (local)
git reset --soft HEAD~1

# Último commit (público)
git revert HEAD

# Múltiples commits (local)
git reset --hard HEAD~3

# Working directory completo
git reset --hard HEAD
git clean -fxd
```

### Recuperar trabajo perdido

```bash
# Ver reflog
git reflog

# Recuperar commit
git checkout abc123
git branch rescue-branch

# Recuperar después de reset
git reset --hard HEAD@{2}
```

### Conflictos de merge

```bash
# Durante merge
git status  # Ver conflictos
# Editar archivos
git add archivo-resuelto
git commit

# O abortar
git merge --abort

# Elegir versión completa
git checkout --ours archivo.txt
git checkout --theirs archivo.txt
```

### Problemas con push

```bash
# Push rechazado
git pull --rebase
git push

# Necesitas force push (feature branch)
git push --force-with-lease

# Remoto cambió
git fetch origin
git reset --hard origin/main
```

---

## 22. Referencias y Placeholders de Formato
[⬆️ Top](#tabla-de-contenidos)

**¿Qué son?**
Son variables internas que Git expone para personalizar la salida de comandos como `git log`, `git for-each-ref`, `git show-ref`, etc. Permiten crear formatos personalizados para scripts, informes y automatización.

**¿Dónde se usan?**
- En comandos con la opción `--format="..."`
- En plantillas de hooks
- En scripts para procesar información de Git
- Para exportar datos estructurados

> 📖 **REFERENCIAS CRUZADAS:** Esta sección proporciona la referencia completa de placeholders.
> Para ejemplos específicos de cada comando, consulta:
> - **[Sección 5: git log](#5-git-log---explorando-la-historia)** - Formatos personalizados con `--pretty`
> - **[Sección 6: git branch](#6-git-branch---gestionando-líneas-de-desarrollo)** - Formato con `--format`
> - **[Sección 17: git tag](#17-git-tag---marcando-versiones)** - Listar tags con formato personalizado

---

### Comandos que usan placeholders

#### 1. git for-each-ref

**Descripción:** Itera sobre todas las referencias (ramas, tags, etc.) y muestra información personalizada.

**Sintaxis:**
```bash
git for-each-ref [<opciones>] [<patrón>]
```

**Placeholders principales:**

```bash
# Información de la referencia
%(refname)           # Nombre completo: refs/heads/main
%(refname:short)     # Nombre corto: main
%(refname:lstrip=N)  # Elimina N componentes del inicio
%(refname:rstrip=N)  # Elimina N componentes del final

# Información del objeto
%(objecttype)        # Tipo: commit, tag, tree, blob
%(objectsize)        # Tamaño del objeto en bytes
%(objectname)        # Hash SHA-1 completo
%(objectname:short)  # Hash SHA-1 abreviado (7 caracteres)
%(objectname:short=N) # Hash abreviado con N caracteres

# Información del commit/tag
%(tree)              # Hash del árbol
%(parent)            # Hash(es) del/los padre(s)
%(author)            # Autor completo: Nombre <email>
%(authorname)        # Solo el nombre del autor
%(authoremail)       # Solo el email del autor
%(authordate)        # Fecha del autor (formato por defecto)
%(committer)         # Committer completo: Nombre <email>
%(committername)     # Solo el nombre del committer
%(committeremail)    # Solo el email del committer
%(committerdate)     # Fecha del committer
%(subject)           # Primera línea del mensaje de commit
%(body)              # Cuerpo del mensaje (sin el subject)
%(contents)          # Mensaje completo (subject + body)

# Información de tracking
%(upstream)          # Rama remota asociada (upstream)
%(upstream:short)    # Nombre corto de la rama remota
%(upstream:track)    # Estado de tracking: [ahead N, behind M]
%(upstream:trackshort) # Estado abreviado: >, <, <>, =

# Información adicional
%(HEAD)              # '*' si es la rama actual, ' ' si no
%(color:...)         # Aplicar color
%(align:...)         # Alinear texto
%(if:...)%(then)%(else)%(end) # Condicionales
```

**Ejemplos prácticos:**

```bash
# 1. Listar todas las ramas con sus hashes
git for-each-ref --format="%(refname:short) %(objectname:short)" refs/heads/

# Salida:
# main a1b2c3d
# develop e4f5g6h
# feature/login i7j8k9l

# 2. Ramas con información de tracking
git for-each-ref --format="%(refname:short) %(upstream:short) %(upstream:track)" refs/heads/

# Salida:
# main origin/main [ahead 2, behind 1]
# develop origin/develop [ahead 5]
# feature/login  

# 3. Listar tags con fechas y autores
git for-each-ref --format="%(refname:short) %(authordate:short) %(authorname)" refs/tags/

# Salida:
# v1.0.0 2024-01-15 Juan Pérez
# v1.1.0 2024-02-20 María García

# 4. Información completa formateada
git for-each-ref --format="Rama: %(refname:short)
  Último commit: %(objectname:short)
  Autor: %(authorname)
  Fecha: %(authordate:relative)
  Mensaje: %(subject)
  Tracking: %(upstream:track)
" refs/heads/

# 5. Con colores
git for-each-ref --format="%(color:green)%(refname:short)%(color:reset) - %(subject)" refs/heads/

# 6. Ordenar por fecha de commit
git for-each-ref --sort=-committerdate --format="%(committerdate:short) %(refname:short)" refs/heads/

# Salida:
# 2024-03-01 feature/new-ui
# 2024-02-28 develop
# 2024-02-15 main

# 7. Filtrar ramas remotas
git for-each-ref --format="%(refname:short)" refs/remotes/origin/

# 8. Ramas con ahead/behind visual
git for-each-ref --format="%(refname:short) %(upstream:trackshort)" refs/heads/

# Salida:
# main <>    (divergente: tengo commits y hay remotos)
# develop >  (ahead: tengo commits para subir)
# feature <  (behind: hay commits remotos para traer)
```

**Opciones de formato de fecha:**

```bash
%(authordate:relative)    # "2 days ago"
%(authordate:short)       # "2024-02-13"
%(authordate:local)       # En zona horaria local
%(authordate:iso)         # ISO 8601: "2024-02-13 14:30:45 +0100"
%(authordate:iso-strict)  # ISO 8601 estricto
%(authordate:rfc)         # RFC 2822
%(authordate:raw)         # Unix timestamp + zona
%(authordate:unix)        # Solo Unix timestamp
%(authordate:format:...)  # Formato personalizado (strftime)
```

**Opciones de git for-each-ref:**

```bash
--count=<n>              # Limitar a n referencias
--sort=<key>             # Ordenar por campo (- para descending)
--format=<format>        # Formato de salida personalizado
--shell                  # Formato para shell scripts
--perl                   # Formato para Perl
--python                 # Formato para Python
--tcl                    # Formato para Tcl
--points-at=<object>     # Solo refs que apuntan a objeto
--merged[=<commit>]      # Solo refs fusionadas en commit
--no-merged[=<commit>]   # Solo refs NO fusionadas en commit
--contains[=<commit>]    # Solo refs que contienen commit
--no-contains[=<commit>] # Solo refs que NO contienen commit
```

---

#### 2. git log con --format

**Placeholders para commits:**

```bash
# Hash del commit
%H    # Hash completo (40 caracteres)
%h    # Hash abreviado
%T    # Hash del tree
%t    # Hash del tree abreviado
%P    # Hashes de los padres (completos)
%p    # Hashes de los padres (abreviados)

# Información del autor
%an   # Nombre del autor
%ae   # Email del autor
%aE   # Email del autor (respetando .mailmap)
%ad   # Fecha del autor (formato según --date)
%aD   # Fecha del autor (RFC2822)
%ar   # Fecha del autor (relativa)
%at   # Fecha del autor (timestamp UNIX)
%ai   # Fecha del autor (ISO 8601)
%aI   # Fecha del autor (ISO 8601 estricto)

# Información del committer
%cn   # Nombre del committer
%ce   # Email del committer
%cE   # Email del committer (respetando .mailmap)
%cd   # Fecha del committer
%cD   # Fecha del committer (RFC2822)
%cr   # Fecha del committer (relativa)
%ct   # Fecha del committer (timestamp UNIX)
%ci   # Fecha del committer (ISO 8601)
%cI   # Fecha del committer (ISO 8601 estricto)

# Referencias (ramas/tags)
%d    # Nombres de ref (como --decorate)
%D    # Nombres de ref sin los paréntesis
%S    # Ref name (dada en la línea de comando)

# Mensaje del commit
%s    # Subject (primera línea)
%f    # Subject sanitizado (para nombre de archivo)
%b    # Body (resto del mensaje)
%B    # Body raw (sin procesar)
%N    # Notas del commit
%GG   # Mensaje raw de verificación GPG
%G?   # Estado de firma GPG
%GS   # Nombre del firmante GPG
%GK   # Key usada para firmar

# Colores
%Cred       # Cambiar a rojo
%Cgreen     # Cambiar a verde
%Cblue      # Cambiar a azul
%Creset     # Reset color
%C(...)     # Color específico (por nombre o código)

# Otros
%n    # Nueva línea
%x00  # Byte nulo
%%    # Literal '%'
```

**Ejemplos prácticos con git log:**

```bash
# 1. Log personalizado básico
git log --format="%h - %an, %ar : %s"

# Salida:
# a1b2c3d - Juan Pérez, 2 days ago : Add login feature
# e4f5g6h - María García, 1 week ago : Fix bug in payment

# 2. Con colores
git log --format="%C(yellow)%h%C(reset) - %C(cyan)%an%C(reset), %ar : %s"

# 3. Formato completo estilo GitHub
git log --format="%C(auto)%h%d %s %C(black)%C(bold)%cr by %an"

# 4. Para exportar a CSV
git log --format="%h,%an,%ae,%ad,%s" --date=short > commits.csv

# 5. Con estadísticas
git log --format="%h %an %s" --stat

# 6. Commits con firma GPG
git log --format="%h %s %G? %GS"
# %G? muestra: G (buena), B (mala), U (sin verificar), N (sin firma)

# 7. Log detallado para análisis
git log --format="Commit: %H
Autor: %an <%ae>
Fecha: %ad
Committer: %cn <%ce>
Fecha Commit: %cd

%s

%b
---"

# 8. Log tipo GitHub/GitLab
git log --graph --format="%C(yellow)%h%C(reset) %C(bold blue)%an%C(reset) %C(dim white)%ar%C(reset) %s %C(auto)%d"

# 9. Solo hash y subject
git log --format="%h %s" -10

# 10. Log con información de merge
git log --format="%h %s (padres: %p)" --merges
```

**Opciones de --date para git log:**

```bash
git log --format="%h %ad %s" --date=relative   # "2 hours ago"
git log --format="%h %ad %s" --date=local      # Zona horaria local
git log --format="%h %ad %s" --date=iso        # ISO 8601
git log --format="%h %ad %s" --date=iso-strict # ISO 8601 estricto
git log --format="%h %ad %s" --date=rfc        # RFC 2822
git log --format="%h %ad %s" --date=short      # YYYY-MM-DD
git log --format="%h %ad %s" --date=raw        # Timestamp + zona
git log --format="%h %ad %s" --date=unix       # Timestamp UNIX
git log --format="%h %ad %s" --date=format:"%Y-%m-%d %H:%M"  # Personalizado
git log --format="%h %ad %s" --date=human      # Formato legible
git log --format="%h %ad %s" --date=default    # Formato por defecto
```

---

#### 3. git show-ref

**Descripción:** Muestra referencias disponibles en el repositorio local.

**Sintaxis:**
```bash
git show-ref [<opciones>] [<patrón>]
```

**Salida por defecto:**
```bash
git show-ref

# Formato: <hash> <refname>
a1b2c3d4... refs/heads/main
e5f6g7h8... refs/heads/develop
i9j0k1l2... refs/remotes/origin/main
m3n4o5p6... refs/tags/v1.0.0
```

**Opciones:**

```bash
--head              # Incluir HEAD
--heads             # Solo ramas locales (refs/heads/)
--tags              # Solo tags (refs/tags/)
-d, --dereference   # Mostrar objeto al que apunta un tag anotado
--hash[=<n>]        # Solo mostrar hash (opcionalmente primeros n chars)
--abbrev[=<n>]      # Abreviar hash a n caracteres
--quiet             # No mostrar nada, solo retornar código de salida
--verify            # Verificar que existe exactamente una referencia
--exclude-existing  # Filtrar refs que ya existen
```

**Ejemplos:**

```bash
# 1. Ver todas las referencias
git show-ref

# 2. Solo ramas locales
git show-ref --heads

# 3. Solo tags
git show-ref --tags

# 4. Buscar una rama específica
git show-ref main
# a1b2c3d4... refs/heads/main
# i9j0k1l2... refs/remotes/origin/main

# 5. Solo el hash
git show-ref --hash refs/heads/main
# a1b2c3d4e5f6...

# 6. Hash abreviado
git show-ref --hash --abbrev refs/heads/main
# a1b2c3d

# 7. Verificar que existe una referencia
git show-ref --verify refs/heads/main
# Retorna 0 si existe, 1 si no

# 8. Con HEAD
git show-ref --head

# 9. Tags con dereferencia (objeto apuntado)
git show-ref --tags --dereference
# m3n4o5p6... refs/tags/v1.0.0
# a1b2c3d4... refs/tags/v1.0.0^{}  (commit al que apunta)
```

---

#### 4. Otros comandos que usan placeholders

##### git branch --format

```bash
# Formato personalizado para ramas
git branch --format="%(refname:short) → %(upstream:short) %(upstream:track)"

# Con colores
git branch --format="%(color:green)%(refname:short)%(color:reset) %(upstream:trackshort)"
```

##### git tag --format

```bash
# Listar tags con información
git tag --format="%(refname:short): %(subject) - %(authorname)"

# Tags ordenados por fecha
git tag --sort=-creatordate --format="%(creatordate:short) %(refname:short)"
```

---

### Formato avanzado: condicionales y alineación

#### Condicionales

```bash
# Sintaxis: %(if:condition)%(then)TEXTO%(else)OTRO%(end)

# Ejemplo: mostrar upstream solo si existe
git for-each-ref --format="%(refname:short) %(if)%(upstream)%(then)→ %(upstream:short)%(end)" refs/heads/

# Ejemplo: color según si está merged
git for-each-ref --format="%(if:equals=refs/heads/main)%(refname)%(then)%(color:green)%(end)%(refname:short)%(color:reset)"
```

#### Alineación

```bash
# %(align:<width>,<position>)TEXTO%(end)
# position: left, right, middle

# Ejemplo: tabla alineada
git for-each-ref --format="%(align:20,left)%(refname:short)%(end) %(align:10,right)%(objectname:short)%(end) %(subject)" refs/heads/

# Salida:
# main                  a1b2c3d    Initial commit
# develop               e4f5g6h    Add feature
# feature/login         i7j8k9l    Login page
```

---

### Casos de uso prácticos

#### 1. Listar ramas desactualizadas

```bash
git for-each-ref --sort=-committerdate --format="%(committerdate:short) %(refname:short)" refs/heads/ | head -10
```

#### 2. Encontrar ramas sin upstream

```bash
git for-each-ref --format="%(if)%(upstream)%(then)%(else)%(refname:short)%(end)" refs/heads/ | grep -v '^$'
```

#### 3. Exportar historial para análisis

```bash
git log --format="%H,%an,%ae,%ad,%s" --date=iso-strict --all > commits.csv
```

#### 4. Verificar firmas GPG

```bash
git log --format="%h %G? %GS: %s" --show-signature
```

#### 5. Listar ramas merged y no merged

```bash
# Merged en main
git for-each-ref --merged=main --format="✓ %(refname:short)" refs/heads/

# No merged en main
git for-each-ref --no-merged=main --format="✗ %(refname:short)" refs/heads/
```

#### 6. Ver estado de tracking de todas las ramas

```bash
git for-each-ref --format="%(refname:short)%(if)%(upstream)%(then) → %(upstream:short) %(upstream:track)%(else) (sin upstream)%(end)" refs/heads/

# Salida:
# main → origin/main [ahead 2, behind 1]
# develop → origin/develop [up to date]
# feature/login (sin upstream)
```

#### 7. Buscar quién hizo el último commit en cada rama

```bash
git for-each-ref --format="%(refname:short): %(authorname) - %(authordate:relative)" refs/heads/
```

#### 8. Script para limpiar ramas merged

```bash
#!/bin/bash
# Listar ramas merged (excepto main/develop) y eliminarlas
git for-each-ref --format="%(refname:short)" --merged=main refs/heads/ | \
  grep -v -E '^(main|develop)$' | \
  xargs -r git branch -d
```

#### 9. Log estilo commit convencional

```bash
git log --format="%C(yellow)%h%C(reset) %C(blue)%ad%C(reset) %C(green)%an%C(reset)%n  %s%n" --date=short
```

#### 10. Comparar fechas de autor vs committer

```bash
git log --format="Commit: %h%nAutor fecha: %ai%nCommitter fecha: %ci%nDiferencia: %ar vs %cr%n---"
```

---

### Resumen de placeholders más usados

| Placeholder | Descripción | Ejemplo de salida |
|-------------|-------------|-------------------|
| `%(refname)` | Nombre completo de ref | `refs/heads/main` |
| `%(refname:short)` | Nombre corto de ref | `main` |
| `%(objectname)` | Hash SHA-1 completo | `a1b2c3d4e5f6...` |
| `%(objectname:short)` | Hash abreviado | `a1b2c3d` |
| `%(upstream)` | Rama upstream | `refs/remotes/origin/main` |
| `%(upstream:short)` | Upstream corto | `origin/main` |
| `%(upstream:track)` | Estado tracking | `[ahead 2, behind 1]` |
| `%(upstream:trackshort)` | Estado abreviado | `<>` |
| `%(authorname)` | Nombre del autor | `Juan Pérez` |
| `%(authoremail)` | Email del autor | `juan@example.com` |
| `%(authordate)` | Fecha del autor | `Tue Feb 13 14:30:00 2024` |
| `%(subject)` | Primera línea mensaje | `Add login feature` |
| `%(contents)` | Mensaje completo | Todo el mensaje |
| `%h` (git log) | Hash abreviado | `a1b2c3d` |
| `%s` (git log) | Subject | `Fix bug` |
| `%an` (git log) | Autor nombre | `Juan Pérez` |
| `%ad` (git log) | Autor fecha | Según `--date` |
| `%d` (git log) | Refs decoradas | `(HEAD -> main, origin/main)` |

---

### Mejores prácticas

✅ **Usar en scripts:** Ideal para automatización y CI/CD
✅ **Exportar datos:** CSV, JSON-like para análisis
✅ **Personalizar salidas:** Adaptar a tus necesidades
✅ **Combinación con otros comandos:** Potente con grep, awk, etc.

❌ **No abusar de colores en scripts:** Solo para terminal
❌ **No confiar en orden sin --sort:** Especifica orden explícitamente
❌ **Cuidado con caracteres especiales:** Sanitizar para shell

---

### Troubleshooting

**Problema:** El formato no muestra lo esperado
```bash
# Verificar que el campo existe
git for-each-ref --format="%(refname) %(upstream)" refs/heads/
# Si upstream está vacío, esa rama no tiene tracking
```

**Problema:** Fechas en formato incorrecto
```bash
# Especificar formato de fecha
git for-each-ref --format="%(authordate:short)" refs/heads/
```

**Problema:** Condicionales no funcionan
```bash
# Asegurar sintaxis correcta
git for-each-ref --format="%(if)%(upstream)%(then)Tiene upstream%(else)Sin upstream%(end)" refs/heads/
```

---

### Recursos adicionales

- `git help for-each-ref` - Documentación completa de placeholders
- `git help log` - Sección PRETTY FORMATS
- [Git Documentation - for-each-ref](https://git-scm.com/docs/git-for-each-ref)
- [Git Documentation - log formats](https://git-scm.com/docs/git-log#_pretty_formats)

---

## CONFIGURACIÓN RECOMENDADA

```bash
# Identidad
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"

# Editor
git config --global core.editor "code --wait"

# Diff/merge tool
git config --global diff.tool vscode
git config --global merge.tool vscode

# Pull con rebase
git config --global pull.rebase true

# Push solo rama actual
git config --global push.default current

# Auto-setup tracking
git config --global push.autoSetupRemote true

# Fetch con prune
git config --global fetch.prune true

# Colores
git config --global color.ui auto

# Alias útiles
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.lg "log --graph --oneline --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
```

---

## RECURSOS ADICIONALES

**Documentación relacionada:**
- `GIT_FUNCIONAMIENTO_INTERNO.md` - Teoría y arquitectura de Git
- `GITHUB_ACTIONS_*.md` - Integración con CI/CD

**Aprend más:**
- [Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)

---

**Última actualización:** Febrero 2026
**Versión:** 1.0.0

Este documento cubre los 21 comandos Git más importantes con ejemplos prácticos del mundo real. Para entender el funcionamiento interno de Git, consulta `GIT_FUNCIONAMIENTO_INTERNO.md`.


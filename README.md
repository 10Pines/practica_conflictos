# Práctica de Git: Conflictos, Merge y Rebase

Este repo tiene ramas y escenarios pre-armados para practicar resolución de conflictos.

> **Regla del repo:** no se puede commitear directamente a `main` (hook de pre-commit).
> Todos los cambios entran vía PR desde una rama de feature.

---

## Ejercicio 1 — Merge con conflicto

**Situación:** Dos personas subieron el nivel de Aragorn en ramas distintas. Antes de abrir un PR, tenés que actualizar tu feature branch con los últimos cambios de main y resolver el conflicto ahí.

```bash
git checkout feature/conflicto-merge
git merge main
# → Conflicto en personajes.txt
# Resolvé el conflicto manualmente, luego:
git add personajes.txt
git commit -m "Merge: actualizo feature con cambios de main"
```

Para cancelar en cualquier momento: `git merge --abort`

> En GitFlow no se pushea directo a main. El flujo es: resolver conflictos en la feature branch → push → PR → merge a main.

---

## Ejercicio 2 — Rebase con conflicto

**Situación:** La rama `feature/conflicto-rebase` quedó desactualizada respecto a main.

```bash
git checkout feature/conflicto-rebase
git rebase main
# → Conflicto: resolvé en personajes.txt, luego:
git add personajes.txt
git rebase --continue
# Si querés cancelar: git rebase --abort
```

**Pregunta para pensar:** ¿Qué diferencia hay en el historial resultante vs el ejercicio 1?

---

## Ejercicio 3 — Squash: limpiar commits sucios

**Situación:** La rama `feature/commits-sucios` tiene 5 commits WIP que queremos unificar en uno.

```bash
git checkout feature/commits-sucios
git log --oneline   # mirá los 5 commits sucios
git rebase -i HEAD~5
# En el editor: cambiá "pick" por "squash" (o "s") en los commits 2 al 5
# Guardá → escribí el mensaje final del commit unificado
```

**Alternativa al mergear:**
```bash
git checkout main
git merge --squash feature/commits-sucios
git commit -m "Agrega sección de historias (squash)"
```

---

## Ejercicio 4 — Cherry-pick: traer un commit específico

**Situación:** Hay un hotfix en `main` que también necesita `release/v1.0`.

```bash
# Encontrá el hash del hotfix en main:
git log main --oneline

# Aplicalo en release/v1.0:
git checkout release/v1.0
git cherry-pick <hash-del-hotfix>
```

**Bonus:** ¿Qué pasa si hay conflicto? Resolvelo y hacé `git cherry-pick --continue`.

---

## Ejercicio 5 — Reflog: recuperar commits "perdidos"

**Situación:** Simulá perder commits con un reset duro y recuperalos con reflog.

```bash
# Simulá el "accidente":
git checkout feature/commits-sucios
git reset --hard HEAD~3   # "perdés" 3 commits

# Recuperate:
git reflog                # encontrá el hash del estado anterior
git reset --hard <hash>   # restaurá

# Alternativa segura (crea rama con lo recuperado):
git checkout -b recuperado <hash>
```

---

## Ejercicio 6 — Feature branch sobre otra feature branch

**Situación:** `feature/saruman-extension` salió de `feature/saruman-base`. Si mergeás la base a main, ¿podés después mergear la extensión sin conflictos?

**Depende de cómo mergeás la primera rama.**

### Caso A — Merge regular (funciona bien)

```bash
git checkout main
git merge feature/saruman-base

git merge feature/saruman-extension
# → Sin conflicto: git reconoce que los commits de saruman-base
#   ya están en main y solo aplica el commit nuevo de la extensión
```

### Caso B — Squash merge (genera conflicto)

```bash
git checkout main
git merge --squash feature/saruman-base
git commit -m "Agrega Saruman (squash)"

git merge feature/saruman-extension
# → CONFLICTO: git no reconoce la relación entre el squash commit
#   y los commits originales de saruman-base
```

### Solución A — `rebase -i` con drop (más didáctico)

```bash
git checkout feature/saruman-extension
git rebase -i main
# En el editor: "drop" el commit de saruman-base, conservá solo el de la extensión
git push --force-with-lease
```

### Solución B — `rebase --onto` (más quirúrgico)

```bash
# "Tomá los commits de saruman-extension que no están en saruman-base
#  y ponelos sobre main"
git rebase --onto main feature/saruman-base feature/saruman-extension
git push --force-with-lease
```

---

## Flujos reales

### GitHub Flow vs GitFlow

| | GitHub Flow | GitFlow |
|---|---|---|
| **Complejidad** | Simple | Estructurado |
| **Ramas principales** | `main` | `main` + `develop` |
| **Ramas de trabajo** | `feature/*` | `feature/*`, `release/*`, `hotfix/*` |
| **Deploy** | Cada merge a main | Solo desde `release/*` o `hotfix/*` |
| **Ideal para** | Deploy continuo, SaaS | Releases versionadas, mobile, libs |

**GitHub Flow** en la práctica:
```
main ──────────────────────────────────►
         \                    /
          feature/nueva-cosa ─
```

**GitFlow** en la práctica:
```
main    ─────────────────────────────────►  (solo releases)
           \              /
develop ────────────────────────────────►  (integración)
              \      /    \       /
               feature/*   release/*
```

> Este repo usa **GitFlow**: el hook de pre-commit bloquea commits directos a `main`.

### La regla de oro del rebase

> **Nunca hagas rebase de una rama que ya fue publicada y que otros están usando.**

```bash
# MAL: rebase de una rama que ya está en el remoto
git rebase main
git push --force   # ← destruye el historial del remoto

# BIEN: rebase solo en ramas locales antes del primer push
git rebase main
git push          # primer push, sin --force
```

---

## Referencia rápida

| Comando | Para qué |
|---|---|
| `git merge --abort` | Cancelar un merge en curso |
| `git rebase --abort` | Cancelar un rebase en curso |
| `git rebase --continue` | Continuar tras resolver conflicto en rebase |
| `git rebase -i HEAD~N` | Rebase interactivo de los últimos N commits |
| `git cherry-pick <hash>` | Traer un commit específico a la rama actual |
| `git reflog` | Ver historial completo de movimientos de HEAD |
| `git rebase --onto <nueva> <vieja> <rama>` | Trasplantar commits de una base a otra |

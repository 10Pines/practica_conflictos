# Práctica de Git: Conflictos, Merge y Rebase

Este repo tiene ramas y escenarios pre-armados para practicar resolución de conflictos.

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
Todo sale de `main`, todo vuelve a `main` vía PR. Simple y efectivo para equipos con CI/CD.

**GitFlow** en la práctica:
```
main    ─────────────────────────────────►  (solo releases)
           \              /
develop ────────────────────────────────►  (integración)
              \      /    \       /
               feature/*   release/*
```
`develop` es la rama de integración. Los features van ahí. Solo se toca `main` al liberar una versión.

> Este repo usa **GitFlow**: el hook de pre-commit bloquea commits directos a `main`.
> Los cambios entran via PR desde `feature/*`, `hotfix/*` o `release/*`.

---

### La regla de oro del rebase

> **Nunca hagas rebase de una rama que ya fue publicada (pusheada) y que otros están usando.**

**Por qué:** el rebase reescribe el historial (cambia los hashes de los commits). Si otra persona tiene esa rama, su historial y el tuyo van a divergir y el próximo `push` o `pull` va a ser un caos.

```
# MAL: rebase de una rama que ya está en el remoto y que otros tienen
git checkout feature/mi-rama
git rebase main
git push --force   # ← destruye el historial del remoto, rompe el repo de tus compañeros
```

```
# BIEN: rebase solo en ramas locales que nadie más tiene
git checkout feature/mi-rama-local
git rebase main   # ← nadie más tiene esta rama, es seguro
git push          # primer push, sin --force
```

**Regla práctica:** si ya hiciste `push` de la rama, usá `merge` para integrar cambios. Guardá el `rebase` para limpiar *antes* del primer push.

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

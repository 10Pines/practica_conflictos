# Práctica de Git: Conflictos, Merge y Rebase

Este repo tiene ramas y escenarios pre-armados para practicar resolución de conflictos.

---

## Ejercicio 1 — Merge con conflicto

**Situación:** Dos personas subieron el nivel de Aragorn en ramas distintas.

```bash
git checkout main
git merge feature/conflicto-merge
# → Conflicto en personajes.txt
# Resolvé el conflicto manualmente, luego:
git add personajes.txt
git commit -m "Merge: resuelvo conflicto de nivel de Aragorn"
```

Para cancelar en cualquier momento: `git merge --abort`

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

## Referencia rápida

| Comando | Para qué |
|---|---|
| `git merge --abort` | Cancelar un merge en curso |
| `git rebase --abort` | Cancelar un rebase en curso |
| `git rebase --continue` | Continuar tras resolver conflicto en rebase |
| `git rebase -i HEAD~N` | Rebase interactivo de los últimos N commits |
| `git cherry-pick <hash>` | Traer un commit específico a la rama actual |
| `git reflog` | Ver historial completo de movimientos de HEAD |

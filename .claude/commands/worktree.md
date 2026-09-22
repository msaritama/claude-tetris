---
description: Crea un git worktree en .trees/<nombre> (nombre inferido del requerimiento) y ejecuta ahí las instrucciones recibidas
argument-hint: <instrucciones a ejecutar en el nuevo worktree>
---

Vas a aislar el siguiente trabajo en un nuevo git worktree y luego ejecutarlo ahí dentro.

Instrucciones recibidas para ejecutar en el worktree:

$ARGUMENTS

Sigue estos pasos en orden:

1. **Determina el nombre.** A partir del requerimiento anterior, elige un nombre corto en kebab-case (minúsculas, sin acentos ni caracteres especiales, palabras separadas por guiones, idealmente 2-5 palabras) que lo resuma. Este nombre se usará como carpeta y como rama, p. ej. `fix-ghost-piece-color`, `add-hold-piece`, `feature-multiplayer`.

2. **Revisa el estado del repo.** Ejecuta `git status` antes de crear nada, para no arrastrar cambios pendientes sin commitear al worktree por accidente (o para decidir si conviene commitearlos/stashearlos primero).

3. **Crea el worktree** desde la raíz de este repo (la carpeta que contiene este `CLAUDE.md`), con una rama nueva:
   ```
   git worktree add .trees/<nombre> -b <nombre>
   ```
   Si la rama `<nombre>` ya existe, usa en su lugar:
   ```
   git worktree add .trees/<nombre> <nombre>
   ```
   `git worktree add` crea `.trees/` automáticamente si no existe.

4. **Asegura `.trees/` en `.gitignore`.** Si el archivo `.gitignore` de este proyecto no incluye una entrada para `.trees/` o `.trees`, agrégala — los worktrees no deben versionarse.

5. **Cambia al worktree.** Usa `.trees/<nombre>` como directorio de trabajo para todo lo que sigue (rutas relativas, comandos, etc.).

6. **Ejecuta las instrucciones recibidas** dentro de ese worktree como lo harías normalmente en el proyecto: lee el `CLAUDE.md` que aplique, implementa los cambios, corre lo que haga falta para verificarlos.

7. **Al terminar**, resume brevemente al usuario: nombre de la carpeta y rama creadas (`.trees/<nombre>`, rama `<nombre>`), qué se hizo ahí, y recuérdale que puede eliminarlo cuando ya no lo necesite con:
   ```
   git worktree remove .trees/<nombre>
   ```

No uses las herramientas `EnterWorktree`/`ExitWorktree` para esto: esas crean worktrees dentro de `.claude/worktrees/`, pero aquí el flujo pedido es explícitamente manual con `git worktree add .trees/<nombre>`.

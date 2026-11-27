# Flujo de desarrollo

Este documento explica cómo se organiza el trabajo en el repositorio: qué ramas existen, cómo se nombran, cómo se escriben los commits y qué pasos se siguen desde que empieza una funcionalidad hasta que se publica como una versión.

## 1. Estrategia de ramas

Se usa una versión simplificada de **Git Flow**:

| Rama | Para qué sirve | Quién escribe en ella |
| --- | --- | --- |
| `main` | Versión estable. Cada versión publicada queda aquí, marcada con una etiqueta (`v0.1.0`, `v0.2.0`…). | Solo recibe pull requests desde `develop`. |
| `develop` | Rama de integración. Contiene el trabajo terminado que aún no se ha publicado como versión. | Solo recibe pull requests desde ramas de trabajo (`feature/*`, `docs/*`, `fix/*`). |
| `feature/<nombre>` | Una rama por funcionalidad o tarea. | Aquí se hacen los commits del día a día. |

```
main      ●───────────────────────────●  v0.1.0
           \                         /
develop     ●───────●───────────────●
                     \             /
feature/xxx           ●───●───●───●
```

**Reglas:**

- `main` y `develop` están **protegidas** en GitHub: no se puede hacer push directo, todo entra por pull request.
- Las ramas `feature/*` siempre se crean **a partir de `develop`** actualizada.
- Una rama `feature/*` se borra después de fusionarse.

### Nombres de rama

`feature/` seguido de una descripción corta en minúsculas y con guiones:

- `feature/estructura-inicial`
- `feature/crud-estudiantes`
- `feature/login-jwt`
- `feature/registro-calificaciones`

Si la tarea es solo documentación se puede usar `docs/<nombre>`, y si es una corrección, `fix/<nombre>`. Ambas siguen el mismo flujo que `feature/*`.

## 2. Mensajes de commit

Se usa **Conventional Commits**:

```
<tipo>: <descripción corta en presente>
```

| Tipo | Cuándo se usa | Ejemplo |
| --- | --- | --- |
| `feat` | Nueva funcionalidad | `feat: agregar endpoint para crear estudiantes` |
| `fix` | Corrección de un error | `fix: validar que la nota esté entre 1.0 y 5.0` |
| `docs` | Solo documentación | `docs: agregar flujo de desarrollo` |
| `test` | Agregar o corregir pruebas | `test: pruebas unitarias de CalificacionService` |
| `refactor` | Cambio de código sin cambiar el comportamiento | `refactor: extraer cálculo de nota final a un método` |
| `chore` | Configuración, dependencias, estructura | `chore: configurar Spring Boot y Maven` |
| `ci` | Cambios en GitHub Actions | `ci: ejecutar pruebas en cada pull request` |

**Buenas prácticas:**

- Un commit = un cambio con sentido propio. Mejor varios commits pequeños que uno gigante.
- La descripción va en minúscula y sin punto final.
- Nunca se suben contraseñas, archivos `.env` ni carpetas generadas (`target/`, `node_modules/`). El `.gitignore` ya los excluye.

## 3. Paso a paso de una funcionalidad (en VS Code)

Todo el trabajo con Git se hace desde la vista **Source Control** de VS Code (icono de ramas en la barra izquierda, o `Ctrl+Shift+G`) y desde github.com.

1. **Actualizar `develop`.** Clic en el nombre de la rama abajo a la izquierda → elegir `develop`. Luego en Source Control → menú `···` → **Pull**.
2. **Crear la rama.** Clic en el nombre de la rama abajo a la izquierda → **Create new branch from...** → elegir `develop` → escribir el nombre, por ejemplo `feature/crud-estudiantes`.
3. **Programar** la funcionalidad con sus pruebas.
4. **Hacer commit.** En Source Control, revisar los archivos cambiados, pasar el mouse sobre **Changes** y clic en **+** para prepararlos (stage). Escribir el mensaje arriba y clic en **Commit**. Repetir cada vez que se termine un avance con sentido.
5. **Publicar la rama.** La primera vez aparece el botón **Publish Branch**; después, **Sync Changes**.
6. **Abrir el pull request** en github.com: aparece el aviso amarillo **Compare & pull request**. Base: `develop`, compare: la rama `feature/*`. Título con el mismo formato de los commits y una descripción corta de qué se hizo.
7. **Revisar el PR**: que los checks de GitHub Actions estén en verde (cuando estén configurados) y que la pestaña **Files changed** solo tenga lo esperado.
8. **Fusionar** con **Merge pull request** y luego **Delete branch**.
9. **Volver a `develop`** en VS Code y hacer **Pull** para traer lo fusionado. La rama local vieja se puede borrar desde la Command Palette (`Ctrl+Shift+P`) → **Git: Delete Branch...**.

## 4. Pull requests

Un PR está listo para fusionarse cuando:

- Va de `feature/*` hacia `develop` (o de `develop` hacia `main` al publicar una versión).
- El título sigue Conventional Commits.
- Compila y las pruebas pasan.
- No incluye archivos que no tienen que ver con la tarea.

## 5. Versiones

Se usa **versionado semántico** (`vMAYOR.MENOR.PARCHE`):

| Parte | Cuándo sube | Ejemplo |
| --- | --- | --- |
| MENOR | Se agrega un conjunto de funcionalidades | `v0.1.0` → `v0.2.0` |
| PARCHE | Solo correcciones, sin funcionalidades nuevas | `v0.1.0` → `v0.1.1` |
| MAYOR | Primera versión completa, o un cambio grande | `v0.x` → `v1.0.0` |

Mientras el proyecto está en desarrollo, las versiones empiezan por `0`. La `v1.0.0` corresponde a la versión final del proyecto.

| Versión | Contenido |
| --- | --- |
| `v0.1.0` | Análisis, diseño UML, arquitectura y estructura del repositorio |
| `v0.2.0` | Enfoque en colegios: análisis, UML y arquitectura actualizados |

Para publicar una versión, cuando el trabajo está completo en `develop`:

1. Abrir un pull request de `develop` hacia `main` con título `chore: release vX.Y.Z`. **No** borrar `develop` después del merge.
2. Fusionarlo con **Create a merge commit**.
3. En github.com → **Releases** → **Draft a new release** → crear la etiqueta `vX.Y.Z` con target `main`, describir qué incluye y hacer clic en **Publish release**.

Solo lo que está en `main` se despliega a producción.

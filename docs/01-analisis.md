# Análisis del sistema

## 1. Introducción

Este documento define qué debe hacer Platform antes de empezar a programarlo: quiénes lo usan, qué puede hacer cada uno, los casos de uso y los requerimientos funcionales y no funcionales. Es la base de los diagramas UML (`docs/uml/`) y de la arquitectura (`03-arquitectura.md`).

Cada requisito tiene un código único (CU-xx, RF-xx, RNF-xx). Los demás documentos se refieren a ellos por ese código, para que cada regla se defina en un solo lugar.

## 2. Actores

| Actor | Tipo | Descripción |
|---|---|---|
| Administrador académico | Humano | Gestiona estudiantes, docentes, cursos, periodos, matrículas y las cuentas de usuario. |
| Docente | Humano | Define las actividades evaluativas de sus cursos y registra notas y asistencia. |
| Estudiante | Humano | Consulta su información académica y sus alertas de riesgo. |
| Acudiente | Humano | Consulta la información académica de los estudiantes a su cargo. |
| Planificador | Sistema | Tarea automática que calcula el riesgo académico cada noche y envía alertas. |

Todos los actores humanos son **usuarios** del sistema: entran con correo y contraseña, y lo que ven depende de su rol. **No existe el autorregistro**: el Administrador crea todas las cuentas.

### 2.1 Matriz de permisos

✔ = puede hacerlo · **propio** = solo sobre sus cursos, o sobre sus propios datos · — = no tiene acceso

| Operación | Administrador | Docente | Estudiante | Acudiente |
|---|:-:|:-:|:-:|:-:|
| Iniciar y cerrar sesión | ✔ | ✔ | ✔ | ✔ |
| Crear, editar e inactivar cuentas de usuario | ✔ | — | — | — |
| Gestionar estudiantes | ✔ | — | — | — |
| Gestionar docentes y cursos | ✔ | — | — | — |
| Abrir y cerrar periodos | ✔ | — | — | — |
| Matricular estudiantes | ✔ | — | — | — |
| Definir actividades y porcentajes | — | propio | — | — |
| Registrar y modificar calificaciones | — | propio | — | — |
| Tomar asistencia | — | propio | — | — |
| Consultar notas, asistencia y horario | ✔ | propio | propio | propio |
| Usar el simulador de nota necesaria | — | — | propio | propio |
| Ver alertas de riesgo | ✔ | propio | propio | propio |
| Generar boletín y reportes | ✔ | propio | — | — |
| Consultar el historial de auditoría | ✔ | — | — | — |

Para el Acudiente, **propio** significa los estudiantes que tiene asociados.

## 3. Casos de uso

| Código | Caso de uso | Actor principal |
|---|---|---|
| CU-01 | Iniciar sesión y cerrar sesión | Todos |
| CU-02 | Gestionar estudiantes (crear, editar, buscar, inactivar) | Administrador |
| CU-03 | Gestionar docentes y cursos | Administrador |
| CU-04 | Abrir y cerrar periodo académico | Administrador |
| CU-05 | Matricular estudiante en curso | Administrador |
| CU-06 | Definir actividades evaluativas y sus porcentajes | Docente |
| CU-07 | Registrar calificaciones | Docente |
| CU-08 | Tomar asistencia | Docente |
| CU-09 | Consultar notas, asistencia y horario | Estudiante, Acudiente |
| CU-10 | Generar boletín y reportes | Administrador, Docente |
| CU-11 | Calcular riesgo académico y enviar alertas | Sistema (planificador) |

El diagrama de secuencia se hace sobre **CU-07 Registrar calificaciones**, porque recorre todas las capas: frontend, controlador, servicio, repositorio y el recálculo de la nota final.

### 3.1 Especificación de los casos de uso

#### CU-01 Iniciar sesión y cerrar sesión
- **Actor:** todos los usuarios.
- **Objetivo:** entrar al sistema y ver solo las opciones de su rol.
- **Precondición:** el usuario tiene una cuenta activa.
- **Flujo principal:**
  1. El usuario escribe su correo y su contraseña.
  2. El sistema verifica las credenciales y entrega un token de sesión.
  3. El sistema muestra el panel que corresponde al rol.
- **Flujos alternos:**
  - Las credenciales son incorrectas: el sistema muestra "Correo o contraseña incorrectos", sin decir cuál de los dos falló, y registra el intento fallido.
  - La cuenta está inactiva: el sistema no deja entrar.
- **Resultado:** el usuario queda con sesión iniciada. Al cerrar sesión, el token deja de usarse en el navegador.

#### CU-02 Gestionar estudiantes
- **Actor:** Administrador.
- **Objetivo:** mantener al día los datos de los estudiantes.
- **Flujo principal:**
  1. El Administrador busca un estudiante por documento o por nombre.
  2. Crea un estudiante, o edita el que encontró. Al crearlo, el sistema crea también su cuenta de usuario.
  3. Si es necesario, lo inactiva.
- **Flujos alternos:**
  - El documento ya existe: el sistema rechaza el registro.
  - Faltan datos obligatorios: el sistema indica cuáles.
- **Resultado:** los datos quedan guardados. **Inactivar no borra**: el historial académico se conserva.

#### CU-03 Gestionar docentes y cursos
- **Actor:** Administrador.
- **Objetivo:** registrar a los docentes y los cursos de cada periodo.
- **Flujo principal:**
  1. El Administrador registra al docente, y el sistema crea su cuenta.
  2. Crea el curso con código, nombre, créditos, cupo máximo y horario.
  3. Asigna el periodo, el docente y los cursos que son prerrequisito.
- **Flujos alternos:**
  - El código del curso ya existe en el periodo: el sistema rechaza el registro.
  - El periodo está CERRADO: no se pueden crear cursos en él.
- **Resultado:** el curso queda disponible para matrículas.

#### CU-04 Abrir y cerrar periodo académico
- **Actor:** Administrador.
- **Objetivo:** controlar en qué periodo se puede trabajar.
- **Flujo principal:**
  1. El Administrador crea el periodo (por ejemplo, 2026-2) con fecha de inicio y de fin, y lo deja ABIERTO.
  2. Al terminar el periodo, lo cierra.
  3. Al cerrarlo, cada matrícula ACTIVA pasa a APROBADA si la nota final es 3.0 o más, y a REPROBADA si es menor.
- **Flujo alterno:** las fechas son inválidas (el fin queda antes del inicio): el sistema rechaza el periodo.
- **Resultado:** un periodo CERRADO ya no admite matrículas, notas ni asistencia.

#### CU-05 Matricular estudiante en curso
- **Actor:** Administrador.
- **Objetivo:** inscribir a un estudiante en un curso del periodo abierto.
- **Precondición:** el estudiante está ACTIVO y el periodo del curso está ABIERTO.
- **Flujo principal:**
  1. El Administrador elige al estudiante y el curso.
  2. El sistema valida el cupo y los prerrequisitos («include»).
  3. El sistema crea la matrícula en estado ACTIVA.
- **Flujos alternos:**
  - El curso no tiene cupo: el sistema rechaza la matrícula.
  - Al estudiante le falta un prerrequisito, es decir, no tiene ese curso APROBADO: el sistema rechaza la matrícula e indica cuál falta.
  - Ya existe una matrícula ACTIVA del estudiante en ese curso: el sistema la rechaza.
- **Resultado:** el estudiante aparece en las listas del curso.

#### CU-06 Definir actividades evaluativas
- **Actor:** Docente.
- **Objetivo:** definir cómo se evalúa su curso.
- **Flujo principal:**
  1. El Docente crea las actividades (por ejemplo, Parcial 1, Taller, Final), cada una con su porcentaje y fecha.
  2. El sistema valida que los porcentajes sumen 100% («include»).
- **Flujo alterno:** los porcentajes no suman 100%: el sistema muestra cuánto falta o cuánto sobra, y no deja publicar la evaluación.
- **Resultado:** el curso queda con su plan de evaluación.

#### CU-07 Registrar calificaciones
- **Actor:** Docente.
- **Objetivo:** registrar o corregir la nota de un estudiante en una actividad.
- **Precondición:** la matrícula está ACTIVA y la actividad pertenece al curso del docente.
- **Flujo principal:**
  1. El Docente elige el curso y la actividad, y escribe las notas.
  2. El sistema valida que cada nota esté entre 0.0 y 5.0.
  3. El sistema guarda la nota: la crea si no existía o la actualiza si ya existía.
  4. El sistema registra el cambio en la auditoría («include»), con el valor anterior, el valor nuevo y el usuario.
  5. El sistema recalcula la nota final de la matrícula («include»).
- **Flujos alternos:**
  - La nota está fuera de rango: el sistema responde con un error y no guarda nada.
  - La matrícula no está ACTIVA: el sistema responde con un error.
- **Resultado:** la nota y la nota final quedan actualizadas. El detalle de este flujo está en el diagrama de secuencia.

#### CU-08 Tomar asistencia
- **Actor:** Docente.
- **Objetivo:** registrar la asistencia de cada sesión.
- **Flujo principal:**
  1. El Docente elige el curso y la fecha de la sesión.
  2. Marca a cada estudiante como PRESENTE, AUSENTE, TARDE o EXCUSA.
  3. El sistema guarda los registros.
- **Flujo alterno:** ya existe asistencia para esa matrícula y esa fecha: el sistema actualiza el registro en vez de duplicarlo.
- **Resultado:** la asistencia queda disponible para las consultas y para el cálculo de riesgo.

#### CU-09 Consultar notas, asistencia y horario
- **Actores:** Estudiante y Acudiente.
- **Objetivo:** conocer la situación académica.
- **Flujo principal:**
  1. El usuario abre su panel.
  2. El sistema muestra los cursos, las notas por actividad, la nota final parcial, el porcentaje de asistencia, el horario y el nivel de riesgo con su color.
- **Extensión («extend»), Simular nota necesaria:** el usuario consulta qué nota necesita en las actividades que faltan para llegar a 3.0 (RF-14).
- **Restricción:** el Estudiante solo ve sus propios datos. El Acudiente solo ve los de los estudiantes que tiene asociados.

#### CU-10 Generar boletín y reportes
- **Actores:** Administrador y Docente.
- **Objetivo:** obtener información consolidada.
- **Flujo principal:**
  1. El usuario elige el tipo de reporte: boletín de un estudiante en PDF, promedio por curso, o estudiantes en riesgo.
  2. El sistema genera el reporte.
- **Restricción:** el Docente solo puede generar reportes de sus propios cursos.

#### CU-11 Calcular riesgo académico y enviar alertas
- **Actor:** Planificador (sistema).
- **Objetivo:** detectar a tiempo a los estudiantes en riesgo.
- **Disparador:** todas las noches, a una hora configurada.
- **Flujo principal:**
  1. El Planificador recorre las matrículas ACTIVAS de los periodos abiertos.
  2. Calcula el nivel de riesgo de cada una (RF-08).
  3. Si el nivel subió, por ejemplo de BAJO a MEDIO o de MEDIO a ALTO, envía un correo al estudiante y al docente.
- **Resultado:** cada matrícula tiene su nivel de riesgo actualizado.

## 4. Requerimientos funcionales

| Código | Requerimiento | CU |
|---|---|---|
| RF-01 | El sistema autentica usuarios y restringe cada pantalla y cada operación de la API según su rol (ver la matriz de permisos). | CU-01 |
| RF-02 | El administrador puede crear, editar, buscar e inactivar estudiantes, sin borrarlos físicamente. | CU-02 |
| RF-03 | No se permite matricular si el curso no tiene cupo o si faltan prerrequisitos. | CU-05 |
| RF-04 | Los porcentajes de las actividades de un curso deben sumar 100%. | CU-06 |
| RF-05 | La nota final se calcula sola, en escala de 0.0 a 5.0, con 3.0 como nota aprobatoria. Fórmula: nota final = Σ (valor × porcentaje). | CU-07 |
| RF-06 | El docente registra la asistencia por sesión: presente, ausente, tarde o excusa. | CU-08 |
| RF-07 | El sistema genera el boletín del estudiante en PDF. | CU-10 |
| RF-08 | El sistema marca en riesgo a un estudiante y notifica al docente y al estudiante. El nivel es **ALTO** si el promedio proyectado es menor que 3.0 o la inasistencia supera el 20%; **MEDIO** si el promedio proyectado está entre 3.0 y 3.5, o si las dos últimas notas bajan; **BAJO** en los demás casos. | CU-11 |
| RF-09 | Toda modificación de notas queda en un historial de auditoría con el valor anterior, el valor nuevo, el usuario y la fecha. | CU-07 |
| RF-10 | Solo el Administrador crea, edita e inactiva cuentas de usuario. No hay autorregistro. | CU-02, CU-03 |
| RF-11 | El administrador puede crear, editar y consultar docentes y cursos, incluyendo el cupo, el horario y los prerrequisitos de cada curso. | CU-03 |
| RF-12 | El administrador puede abrir y cerrar periodos. Al cerrar uno, las matrículas activas pasan a APROBADA o REPROBADA según la nota final. | CU-04 |
| RF-13 | En un periodo CERRADO no se permiten matrículas, ni cambios de notas o de asistencia. | CU-04, CU-05, CU-07, CU-08 |
| RF-14 | El simulador calcula la nota necesaria en las actividades que faltan: (3.0 − Σ valor × porcentaje de las ya calificadas) ÷ porcentaje restante. Si el resultado es mayor que 5.0, informa que ya no es posible aprobar. | CU-09 |
| RF-15 | El estudiante y el acudiente ven las notas, la asistencia, el horario y el nivel de riesgo, solo de los estudiantes que les corresponden. | CU-09 |
| RF-16 | El sistema genera reportes de promedio por curso y la lista de estudiantes en riesgo. | CU-10 |
| RF-17 | El promedio proyectado es la nota acumulada dividida por el porcentaje ya evaluado. La inasistencia es el porcentaje de sesiones marcadas como AUSENTE. | CU-11 |

## 5. Requerimientos no funcionales

| Código | Tipo | Requerimiento |
|---|---|---|
| RNF-01 | Seguridad | Contraseñas cifradas con BCrypt y autenticación con JWT. |
| RNF-02 | Rendimiento | Las consultas comunes responden en menos de 2 segundos con hasta 5.000 estudiantes registrados. |
| RNF-03 | Usabilidad | Interfaz responsive que funcione en celular. |
| RNF-04 | Mantenibilidad | Arquitectura en capas y al menos 70% de cobertura de pruebas en la capa de servicios. |
| RNF-05 | Portabilidad | El sistema se levanta con un solo comando usando Docker Compose. |
| RNF-06 | Documentación | La API queda documentada y se puede probar desde Swagger. |
| RNF-07 | Integridad | Las reglas de datos se validan en el backend, no solo en la pantalla: una sola matrícula activa por estudiante y curso, una sola nota por matrícula y actividad, notas entre 0.0 y 5.0, y porcentajes que suman 100%. |
| RNF-08 | Compatibilidad | Funciona en las dos últimas versiones de Chrome, Edge y Firefox. |
| RNF-09 | Confiabilidad | Los errores se muestran al usuario con mensajes claros, nunca con trazas técnicas. La base de datos tiene copia de seguridad diaria. |

## 6. Diagramas

Los diagramas UML están en `docs/uml/`:

| Diagrama | Archivo |
|---|---|
| Casos de uso | [01-casos-de-uso.png](uml/01-casos-de-uso.png) |
| Clases | [02-clases.png](uml/02-clases.png) |
| Secuencia de CU-07 | [03-secuencia-cu07.png](uml/03-secuencia-cu07.png) |
| Arquitectura | [04-arquitectura.png](uml/04-arquitectura.png) |

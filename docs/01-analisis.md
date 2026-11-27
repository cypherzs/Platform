# Análisis del sistema

## 1. Introducción

Platform es un sistema de gestión académica para **colegios** de preescolar, básica y media. Este documento define qué debe hacer el sistema antes de programarlo: quiénes lo usan, qué puede hacer cada uno, los casos de uso y los requerimientos funcionales y no funcionales. Es la base de los diagramas UML (`docs/uml/`) y de la arquitectura (`03-arquitectura.md`).

Cada requisito tiene un código único (CU-xx, RF-xx, RNF-xx). Los demás documentos se refieren a ellos por ese código, para que cada regla se defina en un solo lugar.

### 1.1 Alcance

- **Incluye:** la gestión académica del colegio: estudiantes, acudientes, docentes, asignaturas, grupos, periodos, matrícula en grupo, calificaciones, asistencia, boletines, promoción y alertas de riesgo.
- **No reemplaza a SIMAT.** SIMAT (Sistema Integrado de Matrícula del Ministerio de Educación) es el registro oficial de qué estudiante tiene cupo en qué colegio, grado y jornada. Platform toma esos datos, importando el reporte de matriculados en Excel o CSV, y agrega lo que SIMAT no maneja: notas, asistencia, boletines y alertas.
- **No incluye en esta versión:** pagos, aplicación móvil nativa, conexión automática con SIMAT (que no ofrece una API pública) ni asistente con IA.

### 1.2 Conceptos del dominio

| Concepto | Significado en Platform |
|---|---|
| Año lectivo | Año escolar (por ejemplo, 2027). Se divide en periodos. |
| Periodo | Parte del año con su porcentaje de la nota final. Por defecto, 4 periodos de 25%. |
| Grado | Nivel escolar: transición, 1° a 11°. |
| Grupo | Curso concreto de un grado en un año, como 6A, con jornada, cupo y director de grupo. |
| Asignatura | Materia del plan de estudios (Matemáticas, Lengua Castellana…), con su área. |
| Clase | Carga académica: una asignatura dictada por un docente a un grupo, con su intensidad horaria y horario. |
| Matrícula | Inscripción de un estudiante en un grupo durante un año lectivo. |
| Desempeño | Escala nacional del Decreto 1290 de 2009: Superior, Alto, Básico y Bajo. Cada colegio fija los rangos en su SIEE. |
| SIEE | Sistema Institucional de Evaluación de los Estudiantes: las reglas de evaluación y promoción de cada colegio. |

## 2. Actores

| Actor | Tipo | Descripción |
|---|---|---|
| Administrador académico | Humano | Coordinación o secretaría académica. Gestiona estudiantes, acudientes, docentes, asignaturas, grupos, periodos, matrículas y cuentas de usuario. |
| Docente | Humano | Define las actividades evaluativas de sus clases y registra notas y asistencia. Puede ser director de grupo. |
| Estudiante | Humano | Consulta su información académica y sus alertas. La cuenta es opcional en los grados más pequeños. |
| Acudiente | Humano | Padre, madre o responsable del estudiante. Consulta su información académica y recibe las alertas. |
| Planificador | Sistema | Tarea automática que calcula el riesgo académico cada noche y envía alertas. |

Todos los actores humanos son **usuarios** del sistema: entran con correo y contraseña, y lo que ven depende de su rol. **No existe el autorregistro**: el Administrador crea todas las cuentas.

### 2.1 Matriz de permisos

✔ = puede hacerlo · **propio** = solo sobre sus clases o grupos, o sobre sus propios datos · — = no tiene acceso

| Operación | Administrador | Docente | Estudiante | Acudiente |
|---|:-:|:-:|:-:|:-:|
| Iniciar y cerrar sesión | ✔ | ✔ | ✔ | ✔ |
| Crear, editar e inactivar cuentas de usuario | ✔ | — | — | — |
| Gestionar estudiantes y acudientes | ✔ | — | — | — |
| Gestionar docentes, asignaturas y grupos | ✔ | — | — | — |
| Abrir y cerrar periodos y año lectivo | ✔ | — | — | — |
| Matricular estudiantes e importar desde SIMAT | ✔ | — | — | — |
| Definir actividades y porcentajes | — | propio | — | — |
| Registrar y modificar calificaciones | — | propio | — | — |
| Tomar asistencia | — | propio | — | — |
| Consultar notas, asistencia y horario | ✔ | propio | propio | propio |
| Usar el simulador de nota necesaria | — | — | propio | propio |
| Ver alertas de riesgo | ✔ | propio | propio | propio |
| Generar boletín y reportes | ✔ | propio | — | — |
| Consultar el historial de auditoría | ✔ | — | — | — |

Para el Docente, **propio** son sus clases y, si es director de grupo, todo su grupo. Para el Acudiente, son los estudiantes que tiene asociados.

## 3. Casos de uso

| Código | Caso de uso | Actor principal |
|---|---|---|
| CU-01 | Iniciar sesión y cerrar sesión | Todos |
| CU-02 | Gestionar estudiantes y acudientes | Administrador |
| CU-03 | Gestionar docentes, asignaturas y grupos | Administrador |
| CU-04 | Abrir y cerrar periodos y año lectivo | Administrador |
| CU-05 | Matricular estudiante en grupo | Administrador |
| CU-06 | Definir actividades evaluativas y sus porcentajes | Docente |
| CU-07 | Registrar calificaciones | Docente |
| CU-08 | Tomar asistencia | Docente |
| CU-09 | Consultar notas, asistencia y horario | Estudiante, Acudiente |
| CU-10 | Generar boletín de periodo y reportes | Administrador, Docente |
| CU-11 | Calcular riesgo académico y enviar alertas | Sistema (planificador) |

El diagrama de secuencia se hace sobre **CU-07 Registrar calificaciones**, porque recorre todas las capas: frontend, controlador, servicio, repositorio y el cálculo de la nota del periodo.

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

#### CU-02 Gestionar estudiantes y acudientes
- **Actor:** Administrador.
- **Objetivo:** mantener al día los datos de los estudiantes y de sus acudientes.
- **Flujo principal:**
  1. El Administrador busca un estudiante por tipo y número de documento, o por nombre.
  2. Crea o edita el estudiante: tipo de documento (RC, TI, CC, CE, PPT), documento, nombres, apellidos y fecha de nacimiento.
  3. Asocia uno o más acudientes, con su parentesco y teléfono, y si se necesita crea sus cuentas.
  4. Si el estudiante deja el colegio, lo marca como RETIRADO.
- **Flujos alternos:**
  - El tipo y número de documento ya existen: el sistema rechaza el registro.
  - El estudiante queda sin acudiente: el sistema no deja guardar, porque todo estudiante debe tener al menos uno.
- **Resultado:** los datos quedan guardados. **Retirar no borra**: el historial académico se conserva.

#### CU-03 Gestionar docentes, asignaturas y grupos
- **Actor:** Administrador.
- **Objetivo:** organizar el año: quién enseña qué, y a qué grupo.
- **Flujo principal:**
  1. El Administrador registra a los docentes, y el sistema crea sus cuentas.
  2. Registra las asignaturas del plan de estudios con su área.
  3. Crea los grupos del año: grado, nombre (6A), jornada, cupo máximo y director de grupo.
  4. Arma la carga académica de cada grupo: para cada asignatura, el docente, la intensidad horaria semanal y el horario.
- **Flujos alternos:**
  - Ya existe un grupo con ese nombre en el mismo año: el sistema lo rechaza.
  - Un docente queda con dos clases a la misma hora: el sistema avisa del cruce de horario.
- **Resultado:** cada grupo queda con su carga académica completa.

#### CU-04 Abrir y cerrar periodos y año lectivo
- **Actor:** Administrador.
- **Objetivo:** controlar en qué periodo se registran notas, y cerrar el año.
- **Flujo principal:**
  1. El Administrador crea los periodos del año, con su número, porcentaje y fechas. Los porcentajes deben sumar 100%.
  2. Abre el periodo actual. Al terminar, lo cierra.
  3. Al cerrar el último periodo, el sistema cierra el año y calcula la promoción («include», RF-13): cada matrícula pasa a PROMOVIDA o NO_PROMOVIDA.
- **Flujo alterno:** los porcentajes de los periodos no suman 100%, o las fechas se cruzan: el sistema no deja guardarlos.
- **Resultado:** un periodo CERRADO ya no admite cambios de notas ni de asistencia.

#### CU-05 Matricular estudiante en grupo
- **Actor:** Administrador.
- **Objetivo:** ubicar a cada estudiante en su grupo del año.
- **Precondición:** el estudiante está ACTIVO y el grupo pertenece al año lectivo en curso.
- **Flujo principal:**
  1. El Administrador elige al estudiante y el grupo.
  2. El sistema valida el cupo del grupo («include») y que el estudiante no tenga otra matrícula activa ese año.
  3. El sistema crea la matrícula en estado ACTIVA.
- **Extensión («extend»), Importar desde SIMAT:** el Administrador carga el reporte de matriculados de SIMAT en Excel o CSV. El sistema crea o actualiza cada estudiante por su tipo y número de documento, lo matricula en el grupo indicado y muestra un resumen: creados, actualizados y filas con error.
- **Flujos alternos:**
  - El grupo no tiene cupo: el sistema rechaza la matrícula.
  - El estudiante ya tiene una matrícula activa ese año: el sistema la rechaza. Para cambiarlo de grupo, primero se retira la matrícula anterior.
- **Resultado:** el estudiante aparece en las listas de todas las clases de su grupo.

#### CU-06 Definir actividades evaluativas
- **Actor:** Docente.
- **Objetivo:** definir cómo se evalúa cada clase en cada periodo.
- **Flujo principal:**
  1. El Docente elige la clase y el periodo.
  2. Crea las actividades (por ejemplo, Taller 1, Quiz, Evaluación de periodo), cada una con su porcentaje y fecha.
  3. El sistema valida que los porcentajes sumen 100% («include»).
- **Flujo alterno:** los porcentajes no suman 100%: el sistema muestra cuánto falta o cuánto sobra, y no deja publicar la evaluación.
- **Resultado:** la clase queda con su plan de evaluación del periodo.

#### CU-07 Registrar calificaciones
- **Actor:** Docente.
- **Objetivo:** registrar o corregir la nota de un estudiante en una actividad.
- **Precondición:** la matrícula está ACTIVA, la actividad pertenece a una clase del docente y su periodo está ABIERTO.
- **Flujo principal:**
  1. El Docente elige la clase y la actividad, y escribe las notas.
  2. El sistema valida que cada nota esté entre 1.0 y 5.0.
  3. El sistema guarda la nota: la crea si no existía o la actualiza si ya existía.
  4. El sistema registra el cambio en la auditoría («include»), con el registro modificado, el valor anterior, el valor nuevo y el usuario.
  5. El sistema calcula la nota del periodo en esa asignatura y su desempeño («include»).
- **Flujos alternos:**
  - La nota está fuera de rango: el sistema responde con un error y no guarda nada.
  - La matrícula no está ACTIVA o el periodo está CERRADO: el sistema responde con un error.
- **Resultado:** la nota queda guardada, y la nota del periodo y el desempeño quedan actualizados. El detalle de este flujo está en el diagrama de secuencia.

#### CU-08 Tomar asistencia
- **Actor:** Docente.
- **Objetivo:** registrar la asistencia de cada sesión de clase.
- **Flujo principal:**
  1. El Docente elige la clase y la fecha.
  2. Marca a cada estudiante como PRESENTE, AUSENTE, TARDE o EXCUSA.
  3. El sistema guarda los registros.
- **Flujo alterno:** ya existe asistencia para esa matrícula, clase y fecha: el sistema actualiza el registro en vez de duplicarlo.
- **Resultado:** la asistencia queda disponible para las consultas, el riesgo y la promoción.

#### CU-09 Consultar notas, asistencia y horario
- **Actores:** Estudiante y Acudiente.
- **Objetivo:** conocer la situación académica.
- **Flujo principal:**
  1. El usuario abre su panel. Si es acudiente de varios estudiantes, elige uno.
  2. El sistema muestra, por asignatura, las notas por actividad, la nota y el desempeño de cada periodo, el porcentaje de asistencia y el nivel de riesgo con su color. También muestra el horario del grupo.
- **Extensión («extend»), Simular nota necesaria:** el usuario consulta qué nota necesita en lo que falta del año para terminar la asignatura con 3.0 (RF-15).
- **Restricción:** el Estudiante solo ve sus propios datos. El Acudiente solo ve los de los estudiantes que tiene asociados.

#### CU-10 Generar boletín de periodo y reportes
- **Actores:** Administrador y Docente.
- **Objetivo:** entregar a las familias el boletín de cada periodo, y obtener información consolidada.
- **Flujo principal:**
  1. El usuario elige el tipo de reporte: boletín de periodo en PDF (por estudiante o de todo un grupo), consolidado de notas del grupo, o estudiantes en riesgo.
  2. El sistema genera el reporte.
- **Contenido del boletín:** por cada asignatura, la nota y el desempeño del periodo, el acumulado del año y las inasistencias.
- **Restricción:** el Docente solo genera reportes de sus clases, o de su grupo si es director de grupo.

#### CU-11 Calcular riesgo académico y enviar alertas
- **Actor:** Planificador (sistema).
- **Objetivo:** detectar a tiempo a los estudiantes en riesgo de perder asignaturas o el año.
- **Disparador:** todas las noches, a una hora configurada.
- **Flujo principal:**
  1. El Planificador recorre las matrículas ACTIVAS del año en curso.
  2. Calcula el nivel de riesgo de cada una (RF-12).
  3. Si el nivel subió, por ejemplo de BAJO a MEDIO, envía un correo a los acudientes y al director de grupo.
- **Resultado:** cada matrícula tiene su nivel de riesgo actualizado.

## 4. Requerimientos funcionales

Los valores marcados como **configurables** son los que propone el sistema por defecto. Cada colegio los ajusta según su SIEE.

| Código | Requerimiento | CU |
|---|---|---|
| RF-01 | El sistema autentica usuarios y restringe cada pantalla y cada operación de la API según su rol (ver la matriz de permisos). | CU-01 |
| RF-02 | El administrador puede crear, editar, buscar y retirar estudiantes, sin borrarlos físicamente. Cada estudiante tiene al menos un acudiente. | CU-02 |
| RF-03 | No se permite matricular si el grupo no tiene cupo, o si el estudiante ya tiene una matrícula activa en el mismo año. | CU-05 |
| RF-04 | Los porcentajes de las actividades de una clase en un periodo deben sumar 100%. Los porcentajes de los periodos de un año también. | CU-04, CU-06 |
| RF-05 | Las notas van de 1.0 a 5.0. La nota del periodo en una asignatura es Σ (nota × porcentaje de la actividad). La nota final de la asignatura es Σ (nota del periodo × porcentaje del periodo). La nota aprobatoria es 3.0 (configurable). | CU-07 |
| RF-06 | El docente registra la asistencia por clase y fecha: presente, ausente, tarde o excusa. | CU-08 |
| RF-07 | El sistema genera el boletín de periodo en PDF, por estudiante o por grupo. | CU-10 |
| RF-08 | Cada nota de periodo y nota final se acompaña de su desempeño en la escala nacional: Superior (4.6 a 5.0), Alto (4.0 a 4.5), Básico (3.0 a 3.9) y Bajo (1.0 a 2.9). Rangos configurables. | CU-07, CU-10 |
| RF-09 | Toda modificación de notas queda en un historial de auditoría que indica qué registro se modificó (`entidad` y `registroId`), el valor anterior, el valor nuevo, el usuario y la fecha. | CU-07 |
| RF-10 | Solo el Administrador crea, edita e inactiva cuentas de usuario. No hay autorregistro. | CU-02, CU-03 |
| RF-11 | El administrador puede importar el reporte de matriculados de SIMAT en Excel o CSV. El sistema cruza a cada estudiante por tipo y número de documento, lo crea o actualiza, lo matricula en su grupo y entrega un resumen con las filas que fallaron. | CU-05 |
| RF-12 | El planificador calcula el riesgo de cada matrícula. Es **ALTO** si alguna asignatura va en desempeño Bajo o la inasistencia supera el 20%; **MEDIO** si alguna asignatura está entre 3.0 y 3.5, o si las dos últimas notas de una asignatura bajan; **BAJO** en los demás casos. Umbrales configurables. | CU-11 |
| RF-13 | Al cerrar el año, una matrícula queda NO_PROMOVIDA si el estudiante pierde 3 o más asignaturas (nota final menor que 3.0) o si su inasistencia supera el 25%. Si no, queda PROMOVIDA. Valores configurables. | CU-04 |
| RF-14 | En un periodo CERRADO no se permiten cambios de notas ni de asistencia. | CU-04, CU-07, CU-08 |
| RF-15 | El simulador calcula la nota necesaria en lo que falta del año para terminar la asignatura con la nota aprobatoria. Si el resultado es mayor que 5.0, informa que ya no es posible aprobar. | CU-09 |
| RF-16 | El estudiante y el acudiente ven las notas, la asistencia, el horario y el nivel de riesgo, solo de los estudiantes que les corresponden. | CU-09 |
| RF-17 | El administrador gestiona docentes, asignaturas, grupos y la carga académica de cada grupo. El sistema avisa si un docente queda con dos clases a la misma hora. | CU-03 |
| RF-18 | El sistema genera el consolidado de notas por grupo y la lista de estudiantes en riesgo. | CU-10 |

## 5. Requerimientos no funcionales

| Código | Tipo | Requerimiento |
|---|---|---|
| RNF-01 | Seguridad | Contraseñas cifradas con BCrypt y autenticación con JWT. |
| RNF-02 | Rendimiento | Las consultas comunes responden en menos de 2 segundos con hasta 3.000 estudiantes matriculados. |
| RNF-03 | Usabilidad | Interfaz responsive que funcione en celular, pensada también para acudientes sin experiencia técnica. |
| RNF-04 | Mantenibilidad | Arquitectura en capas y al menos 70% de cobertura de pruebas en la capa de servicios. |
| RNF-05 | Portabilidad | El sistema se levanta con un solo comando usando Docker Compose. |
| RNF-06 | Documentación | La API queda documentada y se puede probar desde Swagger. |
| RNF-07 | Integridad | Las reglas de datos se validan en el backend, no solo en la pantalla: una sola matrícula activa por estudiante y año, una sola nota por matrícula y actividad, notas entre 1.0 y 5.0, y porcentajes que suman 100%. |
| RNF-08 | Compatibilidad | Funciona en las dos últimas versiones de Chrome, Edge y Firefox. |
| RNF-09 | Confiabilidad | Los errores se muestran al usuario con mensajes claros, nunca con trazas técnicas. La base de datos tiene copia de seguridad diaria. |
| RNF-10 | Privacidad | Los datos son de menores de edad: el sistema solo muestra a cada usuario lo que su rol permite, y el tratamiento de datos personales sigue la Ley 1581 de 2012. |

## 6. Diagramas

Los diagramas UML están en `docs/uml/`:

| Diagrama | Archivo |
|---|---|
| Casos de uso | [01-casos-de-uso.png](uml/01-casos-de-uso.png) |
| Clases | [02-clases.png](uml/02-clases.png) |
| Secuencia de CU-07 | [03-secuencia-cu07.png](uml/03-secuencia-cu07.png) |
| Arquitectura | [04-arquitectura.png](uml/04-arquitectura.png) |

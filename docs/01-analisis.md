# Análisis del sistema

## Actores

| Actor | Tipo | Descripción |
|---|---|---|
| Administrador académico | Humano | Gestiona estudiantes, docentes, cursos, periodos y matrículas |
| Docente | Humano | Registra actividades, notas y asistencia de sus cursos |
| Estudiante | Humano | Consulta su información académica y sus alertas |
| Acudiente | Humano | Consulta el avance de su estudiante |
| Planificador | Sistema | Calcula el riesgo académico cada noche y envía alertas |

## Casos de uso

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

El diagrama de secuencia se hará sobre **CU-07 Registrar calificaciones**, porque recorre todas las capas: frontend, controlador, servicio, repositorio y el recálculo de la nota final.

## Requerimientos funcionales

| Código | Requerimiento |
|---|---|
| RF-01 | El sistema autentica usuarios y restringe cada pantalla según su rol. |
| RF-02 | El administrador puede crear, editar, buscar e inactivar estudiantes, sin borrarlos físicamente. |
| RF-03 | No se permite matricular si el curso no tiene cupo o faltan prerrequisitos. |
| RF-04 | Los porcentajes de las actividades de un curso deben sumar 100%. |
| RF-05 | La nota final se calcula sola en escala 0.0 a 5.0, con 3.0 como nota aprobatoria. |
| RF-06 | El docente registra asistencia por sesión: presente, ausente, tarde o excusa. |
| RF-07 | El sistema genera el boletín del estudiante en PDF. |
| RF-08 | El sistema marca en riesgo a un estudiante y notifica al docente y al estudiante. |
| RF-09 | Toda modificación de notas queda en un historial de auditoría. |

## Requerimientos no funcionales

| Código | Tipo | Requerimiento |
|---|---|---|
| RNF-01 | Seguridad | Contraseñas cifradas con BCrypt y autenticación con JWT. |
| RNF-02 | Rendimiento | Las consultas comunes responden en menos de 2 segundos. |
| RNF-03 | Usabilidad | Interfaz responsive que funcione en celular. |
| RNF-04 | Mantenibilidad | Arquitectura en capas y al menos 70% de cobertura de pruebas en la capa de servicios. |
| RNF-05 | Portabilidad | El sistema se levanta con un solo comando usando Docker Compose. |
| RNF-06 | Documentación | La API queda documentada y probable en Swagger. |

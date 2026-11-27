# Platform: Sistema de Gestión de Estudiantes

Platform es una aplicación web que centraliza la información académica de un colegio (preescolar, básica y media): estudiantes, acudientes, docentes, grupos, matrículas, calificaciones y asistencia. Además, analiza las notas y la asistencia para detectar a tiempo a los estudiantes en riesgo de perder una asignatura o el año.

## Que resuelve?

En muchos colegios las notas, la asistencia y las matrículas se manejan en hojas de cálculo o en papel, por separado. Esto provoca errores al calcular promedios, información duplicada y que los casos de bajo rendimiento se descubran cuando ya es tarde.

## Usuarios

- **Administrador académico:** gestiona estudiantes, acudientes, docentes, asignaturas, grupos, periodos y matrículas.
- **Docente:** registra calificaciones y asistencia.
- **Estudiante:** consulta sus notas, asistencia y alertas.
- **Acudiente:** consulta notas, asistencia y alertas de su estudiante, y recibe los avisos de riesgo.

## Funcionalidades principales

- Inicio de sesión con roles
- Gestión de estudiantes, acudientes, docentes, asignaturas, grupos y periodos
- Matrícula en grupo con validación de cupo e importación del reporte de SIMAT
- Registro de calificaciones con cálculo automático de la nota del periodo y el desempeño (Superior, Alto, Básico, Bajo)
- Control de asistencia
- Alerta temprana de riesgo académico
- Simulador "¿qué nota necesito para aprobar?"
- Boletín de periodo en PDF, reportes y cálculo de promoción al cierre del año

## Tecnologías

| Capa | Tecnología |
| --- | --- |
| Backend | Java 21, Spring Boot 3, Maven |
| Frontend | React, Vite |
| Base de datos | PostgreSQL (H2 para pruebas) |
| Pruebas | JUnit 5, Mockito |
| CI/CD | GitHub Actions |

## Estructura del repositorio

```
├── backend/    # API REST en Spring Boot
├── frontend/   # Aplicación web en React
├── docs/       # Análisis, diagramas UML y documentos del proceso
└── README.md
```

## Ramas

- `main`: versión estable; cada versión publicada lleva su etiqueta (`v0.1.0`, `v0.2.0`…).
- `develop`: integración del trabajo del equipo.
- `feature/<nombre>`: una rama por funcionalidad, que se integra a `develop` mediante pull request.

## Documentación

| Documento | Contenido |
| --- | --- |
| [Análisis del sistema](docs/01-analisis.md) | Actores, permisos, casos de uso, requerimientos funcionales y no funcionales |
| [Flujo de desarrollo](docs/02-flujo-desarrollo.md) | Estrategia de ramas, commits, pull requests y versiones |
| [Arquitectura](docs/03-arquitectura.md) | Estilo de arquitectura, componentes, seguridad, manejo de errores y despliegue |
| [Diagramas UML](docs/uml/) | Casos de uso, clases, secuencia de CU-07 y arquitectura |



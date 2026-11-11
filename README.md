# Platform: Sistema de Gestión de Estudiantes

Platform es una aplicación web que centraliza la información académica de una institución educativa ya sea de estudiantes, docentes, cursos, matrículas, calificaciones y asistencia. Además, analiza las notas y la asistencia para detectar a tiempo a los estudiantes en riesgo de perder una materia.

## Que resuelve?

En muchas instituciones las notas, la asistencia y las matrículas se manejan en hojas de cálculo o en papel, por separado. Esto provoca errores al calcular promedios, información duplicada y que los casos de bajo rendimiento se descubran cuando ya es tarde.

## Usuarios

- **Administrador académico:** gestiona estudiantes, docentes, cursos, periodos y matrículas.
- **Docente:** registra calificaciones y asistencia.
- **Estudiante:** consulta sus notas, asistencia y alertas.
- **Acudiente (Opcional):** consulta notas, asistencia y alertas de su estudiante.

## Funcionalidades principales

- Inicio de sesión con roles
- Gestión de estudiantes, docentes, cursos y periodos
- Matrícula con validación de cupos y prerrequisitos
- Registro de calificaciones con cálculo automático de la nota final
- Control de asistencia
- Alerta temprana de riesgo académico
- Simulador "¿qué nota necesito para aprobar?"
- Boletín en PDF y reportes

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

- `main`: versión estable de cada entrega.
- `develop`: integración del trabajo del equipo.
- `feature/<nombre>`: una rama por funcionalidad, que se integra a `develop` mediante pull request.



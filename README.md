#  Manual de LINQ — Proyecto GestionUsuarios

Manual completo basado en el proyecto de clase **GestionUsuariosV2**, un sistema de gestión de pacientes construido en C# con arquitectura en capas.

---

##  Estructura del Manual

```
manual-linq/
├── README.md                          ← Este archivo (índice principal)
├── 01-arquitectura-capas/
│   └── README.md                      ← ¿Qué es la arquitectura en capas?
├── 02-entidades/
│   └── README.md                      ← Capa de Entidades (modelos)
├── 03-capa-datos-sql/
│   └── README.md                      ← Acceso a datos con ADO.NET puro
├── 04-capa-datos-linq/
│   └── README.md                      ← Acceso a datos con LINQ to SQL
├── 05-capa-datos-ef/
│   └── README.md                      ← Acceso a datos con Entity Framework
├── 06-logica-negocio/
│   └── README.md                      ← Capa de Lógica de Negocio
├── 07-presentacion/
│   └── README.md                      ← Capa de Presentación (WinForms)
└── 08-linq-consultas-avanzadas/
    └── README.md                      ← Consultas LINQ avanzadas aplicadas al proyecto
```

---

##  Arquitectura del Proyecto

El proyecto usa **5 capas** claramente separadas:

```
GestionUsuarios.slnx
│
├── GestionUsuariosEntidades          ← Capa 1: Modelos / DTOs
├── GestionUsuariosDatos              ← Capa 2a: Datos con ADO.NET (SQL puro)
├── GestionUsuariosDatosLINQ          ← Capa 2b: Datos con LINQ to SQL
├── GestionUsuariosDatosEF            ← Capa 2c: Datos con Entity Framework
├── GestionUsuariosLogicaNegocio      ← Capa 3: Reglas de negocio
└── GestionUsuariosPresentacion       ← Capa 4: Interfaz de usuario (WinForms)
```

---

##  Índice de Temas

| Capítulo | Tema | Archivo |
|----------|------|---------|
| 01 | Arquitectura en capas | [Ver →](./01-arquitectura-capas/README.md) |
| 02 | Capa de Entidades | [Ver →](./02-entidades/README.md) |
| 03 | Capa de Datos con ADO.NET | [Ver →](./03-capa-datos-sql/README.md) |
| 04 | Capa de Datos con LINQ to SQL | [Ver →](./04-capa-datos-linq/README.md) |
| 05 | Capa de Datos con Entity Framework | [Ver →](./05-capa-datos-ef/README.md) |
| 06 | Lógica de Negocio | [Ver →](./06-logica-negocio/README.md) |
| 07 | Capa de Presentación | [Ver →](./07-presentacion/README.md) |
| 08 | Consultas LINQ Avanzadas | [Ver →](./08-linq-consultas-avanzadas/README.md) |
| 09 | Comparativa: ADO.NET vs LINQ vs EF | [Ver →](./09-comparativa-tecnologias/README.md) |
| 10 | Errores Comunes y Soluciones | [Ver →](./10-errores-comunes/README.md) |
| 11 | Ejercicios Resueltos | [Ver →](./11-ejercicios-resueltos/README.md) |
| 12 | Git Workflow para el Proyecto | [Ver →](./12-git-workflow/README.md) |


---

*Manual LINQ basado en el proyecto GestionUsuariosV2 comparando con las diferentes tecnologías de manipulacion de datos en la capa de datos — Programación Avanzada*

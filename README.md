# Manual Integral de LINQ en C# — Proyecto GestionUsuariosV2

Bienvenido a este manual completo y práctico sobre **LINQ (Language Integrated Query)** en C#, aplicado a un entorno de desarrollo real mediante arquitectura en capas.

---

## ¿Qué es LINQ y por qué aprenderlo?

**LINQ** es una característica fundamental de .NET que permite realizar consultas a diferentes fuentes de datos utilizando una sintaxis uniforme, declarativa e integrada directamente en el lenguaje C#. Gracias a LINQ, las consultas pasan a formar parte del propio lenguaje, permitiendo escribir código más legible, mantenible y validado en tiempo de compilación.

Con LINQ es posible consultar diferentes fuentes de datos, entre ellas:

* Colecciones en memoria (`List<T>`)
* Bases de datos SQL Server
* Entity Framework
* Archivos XML
* Datos en formato JSON
* Servicios externos
* Cualquier origen compatible con `IEnumerable` o `IQueryable`

**Por ejemplo:**

```csharp
var pacientesMayores = pacientes
    .Where(p => p.Edad >= 18)
    .OrderBy(p => p.Nombre);

```

Este enfoque permite reemplazar estructuras repetitivas y consultas SQL escritas manualmente por código más limpio, expresivo y fácil de mantener.

Aprender LINQ transforma la forma en que interactúas con los datos: pasas de escribir bucles complejos y consultas SQL en formato de texto (propensas a errores), a escribir código fuertemente tipado, limpio, legible y validado en tiempo de compilación.

---

## ¿Por qué basar este manual en un proyecto de clase?

La teoría por sí sola rara vez es suficiente. Este manual no utiliza ejemplos abstractos, sino que está construido directamente sobre **GestionUsuariosV2**, un sistema CRUD real de gestión de pacientes desarrollado en clases.

Aplicar LINQ sobre un proyecto estructurado en capas (Entidades, Datos, Lógica de Negocio y Presentación WinForms) permite entender cómo se comunican los componentes en una aplicación de nivel empresarial y cómo LINQ facilita el flujo de información entre la base de datos y la interfaz gráfica.

---

## El valor de la comparativa: ADO.NET vs LINQ vs Entity Framework

Una de las fortalezas de este manual es la comparación directa entre tres tecnologías de acceso a datos dentro de la misma arquitectura:

### ADO.NET Puro

Representa el acceso tradicional a bases de datos mediante componentes clave. Este enfoque permite comprender los fundamentos de la comunicación entre una aplicación .NET y SQL Server.

* `SqlConnection`
* `SqlCommand`
* `SqlDataReader`
* Consultas SQL escritas manualmente

### LINQ to SQL

Introduce el concepto de mapeo objeto-relacional (ORM) y constituye un puente entre ADO.NET tradicional y las tecnologías ORM modernas, permitiendo:

* Mapeo automático de tablas a clases
* Consultas utilizando sintaxis de C#
* Reducción significativa de código repetitivo
* Integración directa con LINQ

### Entity Framework

Es la tecnología ORM más utilizada en el ecosistema .NET y amplía las capacidades de LINQ. Su uso permite desarrollar aplicaciones más mantenibles, escalables y alineadas con las prácticas actuales de la industria del software mediante:

* `DbContext`
* `DbSet`
* Migraciones
* Consultas LINQ avanzadas
* Gestión automática de relaciones entre entidades

> **Conclusión:** Comparar estas tecnologías proporciona criterio técnico para comprender qué herramienta utilizar, cuándo utilizarla y por qué.

## Estructura del Manual

```text
manual-linq/
├── README.md
├── 01-arquitectura-capas/
│   └── README.md
├── 02-entidades/
│   └── README.md
├── 03-capa-datos-sql/
│   └── README.md
├── 04-capa-datos-linq/
│   └── README.md
├── 05-capa-datos-ef/
│   └── README.md
├── 06-logica-negocio/
│   └── README.md
├── 07-presentacion/
│   └── README.md
└── 08-linq-consultas-avanzadas/
    └── README.md
```

---

## Arquitectura del Proyecto

El proyecto utiliza una arquitectura de cinco capas:

```text
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

## Índice de Temas

| Capítulo | Tema | Archivo |
|----------|------|---------|
| 01 | Arquitectura en capas | [Ver →](./manual-linq-PA/01-arquitectura-capas/README.md) |
| 02 | Capa de Entidades | [Ver →](./manual-linq-PA/02-entidades/README.md) |
| 03 | Capa de Datos con ADO.NET | [Ver →](./manual-linq-PA/03-capa-datos-sql/README.md) |
| 04 | Capa de Datos con LINQ to SQL | [Ver →](./manual-linq-PA/04-capa-datos-linq/README.md) |
| 05 | Capa de Datos con Entity Framework | [Ver →](./manual-linq-PA/05-capa-datos-ef/README.md) |
| 06 | Lógica de Negocio | [Ver →](./manual-linq-PA/06-logica-negocio/README.md) |
| 07 | Capa de Presentación | [Ver →](./manual-linq-PA/07-presentacion/README.md) |
| 08 | Consultas LINQ Avanzadas | [Ver →](./manual-linq-PA/08-linq-consultas-avanzadas/README.md) |
| 09 | Comparativa: ADO.NET vs LINQ vs EF | [Ver →](./manual-linq-PA/09-comparativa-tecnologias/README.md) |
| 10 | Errores Comunes y Soluciones | [Ver →](./manual-linq-PA/10-errores-comunes/README.md) |
| 11 | Ejercicios Resueltos | [Ver →](./manual-linq-PA/11-ejercicios-resueltos/README.md) |
| 12 | Git Workflow para el Proyecto | [Ver →](./manual-linq-PA/12-git-workflow/README.md) |

---

**Desarrollado por:** Mauro Sebastián Pico Solís

**Estudiante de Ingeniería en Tecnologías de la Información**
Universidad Técnica de Ambato (UTA)
Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI)

*Manual LINQ basado en el proyecto GestionUsuariosV2 comparando diferentes tecnologías de manipulación de datos en la capa de acceso a datos — Programación Avanzada.*

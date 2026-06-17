# Manual Integral de LINQ en C# — Proyecto GestionUsuariosV2

Bienvenido a este manual completo y práctico sobre **LINQ (Language Integrated Query)** en C#, aplicado a un entorno de desarrollo real mediante arquitectura en capas.

---

## ¿Qué es LINQ y por qué aprenderlo?

**LINQ** es una característica fundamental de .NET que permite realizar consultas a diferentes fuentes de datos (colecciones en memoria, bases de datos SQL, archivos XML, etc.) utilizando una sintaxis uniforme, declarativa e integrada directamente en el lenguaje C#. 

Aprender LINQ transforma la forma en que interactúas con los datos: pasas de escribir bucles complejos y consultas SQL en formato de texto (propensas a errores), a escribir código fuertemente tipado, limpio, legible y validado en tiempo de compilación.

## ¿Por qué basar este manual en un proyecto de clase?

La teoría por sí sola rara vez es suficiente. Este manual no utiliza ejemplos abstractos, sino que está construido directamente sobre **GestionUsuariosV2**, un sistema CRUD real de gestión de pacientes desarrollado en clases. 

Aplicar LINQ sobre un proyecto estructurado en capas (Entidades, Datos, Lógica de Negocio y Presentación WinForms) permite entender cómo se comunican los componentes en una aplicación de nivel empresarial y cómo LINQ facilita el flujo de información entre la base de datos y la interfaz gráfica.

## El valor de la comparativa: ADO.NET vs LINQ vs EF

Una de las joyas de este manual es la comparación directa entre tres tecnologías de acceso a datos en la misma arquitectura:
1. **ADO.NET Puro:** Entender la base. Escribir comandos SQL a mano, usar `SqlDataReader` y manejar conexiones manualmente.
2. **LINQ to SQL:** El puente. Ver cómo el mapeo objeto-relacional (ORM) básico nos permite usar sintaxis de C# para filtrar y proyectar datos desde SQL Server.
3. **Entity Framework (EF):** El estándar moderno. Descubrir cómo evolucionó LINQ para manejar contextos completos y migraciones.

Comparar estas tecnologías te da el criterio técnico necesario para saber **qué herramienta usar, cuándo usarla y por qué**, un conocimiento invaluable para cualquier ingeniero de software.

---

## Estructura del Manual

```text
manual-linq/
├── README.md                      ← Este archivo (índice principal)
├── 01-arquitectura-capas/
│   └── README.md                  ← ¿Qué es la arquitectura en capas?
├── 02-entidades/
│   └── README.md                  ← Capa de Entidades (modelos)
├── 03-capa-datos-sql/
│   └── README.md                  ← Acceso a datos con ADO.NET puro
├── 04-capa-datos-linq/
│   └── README.md                  ← Acceso a datos con LINQ to SQL
├── 05-capa-datos-ef/
│   └── README.md                  ← Acceso a datos con Entity Framework
├── 06-logica-negocio/
│   └── README.md                  ← Capa de Lógica de Negocio
├── 07-presentacion/
│   └── README.md                  ← Capa de Presentación (WinForms)
└── 08-linq-consultas-avanzadas/
    └── README.md                  ← Consultas LINQ avanzadas aplicadas
---

##  Arquitectura del Proyecto

El proyecto usa **5 capas** separadas:

GestionUsuarios.slnx
│
├── GestionUsuariosEntidades          ← Capa 1: Modelos / DTOs
├── GestionUsuariosDatos              ← Capa 2a: Datos con ADO.NET (SQL puro)
├── GestionUsuariosDatosLINQ          ← Capa 2b: Datos con LINQ to SQL
├── GestionUsuariosDatosEF            ← Capa 2c: Datos con Entity Framework
├── GestionUsuariosLogicaNegocio      ← Capa 3: Reglas de negocio
└── GestionUsuariosPresentacion       ← Capa 4: Interfaz de usuario (WinForms)

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

*Desarrollado por: Mauro Sebastián Pico Solís

Estudiante de Ingeniería en Tecnologías de la Información Universidad Técnica de Ambato (UTA) - Facultad de Ingeniería en Sistemas, Electrónica e Industrial (FISEI)*

*Manual LINQ basado en el proyecto GestionUsuariosV2 comparando con las diferentes tecnologías de manipulacion de datos en la capa de datos — Programación Avanzada*

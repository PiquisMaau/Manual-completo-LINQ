# 01 — Arquitectura en Capas

## ¿Qué es la arquitectura en capas?

Es un patrón de diseño que **separa las responsabilidades** del sistema en proyectos independientes. Cada capa solo "habla" con la capa que tiene debajo, nunca saltándose niveles.

## Diagrama del proyecto GestionUsuarios

```
┌─────────────────────────────────────────────────┐
│         GestionUsuariosPresentacion              │  ← WinForms (.exe)
│       (Form_Paciente.cs, Form1.cs)               │     El usuario interactúa aquí
└───────────────────┬─────────────────────────────┘
                    │ llama a
┌───────────────────▼─────────────────────────────┐
│        GestionUsuariosLogicaNegocio              │  ← Validaciones y reglas
│   (PacienteNegocio.cs, GeneroNegocio.cs)         │
└───────────────────┬─────────────────────────────┘
                    │ llama a (según configuración)
       ┌────────────┼──────────────┐
       ▼            ▼              ▼
┌────────────┐ ┌──────────┐ ┌───────────────┐
│ Datos SQL  │ │Datos LINQ│ │   Datos EF    │  ← 3 implementaciones del mismo contrato
│(ADO.NET)   │ │(LINQ to  │ │(Entity Frame- │
│            │ │  SQL)    │ │   work 6)     │
└────────────┘ └──────────┘ └───────────────┘
       │              │              │
       └──────────────┴──────────────┘
                      │ acceden a
┌─────────────────────▼───────────────────────────┐
│              SQL Server                          │  ← Base de datos
│    (BD: ProgramacionAvanzada)                    │
└─────────────────────────────────────────────────┘
       ▲
       │ todas las capas usan
┌──────┴──────────────────────────────────────────┐
│         GestionUsuariosEntidades                 │  ← DTOs / Modelos compartidos
│  (PacienteEntidades, GeneroEntidades, ...)       │     ¡Esta capa no depende de nadie!
└─────────────────────────────────────────────────┘
```

## ¿Por qué separar en capas?

| Ventaja | Ejemplo en el proyecto |
|---------|----------------------|
| **Cambiar tecnología sin tocar UI** | Se cambió de `GestionUsuariosDatos` (SQL puro) → `GestionUsuariosDatosLINQ` → `GestionUsuariosDatosEF` sin tocar el Form |
| **Reutilización** | `PacienteNegocio` puede ser llamado desde WinForms, API web o consola |
| **Testabilidad** | Se puede probar la lógica de negocio sin abrir la pantalla |
| **Mantenimiento** | Un bug en el SQL no obliga a tocar la pantalla |

## Cómo está referenciado en el proyecto

```
GestionUsuariosPresentacion
    ↳ Referencia: GestionUsuariosLogicaNegocio
    ↳ Referencia: GestionUsuariosEntidades

GestionUsuariosLogicaNegocio
    ↳ Referencia: GestionUsuariosDatosEF     ← actualmente activo
    ↳ Referencia: GestionUsuariosEntidades
    // Las referencias a GestionUsuariosDatos y GestionUsuariosDatosLINQ
    // están comentadas (se cambia solo aquí para cambiar toda la tecnología)

GestionUsuariosDatosEF
    ↳ Referencia: GestionUsuariosEntidades
    ↳ Paquete NuGet: EntityFramework 6.5.2
```

## Flujo de una operación completa (Guardar Paciente)

```
[Usuario hace clic en "Guardar"]
        ↓
Form_Paciente.cs → btn_Guardar_Click()
        ↓
        Construye objeto PacienteEntidades con los valores del formulario
        ↓
PacienteNegocio.GuardarPaciente(paciente)   ← Lógica de Negocio
        ↓
        Valida: ¿Id == 0? → Nuevo | Id > 0 → Actualizar
        ↓
PacienteDatosEF.Nuevo(paciente)             ← Capa de Datos EF
        ↓
        Mapea PacienteEntidades → Paciente (EF)
        contexto.Paciente.Add(pacienteEF)
        contexto.SaveChanges()
        ↓
        [INSERT ejecutado en SQL Server]
        ↓
        Retorna PacienteEntidades con Id asignado
        ↓
Form_Paciente.cs → txtb_Id.Text = paciente.Id.ToString()
        ↓
[MessageBox: "Los datos se almacenaron correctamente"]
```

---

[← Inicio](../README.md) | [Siguiente: Entidades →](../02-entidades/README.md)

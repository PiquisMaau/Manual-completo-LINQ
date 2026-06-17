# 06 — Capa de Lógica de Negocio

## Proyecto: `GestionUsuariosLogicaNegocio`

Esta capa es el **árbitro** del sistema. Recibe peticiones de la Presentación, aplica las reglas del dominio y delega a la capa de Datos. Nunca interactúa directamente con la interfaz ni con SQL.

>  **Responsabilidad principal:** Validar que los datos sean correctos ANTES de tocar la base de datos.

---

## PacienteNegocio.cs — con TransactionScope

```csharp
// Archivo: GestionUsuariosLogicaNegocio/PacienteNegocio.cs
using GestionUsuariosEntidades;
using GestionUsuariosDatosEF;
using System.Transactions;

public static class PacienteNegocio
{
    public static PacienteEntidades GuardarPaciente(PacienteEntidades paciente)
    {
        // TransactionScope: si algo falla, TODOS los cambios se revierten (rollback)
        using (TransactionScope scope = new TransactionScope())
        {
            PacienteEntidades resultado;
            try
            {
                // Regla de negocio: si Id == 0 es nuevo; si Id > 0 es actualización
                if (paciente.Id == 0)
                {
                    resultado = PacienteDatosEF.Nuevo(paciente);
                }
                else
                {
                    resultado = PacienteDatosEF.Actualizar(paciente);
                }

                scope.Complete(); // Confirma la transacción (commit)
                return resultado;
            }
            catch (Exception)
            {
                // Al salir del using sin Complete(), el TransactionScope hace rollback
                throw;
            }
        }
    }

    public static List<PacienteEntidades> DevolverListaPaciente()
    {
        // La capa de negocio simplemente delega — no hay reglas adicionales aquí
        return PacienteDatosEF.DevolverListaEntidades();
    }

    public static PacienteEntidades CargarPacientePorId(int id)
    {
        return PacienteDatosEF.CargarPacientePorID(id);
    }

    public static bool EliminarPacientePorId(int id)
    {
        return PacienteDatosEF.EliminarPacientePorId(id);
    }
}
```

---

## Paciente_Enfermedades_Negocio.cs — con validaciones reales

Esta es la clase más completa del proyecto en cuanto a reglas de negocio:

```csharp
// Archivo: GestionUsuariosLogicaNegocio/Paciente_Enfermedades_Negocio.cs
using GestionUsuariosDatosEF;
using GestionUsuariosEntidades;

public class Paciente_Enfermedades_Negocio
{
    /// <summary>
    /// Solicita la lista de enfermedades de un paciente.
    /// Regla: El ID del paciente debe ser válido.
    /// </summary>
    public static List<Paciente_Enfermedad_Entidades> CargarDetallePorPaciente(int id_Paciente)
    {
        // Validación antes de ir a la BD
        if (id_Paciente <= 0)
        {
            throw new ArgumentException(
                "El ID del paciente no es válido para buscar su historial.");
        }

        return Paciente_Enfermedades_Datos.CargarDetallePorPaciente(id_Paciente);
    }

    /// <summary>
    /// Valida y registra una nueva enfermedad para un paciente.
    /// </summary>
    public static Paciente_Enfermedad_Entidades Nuevo(Paciente_Enfermedad_Entidades detalle)
    {
        // Regla 1: El objeto no puede ser null
        if (detalle == null)
            throw new ArgumentNullException("El objeto detalle no puede estar vacío.");

        // Regla 2: Debe existir un paciente seleccionado
        if (detalle.Id_Paciente <= 0)
            throw new ArgumentException(
                "Debe existir un paciente seleccionado para registrar una enfermedad.");

        // Regla 3: La enfermedad debe ser válida
        if (detalle.Enfermedad == null || detalle.Enfermedad.id <= 0)
            throw new ArgumentException(
                "Debe seleccionar una enfermedad válida de la lista desplegable.");

        // Regla 4: La fecha no puede ser futura
        if (detalle.FechaEnfermedad > DateTime.Now)
            throw new ArgumentException(
                "La fecha de la enfermedad no puede ser en el futuro.");

        // Si todo está correcto, pasar a la capa de datos
        return Paciente_Enfermedades_Datos.Nuevo(detalle);
    }

    /// <summary>
    /// Valida y actualiza un registro de enfermedad existente.
    /// </summary>
    public static Paciente_Enfermedad_Entidades Actualizar(Paciente_Enfermedad_Entidades detalle)
    {
        if (detalle.Id <= 0)
            throw new ArgumentException(
                "No se puede actualizar un registro que no existe (ID inválido).");

        if (detalle.Enfermedad == null || detalle.Enfermedad.id <= 0)
            throw new ArgumentException("Debe seleccionar una enfermedad válida.");

        return Paciente_Enfermedades_Datos.Actualizar(detalle);
    }

    /// <summary>
    /// Elimina un registro del historial médico tras validar el ID.
    /// </summary>
    public static bool EliminarDetallePorId(int id_Detalle)
    {
        if (id_Detalle <= 0)
            throw new ArgumentException(
                "Debe seleccionar un registro válido de la tabla para eliminarlo.");

        return Paciente_Enfermedades_Datos.EliminarDetallePorId(id_Detalle);
    }
}
```

---

## GeneroNegocio.cs — delegación pura

```csharp
// Archivo: GestionUsuariosLogicaNegocio/GeneroNegocio.cs
using GestionUsuariosEntidades;
using GestionUsuariosDatosEF;

public static class GeneroNegocio
{
    // No hay reglas de negocio para géneros — solo delega
    public static List<GeneroEntidades> DevolverListaGeneros()
    {
        return GeneroDatosEF.DevolverListaGeneros();
    }
}
```

---

## ¿Qué va en la capa de Negocio vs en la de Datos?

| Tipo de lógica | ¿Dónde va? | Ejemplo del proyecto |
|---------------|-----------|---------------------|
| Validaciones de campo | Negocio | `if (detalle.Enfermedad == null)` |
| Reglas de fechas | Negocio | `if (FechaEnfermedad > DateTime.Now)` |
| Decisión INSERT vs UPDATE | Negocio | `if (paciente.Id == 0)` |
| Manejo de transacciones | Negocio | `TransactionScope` |
| Llamar a la BD | Datos | `contexto.Paciente.Add(...)` |
| Mapear objetos EF → Entidades | Datos | `new PacienteEntidades(item.id, ...)` |
| SQL / LINQ queries | Datos | `.Where(p => p.id == id)` |

---

## Cómo cambiar la tecnología de datos sin tocar la presentación

Actualmente el proyecto usa EF. Si quisieras volver a LINQ to SQL:

```csharp
// GestionUsuariosLogicaNegocio/PacienteNegocio.cs

// ANTES (EF):
// using GestionUsuariosDatosEF;
// resultado = PacienteDatosEF.Nuevo(paciente);

// DESPUÉS (LINQ to SQL) — solo cambia estas 2 líneas:
using GestionUsuariosDatosLINQ;
resultado = PacienteDatos.Nuevo(paciente);  // ← misma firma de método
```

La capa de Presentación NO se toca. Este es el poder de la arquitectura en capas.

---

[← Entity Framework](../05-capa-datos-ef/README.md) | [Siguiente: Presentación →](../07-presentacion/README.md)

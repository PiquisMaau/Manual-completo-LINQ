# 02 — Capa de Entidades (GestionUsuariosEntidades)

## ¿Qué es la capa de Entidades?

Son clases **POCO** (Plain Old C# Objects) que representan los datos del sistema. No tienen lógica, no acceden a la base de datos: solo transportan información entre capas. También se les llama **DTOs** (Data Transfer Objects).

>  **Regla clave:** Esta capa NO referencia ninguna otra capa del proyecto. Todos la usan, ella no usa a nadie.

## Entidades del proyecto

### `PacienteEntidades.cs`

```csharp
using System;
using System.Collections.Generic;

namespace GestionUsuariosEntidades
{
    public class PacienteEntidades
    {
        // Constructor vacío (para instanciar un paciente nuevo/en blanco)
        public PacienteEntidades() { }

        // Constructor con mensaje de error (para manejar fallos)
        public PacienteEntidades(string error_Texto)
        {
            Error_Texto = error_Texto;
        }

        // Constructor completo (para mapear datos que vienen de la BD)
        public PacienteEntidades(int id, int idGenero, string genero, string nombre,
            string apellido, string cedula, DateTime fechaNacimiento,
            string telefono, string direccion, bool afiliado, string codigoIEES)
        {
            Id             = id;
            IdGenero       = idGenero;
            Genero         = genero;
            Nombre         = nombre;
            Apellido       = apellido;
            Cedula         = cedula;
            FechaNacimiento= fechaNacimiento;
            Telefono       = telefono;
            Direccion      = direccion;
            Afiliado       = afiliado;
            CodigoIEES     = codigoIEES;
        }

        // Propiedades de datos
        public int      Id              { get; set; }
        public int      IdGenero        { get; set; }
        public string   Genero          { get; set; }   // Nombre del género (JOIN)
        public string   Nombre          { get; set; }
        public string   Apellido        { get; set; }
        public string   Cedula          { get; set; }
        public DateTime FechaNacimiento { get; set; }
        public string   Telefono        { get; set; }
        public string   Direccion       { get; set; }
        public bool     Afiliado        { get; set; }
        public string   CodigoIEES      { get; set; }

        // Propiedad especial para transportar errores entre capas
        public string Error_Texto { get; set; }

        // Lista de enfermedades (relación 1:N con PacienteEnfermedad)
        public List<Paciente_Enfermedad_Entidades> ListaEnfermedades { get; set; }
    }
}
```

### `GeneroEntidades.cs`

```csharp
namespace GestionUsuariosEntidades
{
    public class GeneroEntidades
    {
        public int    Id     { get; set; }
        public string Nombre { get; set; }

        public GeneroEntidades() { }

        public GeneroEntidades(int id, string nombre)
        {
            Id     = id;
            Nombre = nombre;
        }
    }
}
```

### `EnfermedadEntidades.cs`

```csharp
namespace GestionUsuariosEntidades
{
    public class EnfermedadEntidades
    {
        public int    id     { get; set; }
        public string nombre { get; set; }

        public EnfermedadEntidades() { }

        public EnfermedadEntidades(int id, string nombre)
        {
            this.id     = id;
            this.nombre = nombre;
        }
    }
}
```

### `Paciente_Enfermedad_Entidades.cs` (tabla intermedia / detalle)

```csharp
using System;

namespace GestionUsuariosEntidades
{
    public class Paciente_Enfermedad_Entidades
    {
        public int    Id              { get; set; }
        public int    Id_Paciente     { get; set; }

        // En lugar de guardar solo el ID de la enfermedad,
        // guardamos el OBJETO completo para acceder a su nombre fácilmente
        public EnfermedadEntidades Enfermedad { get; set; }

        public DateTime FechaEnfermedad { get; set; }
        public string   Observacion     { get; set; }

        public Paciente_Enfermedad_Entidades() { }

        public Paciente_Enfermedad_Entidades(int id, int id_Paciente,
            EnfermedadEntidades enfermedad, DateTime fechaEnfermedad, string observacion)
        {
            Id              = id;
            Id_Paciente     = id_Paciente;
            Enfermedad      = enfermedad;
            FechaEnfermedad = fechaEnfermedad;
            Observacion     = observacion;
        }
    }
}
```

## Patrones usados en las Entidades

### 1. Constructor múltiple para distintos escenarios

```csharp
// Escenario 1: Crear paciente en blanco para el formulario
PacienteEntidades paciente = new PacienteEntidades();

// Escenario 2: Mapear datos que llegan de la base de datos
PacienteEntidades paciente = new PacienteEntidades(
    item.id, (int)item.id_Genero, item.Genero?.nombre ?? "N/A",
    item.nombre, item.apellido, item.cedula,
    item.fechaNacimiento, item.telefono,
    item.direccion, item.afiliado ?? false, item.codigoIEES
);

// Escenario 3: Reportar un error desde la capa de datos
return new PacienteEntidades("Error al conectar con la base de datos");
```

### 2. Entidad con objeto anidado (composición)

```csharp
// En lugar de:
public int Id_Enfermedad { get; set; }       //  Solo el ID, necesitas otro query

// El proyecto usa:
public EnfermedadEntidades Enfermedad { get; set; }  //  El objeto completo

// Uso:
var detalle = new Paciente_Enfermedad_Entidades();
detalle.Enfermedad = new EnfermedadEntidades(3, "Diabetes");
Console.WriteLine(detalle.Enfermedad.nombre); // "Diabetes" — sin query adicional
```

### 3. Propiedad `Error_Texto` para manejo de errores entre capas

```csharp
// En la capa de datos EF:
catch (Exception e)
{
    paciente.Error_Texto = e.Message + " — Paciente método Guardar";
    return paciente;   // Retorna el objeto con el error, sin lanzar excepción
}

// En la presentación:
paciente = PacienteNegocio.GuardarPaciente(paciente);
if (paciente.Error_Texto != null)
{
    MessageBox.Show(paciente.Error_Texto);
}
```

---

[← Arquitectura](../01-arquitectura-capas/README.md) | [Siguiente: Datos SQL →](../03-capa-datos-sql/README.md)

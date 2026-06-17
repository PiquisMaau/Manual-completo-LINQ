# 04 — Capa de Datos con LINQ to SQL

## Proyecto: `GestionUsuariosDatosLINQ`

LINQ to SQL es un ORM ligero de Microsoft que genera clases C# a partir de las tablas de tu base de datos. El archivo `.dbml` (Database Markup Language) define el mapeo. En este proyecto se llama `DataProgramacionAvanzada.dbml`.

> 🔑 **Cuándo usarla:** Proyectos pequeños/medianos con SQL Server, cuando quieres evitar escribir SQL a mano pero no necesitas toda la potencia de EF.

---

## Cómo funciona el DataContext

El archivo `.dbml` genera automáticamente la clase `DataProgramacionAvanzadaDataContext` que representa la conexión y las tablas como propiedades.

```csharp
// Clase generada automáticamente por el diseñador .dbml
// NO editar a mano — se regenera al modificar el .dbml en Visual Studio

public partial class DataProgramacionAvanzadaDataContext : DataContext
{
    // Cada tabla de la BD se convierte en una propiedad Table<T>
    public Table<Paciente>  Pacientes { get; }
    public Table<Genero>    Generos   { get; }
    // ... otras tablas
}
```

---

## Guardar un nuevo Paciente

```csharp
// Archivo: GestionUsuariosDatosLINQ/PacienteDatos.cs
using GestionUsuariosEntidades;

public static PacienteEntidades Nuevo(PacienteEntidades paciente)
{
    try
    {
        // 1. Crear el objeto del modelo LINQ (generado por el .dbml)
        Paciente pacienteLinQ = new Paciente();
        pacienteLinQ.id              = paciente.Id;
        pacienteLinQ.id_Genero       = paciente.IdGenero;
        pacienteLinQ.nombre          = paciente.Nombre;
        pacienteLinQ.apellido        = paciente.Apellido;
        pacienteLinQ.cedula          = paciente.Cedula;
        pacienteLinQ.fechaNacimiento = paciente.FechaNacimiento;
        pacienteLinQ.telefono        = paciente.Telefono;
        pacienteLinQ.direccion       = paciente.Direccion;
        pacienteLinQ.afiliado        = paciente.Afiliado;
        pacienteLinQ.codigoIEES      = paciente.CodigoIEES;

        // 2. Usar el DataContext dentro de un using (cierra la conexión automáticamente)
        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            contexto.Pacientes.InsertOnSubmit(pacienteLinQ); // "Encola" el INSERT
            contexto.SubmitChanges();                        // Ejecuta el INSERT en la BD
        }

        // 3. El ID generado en BD fue asignado al objeto pacienteLinQ automáticamente
        paciente.Id = pacienteLinQ.id;
        return paciente;
    }
    catch (Exception)
    {
        throw;
    }
}
```

---

## Listar todos los Pacientes

```csharp
public static List<PacienteEntidades> DevolverListaPaciente()
{
    try
    {
        List<PacienteEntidades> listaPacienteEntidades = new List<PacienteEntidades>();
        List<Paciente>          listaPacienteLinQ      = new List<Paciente>();

        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            // LINQ Query Syntax — como si fuera SQL
            var resultado = from p in contexto.Pacientes
                            select p;

            // ToList() materializa la consulta (ejecuta el SELECT en la BD)
            listaPacienteLinQ = resultado.ToList();
        }

        // Fuera del using: mapeamos los objetos LINQ a nuestras Entidades
        foreach (var item in listaPacienteLinQ)
        {
            listaPacienteEntidades.Add(new PacienteEntidades(
                item.id,
                (int)item.id_Genero,
                // Llamamos a otro método para obtener el nombre del género
                GeneroDatos.DevolverNombreGeneroPorID((int)item.id_Genero),
                item.nombre,
                item.apellido,
                item.cedula,
                item.fechaNacimiento,
                item.telefono,
                item.direccion,
                (bool)item.afiliado,
                item.codigoIEES));
        }

        return listaPacienteEntidades;
    }
    catch (Exception e)
    {
        var error = e.Message;
        throw;
    }
}
```

---

## Cargar un Paciente por ID (FirstOrDefault)

```csharp
public static PacienteEntidades CargarPacientePorId(int id)
{
    try
    {
        PacienteEntidades pacienteEntidad = new PacienteEntidades();

        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            // FirstOrDefault con lambda: equivalente a WHERE id = @id
            // Devuelve null si no encuentra nada (a diferencia de First() que lanza excepción)
            Paciente pacienteLinQ = contexto.Pacientes
                                            .FirstOrDefault(p => p.id == id);

            pacienteEntidad.Id              = pacienteLinQ.id;
            pacienteEntidad.IdGenero        = (int)pacienteLinQ.id_Genero;
            pacienteEntidad.Nombre          = pacienteLinQ.nombre;
            pacienteEntidad.Apellido        = pacienteLinQ.apellido;
            pacienteEntidad.Cedula          = pacienteLinQ.cedula;
            pacienteEntidad.FechaNacimiento = (DateTime)pacienteLinQ.fechaNacimiento;
            pacienteEntidad.Telefono        = pacienteLinQ.telefono;
            pacienteEntidad.Direccion       = pacienteLinQ.direccion;
            pacienteEntidad.Afiliado        = (bool)pacienteLinQ.afiliado;
            pacienteEntidad.CodigoIEES      = pacienteLinQ.codigoIEES;
        }

        return pacienteEntidad;
    }
    catch (Exception e)
    {
        throw;
    }
}
```

---

## Actualizar un Paciente

```csharp
public static PacienteEntidades Actualizar(PacienteEntidades paciente)
{
    try
    {
        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            // 1. PRIMERO buscar el registro actual en la BD
            Paciente pacienteLinQ = contexto.Pacientes
                                            .FirstOrDefault(p => p.id == paciente.Id);

            // 2. Modificar sus propiedades (el DataContext rastrea los cambios)
            pacienteLinQ.id_Genero       = paciente.IdGenero;
            pacienteLinQ.nombre          = paciente.Nombre;
            pacienteLinQ.apellido        = paciente.Apellido;
            pacienteLinQ.cedula          = paciente.Cedula;
            pacienteLinQ.fechaNacimiento = paciente.FechaNacimiento;
            pacienteLinQ.telefono        = paciente.Telefono;
            pacienteLinQ.direccion       = paciente.Direccion;
            pacienteLinQ.afiliado        = paciente.Afiliado;
            pacienteLinQ.codigoIEES      = paciente.CodigoIEES;

            // 3. SubmitChanges genera el UPDATE automáticamente
            contexto.SubmitChanges();
            return paciente;
        }
    }
    catch (Exception)
    {
        throw;
    }
}
```

---

## Eliminar un Paciente

```csharp
public static bool EliminarPacientePorId(int id)
{
    try
    {
        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            Paciente pacienteLinQ = contexto.Pacientes
                                            .FirstOrDefault(p => p.id == id);

            contexto.Pacientes.DeleteOnSubmit(pacienteLinQ); // "Encola" el DELETE
            contexto.SubmitChanges();                        // Ejecuta el DELETE
            return true;
        }
    }
    catch (Exception e)
    {
        throw;
    }
}
```

---

## Listado de Géneros con LINQ Query Syntax

```csharp
// Archivo: GestionUsuariosDatosLINQ/GeneroDatos.cs

public static List<GeneroEntidades> DevolverListaGeneros()
{
    try
    {
        List<GeneroEntidades> listaGenerosEntidades = new List<GeneroEntidades>();
        List<Genero>          listaGenero           = new List<Genero>();

        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            // Query Syntax — más parecida a SQL
            var resultados = from g in contexto.Generos
                             select g;

            listaGenero = resultados.ToList();
        }

        foreach (var item in listaGenero)
        {
            listaGenerosEntidades.Add(new GeneroEntidades(item.id, item.nombre));
        }

        return listaGenerosEntidades;
    }
    catch (Exception ex)
    {
        return null;
    }
}

// Obtener solo el nombre de un género por su ID
public static string DevolverNombreGeneroPorID(int ID_Genero)
{
    try
    {
        using (DataProgramacionAvanzadaDataContext contexto =
               new DataProgramacionAvanzadaDataContext())
        {
            // Method Syntax con lambda — más compacto
            var resultado = contexto.Generos.FirstOrDefault(g => g.id == ID_Genero);
            return resultado.nombre;
        }
    }
    catch (Exception ex)
    {
        throw;
    }
}
```

---

## Resumen: LINQ to SQL vs SQL Puro

| Aspecto | SQL Puro (ADO.NET) | LINQ to SQL |
|---------|-------------------|-------------|
| Código SQL | Escrito a mano como string | Generado automáticamente |
| INSERT | `cmd.CommandText = "INSERT..."` | `InsertOnSubmit()` + `SubmitChanges()` |
| DELETE | `cmd.CommandText = "DELETE..."` | `DeleteOnSubmit()` + `SubmitChanges()` |
| UPDATE | `cmd.CommandText = "UPDATE..."` | Modificar propiedades + `SubmitChanges()` |
| SELECT filtrado | Parámetros `@id` en el SQL | Lambda: `.FirstOrDefault(p => p.id == id)` |
| Mapping | Manual (`dr["campo"]`) | Automático por el DataContext |
| Tipado | Débil (convierte con `Convert.ToInt32`) | Fuerte (directamente tipado) |

---

[← SQL Puro](../03-capa-datos-sql/README.md) | [Siguiente: Entity Framework →](../05-capa-datos-ef/README.md)

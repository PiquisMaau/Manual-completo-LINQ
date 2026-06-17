# 05 — Capa de Datos con Entity Framework 6

## Proyecto: `GestionUsuariosDatosEF`

Entity Framework (EF) es el ORM completo de Microsoft. En este proyecto se usa **EF6 Database-First**: el modelo se genera desde una base de datos existente (archivo `.edmx`).

>  **Cuándo usarla:** Proyectos medianos/grandes donde quieres aprovechar el Change Tracking, relaciones automáticas, Migrations, y consultas más expresivas.

---

## El DbContext del proyecto

```csharp
// Archivo: GestionUsuariosDatosEF/Model1.Context.cs (generado por el .edmx)
// El context representa la sesión con la base de datos

public partial class ProgramacionAvanzadaEntities : DbContext
{
    public ProgramacionAvanzadaEntities()
        : base("name=ProgramacionAvanzadaEntities")  // Lee el connection string del App.Config
    {
    }

    // Cada DbSet<T> representa una tabla
    public virtual DbSet<Enfermedad>         Enfermedad          { get; set; }
    public virtual DbSet<Genero>             Genero              { get; set; }
    public virtual DbSet<Paciente>           Paciente            { get; set; }
    public virtual DbSet<Paciente_Enfermedad>Paciente_Enfermedad { get; set; }
}
```

---

## Guardar Paciente con EF (con manejo de detalle)

```csharp
// Archivo: GestionUsuariosDatosEF/PacienteDatosEF.cs

public static PacienteEntidades Nuevo(PacienteEntidades paciente)
{
    try
    {
        // 1. Crear objeto EF y mapear desde la entidad
        Paciente pacienteEF = new Paciente();
        pacienteEF.id              = paciente.Id;
        pacienteEF.id_Genero       = paciente.IdGenero;
        pacienteEF.nombre          = paciente.Nombre;
        pacienteEF.apellido        = paciente.Apellido;
        pacienteEF.cedula          = paciente.Cedula;
        pacienteEF.fechaNacimiento = paciente.FechaNacimiento;
        pacienteEF.telefono        = paciente.Telefono;
        pacienteEF.direccion       = paciente.Direccion;
        pacienteEF.afiliado        = paciente.Afiliado;
        pacienteEF.codigoIEES      = paciente.CodigoIEES;

        using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
        {
            contexto.Paciente.Add(pacienteEF); // Agrega al DbSet (estado: Added)
            contexto.SaveChanges();             // Genera el INSERT y guarda en BD
        }

        // EF asigna automáticamente el ID generado por IDENTITY al objeto
        paciente.Id = pacienteEF.id;

        // Actualizar el Id_Paciente en la lista de enfermedades (detalle)
        foreach (var item in paciente.ListaEnfermedades)
        {
            item.Id_Paciente = pacienteEF.id;
        }

        return paciente;
    }
    catch (Exception e)
    {
        // En lugar de lanzar, se guarda el error en la entidad y se retorna
        paciente.Error_Texto = e.Message + " — Paciente método Guardar";
        return paciente;
    }
}
```

---

## Listar Pacientes con Include (Eager Loading)

```csharp
public static List<PacienteEntidades> DevolverListaEntidades()
{
    try
    {
        List<PacienteEntidades> listaPacienteEF = new List<PacienteEntidades>();

        using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
        {
            // Include("Genero"): carga la entidad relacionada en la misma consulta
            // Genera: SELECT p.*, g.* FROM Paciente p INNER JOIN Genero g ON ...
            var lista = contexto.Paciente.Include("Genero").ToList();

            foreach (var item in lista)
            {
                listaPacienteEF.Add(new PacienteEntidades(
                    item.id,
                    item.id_Genero ?? 0,
                    item.Genero?.nombre ?? "N/A",  // Navegación: accede directo al objeto relacionado
                    item.nombre,
                    item.apellido,
                    item.cedula,
                    item.fechaNacimiento,
                    item.telefono,
                    item.direccion,
                    item.afiliado ?? false,
                    item.codigoIEES));
            }
        }

        return listaPacienteEF;
    }
    catch (Exception e)
    {
        var error = e.Message;
        throw;
    }
}
```

---

## Buscar por ID con Include y FirstOrDefault

```csharp
public static PacienteEntidades CargarPacientePorID(int id_Consultado)
{
    try
    {
        PacienteEntidades pacienteEntidades = new PacienteEntidades();

        using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
        {
            // Include + Where: SELECT con JOIN y filtro en una sola llamada
            var pacienteEF = contexto.Paciente
                                     .Include("Genero")
                                     .FirstOrDefault(p => p.id == id_Consultado);

            pacienteEntidades.Id              = pacienteEF.id;
            pacienteEntidades.IdGenero        = pacienteEF.id_Genero ?? 0;
            pacienteEntidades.Nombre          = pacienteEF.nombre;
            pacienteEntidades.Apellido        = pacienteEF.apellido;
            pacienteEntidades.Cedula          = pacienteEF.cedula;
            pacienteEntidades.FechaNacimiento = pacienteEF.fechaNacimiento;
            pacienteEntidades.Telefono        = pacienteEF.telefono;
            pacienteEntidades.Direccion       = pacienteEF.direccion;
            pacienteEntidades.Afiliado        = pacienteEF.afiliado ?? false;
            pacienteEntidades.CodigoIEES      = pacienteEF.codigoIEES;
        }

        return pacienteEntidades;
    }
    catch (Exception)
    {
        throw;
    }
}
```

---

## Actualizar con AddOrUpdate

```csharp
public static PacienteEntidades Actualizar(PacienteEntidades paciente)
{
    try
    {
        Paciente pacienteEF = new Paciente();
        pacienteEF.id              = paciente.Id;
        pacienteEF.id_Genero       = paciente.IdGenero;
        pacienteEF.nombre          = paciente.Nombre;
        pacienteEF.apellido        = paciente.Apellido;
        pacienteEF.cedula          = paciente.Cedula;
        pacienteEF.fechaNacimiento = paciente.FechaNacimiento;
        pacienteEF.telefono        = paciente.Telefono;
        pacienteEF.direccion       = paciente.Direccion;
        pacienteEF.afiliado        = paciente.Afiliado;
        pacienteEF.codigoIEES      = paciente.CodigoIEES;

        using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
        {
            // AddOrUpdate: INSERT si no existe, UPDATE si ya existe (por PK)
            // Requiere: using System.Data.Entity.Migrations;
            contexto.Paciente.AddOrUpdate(pacienteEF);
            contexto.SaveChanges();
        }

        return paciente;
    }
    catch (Exception)
    {
        throw;
    }
}
```

---

## Eliminar con Remove

```csharp
public static bool EliminarPacientePorId(int id_Consultado)
{
    try
    {
        using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
        {
            // 1. Primero buscar el objeto a eliminar
            var pacienteEF = contexto.Paciente
                                     .FirstOrDefault(p => p.id == id_Consultado);

            // 2. Marcar para eliminación y guardar
            contexto.Paciente.Remove(pacienteEF);
            contexto.SaveChanges();  // Genera el DELETE
            return true;
        }
    }
    catch (Exception)
    {
        throw;
    }
}
```

---

## Cargar Detalle de Enfermedades con Include y Where

```csharp
// Archivo: GestionUsuariosDatosEF/Paciente_Enfermedades_Datos.cs

public static List<Paciente_Enfermedad_Entidades> CargarDetallePorPaciente(int id_Paciente)
{
    try
    {
        List<Paciente_Enfermedad_Entidades> listaDetalleEntidades =
            new List<Paciente_Enfermedad_Entidades>();

        using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
        {
            // Include("Enfermedad"): trae la entidad Enfermedad relacionada
            // Where filtra por el paciente
            // Genera: SELECT pe.*, e.* FROM Paciente_Enfermedad pe
            //         INNER JOIN Enfermedad e ON pe.id_Enfermedad = e.id
            //         WHERE pe.id_Paciente = @id_Paciente
            var listaEF = contexto.Paciente_Enfermedad
                                  .Include("Enfermedad")
                                  .Where(pe => pe.id_Paciente == id_Paciente)
                                  .ToList();

            foreach (var item in listaEF)
            {
                // Crear la entidad de enfermedad usando las propiedades de navegación
                EnfermedadEntidades enfermedadAsociada = new EnfermedadEntidades(
                    item.id_Enfermedad ?? 0,
                    item.Enfermedad?.nombre ?? "N/A"
                );

                listaDetalleEntidades.Add(new Paciente_Enfermedad_Entidades(
                    item.id,
                    (int)item.id_Paciente,
                    enfermedadAsociada,
                    item.fechaEnfermedad ?? DateTime.Now,
                    item.observacion
                ));
            }
        }

        return listaDetalleEntidades;
    }
    catch (Exception)
    {
        throw;
    }
}
```

---

## Resumen de operaciones EF6

| Operación | Método EF | Cuándo llamarlo |
|-----------|-----------|----------------|
| INSERT | `DbSet.Add()` | Antes de `SaveChanges()` |
| INSERT o UPDATE | `DbSet.AddOrUpdate()` | Antes de `SaveChanges()` (requiere EF Migrations) |
| DELETE | `DbSet.Remove()` | Antes de `SaveChanges()` |
| SELECT todos | `DbSet.ToList()` | Materializa la consulta |
| SELECT filtrado | `DbSet.Where(lambda).ToList()` | Filtra en BD |
| SELECT uno | `DbSet.FirstOrDefault(lambda)` | Devuelve null si no existe |
| JOIN eager | `DbSet.Include("NombrePropiedad")` | Carga relación en una consulta |

---

[← LINQ to SQL](../04-capa-datos-linq/README.md) | [Siguiente: Lógica de Negocio →](../06-logica-negocio/README.md)

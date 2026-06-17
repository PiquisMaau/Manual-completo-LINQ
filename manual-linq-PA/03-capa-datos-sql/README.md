# 03 — Capa de Datos con ADO.NET (SQL Puro)

## Proyecto: `GestionUsuariosDatos`

Esta es la implementación más básica: se usa `SqlConnection` y `SqlCommand` para ejecutar SQL directamente. No hay ORM. El desarrollador escribe el SQL a mano.

>  **Cuándo usarla:** Cuando necesitas control total del SQL, consultas muy complejas o rendimiento máximo.

---

## Guardar un nuevo Paciente (INSERT)

```csharp
// Archivo: GestionUsuariosDatos/PacienteDatos.cs
using System.Data;
using System.Data.SqlClient;
using GestionUsuariosEntidades;

public static PacienteEntidades Nuevo(PacienteEntidades paciente)
{
    try
    {
        // 1. Crear la conexión (string guardado en Settings)
        SqlConnection conexion = new SqlConnection(
            Properties.Settings.Default.ConexionBDD);
        conexion.Open();

        // 2. Preparar el comando SQL con parámetros (evita SQL Injection)
        SqlCommand cmd = new SqlCommand();
        cmd.Connection   = conexion;
        cmd.CommandType  = CommandType.Text;
        cmd.CommandText  = @"
            INSERT INTO [dbo].[Paciente]
                ([id_Genero], [nombre], [apellido], [cedula],
                 [fechaNacimiento], [telefono], [direccion],
                 [afiliado], [codigoIEES])
            VALUES
                (@id_Genero, @nombre, @apellido, @cedula,
                 @fechaNacimiento, @telefono, @direccion,
                 @afiliado, @codigoIEES);
            SELECT SCOPE_IDENTITY()";  // Devuelve el ID generado

        // 3. Vincular los parámetros con las propiedades de la entidad
        cmd.Parameters.AddWithValue("@id_Genero",        paciente.IdGenero);
        cmd.Parameters.AddWithValue("@nombre",           paciente.Nombre);
        cmd.Parameters.AddWithValue("@apellido",         paciente.Apellido);
        cmd.Parameters.AddWithValue("@cedula",           paciente.Cedula);
        cmd.Parameters.AddWithValue("@fechaNacimiento",  paciente.FechaNacimiento);
        cmd.Parameters.AddWithValue("@telefono",         paciente.Telefono);
        cmd.Parameters.AddWithValue("@direccion",        paciente.Direccion);
        cmd.Parameters.AddWithValue("@afiliado",         paciente.Afiliado);
        cmd.Parameters.AddWithValue("@codigoIEES",       paciente.CodigoIEES);

        // 4. ExecuteScalar devuelve el primer campo de la primera fila (el nuevo ID)
        int IdPaciente = Convert.ToInt32(cmd.ExecuteScalar());
        paciente.Id = IdPaciente;

        conexion.Close();
        return paciente;
    }
    catch (Exception e)
    {
        var error = e.Message;
        return null;
    }
}
```

---

## Listar todos los Pacientes con JOIN (SELECT)

```csharp
public static List<PacienteEntidades> DevolverListaPaciente()
{
    try
    {
        List<PacienteEntidades> listaPacientes = new List<PacienteEntidades>();

        SqlConnection conexion = new SqlConnection(
            Properties.Settings.Default.ConexionBDD);
        conexion.Open();

        SqlCommand cmd = new SqlCommand();
        cmd.Connection  = conexion;
        cmd.CommandType = CommandType.Text;

        // JOIN para obtener el nombre del género sin hacer otra consulta
        cmd.CommandText = @"
            SELECT p.[id]
                  ,p.[id_Genero]
                  ,g.[nombre] as genero    -- alias para distinguirlo de p.nombre
                  ,p.[nombre]
                  ,p.[apellido]
                  ,p.[cedula]
                  ,p.[fechaNacimiento]
                  ,p.[telefono]
                  ,p.[direccion]
                  ,p.[afiliado]
                  ,p.[codigoIEES]
            FROM [dbo].[Paciente] p
            INNER JOIN GENERO g ON p.id_Genero = g.id";

        // SqlDataReader: lee fila por fila de forma eficiente (forward-only)
        using (var dr = cmd.ExecuteReader())
        {
            while (dr.Read())
            {
                PacienteEntidades paciente = new PacienteEntidades();
                paciente.Id              = Convert.ToInt32(dr["id"].ToString());
                paciente.IdGenero        = Convert.ToInt32(dr["id_Genero"].ToString());
                paciente.Genero          = dr["genero"].ToString();
                paciente.Nombre          = dr["nombre"].ToString();
                paciente.Apellido        = dr["apellido"].ToString();
                paciente.Cedula          = dr["cedula"].ToString();
                paciente.FechaNacimiento = Convert.ToDateTime(dr["fechaNacimiento"].ToString());
                paciente.Telefono        = dr["telefono"].ToString();
                paciente.Direccion       = dr["direccion"].ToString();
                paciente.Afiliado        = Convert.ToBoolean(dr["afiliado"].ToString());
                paciente.CodigoIEES      = dr["codigoIEES"].ToString();

                listaPacientes.Add(paciente);
            }
        }

        conexion.Close();
        return listaPacientes;
    }
    catch
    {
        return null;
    }
}
```

---

## Actualizar un Paciente (UPDATE)

```csharp
public static PacienteEntidades Actualizar(PacienteEntidades paciente)
{
    try
    {
        SqlConnection conexion = new SqlConnection(
            Properties.Settings.Default.ConexionBDD);
        conexion.Open();

        SqlCommand cmd = new SqlCommand();
        cmd.Connection  = conexion;
        cmd.CommandType = CommandType.Text;
        cmd.CommandText = @"
            UPDATE [dbo].[Paciente]
            SET [id_Genero]       = @id_Genero
               ,[nombre]          = @nombre
               ,[apellido]        = @apellido
               ,[cedula]          = @cedula
               ,[fechaNacimiento] = @fechaNacimiento
               ,[telefono]        = @telefono
               ,[direccion]       = @direccion
               ,[afiliado]        = @afiliado
               ,[codigoIEES]      = @codigoIEES
            WHERE id = @id";   // ← El WHERE es crítico

        cmd.Parameters.AddWithValue("@id_Genero",        paciente.IdGenero);
        cmd.Parameters.AddWithValue("@nombre",           paciente.Nombre);
        cmd.Parameters.AddWithValue("@apellido",         paciente.Apellido);
        cmd.Parameters.AddWithValue("@cedula",           paciente.Cedula);
        cmd.Parameters.AddWithValue("@fechaNacimiento",  paciente.FechaNacimiento);
        cmd.Parameters.AddWithValue("@telefono",         paciente.Telefono);
        cmd.Parameters.AddWithValue("@direccion",        paciente.Direccion);
        cmd.Parameters.AddWithValue("@afiliado",         paciente.Afiliado);
        cmd.Parameters.AddWithValue("@codigoIEES",       paciente.CodigoIEES);
        cmd.Parameters.AddWithValue("@id",               paciente.Id);

        // ExecuteNonQuery: para INSERT, UPDATE, DELETE — devuelve filas afectadas
        cmd.ExecuteNonQuery();
        conexion.Close();
        return paciente;
    }
    catch (Exception e)
    {
        string error = e.Message;
        return null;
    }
}
```

---

## Eliminar un Paciente (DELETE)

```csharp
public static bool EliminarPacientePorId(int id)
{
    try
    {
        SqlConnection conexion = new SqlConnection(
            Properties.Settings.Default.ConexionBDD);
        conexion.Open();

        SqlCommand cmd = new SqlCommand();
        cmd.Connection  = conexion;
        cmd.CommandType = CommandType.Text;
        cmd.CommandText = @"DELETE FROM [dbo].[Paciente] WHERE id = @id";
        cmd.Parameters.AddWithValue("@id", id);

        // ExecuteNonQuery devuelve el número de filas afectadas
        var numeroFilaAfectadas = cmd.ExecuteNonQuery();

        return numeroFilaAfectadas > 0;  // true si eliminó al menos 1 fila
    }
    catch (Exception e)
    {
        throw;
    }
}
```

---

## Comparación de métodos de ejecución SQL

| Método | Cuándo usarlo | Devuelve |
|--------|--------------|----------|
| `ExecuteScalar()` | SELECT que devuelve un solo valor (COUNT, MAX, nuevo ID) | `object` |
| `ExecuteNonQuery()` | INSERT, UPDATE, DELETE | `int` (filas afectadas) |
| `ExecuteReader()` | SELECT que devuelve múltiples filas | `SqlDataReader` |

---

[← Entidades](../02-entidades/README.md) | [Siguiente: LINQ to SQL →](../04-capa-datos-linq/README.md)

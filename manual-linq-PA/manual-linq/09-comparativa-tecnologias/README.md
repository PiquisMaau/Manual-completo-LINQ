# 09 — Comparativa: ADO.NET vs LINQ to SQL vs Entity Framework

El proyecto GestionUsuarios implementa la **misma operación** de tres formas distintas.
Aquí las vemos lado a lado para entender qué cambia y qué permanece igual.

---

## La misma operación en las 3 tecnologías: Listar Pacientes

### 🟥 ADO.NET (SQL Puro) — `GestionUsuariosDatos/PacienteDatos.cs`

```csharp
public static List<PacienteEntidades> DevolverListaPaciente()
{
    List<PacienteEntidades> listaPacientes = new List<PacienteEntidades>();

    SqlConnection conexion = new SqlConnection(
        Properties.Settings.Default.ConexionBDD);
    conexion.Open();

    SqlCommand cmd = new SqlCommand();
    cmd.Connection  = conexion;
    cmd.CommandType = CommandType.Text;

    // SQL escrito a mano — tú controlas exactamente qué se envía a la BD
    cmd.CommandText = @"
        SELECT p.id, p.id_Genero, g.nombre as genero,
               p.nombre, p.apellido, p.cedula,
               p.fechaNacimiento, p.telefono,
               p.direccion, p.afiliado, p.codigoIEES
        FROM [dbo].[Paciente] p
        INNER JOIN GENERO g ON p.id_Genero = g.id";

    using (var dr = cmd.ExecuteReader())
    {
        while (dr.Read())
        {
            // Mapeo MANUAL columna por columna
            listaPacientes.Add(new PacienteEntidades(
                Convert.ToInt32(dr["id"]),
                Convert.ToInt32(dr["id_Genero"]),
                dr["genero"].ToString(),
                dr["nombre"].ToString(),
                dr["apellido"].ToString(),
                dr["cedula"].ToString(),
                Convert.ToDateTime(dr["fechaNacimiento"]),
                dr["telefono"].ToString(),
                dr["direccion"].ToString(),
                Convert.ToBoolean(dr["afiliado"]),
                dr["codigoIEES"].ToString()
            ));
        }
    }
    conexion.Close();
    return listaPacientes;
}
```

---

### 🟨 LINQ to SQL — `GestionUsuariosDatosLINQ/PacienteDatos.cs`

```csharp
public static List<PacienteEntidades> DevolverListaPaciente()
{
    List<PacienteEntidades> listaPacienteEntidades = new List<PacienteEntidades>();

    using (DataProgramacionAvanzadaDataContext contexto =
           new DataProgramacionAvanzadaDataContext())
    {
        // LINQ Query Syntax — el DataContext traduce esto a SQL
        var resultado = from p in contexto.Pacientes
                        select p;

        // ToList() ejecuta el SELECT
        var listaPacienteLinQ = resultado.ToList();

        // Mapeo todavía manual, pero los objetos ya son tipados
        foreach (var item in listaPacienteLinQ)
        {
            listaPacienteEntidades.Add(new PacienteEntidades(
                item.id,
                (int)item.id_Genero,
                // Consulta adicional para obtener el nombre del género
                GeneroDatos.DevolverNombreGeneroPorID((int)item.id_Genero),
                item.nombre, item.apellido, item.cedula,
                item.fechaNacimiento, item.telefono,
                item.direccion, (bool)item.afiliado, item.codigoIEES));
        }
    }

    return listaPacienteEntidades;
}
```

---

### 🟩 Entity Framework — `GestionUsuariosDatosEF/PacienteDatosEF.cs`

```csharp
public static List<PacienteEntidades> DevolverListaEntidades()
{
    List<PacienteEntidades> listaPacienteEF = new List<PacienteEntidades>();

    using (ProgramacionAvanzadaEntities contexto =
           new ProgramacionAvanzadaEntities())
    {
        // Include carga automáticamente la entidad relacionada Genero
        // Genera el JOIN internamente — no necesitas escribir SQL
        var lista = contexto.Paciente.Include("Genero").ToList();

        foreach (var item in lista)
        {
            listaPacienteEF.Add(new PacienteEntidades(
                item.id,
                item.id_Genero ?? 0,
                item.Genero?.nombre ?? "N/A",  // Navegación directa al objeto relacionado
                item.nombre, item.apellido, item.cedula,
                item.fechaNacimiento, item.telefono,
                item.direccion, item.afiliado ?? false, item.codigoIEES));
        }
    }

    return listaPacienteEF;
}
```

---

## Comparativa: Guardar un nuevo Paciente

| Paso | ADO.NET | LINQ to SQL | Entity Framework |
|------|---------|-------------|-----------------|
| Crear objeto | `SqlCommand cmd = new SqlCommand()` | `Paciente p = new Paciente()` | `Paciente p = new Paciente()` |
| Mapear datos | `cmd.Parameters.AddWithValue(...)` × 9 | `p.nombre = paciente.Nombre` × 9 | `p.nombre = paciente.Nombre` × 9 |
| Guardar | `cmd.ExecuteScalar()` | `InsertOnSubmit()` + `SubmitChanges()` | `Add()` + `SaveChanges()` |
| Obtener ID | `Convert.ToInt32(cmd.ExecuteScalar())` | `pacienteLinQ.id` (auto-asignado) | `pacienteEF.id` (auto-asignado) |
| SQL generado | Escrito por ti | Generado automáticamente | Generado automáticamente |

---

## Comparativa: Eliminar un Paciente

| Tecnología | Código |
|-----------|--------|
| **ADO.NET** | `cmd.CommandText = "DELETE FROM Paciente WHERE id = @id"` + `ExecuteNonQuery()` |
| **LINQ to SQL** | `contexto.Pacientes.DeleteOnSubmit(obj)` + `SubmitChanges()` |
| **EF** | `contexto.Paciente.Remove(obj)` + `SaveChanges()` |

---

## Tabla de decisión: ¿Cuál usar?

| Criterio | ADO.NET | LINQ to SQL | Entity Framework |
|----------|---------|-------------|-----------------|
| **Velocidad de escritura** | ❌ Lenta (mucho código) | ✅ Rápida | ✅ Rápida |
| **Control total del SQL** | ✅ Total | ⚠️ Parcial | ⚠️ Parcial |
| **Relaciones (JOIN/Include)** | Manual (SQL) | Limitado | ✅ Potente (Include) |
| **Transacciones** | Manual (`SqlTransaction`) | Manual | ✅ `TransactionScope` |
| **Curva de aprendizaje** | Baja | Media | Media-Alta |
| **Rendimiento** | ✅ Máximo | ✅ Bueno | ⚠️ Overhead del ORM |
| **Mantenimiento** | ❌ Difícil (strings SQL) | ✅ Fácil | ✅ Fácil |
| **Proyectos grandes** | ❌ Difícil de escalar | ⚠️ Limitado | ✅ Ideal |

---

## Cómo el proyecto cambia de tecnología con UNA sola línea

En `GestionUsuariosLogicaNegocio/PacienteNegocio.cs`:

```csharp
// ═══════════════════════════════════════════════
// OPCIÓN A — Usar ADO.NET puro
// ═══════════════════════════════════════════════
using GestionUsuariosDatos;
// resultado = PacienteDatos.Nuevo(paciente);
// resultado = PacienteDatos.DevolverListaPaciente();

// ═══════════════════════════════════════════════
// OPCIÓN B — Usar LINQ to SQL
// ═══════════════════════════════════════════════
using GestionUsuariosDatosLINQ;
// resultado = PacienteDatos.Nuevo(paciente);
// resultado = PacienteDatos.DevolverListaPaciente();

// ═══════════════════════════════════════════════
// OPCIÓN C — Usar Entity Framework (ACTUALMENTE ACTIVO)
// ═══════════════════════════════════════════════
using GestionUsuariosDatosEF;
resultado = PacienteDatosEF.Nuevo(paciente);
resultado = PacienteDatosEF.DevolverListaEntidades();
```

> 💡 Esto es posible porque todas las capas de datos **devuelven el mismo tipo**: `PacienteEntidades`. La capa de Negocio no sabe (ni le importa) cuál tecnología usa la capa de Datos.

---

[← Consultas Avanzadas](../08-linq-consultas-avanzadas/README.md) | [Siguiente: Errores Comunes →](../10-errores-comunes/README.md)

# 10 — Errores Comunes y Cómo Resolverlos

Todos estos errores aparecieron (o pueden aparecer) en el proyecto GestionUsuariosV2.

---

## ❌ Error 1: `NullReferenceException` al hacer clic en la grilla

**Dónde aparece:** `Form_Paciente.cs` → `dataGridView1_CellClick`

**Causa:** Se hace clic en el encabezado de la columna (fila -1), o la celda "id" no existe.

```csharp
// ❌ Código problemático
private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
{
    var id = Convert.ToInt32(dataGridView1.Rows[e.RowIndex].Cells["id"].Value.ToString());
    // Si e.RowIndex = -1 (encabezado) → IndexOutOfRangeException
    // Si Value es null → NullReferenceException
}
```

**✅ Solución:** Validar el índice de fila antes de procesar

```csharp
private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
{
    // Salir si se hace clic en el encabezado (RowIndex = -1)
    if (e.RowIndex < 0) return;

    try
    {
        var cellValue = dataGridView1.Rows[e.RowIndex].Cells["id"].Value;
        
        // Verificar que el valor no sea null antes de convertir
        if (cellValue == null || cellValue == DBNull.Value) return;
        
        var id = Convert.ToInt32(cellValue.ToString());
        CargarValoresPacientePorId(id);
    }
    catch (Exception ex)
    {
        MessageBox.Show("Error al seleccionar el elemento: " + ex.Message);
    }
}
```

---

## ❌ Error 2: `InvalidOperationException` — ComboBox sin selección al guardar

**Dónde aparece:** `Form_Paciente.cs` → `GuardarPaciente()`

**Causa:** El usuario no seleccionó un género, y `cb_Genero.SelectedValue` es null.

```csharp
// ❌ Código problemático
paciente.IdGenero = Convert.ToInt32(cb_Genero.SelectedValue);
// Si SelectedValue es null → Convert.ToInt32(null) → FormatException
```

**✅ Solución:** Verificar antes de convertir

```csharp
// ✅ Validar que haya selección
if (cb_Genero.SelectedValue == null)
{
    MessageBox.Show("Debe seleccionar un género.", "Campo requerido",
                    MessageBoxButtons.OK, MessageBoxIcon.Warning);
    cb_Genero.Focus();
    return; // Detener el guardado
}

paciente.IdGenero = Convert.ToInt32(cb_Genero.SelectedValue);
```

---

## ❌ Error 3: `ListaEnfermedades` es null al guardar

**Dónde aparece:** `GestionUsuariosDatosEF/PacienteDatosEF.cs` → `Nuevo()`

**Causa:** `paciente.ListaEnfermedades` nunca fue inicializado y se intenta hacer `foreach` sobre él.

```csharp
// ❌ Código problemático
foreach (var item in paciente.ListaEnfermedades) // NullReferenceException si es null
{
    item.Id_Paciente = pacienteEF.id;
}
```

**✅ Solución A (en la capa de datos):** Null check antes del foreach

```csharp
// Verificar que la lista no sea null antes de iterar
if (paciente.ListaEnfermedades != null)
{
    foreach (var item in paciente.ListaEnfermedades)
    {
        item.Id_Paciente = pacienteEF.id;
    }
}
```

**✅ Solución B (en el constructor de la entidad):** Inicializar la lista

```csharp
public PacienteEntidades()
{
    // Inicializar siempre la lista para evitar null
    ListaEnfermedades = new List<Paciente_Enfermedad_Entidades>();
}
```

---

## ❌ Error 4: `SubmitChanges()` falla sin mensaje claro

**Dónde aparece:** `GestionUsuariosDatosLINQ/PacienteDatos.cs`

**Causa:** La restricción de la BD rechaza el dato (cédula duplicada, FK inválida, etc.) pero el `catch (Exception){ throw; }` re-lanza sin contexto.

```csharp
// ❌ Código problemático — pierde información del error
catch (Exception)
{
    throw; // Solo relanza, no agrega contexto
}
```

**✅ Solución:** Capturar y enriquecer el error

```csharp
catch (Exception ex)
{
    // Agregar contexto antes de relanzar
    throw new Exception($"Error al guardar paciente en LINQ to SQL: {ex.Message}", ex);
}
```

---

## ❌ Error 5: `ObjectDisposedException` — usar el contexto fuera del `using`

**Dónde aparece:** Al intentar acceder a propiedades de navegación EF fuera del contexto.

```csharp
// ❌ Código problemático
List<Paciente> lista;
using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
{
    lista = contexto.Paciente.ToList(); // ← SIN Include
}

// ❌ AQUÍ el contexto ya está cerrado (disposed)
foreach (var p in lista)
{
    // Lazy loading intenta abrir conexión → ObjectDisposedException
    string nombreGenero = p.Genero.nombre;
}
```

**✅ Solución:** Usar `Include()` DENTRO del `using`

```csharp
List<Paciente> lista;
using (ProgramacionAvanzadaEntities contexto = new ProgramacionAvanzadaEntities())
{
    // Include carga todo lo necesario ANTES de cerrar el contexto
    lista = contexto.Paciente.Include("Genero").ToList();
}

// ✅ Ahora Genero ya está en memoria, no necesita conexión
foreach (var p in lista)
{
    string nombreGenero = p.Genero?.nombre ?? "N/A"; // ✅ Seguro
}
```

---

## ❌ Error 6: `First()` lanza excepción cuando no hay resultados

**Dónde aparece:** En cualquier consulta que use `.First()` en lugar de `.FirstOrDefault()`

```csharp
// ❌ Si no existe el paciente con ese ID → InvalidOperationException
Paciente p = contexto.Pacientes.First(x => x.id == id);
```

**✅ Solución:** Siempre usar `FirstOrDefault()` y verificar null

```csharp
// ✅ Devuelve null si no existe, en lugar de lanzar excepción
Paciente p = contexto.Pacientes.FirstOrDefault(x => x.id == id);

if (p == null)
{
    throw new Exception($"No existe el paciente con ID {id}");
}

// Ahora sí es seguro usar p
```

---

## ❌ Error 7: `ToList()` antes de filtrar (rendimiento con EF)

```csharp
// ❌ Trae TODOS los pacientes a memoria y luego filtra en C#
var todosPacientes = contexto.Paciente.ToList();
var ambatenos = todosPacientes.Where(p => p.direccion == "Ambato");
// Si hay 100,000 pacientes → carga 100,000 filas para quedarse con 5
```

**✅ Solución:** Filtrar ANTES de `ToList()`

```csharp
// ✅ El filtro se ejecuta en la BD — solo trae los que coinciden
var ambatenos = contexto.Paciente
                        .Where(p => p.direccion == "Ambato")
                        .ToList();
// SQL generado: SELECT * FROM Paciente WHERE direccion = 'Ambato'
```

---

## ❌ Error 8: Conexión que nunca se cierra (ADO.NET)

```csharp
// ❌ Si ocurre una excepción entre Open() y Close(), la conexión queda abierta
SqlConnection conexion = new SqlConnection(connectionString);
conexion.Open();
// ... código que puede fallar ...
conexion.Close(); // ← Nunca llega aquí si hay excepción arriba
```

**✅ Solución:** Usar `using` o `try/finally`

```csharp
// Opción A — using (recomendado): cierra automáticamente al salir del bloque
using (SqlConnection conexion = new SqlConnection(connectionString))
{
    conexion.Open();
    // ... si falla aquí, using hace Dispose() que cierra la conexión
}

// Opción B — try/finally: garantiza el cierre incluso con excepción
SqlConnection conexion = new SqlConnection(connectionString);
try
{
    conexion.Open();
    // ... código ...
}
finally
{
    if (conexion.State == ConnectionState.Open)
        conexion.Close();
}
```

---

## ❌ Error 9: `cb_Genero.SelectedIndex = -1` causa problemas al cargar

**Dónde aparece:** `Form_Paciente.cs` → `CargarComboGenero()`

**Causa:** Poner `SelectedIndex = -1` después de asignar el `DataSource` dispara el evento `SelectedIndexChanged` con valor null.

```csharp
// ❌ El evento SelectedIndexChanged se dispara con valor null
cb_Genero.DataSource      = GeneroNegocio.DevolverListaGeneros();
cb_Genero.DisplayMember   = "nombre";
cb_Genero.ValueMember     = "id";
cb_Genero.SelectedIndex   = -1; // ← Puede causar problemas si hay handler
```

**✅ Solución:** Desuscribir el evento mientras se configura

```csharp
// Desuscribir temporalmente el evento
cb_Genero.SelectedIndexChanged -= cb_Genero_SelectedIndexChanged;

cb_Genero.DataSource     = GeneroNegocio.DevolverListaGeneros();
cb_Genero.DisplayMember  = "nombre";
cb_Genero.ValueMember    = "id";
cb_Genero.SelectedIndex  = -1;

// Volver a suscribir
cb_Genero.SelectedIndexChanged += cb_Genero_SelectedIndexChanged;
```

---

## Tabla resumen de errores

| Error | Causa | Solución rápida |
|-------|-------|----------------|
| `NullReferenceException` al clic en grilla | `RowIndex = -1` | `if (e.RowIndex < 0) return;` |
| `FormatException` al guardar | `SelectedValue == null` | Validar antes de `Convert.ToInt32` |
| `NullReferenceException` en foreach de enfermedades | Lista no inicializada | `ListaEnfermedades = new List<>()` en constructor |
| `ObjectDisposedException` en propiedades navegación | Lazy loading fuera del contexto | Usar `.Include("Genero")` |
| `InvalidOperationException` | `First()` sin resultados | Cambiar a `FirstOrDefault()` |
| Rendimiento lento con EF | `ToList()` antes de `Where` | Filtrar antes de materializar |
| Conexión que no cierra | Excepción antes de `Close()` | Usar `using (SqlConnection ...)` |

---

[← Comparativa](../09-comparativa-tecnologias/README.md) | [Siguiente: Ejercicios Resueltos →](../11-ejercicios-resueltos/README.md)

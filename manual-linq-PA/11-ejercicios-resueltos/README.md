# 11 — Ejercicios Resueltos con el Proyecto

Todos los ejercicios usan las entidades y estructura real de GestionUsuariosV2.

---

## Ejercicio 1 — Filtrar pacientes mayores de edad

**Enunciado:** Obtén la lista de pacientes que tengan 18 años o más, ordenados por edad descendente (mayor primero).

```csharp
// Datos de prueba (los mismos del capítulo 08)
List<PacienteEntidades> pacientes = PacienteNegocio.DevolverListaPaciente();

var mayoresDeEdad = pacientes
    .Where(p => (DateTime.Now.Year - p.FechaNacimiento.Year) >= 18)
    .OrderByDescending(p => p.FechaNacimiento.Year) // Año más antiguo = más viejo
    .Select(p => new
    {
        NombreCompleto = $"{p.Nombre} {p.Apellido}",
        Edad           = DateTime.Now.Year - p.FechaNacimiento.Year,
        Ciudad         = p.Direccion
    });

Console.WriteLine("=== Pacientes mayores de edad ===");
foreach (var p in mayoresDeEdad)
    Console.WriteLine($"  {p.NombreCompleto} | {p.Edad} años | {p.Ciudad}");
```

**Salida esperada:**
```
=== Pacientes mayores de edad ===
  Roberto Cáceres  | 70 años | Loja
  Pedro Salazar    | 60 años | Cuenca
  Juan Torres      | 47 años | Guayaquil
  Carlos Pérez     | 40 años | Ambato
  Lucía Mora       | 35 años | Ambato
  María González   | 33 años | Quito
  Ana Villalba     | 24 años | Ambato
```

---

## Ejercicio 2 — Reporte de afiliados por ciudad

**Enunciado:** Genera un reporte que muestre, por cada ciudad, cuántos pacientes están afiliados al IESS y cuántos no lo están.

```csharp
var reportePorCiudad = pacientes
    .GroupBy(p => p.Direccion)
    .Select(g => new
    {
        Ciudad      = g.Key,
        Afiliados   = g.Count(p => p.Afiliado),
        NoAfiliados = g.Count(p => !p.Afiliado),
        Total       = g.Count(),
        PorcentajeAfiliacion = Math.Round(
            (double)g.Count(p => p.Afiliado) / g.Count() * 100, 1)
    })
    .OrderByDescending(r => r.Total);

Console.WriteLine($"{"Ciudad",-12} {"Afil.",5} {"No Afil.",9} {"Total",6} {"% Afil.",9}");
Console.WriteLine(new string('-', 45));
foreach (var r in reportePorCiudad)
    Console.WriteLine($"{r.Ciudad,-12} {r.Afiliados,5} {r.NoAfiliados,9} " +
                      $"{r.Total,6} {r.PorcentajeAfiliacion,8}%");

// Output:
// Ciudad        Afil. No Afil.  Total  % Afil.
// ---------------------------------------------
// Ambato            2        1      3     66.7%
// Quito             0        1      1      0.0%
// Guayaquil         1        0      1    100.0%
// Cuenca            1        0      1    100.0%
// Loja              0        1      1      0.0%
```

---

## Ejercicio 3 — Validar cédula ecuatoriana única

**Enunciado:** Antes de guardar un paciente nuevo, verificar que su cédula no esté registrada ya en el sistema. Si existe, lanzar un mensaje indicando a quién pertenece.

```csharp
// Agregar este método en PacienteNegocio.cs
public static string ValidarCedulaUnica(string cedula, int idPacienteActual = 0)
{
    var todos = PacienteDatosEF.DevolverListaEntidades();

    // Buscar si la cédula ya pertenece a otro paciente
    // (excluir el propio paciente si estamos actualizando)
    var duplicado = todos.FirstOrDefault(p =>
        p.Cedula == cedula && p.Id != idPacienteActual);

    if (duplicado != null)
        return $"La cédula {cedula} ya está registrada para: " +
               $"{duplicado.Nombre} {duplicado.Apellido} (ID: {duplicado.Id})";

    return null; // null significa que la cédula está disponible
}

// Uso en la capa de Negocio al guardar:
public static PacienteEntidades GuardarPaciente(PacienteEntidades paciente)
{
    // Validar cédula antes de ir a la BD
    string errorCedula = ValidarCedulaUnica(paciente.Cedula, paciente.Id);
    if (errorCedula != null)
    {
        paciente.Error_Texto = errorCedula;
        return paciente;
    }

    using (TransactionScope scope = new TransactionScope())
    {
        PacienteEntidades resultado = paciente.Id == 0
            ? PacienteDatosEF.Nuevo(paciente)
            : PacienteDatosEF.Actualizar(paciente);

        scope.Complete();
        return resultado;
    }
}

// Uso en el formulario:
private void GuardarPaciente()
{
    // ... construir objeto paciente ...

    paciente = PacienteNegocio.GuardarPaciente(paciente);

    if (paciente.Error_Texto != null)
    {
        MessageBox.Show(paciente.Error_Texto, "Cédula duplicada",
                        MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtb_Cedula.Focus();
        return;
    }

    MessageBox.Show("Paciente guardado correctamente.");
    CargarListadoPacientesEnDataGridView();
}
```

---

## Ejercicio 4 — Historial médico completo de un paciente

**Enunciado:** Dado el ID de un paciente, obtener un resumen completo: datos del paciente + todas sus enfermedades ordenadas por fecha descendente.

```csharp
// Método en la capa de Negocio
public static PacienteEntidades ObtenerFichaCompletaPaciente(int idPaciente)
{
    // Cargar datos del paciente
    PacienteEntidades paciente = PacienteDatosEF.CargarPacientePorID(idPaciente);

    if (paciente == null)
        throw new Exception($"No existe el paciente con ID {idPaciente}");

    // Cargar su historial de enfermedades ordenado por fecha
    var historial = Paciente_Enfermedades_Datos
        .CargarDetallePorPaciente(idPaciente);

    // LINQ para ordenar el historial (más reciente primero)
    paciente.ListaEnfermedades = historial
        .OrderByDescending(h => h.FechaEnfermedad)
        .ToList();

    return paciente;
}

// Uso e impresión del resumen
PacienteEntidades ficha = PacienteNegocio.ObtenerFichaCompletaPaciente(1);

Console.WriteLine($"║  FICHA DE PACIENTE — ID: {ficha.Id,-10} ║");
Console.WriteLine($"║  Nombre:   {ficha.Nombre + " " + ficha.Apellido,-26}║");
Console.WriteLine($"║  Cédula:   {ficha.Cedula,-26}║");
Console.WriteLine($"║  Género:   {ficha.Genero,-26}║");
Console.WriteLine($"║  Ciudad:   {ficha.Direccion,-26}║");
Console.WriteLine($"║  IESS:     {(ficha.Afiliado ? ficha.CodigoIEES : "No afiliado"),-26}║");
Console.WriteLine("║  HISTORIAL MÉDICO                    ║");

if (ficha.ListaEnfermedades?.Any() == true)
{
    foreach (var e in ficha.ListaEnfermedades)
    {
        Console.WriteLine($"║  • {e.Enfermedad.nombre,-15} {e.FechaEnfermedad:dd/MM/yyyy}  ║");
        if (!string.IsNullOrEmpty(e.Observacion))
            Console.WriteLine($"║    Obs: {e.Observacion,-29}║");
    }
}
else
{
    Console.WriteLine("!!!!!  Sin enfermedades registradas  !!!!!");
        Console.WriteLine("!!!!!  Disfruta de tu vida :)  !!!!!");
            Console.WriteLine("!!!!!  Que no te importe el qué dirán  !!!!!");
                Console.WriteLine("!!!!!  Te quiero mucho  !!!!!");
                    Console.WriteLine("!!!!!  Recuerda: Tu salud es lo más importante  !!!!!");



}

```

---

## Ejercicio 5 — Buscador dinámico (como un filtro en la grilla)

**Enunciado:** Implementar un buscador que en tiempo real filtre la grilla del formulario según lo que el usuario escribe en un TextBox.

```csharp
// En Form_Paciente.cs — agregar un TextBox llamado txtb_Buscar

// Guardar la lista completa para no re-consultar la BD en cada keystroke
private List<PacienteEntidades> _listaPacientesCompleta;

private void Form_Paciente_Load(object sender, EventArgs e)
{
    _listaPacientesCompleta = PacienteNegocio.DevolverListaPaciente();
    dataGridView1.DataSource = _listaPacientesCompleta;
    CargarComboGenero();
}

// Evento que se dispara con cada tecla presionada
private void txtb_Buscar_TextChanged(object sender, EventArgs e)
{
    FiltrarGrilla(txtb_Buscar.Text);
}

private void FiltrarGrilla(string textoBusqueda)
{
    if (string.IsNullOrWhiteSpace(textoBusqueda))
    {
        // Sin texto: mostrar todos
        dataGridView1.DataSource = _listaPacientesCompleta;
        return;
    }

    string busqueda = textoBusqueda.ToUpper().Trim();

    // LINQ filtra la lista en memoria (ya está cargada, no va a la BD)
    var filtrados = _listaPacientesCompleta
        .Where(p =>
            (p.Nombre   ?? "").ToUpper().Contains(busqueda) ||
            (p.Apellido ?? "").ToUpper().Contains(busqueda) ||
            (p.Cedula   ?? "").Contains(busqueda)           ||
            (p.Direccion?? "").ToUpper().Contains(busqueda) ||
            (p.Genero   ?? "").ToUpper().Contains(busqueda))
        .ToList();

    dataGridView1.DataSource = filtrados;

    // Mostrar cuántos resultados hay
    label_Resultados.Text = $"{filtrados.Count} resultado(s) encontrado(s)";
}
```

---

## Ejercicio 6 — Top 3 pacientes con más enfermedades registradas

**Enunciado:** Usando LINQ, obtener los 3 pacientes que tienen más enfermedades en su historial, mostrando el nombre y el conteo.

```csharp
// Datos (en producción vendrían de la BD)
var pacientes  = PacienteNegocio.DevolverListaPaciente();
var historial  = Paciente_Enfermedades_Negocio.CargarDetallePorPaciente(0);
// En producción cargarías todo el historial, aquí usamos lista local

List<Paciente_Enfermedad_Entidades> todosLosDetalles = new List<Paciente_Enfermedad_Entidades>
{
    new Paciente_Enfermedad_Entidades(1, 1, new EnfermedadEntidades(1, "Diabetes"),     new DateTime(2022, 1, 10), ""),
    new Paciente_Enfermedad_Entidades(2, 1, new EnfermedadEntidades(2, "Hipertensión"),new DateTime(2023, 3, 5),  ""),
    new Paciente_Enfermedad_Entidades(3, 1, new EnfermedadEntidades(3, "Asma"),        new DateTime(2021, 8, 20), ""),
    new Paciente_Enfermedad_Entidades(4, 2, new EnfermedadEntidades(1, "Diabetes"),    new DateTime(2020, 5, 15), ""),
    new Paciente_Enfermedad_Entidades(5, 2, new EnfermedadEntidades(4, "Artritis"),    new DateTime(2019, 11, 3), ""),
    new Paciente_Enfermedad_Entidades(6, 3, new EnfermedadEntidades(2, "Hipertensión"),new DateTime(2024, 2, 28), ""),
};

// Join historial con pacientes, agrupar y contar
var top3 = todosLosDetalles
    .Join(pacientes,
          detalle  => detalle.Id_Paciente,
          paciente => paciente.Id,
          (detalle, paciente) => new { paciente, detalle })
    .GroupBy(x => new { x.paciente.Id, NombreCompleto = $"{x.paciente.Nombre} {x.paciente.Apellido}" })
    .Select(g => new
    {
        g.Key.NombreCompleto,
        CantidadEnfermedades = g.Count()
    })
    .OrderByDescending(x => x.CantidadEnfermedades)
    .Take(3);

Console.WriteLine("=== Top 3 pacientes con más enfermedades ===");
int posicion = 1;
foreach (var item in top3)
    Console.WriteLine($"  #{posicion++} {item.NombreCompleto} — {item.CantidadEnfermedades} enfermedad(es)");

// #1 Carlos Pérez — 3 enfermedades
// #2 María González — 2 enfermedades
// #3 Juan Torres — 1 enfermedad
```

---

## Ejercicio 7 — Exportar lista a formato CSV con LINQ

**Enunciado:** Generar un archivo CSV con todos los pacientes usando LINQ para construir las filas.

```csharp
public static void ExportarPacientesCSV(List<PacienteEntidades> pacientes, string rutaArchivo)
{
    // Encabezado del CSV
    string encabezado = "ID,Nombre,Apellido,Cedula,Genero,FechaNacimiento,Telefono,Ciudad,Afiliado,CodigoIEES";

    // LINQ construye cada fila como string
    var filas = pacientes.Select(p =>
        $"{p.Id}," +
        $"{p.Nombre}," +
        $"{p.Apellido}," +
        $"{p.Cedula}," +
        $"{p.Genero}," +
        $"{p.FechaNacimiento:dd/MM/yyyy}," +
        $"{p.Telefono}," +
        $"{p.Direccion}," +
        $"{(p.Afiliado ? "Sí" : "No")}," +
        $"{p.CodigoIEES ?? ""}"
    );

    // Unir encabezado + filas en un solo string con saltos de línea
    string contenidoCSV = encabezado + Environment.NewLine +
                          string.Join(Environment.NewLine, filas);

    File.WriteAllText(rutaArchivo, contenidoCSV, System.Text.Encoding.UTF8);

    Console.WriteLine($"Exportados {pacientes.Count} pacientes a: {rutaArchivo}");
}

// Uso desde el formulario (botón "Exportar")
private void btn_Exportar_Click(object sender, EventArgs e)
{
    using (SaveFileDialog sfd = new SaveFileDialog())
    {
        sfd.Filter   = "Archivo CSV|*.csv";
        sfd.FileName = $"pacientes_{DateTime.Now:yyyyMMdd_HHmm}.csv";

        if (sfd.ShowDialog() == DialogResult.OK)
        {
            var lista = PacienteNegocio.DevolverListaPaciente();
            ExportarPacientesCSV(lista, sfd.FileName);
            MessageBox.Show($"Archivo guardado correctamente:\n{sfd.FileName}");
        }
    }
}
```

---

## Ejercicio 8 — Estadísticas globales del sistema

**Enunciado:** Calcular métricas generales del sistema que podrían mostrarse en un dashboard.

```csharp
public static void MostrarEstadisticasGlobales(
    List<PacienteEntidades> pacientes,
    List<Paciente_Enfermedad_Entidades> historial)
{
    int totalPacientes = pacientes.Count;
    int afiliados      = pacientes.Count(p => p.Afiliado);
    int noAfiliados    = totalPacientes - afiliados;

    double edadPromedio = pacientes
        .Average(p => DateTime.Now.Year - p.FechaNacimiento.Year);

    var pacienteMasJoven  = pacientes.OrderByDescending(p => p.FechaNacimiento).First();
    var pacienteMasAnciano= pacientes.OrderBy(p => p.FechaNacimiento).First();

    int totalRegistrosHistorial = historial.Count;

    string enfermedadMasFrecuente = historial
        .GroupBy(h => h.Enfermedad.nombre)
        .OrderByDescending(g => g.Count())
        .Select(g => $"{g.Key} ({g.Count()} caso(s))")
        .FirstOrDefault() ?? "N/A";

    string ciudadConMasPacientes = pacientes
        .GroupBy(p => p.Direccion)
        .OrderByDescending(g => g.Count())
        .Select(g => $"{g.Key} ({g.Count()} paciente(s))")
        .FirstOrDefault() ?? "N/A";

    // Imprimir dashboard de texto
    Console.WriteLine("║        ESTADÍSTICAS DEL SISTEMA          ║");
    Console.WriteLine($"║  Total pacientes     : {totalPacientes,-19}║");
    Console.WriteLine($"║  Afiliados IESS      : {afiliados,-19}║");
    Console.WriteLine($"║  No afiliados        : {noAfiliados,-19}║");
    Console.WriteLine($"║  Edad promedio       : {Math.Round(edadPromedio,1),-19}║");
    Console.WriteLine($"║  Paciente más joven  : {pacienteMasJoven.Nombre,-19}║");
    Console.WriteLine($"║  Paciente más anciano: {pacienteMasAnciano.Nombre,-19}║");
    Console.WriteLine($"║  Registros historial : {totalRegistrosHistorial,-19}║");
    Console.WriteLine($"║  Enfermedad frecuente: {enfermedadMasFrecuente,-19}║");
    Console.WriteLine($"║  Ciudad con + pacient: {ciudadConMasPacientes,-19}║");
}
```

---

## Resumen de operadores LINQ usados en los ejercicios

| Operador | Ejercicio | Para qué se usó |
|----------|-----------|----------------|
| `Where` | 1, 2, 5 | Filtrar por condición |
| `OrderBy / OrderByDescending` | 1, 4, 6, 8 | Ordenar resultados |
| `Select` | 1, 2, 7 | Proyectar / transformar datos |
| `GroupBy` | 2, 6, 8 | Agrupar por campo |
| `FirstOrDefault` | 3, 8 | Obtener un único registro |
| `Any` | 3, 4 | Verificar si existe algo |
| `Count` | 2, 8 | Contar registros |
| `Average` | 8 | Calcular promedio |
| `Join` | 6 | Cruzar dos listas |
| `Take` | 6 | Limitar resultados |
| `ToList` | todos | Materializar la consulta |
| `string.Join` | 7 | Unir strings del resultado |

---

[← Errores Comunes](../10-errores-comunes/README.md) | [Siguiente: Git Workflow →](../12-git-workflow/README.md)

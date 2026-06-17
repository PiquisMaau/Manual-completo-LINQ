# 08 — Consultas LINQ Avanzadas aplicadas al Proyecto

Todas las consultas de este capítulo usan las **entidades reales del proyecto** (`PacienteEntidades`, `GeneroEntidades`, `EnfermedadEntidades`, `Paciente_Enfermedad_Entidades`).

---

## Datos de ejemplo del proyecto

```csharp
using GestionUsuariosEntidades;
using System;
using System.Collections.Generic;
using System.Linq;

// Simulamos los datos que normalmente vendrían de la BD
List<PacienteEntidades> pacientes = new List<PacienteEntidades>
{
    new PacienteEntidades(1,  1, "Masculino", "Carlos",    "Pérez",    "1800123456", new DateTime(1985, 3, 15),  "0991234567", "Ambato",  true,  "IESS-001"),
    new PacienteEntidades(2,  2, "Femenino",  "María",     "González", "1800234567", new DateTime(1992, 7, 22),  "0992345678", "Quito",   false, ""),
    new PacienteEntidades(3,  1, "Masculino", "Juan",      "Torres",   "1800345678", new DateTime(1978, 11, 5),  "0993456789", "Guayaquil",true, "IESS-002"),
    new PacienteEntidades(4,  2, "Femenino",  "Ana",       "Villalba", "1800456789", new DateTime(2001, 2, 28),  "0994567890", "Ambato",  false, ""),
    new PacienteEntidades(5,  1, "Masculino", "Pedro",     "Salazar",  "1800567890", new DateTime(1965, 9, 10),  "0995678901", "Cuenca",  true,  "IESS-003"),
    new PacienteEntidades(6,  2, "Femenino",  "Lucía",     "Mora",     "1800678901", new DateTime(1990, 4, 18),  "0996789012", "Ambato",  true,  "IESS-004"),
    new PacienteEntidades(7,  1, "Masculino", "Roberto",   "Cáceres",  "1800789012", new DateTime(1955, 12, 30), "0997890123", "Loja",    false, ""),
};

List<Paciente_Enfermedad_Entidades> historial = new List<Paciente_Enfermedad_Entidades>
{
    new Paciente_Enfermedad_Entidades(1, 1, new EnfermedadEntidades(1, "Diabetes"),    new DateTime(2022, 1, 10),  "Control mensual"),
    new Paciente_Enfermedad_Entidades(2, 1, new EnfermedadEntidades(2, "Hipertensión"),new DateTime(2023, 3, 5),   "Medicación permanente"),
    new Paciente_Enfermedad_Entidades(3, 2, new EnfermedadEntidades(3, "Asma"),        new DateTime(2021, 8, 20),  "Inhalador"),
    new Paciente_Enfermedad_Entidades(4, 3, new EnfermedadEntidades(1, "Diabetes"),    new DateTime(2020, 5, 15),  "Dieta estricta"),
    new Paciente_Enfermedad_Entidades(5, 5, new EnfermedadEntidades(2, "Hipertensión"),new DateTime(2019, 11, 3),  "Control semanal"),
    new Paciente_Enfermedad_Entidades(6, 6, new EnfermedadEntidades(4, "Artritis"),    new DateTime(2024, 2, 28),  "Fisioterapia"),
};
```

---

## 1. Where — Filtrar pacientes afiliados al IESS

```csharp
// Pacientes que tienen afiliación al IESS
var afiliadosIESS = pacientes.Where(p => p.Afiliado == true);

Console.WriteLine("=== Afiliados al IESS ===");
foreach (var p in afiliadosIESS)
    Console.WriteLine($"  {p.Nombre} {p.Apellido} — Código: {p.CodigoIEES}");

// Output:
// Carlos Pérez — Código: IESS-001
// Juan Torres  — Código: IESS-002
// Pedro Salazar — Código: IESS-003
// Lucía Mora   — Código: IESS-004
```

---

## 2. Where + múltiples condiciones — Pacientes de Ambato no afiliados

```csharp
var ambatosNoAfiliados = pacientes
    .Where(p => p.Direccion == "Ambato" && p.Afiliado == false);

foreach (var p in ambatosNoAfiliados)
    Console.WriteLine($"  {p.Nombre} {p.Apellido}");
// Ana Villalba
```

---

## 3. Select — Proyectar solo nombre completo

```csharp
// Crear un objeto anónimo con datos calculados
var resumenPacientes = pacientes.Select(p => new
{
    NombreCompleto = $"{p.Nombre} {p.Apellido}",
    Cedula         = p.Cedula,
    Edad           = DateTime.Now.Year - p.FechaNacimiento.Year,
    EsAfiliado     = p.Afiliado ? "Sí" : "No"
});

Console.WriteLine("=== Resumen de Pacientes ===");
foreach (var r in resumenPacientes)
    Console.WriteLine($"  {r.NombreCompleto} | Cédula: {r.Cedula} | Edad: {r.Edad} | IESS: {r.EsAfiliado}");
```

---

## 4. OrderBy — Listar pacientes por apellido

```csharp
// Como aparecen en el DataGridView ordenados alfabéticamente
var ordenadosPorApellido = pacientes
    .OrderBy(p => p.Apellido)
    .ThenBy(p => p.Nombre);

Console.WriteLine("=== Lista ordenada ===");
foreach (var p in ordenadosPorApellido)
    Console.WriteLine($"  {p.Apellido}, {p.Nombre}");

// Cáceres, Roberto
// González, María
// Mora, Lucía
// Pérez, Carlos
// Salazar, Pedro
// Torres, Juan
// Villalba, Ana
```

---

## 5. GroupBy — Agrupar pacientes por género

```csharp
var porGenero = pacientes.GroupBy(p => p.Genero);

foreach (var grupo in porGenero)
{
    Console.WriteLine($"\n👥 {grupo.Key}: {grupo.Count()} pacientes");
    foreach (var p in grupo)
        Console.WriteLine($"   - {p.Nombre} {p.Apellido}");
}

// 👥 Masculino: 4 pacientes
//    - Carlos Pérez
//    - Juan Torres
//    - Pedro Salazar
//    - Roberto Cáceres
//
// 👥 Femenino: 3 pacientes
//    - María González
//    - Ana Villalba
//    - Lucía Mora
```

---

## 6. GroupBy con proyección — Estadísticas por ciudad

```csharp
var statsPorCiudad = pacientes
    .GroupBy(p => p.Direccion)
    .Select(g => new
    {
        Ciudad        = g.Key,
        TotalPacientes= g.Count(),
        Afiliados     = g.Count(p => p.Afiliado),
        NoAfiliados   = g.Count(p => !p.Afiliado)
    })
    .OrderByDescending(x => x.TotalPacientes);

Console.WriteLine("=== Estadísticas por ciudad ===");
foreach (var s in statsPorCiudad)
    Console.WriteLine($"  {s.Ciudad}: {s.TotalPacientes} pacientes " +
                      $"(IESS: {s.Afiliados} | Sin IESS: {s.NoAfiliados})");

// Ambato: 3 pacientes (IESS: 2 | Sin IESS: 1)
// Quito: 1 pacientes (IESS: 0 | Sin IESS: 1)
// Guayaquil: 1 pacientes (IESS: 1 | Sin IESS: 0)
// Cuenca: 1 pacientes (IESS: 1 | Sin IESS: 0)
// Loja: 1 pacientes (IESS: 0 | Sin IESS: 1)
```

---

## 7. Join — Cruzar pacientes con su historial de enfermedades

```csharp
// Equivalent de la consulta que hace EF con Include("Paciente")
var pacientesConEnfermedad = historial.Join(
    pacientes,
    h => h.Id_Paciente,          // Clave del historial
    p => p.Id,                   // Clave del paciente
    (h, p) => new
    {
        Paciente    = $"{p.Nombre} {p.Apellido}",
        Enfermedad  = h.Enfermedad.nombre,
        Fecha       = h.FechaEnfermedad.ToString("dd/MM/yyyy"),
        Observacion = h.Observacion
    }
);

Console.WriteLine("=== Historial médico ===");
foreach (var item in pacientesConEnfermedad)
    Console.WriteLine($"  {item.Paciente} | {item.Enfermedad} | {item.Fecha} | {item.Observacion}");
```

---

## 8. Any y All — Validaciones de negocio

```csharp
// ¿Hay algún paciente mayor de 70 años?
bool hayAncianosEnSistema = pacientes.Any(
    p => (DateTime.Now.Year - p.FechaNacimiento.Year) > 70);

Console.WriteLine($"¿Hay pacientes mayores de 70? {hayAncianosEnSistema}"); // True

// ¿Todos los pacientes afiliados tienen código IESS?
bool todosAfiliadosTienenCodigo = pacientes
    .Where(p => p.Afiliado)
    .All(p => !string.IsNullOrEmpty(p.CodigoIEES));

Console.WriteLine($"¿Todos afiliados tienen código? {todosAfiliadosTienenCodigo}"); // True

// ¿Existe un paciente con esta cédula? (validación antes de guardar)
string cedulaBuscada = "1800123456";
bool cedulaYaExiste = pacientes.Any(p => p.Cedula == cedulaBuscada);
Console.WriteLine($"¿Cédula {cedulaBuscada} ya registrada? {cedulaYaExiste}"); // True
```

---

## 9. Count y Aggregate — Métricas del sistema

```csharp
// Totales generales
int totalPacientes    = pacientes.Count();
int totalAfiliados    = pacientes.Count(p => p.Afiliado);
int totalNoAfiliados  = pacientes.Count(p => !p.Afiliado);

Console.WriteLine($"Total pacientes: {totalPacientes}");
Console.WriteLine($"Afiliados IESS: {totalAfiliados}");
Console.WriteLine($"Sin IESS: {totalNoAfiliados}");

// Edad promedio de los pacientes
double edadPromedio = pacientes
    .Average(p => DateTime.Now.Year - p.FechaNacimiento.Year);

Console.WriteLine($"Edad promedio: {Math.Round(edadPromedio, 1)} años");

// Paciente más joven y más anciano
var masJoven  = pacientes.OrderByDescending(p => p.FechaNacimiento).First();
var masAnciano= pacientes.OrderBy(p => p.FechaNacimiento).First();

Console.WriteLine($"Más joven:  {masJoven.Nombre} {masJoven.Apellido} ({masJoven.FechaNacimiento.Year})");
Console.WriteLine($"Más anciano:{masAnciano.Nombre} {masAnciano.Apellido} ({masAnciano.FechaNacimiento.Year})");
```

---

## 10. Paginación — Como en el DataGridView con muchos registros

```csharp
// Simular un DataGridView paginado
int paginaActual     = 1;
int registrosPorPagina = 3;

var paginaPacientes = pacientes
    .OrderBy(p => p.Apellido)
    .Skip((paginaActual - 1) * registrosPorPagina)
    .Take(registrosPorPagina)
    .ToList();

Console.WriteLine($"=== Página {paginaActual} de {Math.Ceiling((double)pacientes.Count / registrosPorPagina)} ===");
foreach (var p in paginaPacientes)
    Console.WriteLine($"  [{p.Id}] {p.Apellido}, {p.Nombre}");
```

---

## 11. SelectMany — Obtener todas las enfermedades de los pacientes afiliados

```csharp
// Lista de pacientes afiliados con su historial (objetos anidados)
var pacientesConHistorial = pacientes
    .Where(p => p.Afiliado)
    .Select(p => new
    {
        Paciente = p,
        // Simula el historial (en el proyecto real viene de la BD)
        Enfermedades = historial.Where(h => h.Id_Paciente == p.Id).ToList()
    });

// SelectMany "aplana" todas las enfermedades en una sola lista
var todasEnfermedadesAfiliados = pacientesConHistorial
    .SelectMany(p => p.Enfermedades,
                (pacienteConHistorial, enfermedad) => new
                {
                    Paciente   = $"{pacienteConHistorial.Paciente.Nombre} {pacienteConHistorial.Paciente.Apellido}",
                    Enfermedad = enfermedad.Enfermedad.nombre,
                    Fecha      = enfermedad.FechaEnfermedad
                });

foreach (var item in todasEnfermedadesAfiliados)
    Console.WriteLine($"  {item.Paciente} — {item.Enfermedad} ({item.Fecha:dd/MM/yyyy})");
```

---

## 12. Distinct — Obtener ciudades únicas para un filtro

```csharp
// Lista de ciudades sin repetir (para un ComboBox de filtro)
var ciudadesUnicas = pacientes
    .Select(p => p.Direccion)
    .Distinct()
    .OrderBy(c => c)
    .ToList();

Console.WriteLine("Ciudades disponibles para filtrar:");
ciudadesUnicas.ForEach(c => Console.WriteLine($"  - {c}"));

// Ambato, Cuenca, Guayaquil, Loja, Quito
```

---

## 13. Búsqueda flexible — Como un buscador en el formulario

```csharp
// Simulación de búsqueda en tiempo real con múltiples parámetros opcionales
public static List<PacienteEntidades> BuscarPacientes(
    List<PacienteEntidades> todos,
    string textoBusqueda  = null,
    string ciudadFiltro   = null,
    bool?  soloAfiliados  = null,
    int?   edadMinima     = null)
{
    var query = todos.AsEnumerable();

    // Filtro por texto (nombre o apellido o cédula)
    if (!string.IsNullOrWhiteSpace(textoBusqueda))
    {
        string busqueda = textoBusqueda.ToUpper();
        query = query.Where(p =>
            p.Nombre.ToUpper().Contains(busqueda)   ||
            p.Apellido.ToUpper().Contains(busqueda) ||
            p.Cedula.Contains(busqueda));
    }

    // Filtro por ciudad
    if (!string.IsNullOrWhiteSpace(ciudadFiltro))
        query = query.Where(p => p.Direccion == ciudadFiltro);

    // Filtro por afiliación
    if (soloAfiliados.HasValue)
        query = query.Where(p => p.Afiliado == soloAfiliados.Value);

    // Filtro por edad mínima
    if (edadMinima.HasValue)
        query = query.Where(p =>
            (DateTime.Now.Year - p.FechaNacimiento.Year) >= edadMinima.Value);

    return query.OrderBy(p => p.Apellido).ToList();
}

// Uso:
var resultados = BuscarPacientes(pacientes, textoBusqueda: "ar", ciudadFiltro: "Ambato");
foreach (var p in resultados)
    Console.WriteLine($"{p.Nombre} {p.Apellido} — {p.Direccion}");
// Carlos Pérez — Ambato
```

---

## 14. Enfermedad más frecuente en el historial

```csharp
var enfermedadMasFrecuente = historial
    .GroupBy(h => h.Enfermedad.nombre)
    .Select(g => new { Enfermedad = g.Key, Casos = g.Count() })
    .OrderByDescending(x => x.Casos)
    .FirstOrDefault();

Console.WriteLine($"Enfermedad más frecuente: {enfermedadMasFrecuente?.Enfermedad} " +
                  $"({enfermedadMasFrecuente?.Casos} casos)");
// Enfermedad más frecuente: Diabetes (2 casos)
```

---

[← Presentación](../07-presentacion/README.md) | [← Volver al inicio](../README.md)

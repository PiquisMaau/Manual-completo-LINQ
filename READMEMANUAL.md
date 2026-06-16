# 📘 Manual Completo de LINQ en C#

> **Language Integrated Query (LINQ)** es una característica de C# que permite realizar consultas directamente sobre colecciones, bases de datos, XML y más, usando una sintaxis unificada y tipada.

---

## 📑 Índice

1. [Introducción a LINQ](#1-introducción-a-linq)
2. [Sintaxis: Query vs Method](#2-sintaxis-query-vs-method)
3. [Filtrado con Where](#3-filtrado-con-where)
4. [Proyección con Select](#4-proyección-con-select)
5. [Ordenamiento](#5-ordenamiento)
6. [Agrupación con GroupBy](#6-agrupación-con-groupby)
7. [Joins](#7-joins)
8. [Agregación](#8-agregación)
9. [Operaciones de Conjuntos](#9-operaciones-de-conjuntos)
10. [Cuantificadores](#10-cuantificadores)
11. [Paginación con Skip y Take](#11-paginación-con-skip-y-take)
12. [LINQ to XML](#12-linq-to-xml)
13. [LINQ to Entities (Entity Framework)](#13-linq-to-entities-entity-framework)
14. [Buenas Prácticas](#14-buenas-prácticas)
15. [Ejercicios Propuestos](#15-ejercicios-propuestos)

---

## 1. Introducción a LINQ

LINQ (Language Integrated Query) fue introducido en C# 3.0 y .NET 3.5. Permite consultar cualquier fuente de datos que implemente `IEnumerable<T>` o `IQueryable<T>`.

### Namespaces necesarios

```csharp
using System;
using System.Linq;
using System.Collections.Generic;
using System.Xml.Linq;   // Para LINQ to XML
```

### Modelo de datos usado en los ejemplos

```csharp
public class Producto
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public string Categoria { get; set; }
    public decimal Precio { get; set; }
    public int Stock { get; set; }
}

public class Categoria
{
    public int Id { get; set; }
    public string Nombre { get; set; }
    public string Descripcion { get; set; }
}

// Datos de ejemplo
List<Producto> productos = new List<Producto>
{
    new Producto { Id = 1, Nombre = "Laptop",    Categoria = "Electrónica", Precio = 1200m, Stock = 10 },
    new Producto { Id = 2, Nombre = "Mouse",     Categoria = "Electrónica", Precio = 25m,   Stock = 50 },
    new Producto { Id = 3, Nombre = "Teclado",   Categoria = "Electrónica", Precio = 75m,   Stock = 30 },
    new Producto { Id = 4, Nombre = "Escritorio",Categoria = "Muebles",     Precio = 450m,  Stock = 5  },
    new Producto { Id = 5, Nombre = "Silla",     Categoria = "Muebles",     Precio = 200m,  Stock = 15 },
    new Producto { Id = 6, Nombre = "Monitor",   Categoria = "Electrónica", Precio = 350m,  Stock = 20 },
    new Producto { Id = 7, Nombre = "Cuaderno",  Categoria = "Papelería",   Precio = 5m,    Stock = 200},
    new Producto { Id = 8, Nombre = "Bolígrafo", Categoria = "Papelería",   Precio = 1.5m,  Stock = 500},
};
```

---

## 2. Sintaxis: Query vs Method

LINQ tiene dos formas de escribirse: **Query Syntax** (parecida a SQL) y **Method Syntax** (usando métodos de extensión). Ambas producen el mismo resultado.

### Query Syntax

```csharp
// Obtener productos con precio mayor a 100
var resultado = from p in productos
                where p.Precio > 100
                select p;

foreach (var p in resultado)
    Console.WriteLine($"{p.Nombre} - ${p.Precio}");
```

### Method Syntax (Fluent)

```csharp
// Equivalente con método de extensión
var resultado = productos
    .Where(p => p.Precio > 100)
    .Select(p => p);

foreach (var p in resultado)
    Console.WriteLine($"{p.Nombre} - ${p.Precio}");
```

### ¿Cuál usar?

| Característica | Query Syntax | Method Syntax |
|---------------|--------------|---------------|
| Legibilidad   | Más legible para SQL-lovers | Más compacto |
| Flexibilidad  | Limitada      | Muy flexible  |
| Encadenado    | Difícil       | Fácil         |
| Soporte completo | No todos los métodos | Sí |

> 💡 **Recomendación:** Usa Method Syntax para operaciones simples y encadenadas. Query Syntax es útil para joins complejos.

---

## 3. Filtrado con Where

`Where` filtra elementos según una condición (predicado).

### Filtro simple

```csharp
// Productos de la categoría Electrónica
var electronicos = productos.Where(p => p.Categoria == "Electrónica");

foreach (var p in electronicos)
    Console.WriteLine(p.Nombre);
// Laptop, Mouse, Teclado, Monitor
```

### Filtros múltiples (AND)

```csharp
// Electrónicos con precio entre 50 y 400
var resultado = productos.Where(p => 
    p.Categoria == "Electrónica" && 
    p.Precio >= 50 && 
    p.Precio <= 400);

foreach (var p in resultado)
    Console.WriteLine($"{p.Nombre}: ${p.Precio}");
// Teclado: $75, Monitor: $350
```

### Filtros con OR

```csharp
// Productos baratos O con mucho stock
var resultado = productos.Where(p => p.Precio < 10 || p.Stock > 100);

foreach (var p in resultado)
    Console.WriteLine($"{p.Nombre} - Precio: ${p.Precio} - Stock: {p.Stock}");
// Cuaderno, Bolígrafo
```

### Filtro con índice

```csharp
// Tomar solo los primeros 3 productos que cuesten más de 50
var resultado = productos
    .Where((p, index) => p.Precio > 50)
    .Take(3);

foreach (var p in resultado)
    Console.WriteLine($"[{p.Id}] {p.Nombre}");
```

---

## 4. Proyección con Select

`Select` transforma cada elemento en algo diferente (mapeo).

### Proyección simple

```csharp
// Solo los nombres
var nombres = productos.Select(p => p.Nombre);

foreach (var nombre in nombres)
    Console.WriteLine(nombre);
```

### Tipo anónimo

```csharp
// Crear objetos con propiedades específicas
var resumen = productos.Select(p => new 
{
    p.Nombre,
    p.Precio,
    EsCaro = p.Precio > 500
});

foreach (var item in resumen)
    Console.WriteLine($"{item.Nombre} | ${item.Precio} | ¿Caro? {item.EsCaro}");
```

### Proyección con cálculo

```csharp
// Precio con descuento del 10%
var conDescuento = productos.Select(p => new
{
    p.Nombre,
    PrecioOriginal = p.Precio,
    PrecioConDescuento = Math.Round(p.Precio * 0.90m, 2)
});

foreach (var item in conDescuento)
    Console.WriteLine($"{item.Nombre}: ${item.PrecioOriginal} → ${item.PrecioConDescuento}");
```

### SelectMany (aplanar colecciones)

```csharp
// Ejemplo con listas anidadas
var pedidos = new List<List<string>>
{
    new List<string> { "Laptop", "Mouse" },
    new List<string> { "Silla", "Escritorio" },
    new List<string> { "Cuaderno" }
};

// Aplanar todas las listas en una sola
var todosLosItems = pedidos.SelectMany(orden => orden);

foreach (var item in todosLosItems)
    Console.WriteLine(item);
// Laptop, Mouse, Silla, Escritorio, Cuaderno
```

---

## 5. Ordenamiento

### OrderBy y OrderByDescending

```csharp
// Ordenar por precio ascendente
var porPrecioAsc = productos.OrderBy(p => p.Precio);

Console.WriteLine("=== Por precio (menor a mayor) ===");
foreach (var p in porPrecioAsc)
    Console.WriteLine($"{p.Nombre}: ${p.Precio}");

// Ordenar por precio descendente
var porPrecioDesc = productos.OrderByDescending(p => p.Precio);

Console.WriteLine("\n=== Por precio (mayor a menor) ===");
foreach (var p in porPrecioDesc)
    Console.WriteLine($"{p.Nombre}: ${p.Precio}");
```

### ThenBy (criterio secundario)

```csharp
// Ordenar por categoría, luego por nombre
var ordenMultiple = productos
    .OrderBy(p => p.Categoria)
    .ThenBy(p => p.Nombre);

foreach (var p in ordenMultiple)
    Console.WriteLine($"{p.Categoria} | {p.Nombre}");
```

### Ordenamiento con Query Syntax

```csharp
var resultado = from p in productos
                orderby p.Categoria ascending, p.Precio descending
                select p;

foreach (var p in resultado)
    Console.WriteLine($"{p.Categoria} | {p.Nombre} | ${p.Precio}");
```

---

## 6. Agrupación con GroupBy

`GroupBy` agrupa elementos por una clave.

### Agrupación básica

```csharp
var grupos = productos.GroupBy(p => p.Categoria);

foreach (var grupo in grupos)
{
    Console.WriteLine($"\n📦 Categoría: {grupo.Key}");
    foreach (var p in grupo)
        Console.WriteLine($"   - {p.Nombre}: ${p.Precio}");
}
```

### GroupBy con proyección

```csharp
// Resumen por categoría
var resumenPorCategoria = productos
    .GroupBy(p => p.Categoria)
    .Select(g => new
    {
        Categoria    = g.Key,
        TotalProductos = g.Count(),
        PrecioPromedio = Math.Round(g.Average(p => p.Precio), 2),
        PrecioMinimo   = g.Min(p => p.Precio),
        PrecioMaximo   = g.Max(p => p.Precio)
    });

foreach (var r in resumenPorCategoria)
{
    Console.WriteLine($"Categoría: {r.Categoria}");
    Console.WriteLine($"  Productos: {r.TotalProductos}");
    Console.WriteLine($"  Precio promedio: ${r.PrecioPromedio}");
    Console.WriteLine($"  Rango: ${r.PrecioMinimo} - ${r.PrecioMaximo}");
}
```

### Clave compuesta

```csharp
// Agrupar por categoría Y si es caro (>100)
var gruposCompuestos = productos.GroupBy(p => new 
{ 
    p.Categoria, 
    EsCaro = p.Precio > 100 
});

foreach (var grupo in gruposCompuestos)
{
    Console.WriteLine($"{grupo.Key.Categoria} - ¿Caro? {grupo.Key.EsCaro}: {grupo.Count()} productos");
}
```

---

## 7. Joins

### Join (Inner Join)

```csharp
List<Categoria> categorias = new List<Categoria>
{
    new Categoria { Id = 1, Nombre = "Electrónica",  Descripcion = "Dispositivos electrónicos" },
    new Categoria { Id = 2, Nombre = "Muebles",      Descripcion = "Mobiliario de oficina" },
    new Categoria { Id = 3, Nombre = "Papelería",    Descripcion = "Útiles de escritorio" },
    new Categoria { Id = 4, Nombre = "Deportes",     Descripcion = "Sin productos aún" }
};

List<Producto> productosConId = new List<Producto>
{
    new Producto { Id = 1, Nombre = "Laptop",  Categoria = "Electrónica", Precio = 1200m },
    new Producto { Id = 2, Nombre = "Silla",   Categoria = "Muebles",     Precio = 200m  },
    new Producto { Id = 3, Nombre = "Cuaderno",Categoria = "Papelería",   Precio = 5m    },
};

// Inner Join: solo productos que tienen categoría coincidente
var resultado = productosConId.Join(
    categorias,
    producto  => producto.Categoria,    // Clave del producto
    categoria => categoria.Nombre,      // Clave de la categoría
    (producto, categoria) => new        // Proyección del resultado
    {
        Producto    = producto.Nombre,
        Precio      = producto.Precio,
        Descripcion = categoria.Descripcion
    }
);

foreach (var item in resultado)
    Console.WriteLine($"{item.Producto} ({item.Precio}) → {item.Descripcion}");
```

### GroupJoin (Left Join)

```csharp
// Left Join: todas las categorías, con o sin productos
var leftJoin = categorias.GroupJoin(
    productosConId,
    cat  => cat.Nombre,
    prod => prod.Categoria,
    (cat, prods) => new
    {
        Categoria = cat.Nombre,
        Productos = prods.ToList()
    }
);

foreach (var item in leftJoin)
{
    Console.WriteLine($"\n📂 {item.Categoria}");
    if (item.Productos.Any())
        item.Productos.ForEach(p => Console.WriteLine($"   - {p.Nombre}"));
    else
        Console.WriteLine("   (Sin productos)");
}
```

### Join con Query Syntax

```csharp
var resultado = from p in productosConId
                join c in categorias on p.Categoria equals c.Nombre
                select new
                {
                    Producto    = p.Nombre,
                    Descripcion = c.Descripcion
                };

foreach (var item in resultado)
    Console.WriteLine($"{item.Producto}: {item.Descripcion}");
```

---

## 8. Agregación

### Métodos de agregación básicos

```csharp
// Count
int totalProductos = productos.Count();
Console.WriteLine($"Total productos: {totalProductos}");

// Count con condición
int productosCaros = productos.Count(p => p.Precio > 200);
Console.WriteLine($"Productos caros (>$200): {productosCaros}");

// Sum
decimal totalInventario = productos.Sum(p => p.Precio * p.Stock);
Console.WriteLine($"Valor total inventario: ${totalInventario}");

// Average
decimal precioPromedio = productos.Average(p => p.Precio);
Console.WriteLine($"Precio promedio: ${Math.Round(precioPromedio, 2)}");

// Min y Max
decimal precioMin = productos.Min(p => p.Precio);
decimal precioMax = productos.Max(p => p.Precio);
Console.WriteLine($"Precio mínimo: ${precioMin}");
Console.WriteLine($"Precio máximo: ${precioMax}");
```

### Aggregate (acumulador personalizado)

```csharp
// Concatenar nombres con separador
string nombresUnidos = productos
    .Select(p => p.Nombre)
    .Aggregate((actual, siguiente) => $"{actual}, {siguiente}");

Console.WriteLine($"Productos: {nombresUnidos}");
// Laptop, Mouse, Teclado, Escritorio, Silla, Monitor, Cuaderno, Bolígrafo

// Aggregate con semilla (valor inicial)
decimal totalConImpuesto = productos.Aggregate(
    0m,                                       // Semilla (acumulador inicial)
    (acum, p) => acum + (p.Precio * 1.12m)   // 12% de impuesto
);
Console.WriteLine($"Total con 12% impuesto: ${Math.Round(totalConImpuesto, 2)}");
```

---

## 9. Operaciones de Conjuntos

```csharp
var lista1 = new List<int> { 1, 2, 3, 4, 5 };
var lista2 = new List<int> { 3, 4, 5, 6, 7 };

// Distinct: elimina duplicados
var sinDuplicados = new List<int> { 1, 1, 2, 3, 3, 4 }.Distinct();
Console.WriteLine("Distinct: " + string.Join(", ", sinDuplicados));
// 1, 2, 3, 4

// Union: todos los elementos únicos de ambas listas
var union = lista1.Union(lista2);
Console.WriteLine("Union: " + string.Join(", ", union));
// 1, 2, 3, 4, 5, 6, 7

// Intersect: solo los elementos comunes
var interseccion = lista1.Intersect(lista2);
Console.WriteLine("Intersect: " + string.Join(", ", interseccion));
// 3, 4, 5

// Except: elementos de lista1 que NO están en lista2
var diferencia = lista1.Except(lista2);
Console.WriteLine("Except: " + string.Join(", ", diferencia));
// 1, 2
```

### Concat (sin eliminar duplicados)

```csharp
var concatenado = lista1.Concat(lista2);
Console.WriteLine("Concat: " + string.Join(", ", concatenado));
// 1, 2, 3, 4, 5, 3, 4, 5, 6, 7
```

---

## 10. Cuantificadores

Devuelven `bool`. Útiles para validaciones.

```csharp
// Any: ¿Existe al menos uno que cumpla la condición?
bool hayProductosCaros = productos.Any(p => p.Precio > 1000);
Console.WriteLine($"¿Hay productos >$1000? {hayProductosCaros}"); // True

bool hayProductosSinStock = productos.Any(p => p.Stock == 0);
Console.WriteLine($"¿Hay sin stock? {hayProductosSinStock}"); // False

// All: ¿Todos cumplen la condición?
bool todosConStock = productos.All(p => p.Stock > 0);
Console.WriteLine($"¿Todos tienen stock? {todosConStock}"); // True

bool todosMenosDe2000 = productos.All(p => p.Precio < 2000);
Console.WriteLine($"¿Todos < $2000? {todosMenosDe2000}"); // True

// Contains: ¿Existe un elemento específico?
var nombresProductos = productos.Select(p => p.Nombre).ToList();
bool tieneLaptop = nombresProductos.Contains("Laptop");
Console.WriteLine($"¿Tiene Laptop? {tieneLaptop}"); // True
```

---

## 11. Paginación con Skip y Take

Ideal para implementar paginación en aplicaciones.

```csharp
int paginaActual = 2;
int elementosPorPagina = 3;

var pagina = productos
    .OrderBy(p => p.Id)
    .Skip((paginaActual - 1) * elementosPorPagina)  // Saltar elementos anteriores
    .Take(elementosPorPagina);                        // Tomar solo los de esta página

Console.WriteLine($"=== Página {paginaActual} ===");
foreach (var p in pagina)
    Console.WriteLine($"  [{p.Id}] {p.Nombre}");
```

### TakeLast y SkipLast (.NET Core 2.0+)

```csharp
// Los últimos 3 productos
var ultimos3 = productos.TakeLast(3);

// Todos excepto los últimos 2
var sinUltimos2 = productos.SkipLast(2);
```

### SkipWhile y TakeWhile

```csharp
// Saltar mientras el precio sea menor a 50
var resultado = productos
    .OrderBy(p => p.Precio)
    .SkipWhile(p => p.Precio < 50);

// Tomar mientras el precio sea menor a 200
var baratos = productos
    .OrderBy(p => p.Precio)
    .TakeWhile(p => p.Precio < 200);
```

---

## 12. LINQ to XML

Permite leer, crear y consultar documentos XML.

### Crear XML con LINQ

```csharp
using System.Xml.Linq;

var xmlDoc = new XDocument(
    new XElement("Inventario",
        new XAttribute("fecha", DateTime.Today.ToString("yyyy-MM-dd")),
        productos.Select(p =>
            new XElement("Producto",
                new XAttribute("id", p.Id),
                new XElement("Nombre",    p.Nombre),
                new XElement("Categoria", p.Categoria),
                new XElement("Precio",    p.Precio),
                new XElement("Stock",     p.Stock)
            )
        )
    )
);

xmlDoc.Save("inventario.xml");
Console.WriteLine(xmlDoc.ToString());
```

### Consultar XML con LINQ

```csharp
// Cargar XML desde archivo
XDocument doc = XDocument.Load("inventario.xml");

// Consultar todos los productos
var productosXml = doc.Descendants("Producto")
    .Select(p => new
    {
        Id       = (int)p.Attribute("id"),
        Nombre   = (string)p.Element("Nombre"),
        Categoria= (string)p.Element("Categoria"),
        Precio   = (decimal)p.Element("Precio")
    });

// Filtrar por categoría
var electronicos = doc.Descendants("Producto")
    .Where(p => (string)p.Element("Categoria") == "Electrónica")
    .Select(p => (string)p.Element("Nombre"));

foreach (var nombre in electronicos)
    Console.WriteLine(nombre);
```

---

## 13. LINQ to Entities (Entity Framework)

Con Entity Framework, LINQ se traduce a SQL automáticamente.

### Setup básico

```csharp
// Instalar: dotnet add package Microsoft.EntityFrameworkCore.SqlServer

using Microsoft.EntityFrameworkCore;

public class TiendaContext : DbContext
{
    public DbSet<Producto>  Productos  { get; set; }
    public DbSet<Categoria> Categorias { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlServer("Server=.;Database=Tienda;Trusted_Connection=True;");
}
```

### Consultas con EF + LINQ

```csharp
using var context = new TiendaContext();

// Consulta básica (genera SELECT * FROM Productos WHERE Precio > 100)
var productosCaros = await context.Productos
    .Where(p => p.Precio > 100)
    .OrderBy(p => p.Nombre)
    .ToListAsync();

// Proyección (genera SELECT Id, Nombre, Precio FROM ...)
var resumen = await context.Productos
    .Select(p => new { p.Id, p.Nombre, p.Precio })
    .ToListAsync();

// Join con navegación (si tienes propiedades de navegación)
var conCategoria = await context.Productos
    .Include(p => p.Categoria)  // Eager loading
    .Where(p => p.Precio > 100)
    .ToListAsync();

// Paginación eficiente
int pagina = 1, tamano = 10;
var paginado = await context.Productos
    .OrderBy(p => p.Id)
    .Skip((pagina - 1) * tamano)
    .Take(tamano)
    .ToListAsync();

// Agrupación en base de datos
var stats = await context.Productos
    .GroupBy(p => p.Categoria)
    .Select(g => new
    {
        Categoria = g.Key,
        Total     = g.Count(),
        Promedio  = g.Average(p => p.Precio)
    })
    .ToListAsync();
```

### Diferencia IQueryable vs IEnumerable

```csharp
// IQueryable<T> — Ejecución DIFERIDA en la BD (eficiente)
IQueryable<Producto> query = context.Productos.Where(p => p.Precio > 100);
// Aún no ejecuta SQL

var lista = await query.ToListAsync(); // AQUÍ ejecuta el SQL

// IEnumerable<T> — Trae TODO a memoria y filtra en C# (ineficiente)
IEnumerable<Producto> todoEnMemoria = await context.Productos.ToListAsync();
var filtrado = todoEnMemoria.Where(p => p.Precio > 100); // Filtra en C#
```

> ⚠️ **Importante:** Usa `IQueryable` para filtros con EF. Evita llamar `.ToList()` antes de aplicar filtros.

---

## 14. Buenas Prácticas

### ✅ DO — Hazlo así

```csharp
// 1. Usa var para tipos largos de LINQ
var resultado = productos.Where(p => p.Precio > 100).ToList();

// 2. Materializa cuando necesites iterar múltiples veces
var lista = productos.Where(p => p.Stock > 0).ToList();
Console.WriteLine(lista.Count);       // No re-ejecuta la query
Console.WriteLine(lista.First().Nombre); // No re-ejecuta la query

// 3. Usa métodos async con EF
var productos = await context.Productos.ToListAsync();

// 4. Valida antes de usar First
if (productos.Any(p => p.Categoria == "Deportes"))
{
    var primero = productos.First(p => p.Categoria == "Deportes");
}

// 5. Prefiere FirstOrDefault sobre First para evitar excepciones
var producto = productos.FirstOrDefault(p => p.Id == 99);
if (producto != null)
    Console.WriteLine(producto.Nombre);
```

### ❌ DON'T — Evita esto

```csharp
// 1. No llames ToList() innecesariamente antes de filtrar (con EF)
var todos = context.Productos.ToList(); // Trae TODO a memoria
var filtrado = todos.Where(p => p.Precio > 100); // Filtra en C# (malo)

// 2. No uses First() sin verificar (lanza excepción si está vacío)
var p = productos.First(p => p.Id == 999); // ❌ InvalidOperationException

// 3. No iteres la misma IEnumerable deferred múltiples veces
var query = productos.Where(p => p.Precio > 100); // Deferred
var count = query.Count();    // Ejecuta la query
var first = query.First();    // Ejecuta la query OTRA VEZ
// Solución: materializarla con .ToList() primero
```

---

## 15. Ejercicios Propuestos

Usando la lista de productos del inicio, resuelve los siguientes ejercicios:

1. **Fácil:** Obtén todos los productos con stock mayor a 20, ordenados por stock descendente.

2. **Fácil:** Muestra solo los nombres de los productos de la categoría "Papelería".

3. **Medio:** Crea un resumen de cuántos productos hay por categoría y el valor total (precio × stock) de cada una.

4. **Medio:** Encuentra los 3 productos más caros de cada categoría.

5. **Difícil:** Implementa un sistema de búsqueda que filtre por nombre (contains), categoría, rango de precio mínimo y máximo, usando parámetros opcionales.

6. **Difícil:** Usando dos listas (Productos y Categorías con IDs), haz un Left Join que muestre todas las categorías con sus productos, incluyendo las que no tienen ninguno.

---

## 📚 Recursos Adicionales

- [Documentación oficial de LINQ - Microsoft](https://docs.microsoft.com/es-es/dotnet/csharp/linq/)
- [101 ejemplos de LINQ](https://linqsamples.com/)
- [Entity Framework Core](https://docs.microsoft.com/es-es/ef/core/)

---

## 🤝 Contribuciones

¿Quieres mejorar este manual? ¡Las contribuciones son bienvenidas!

1. Haz un **Fork** del repositorio
2. Crea tu rama: `git checkout -b mejora/nuevo-tema`
3. Haz commit: `git commit -m 'feat: agrega ejemplos de LINQ paralelo'`
4. Push: `git push origin mejora/nuevo-tema`
5. Abre un **Pull Request**

---

*Manual creado con ❤️ para la comunidad de desarrolladores C#*

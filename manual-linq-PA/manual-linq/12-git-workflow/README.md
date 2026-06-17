# 12 — Flujo de Trabajo Git para el Proyecto

Cómo versionar GestionUsuariosV2 correctamente en GitHub: comandos, `.gitignore`, estructura de ramas y convenciones de commits.

---

## Estructura recomendada del repositorio

```
gestion-usuarios-v2/
├── .gitignore                          ← Archivos que NO se suben a GitHub
├── README.md                           ← Descripción del proyecto
├── GestionUsuarios.slnx                ← Solución de Visual Studio
├── GestionUsuariosEntidades/           ← Capa Entidades
├── GestionUsuariosDatos/               ← Capa Datos (ADO.NET)
├── GestionUsuariosDatosLINQ/           ← Capa Datos (LINQ to SQL)
├── GestionUsuariosDatosEF/             ← Capa Datos (Entity Framework)
├── GestionUsuariosLogicaNegocio/       ← Capa Negocio
├── GestionUsuariosPresentacion/        ← Capa Presentación (WinForms)
├── docs/                               ← Documentación adicional
│   └── manual-linq/                    ← Este manual
└── scripts/
    └── BaseDeDatos.sql                 ← Script de creación de la BD
```

---

## `.gitignore` para proyectos C# / Visual Studio

Este archivo le dice a Git qué NO versionar (binarios, configuraciones locales, carpetas generadas).

```gitignore
## Archivo: .gitignore
## Colocarlo en la raíz del repositorio

# ──────────────────────────────────────
# Carpetas de compilación (se generan al hacer build)
# ──────────────────────────────────────
[Bb]in/
[Oo]bj/
[Ll]og/
[Ll]ogs/

# ──────────────────────────────────────
# Carpeta oculta de Visual Studio
# ──────────────────────────────────────
.vs/
*.suo
*.user
*.userosscache
*.sln.docstates

# ──────────────────────────────────────
# Archivos de publicación
# ──────────────────────────────────────
[Pp]ublish/
*.[Pp]ublish.xml
*.azurePubxml
*.pubxml
*.publishproj

# ──────────────────────────────────────
# NuGet (paquetes se restauran con "dotnet restore")
# ──────────────────────────────────────
*.nupkg
*.snupkg
**/[Pp]ackages/*
!**/[Pp]ackages/build/
*.nuget.props
*.nuget.targets

# ──────────────────────────────────────
# Entity Framework — migraciones generadas localmente
# (subir solo si usas Code First con Migrations)
# ──────────────────────────────────────
# **/Migrations/

# ──────────────────────────────────────
# Connection strings con contraseñas
# IMPORTANTE: nunca subir App.config con passwords reales
# ──────────────────────────────────────
# App.config               # ← Descomentar si tiene password en texto plano
# Web.config

# ──────────────────────────────────────
# Archivos de sistema operativo
# ──────────────────────────────────────
.DS_Store
Thumbs.db
Desktop.ini

# ──────────────────────────────────────
# JetBrains Rider / ReSharper
# ──────────────────────────────────────
.idea/
_ReSharper*/
*.DotSettings.user

# ──────────────────────────────────────
# Archivos temporales
# ──────────────────────────────────────
*.tmp
*.temp
~$*
```

---

## Primer commit del proyecto desde cero

```bash
# 1. Ir a la carpeta donde está la solución .slnx
cd C:\Users\TuUsuario\source\repos\GestionUsuariosV2

# 2. Inicializar Git
git init

# 3. Agregar el .gitignore ANTES de cualquier otra cosa
#    (de lo contrario los archivos bin/ ya quedarán trackeados)
# Copiar aquí el .gitignore del bloque de arriba, luego:
git add .gitignore
git commit -m "chore: agrega .gitignore para proyecto C# Visual Studio"

# 4. Agregar todo el código fuente
git add .
git status   # Verificar qué archivos se agregarán (bin/ y obj/ NO deben aparecer)

# 5. Primer commit con el código
git commit -m "feat: proyecto inicial GestionUsuariosV2 con arquitectura en capas"

# 6. Conectar con GitHub
git remote add origin https://github.com/TU-USUARIO/gestion-usuarios-v2.git

# 7. Subir
git push -u origin main
```

---

## Convención de commits (Conventional Commits)

Formato: `tipo(alcance): descripción corta`

| Tipo | Cuándo usarlo | Ejemplo |
|------|--------------|---------|
| `feat` | Nueva funcionalidad | `feat: agrega búsqueda por cédula en grilla` |
| `fix` | Corrección de error | `fix: corrige NullReferenceException al clic en encabezado` |
| `refactor` | Mejora sin cambiar funcionalidad | `refactor: mueve validación de cédula a capa Negocio` |
| `docs` | Solo documentación | `docs: agrega capítulo LINQ avanzado al manual` |
| `chore` | Configuración / tareas | `chore: actualiza .gitignore con carpeta .vs` |
| `style` | Formato de código | `style: aplica indentación correcta en PacienteDatosEF` |
| `test` | Agrega o modifica tests | `test: agrega prueba de validación de cédula duplicada` |

```bash
# Ejemplos reales para este proyecto:
git commit -m "feat: implementa capa de datos con Entity Framework 6"
git commit -m "feat: agrega formulario de historial de enfermedades (Form1)"
git commit -m "fix: corrige mapeo de fechaNacimiento en LINQ to SQL"
git commit -m "refactor: extrae método ValidarCedulaUnica a PacienteNegocio"
git commit -m "docs: agrega manual LINQ con ejemplos del proyecto"
git commit -m "chore: agrega script SQL de creación de tablas"
```

---

## Flujo de ramas recomendado

```
main ──────────────────────────────────────────────── (producción estable)
  │
  ├─── develop ────────────────────────────────────── (integración)
  │       │
  │       ├─── feature/capa-ef ──────────────────── (nueva funcionalidad)
  │       │       └── commits de trabajo
  │       │       └── merge a develop cuando termina
  │       │
  │       ├─── feature/historial-enfermedades ─────
  │       │       └── commits de trabajo
  │       │
  │       └─── fix/null-reference-grilla ──────────
  │               └── commit de corrección
  │               └── merge a develop
  │
  └── merge develop → main cuando el sprint está listo
```

### Comandos para el flujo de ramas

```bash
# Crear rama de funcionalidad desde develop
git checkout develop
git checkout -b feature/exportar-csv

# Trabajar: hacer cambios y commits
git add .
git commit -m "feat: agrega exportación de pacientes a CSV"

# Cuando terminas, mergear a develop
git checkout develop
git merge feature/exportar-csv

# Subir develop a GitHub
git push origin develop

# Cuando develop está estable, mergear a main
git checkout main
git merge develop
git push origin main

# Borrar la rama ya mergeada
git branch -d feature/exportar-csv
git push origin --delete feature/exportar-csv
```

---

## Comandos Git del día a día

```bash
# ──────────────── Ver estado ────────────────
git status                       # ¿Qué cambió?
git log --oneline --graph        # Historial visual compact
git diff                         # Ver exactamente qué cambió en archivos

# ──────────────── Guardar trabajo ────────────────
git add .                        # Agregar todos los cambios
git add GestionUsuariosDatosEF/  # Agregar solo una carpeta
git commit -m "mensaje"          # Crear commit
git push                         # Subir al repositorio remoto

# ──────────────── Descargar cambios ────────────────
git pull                         # Descargar y mergear cambios del remoto
git fetch                        # Solo descargar (sin mergear)

# ──────────────── Deshacer cambios ────────────────
git restore nombre_archivo.cs    # Descartar cambios no committeados
git restore --staged .           # Quitar del staging area (sin borrar cambios)
git revert HEAD                  # Crear commit que deshace el último commit

# ──────────────── Ramas ────────────────
git branch                       # Ver ramas locales
git branch -a                    # Ver todas (incluye remotas)
git checkout -b nombre-rama      # Crear y cambiar a nueva rama
git checkout main                # Cambiar a rama main
git merge nombre-rama            # Fusionar rama en la actual

# ──────────────── Tags para versiones ────────────────
git tag v1.0                     # Crear tag en el commit actual
git tag v1.1 -m "Agrega EF"     # Tag anotado con mensaje
git push origin v1.0             # Subir tag a GitHub
git push origin --tags           # Subir todos los tags
```

---

## README.md del repositorio principal

Así debería verse el README del repositorio en GitHub:

````markdown
# 🏥 GestionUsuarios V2

Sistema de gestión de pacientes desarrollado en **C# .NET Framework 4.8** con **Windows Forms**, implementando arquitectura en capas.

## 🏗️ Arquitectura

```
GestionUsuariosEntidades        → Modelos / DTOs
GestionUsuariosDatos            → Acceso a datos con ADO.NET
GestionUsuariosDatosLINQ        → Acceso a datos con LINQ to SQL
GestionUsuariosDatosEF          → Acceso a datos con Entity Framework 6 ← activo
GestionUsuariosLogicaNegocio    → Reglas de negocio y validaciones
GestionUsuariosPresentacion     → Interfaz de usuario (WinForms)
```

## 🛠️ Requisitos

- Visual Studio 2022
- .NET Framework 4.8
- SQL Server 2019+
- Entity Framework 6.5.2 (NuGet)

## 🚀 Instalación

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/TU-USUARIO/gestion-usuarios-v2.git
   ```

2. Restaurar la base de datos:
   - Abrir SQL Server Management Studio
   - Ejecutar `scripts/BaseDeDatos.sql`

3. Configurar la conexión:
   - Abrir `GestionUsuariosDatosEF/App.config`
   - Modificar la cadena de conexión con tu servidor

4. Compilar y ejecutar:
   - Abrir `GestionUsuarios.slnx` en Visual Studio
   - Presionar `F5`

## 📚 Manual LINQ

Ver el manual completo en [`docs/manual-linq/`](./docs/manual-linq/)

## 📝 Tecnologías

![C#](https://img.shields.io/badge/C%23-239120?style=flat&logo=csharp&logoColor=white)
![.NET](https://img.shields.io/badge/.NET_Framework-4.8-512BD4?style=flat&logo=dotnet)
![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?style=flat&logo=microsoftsqlserver&logoColor=white)
![Entity Framework](https://img.shields.io/badge/Entity_Framework-6.5-512BD4?style=flat)
````

---

[← Ejercicios Resueltos](../11-ejercicios-resueltos/README.md) | [← Volver al inicio](../README.md)

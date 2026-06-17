# 07 — Capa de Presentación (WinForms)

## Proyecto: `GestionUsuariosPresentacion`

La capa de presentación es el formulario que el usuario ve y maneja. En este proyecto es Windows Forms (.NET Framework 4.8). Su única responsabilidad es **capturar datos del usuario** y **mostrar resultados** — NUNCA debe acceder a la BD directamente.

>  **Regla de oro:** El Form solo llama a la capa de Negocio, nunca a la capa de Datos.

---

## Form_Paciente.cs — Flujo completo

```csharp
// Archivo: GestionUsuariosPresentacion/Form_Paciente.cs
using GestionUsuariosEntidades;
using GestionUsuariosLogicaNegocio;  // Solo referencia Negocio, nunca Datos

public partial class Form_Paciente : Form
{
    // Objeto paciente que "vive" mientras el formulario está abierto
    // Sirve como estado de la sesión actual
    PacienteEntidades paciente = new PacienteEntidades();

    public Form_Paciente()
    {
        InitializeComponent();
    }

    // Al cargar el formulario, se inicializan los controles
    private void Form_Paciente_Load(object sender, EventArgs e)
    {
        InicializarValores();
        CargarListadoPacientesEnDataGridView();
        CargarComboGenero();
    }

    // Prepara la lista de enfermedades del paciente actual
    private void InicializarValores()
    {
        paciente.ListaEnfermedades = new List<Paciente_Enfermedad_Entidades>();
    }
}
```

---

## Cargar el ComboBox de Géneros

```csharp
private void CargarComboGenero()
{
    try
    {
        // Obtiene la lista de géneros desde la capa de Negocio
        cb_Genero.DataSource = GeneroNegocio.DevolverListaGeneros();

        // DisplayMember: qué propiedad se MUESTRA al usuario
        cb_Genero.DisplayMember = "nombre";

        // ValueMember: qué propiedad se usa como VALUE cuando el usuario selecciona
        cb_Genero.ValueMember = "id";

        // Inicia sin ninguna selección
        cb_Genero.SelectedIndex = -1;
    }
    catch (Exception ex)
    {
        MessageBox.Show("Error al cargar los géneros: " + ex.Message);
    }
}
```

---

## Llenar el DataGridView con la lista de pacientes

```csharp
private void CargarListadoPacientesEnDataGridView()
{
    // DataSource: asignar una lista directamente a la grilla
    // WinForms lee automáticamente las propiedades de la clase para crear columnas
    dataGridView1.DataSource = PacienteNegocio.DevolverListaPaciente();
}
```

---

## Guardar / Actualizar un Paciente

```csharp
private void btn_Guardar_Click(object sender, EventArgs e)
{
    GuardarPaciente();
}

private void GuardarPaciente()
{
    // 1. Leer los valores de los controles del formulario
    //    y asignarlos al objeto paciente
    paciente.Nombre          = txtb_Nombres.Text.ToUpper();
    paciente.Apellido        = txtb_Apellidos.Text.ToUpper();
    paciente.Cedula          = txtb_Cedula.Text;
    paciente.FechaNacimiento = dtp_FechaNacimiento.Value;
    paciente.Telefono        = txtb_Telefono.Text;
    paciente.Direccion       = txtb_Direccion.Text.ToUpper();
    paciente.Afiliado        = cb_Afiliado.Checked;
    paciente.CodigoIEES      = txtb_CodigoIees.Text;

    // SelectedValue devuelve el ValueMember del ComboBox (el ID del género)
    paciente.IdGenero = Convert.ToInt32(cb_Genero.SelectedValue);

    // Asegurarse de que la lista de enfermedades esté inicializada
    if (paciente.ListaEnfermedades == null)
        paciente.ListaEnfermedades = new List<Paciente_Enfermedad_Entidades>();

    // 2. Llamar a la capa de Negocio (que decidirá INSERT o UPDATE)
    paciente = PacienteNegocio.GuardarPaciente(paciente);

    // 3. Mostrar resultado al usuario
    if (paciente != null)
    {
        // Mostrar el ID asignado por la BD en el campo de texto
        txtb_Id.Text = paciente.Id.ToString();
        MessageBox.Show("Los datos se almacenaron correctamente");

        // Refrescar la grilla con los datos actualizados
        CargarListadoPacientesEnDataGridView();
    }

    // Si hubo error en la capa de datos, mostrar el mensaje
    if (paciente.Error_Texto != null)
    {
        MessageBox.Show(paciente.Error_Texto);
    }
}
```

---

## Seleccionar un Paciente en la grilla (CellClick)

```csharp
private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
{
    try
    {
        // Obtener el ID de la fila seleccionada usando el nombre de la columna
        var id = Convert.ToInt32(
            dataGridView1.Rows[e.RowIndex].Cells["id"].Value.ToString());

        // Leer el género de la fila para pre-seleccionar el ComboBox
        cb_Afiliado.Checked = Convert.ToBoolean(
            dataGridView1.Rows[e.RowIndex].Cells["Afiliado"].Value ?? false);

        int idGenero = Convert.ToInt32(
            dataGridView1.Rows[e.RowIndex].Cells["IdGenero"].Value ?? 0);

        if (idGenero > 0)
            cb_Genero.SelectedValue = idGenero;  // Selecciona el género en el ComboBox
        else
            cb_Genero.SelectedIndex = -1;

        // Cargar todos los datos del paciente desde la BD
        CargarValoresPacientePorId(id);
    }
    catch (Exception ex)
    {
        MessageBox.Show("Error en selección de elemento", ex.Message);
    }
}

private void CargarValoresPacientePorId(int id)
{
    // Cargar el paciente desde la capa de Negocio
    paciente = PacienteNegocio.CargarPacientePorId(id);

    // Rellenar los controles del formulario con los datos del paciente
    txtb_Id.Text               = paciente.Id.ToString();
    cb_Genero.SelectedValue    = paciente.IdGenero;
    txtb_Nombres.Text          = paciente.Nombre;
    txtb_Apellidos.Text        = paciente.Apellido;
    txtb_Cedula.Text           = paciente.Cedula;
    dtp_FechaNacimiento.Value  = paciente.FechaNacimiento;
    txtb_Telefono.Text         = paciente.Telefono;
    txtb_Direccion.Text        = paciente.Direccion;
    cb_Afiliado.Checked        = paciente.Afiliado;
    txtb_CodigoIees.Text       = paciente.CodigoIEES;
}
```

---

## Limpiar el formulario para un nuevo registro

```csharp
private void btn_Nuevo_Click(object sender, EventArgs e)
{
    EncerarCamposParaNuevoRegistro();
}

private void EncerarCamposParaNuevoRegistro()
{
    // Crear una nueva instancia limpia del objeto
    // Esto resetea automáticamente todas las propiedades a sus valores por defecto
    paciente = new PacienteEntidades();

    // Limpiar los controles del formulario
    txtb_Id.Text               = paciente.Id.ToString();   // "0"
    txtb_Nombres.Text          = paciente.Nombre;          // null → ""
    txtb_Apellidos.Text        = paciente.Apellido;
    txtb_Cedula.Text           = paciente.Cedula;
    dtp_FechaNacimiento.Value  = DateTime.Now;
    txtb_Telefono.Text         = paciente.Telefono;
    txtb_Direccion.Text        = paciente.Direccion;
    cb_Afiliado.CheckState     = CheckState.Unchecked;
    txtb_CodigoIees.Text       = paciente.CodigoIEES;
}
```

---

## Eliminar un Paciente con confirmación

```csharp
private void btn_Eliminar_Click(object sender, EventArgs e)
{
    EliminarPaciente();
}

private void EliminarPaciente()
{
    // Buena práctica: confirmar antes de eliminar permanentemente
    if (MessageBox.Show(
            "¿Está usted seguro de eliminar permanentemente este registro?",
            "Eliminar registro",
            MessageBoxButtons.OKCancel,
            MessageBoxIcon.Question) == DialogResult.OK)
    {
        // Llamar a la capa de Negocio con el ID del paciente activo
        if (PacienteNegocio.EliminarPacientePorId(paciente.Id))
        {
            MessageBox.Show(
                "El registro se eliminó correctamente",
                "Eliminar paciente",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);

            // Refrescar la grilla
            CargarListadoPacientesEnDataGridView();
        }
    }
}
```

---

## Abrir la ventana de Historial (Form1)

```csharp
private void button1_Click(object sender, EventArgs e)
{
    // Crear la ventana secundaria pasando datos del paciente actual
    Form1 ventanaHistorial = new Form1(
        Convert.ToInt32(txtb_Id.Text),
        txtb_Nombres.Text,
        txtb_Apellidos.Text);

    // ShowDialog: abre la ventana de forma MODAL (bloquea esta mientras está abierta)
    ventanaHistorial.ShowDialog();
}
```

---

## Resumen de controles usados

| Control | Propiedad clave | Para qué se usa |
|---------|----------------|-----------------|
| `TextBox` | `.Text` | Ingresar/mostrar texto |
| `DateTimePicker` | `.Value` | Seleccionar fecha |
| `CheckBox` | `.Checked` | Valor booleano (Afiliado) |
| `ComboBox` | `.DataSource`, `.DisplayMember`, `.ValueMember`, `.SelectedValue` | Lista desplegable con ID y nombre |
| `DataGridView` | `.DataSource` | Tabla de resultados |
| `MessageBox` | `.Show(...)` | Diálogos de confirmación/error |

---

[← Lógica de Negocio](../06-logica-negocio/README.md) | [Siguiente: Consultas Avanzadas →](../08-linq-consultas-avanzadas/README.md)

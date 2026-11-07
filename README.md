"""
Sistema de Nómina Empresarial
- Herencia, Polimorfismo y Gestión de Métodos en Python
- Interfaz gráfica con Tkinter

Desarrollado por: Lesdy Rivera
"""

# math permite validar números; Tkinter crea la interfaz gráfica
import math
import tkinter as tk
from tkinter import ttk, messagebox, simpledialog


class Bonificable:
    """Clase que permite agregar bonificaciones adicionales a un empleado."""
    def __init__(self):
        self.bonos = []

    def agregar_bonificacion(self, monto):
        """Agrega un monto de bonificación a la lista."""
        self.bonos.append(monto)

    def obtener_bonificaciones(self):
        """Devuelve la suma total de bonificaciones."""
        return sum(self.bonos)


class Empleado:
    """Clase base que representa a un empleado con datos generales."""
    def __init__(self, nombre, identificacion, salario_base):
        self.nombre = nombre
        self.identificacion = identificacion
        self.salario_base = salario_base

    def calcular_salario(self):
        """Calcula el salario; las subclases redefinen este método (polimorfismo)."""
        return self.salario_base

    def nombre_cargo(self):
        """Devuelve el tipo de empleado."""
        return "Empleado"

    def total_bonos(self):
        """Devuelve 0 por defecto; se redefine en clases con bonificaciones."""
        return 0

    def mostrar_informacion(self, tipo="basico", detalle=False):
        """Devuelve un texto formateado con los datos del empleado."""
        texto = f"Nombre: {self.nombre} | Identificación: {self.identificacion}"
        if tipo in ("total", "completo"):
            texto += f" | Salario total: ${self.calcular_salario():,.0f}"
        if tipo == "completo":
            texto += f" | Bonificaciones: ${self.total_bonos():,.0f}"
        if detalle:
            texto += f" | Cargo: {self.nombre_cargo()}"
        return texto


class EmpleadoTiempoCompleto(Empleado):
    """Empleado con salario base más bono por desempeño."""
    def __init__(self, nombre, identificacion, salario_base, bono_desempeno):
        super().__init__(nombre, identificacion, salario_base)
        self.bono_desempeno = bono_desempeno

    def calcular_salario(self):
        return self.salario_base + self.bono_desempeno

    def total_bonos(self):
        return self.bono_desempeno

    def nombre_cargo(self):
        return "Tiempo completo"


class EmpleadoPorHoras(Empleado):
    """Empleado cuyo pago depende de horas trabajadas."""
    def __init__(self, nombre, identificacion, valor_hora, horas):
        super().__init__(nombre, identificacion, valor_hora)
        self.horas = horas

    def calcular_salario(self):
        return self.salario_base * self.horas

    def nombre_cargo(self):
        return "Por horas"


class EmpleadoComision(Empleado):
    """Empleado con salario base más comisión por ventas."""
    def __init__(self, nombre, identificacion, salario_base, ventas):
        super().__init__(nombre, identificacion, salario_base)
        self.ventas = ventas
        self.porcentaje = 0.05  # 5% de comisión

    def calcular_salario(self):
        return self.salario_base + self.ventas * self.porcentaje

    def nombre_cargo(self):
        return "Por comisión"


class EmpleadoTiempoCompletoBonificado(Bonificable, EmpleadoTiempoCompleto):
    """Empleado de tiempo completo que admite bonos adicionales (herencia múltiple)."""
    def __init__(self, nombre, identificacion, salario_base, bono_desempeno):
        EmpleadoTiempoCompleto.__init__(self, nombre, identificacion, salario_base, bono_desempeno)
        Bonificable.__init__(self)

    def calcular_salario(self):
        return super().calcular_salario() + self.obtener_bonificaciones()

    def total_bonos(self):
        return super().total_bonos() + self.obtener_bonificaciones()

    def nombre_cargo(self):
        return "Tiempo completo con bonos"


# Tipos de empleados disponibles en el sistema
TIPOS = {
    "Tiempo completo": (EmpleadoTiempoCompleto, "Salario base", "Bono por desempeño"),
    "Por horas": (EmpleadoPorHoras, "Valor por hora", "Horas trabajadas"),
    "Por comisión": (EmpleadoComision, "Salario base", "Ventas del mes"),
    "Tiempo completo con bonos": (EmpleadoTiempoCompletoBonificado, "Salario base", "Bono por desempeño"),
}


def numero_no_negativo(texto):
    """Valida que el texto sea un número mayor o igual a cero."""
    numero = float(texto)
    if not math.isfinite(numero) or numero < 0:
        raise ValueError("Ingrese un número válido (mayor o igual a cero).")
    return numero


class Aplicacion:
    """Interfaz gráfica principal del sistema de nómina."""
    def __init__(self, ventana):
        self.ventana = ventana
        self.empleados = []
        ventana.title("Sistema de Nómina Empresarial - Lesdy Rivera")
        ventana.geometry("650x550")

        marco = tk.Frame(ventana, padx=15, pady=15)
        marco.pack()

        # Formulario de registro
        formulario = tk.Frame(marco)
        formulario.grid(row=0, column=0, columnspan=2, sticky="w", pady=(0, 10))
        formulario.columnconfigure(1, minsize=220)
        self.entradas = {}
        self.etiqueta_base = None

        for fila, texto in enumerate(["Nombre", "Identificación", "Salario base"]):
            etiqueta = tk.Label(formulario, text=texto, font=("Arial", 10, "bold"))
            etiqueta.grid(row=fila, column=0, sticky="w")
            if texto == "Salario base":
                self.etiqueta_base = etiqueta
            self.entradas[texto] = ttk.Entry(formulario, width=25)
            self.entradas[texto].grid(row=fila, column=1, sticky="w", padx=10, pady=4)

        tk.Label(formulario, text="Tipo de empleado:", font=("Arial", 10, "bold")).grid(row=3, column=0, sticky="w")
        self.tipo = ttk.Combobox(formulario, values=list(TIPOS), state="readonly", width=22)
        self.tipo.current(0)
        self.tipo.grid(row=3, column=1, sticky="w", padx=10, pady=4)
        self.tipo.bind("<<ComboboxSelected>>", self.cambiar_etiqueta)

        self.etiqueta_extra = tk.Label(formulario, text=TIPOS[self.tipo.get()][2], font=("Arial", 10, "bold"))
        self.etiqueta_extra.grid(row=4, column=0, sticky="w")
        self.extra = ttk.Entry(formulario, width=25)
        self.extra.grid(row=4, column=1, sticky="w", padx=10, pady=4)

        tk.Button(formulario, text="Agregar Empleado", command=self.agregar,
                  width=25, font=("Arial", 10, "bold")).grid(row=5, column=1, pady=10)

        # Tabla de empleados
        columnas = ("nombre", "identificacion", "cargo", "salario")
        self.lista = ttk.Treeview(marco, columns=columnas, show="headings", height=7)
        for col, titulo, ancho in [
            ("nombre", "Nombre", 150),
            ("identificacion", "Identificación", 120),
            ("cargo", "Tipo de Empleado", 200),
            ("salario", "Salario Total", 130),
        ]:
            self.lista.heading(col, text=titulo)
            self.lista.column(col, width=ancho, anchor="e" if col == "salario" else "w")
        self.lista.grid(row=6, column=0, columnspan=2, pady=5)

        # Panel de información
        panel_info = tk.LabelFrame(marco, text="Información del Empleado", padx=10, pady=10, font=("Arial", 10, "bold"))
        panel_info.grid(row=7, column=0, sticky="w", pady=10)

        self.nivel = tk.StringVar(value="basico")
        opciones = tk.Frame(panel_info)
        opciones.grid(row=0, column=0, sticky="w")
        for col, (texto, valor) in enumerate([("Básico", "basico"), ("Total", "total"), ("Completo", "completo")]):
            tk.Radiobutton(opciones, text=texto, variable=self.nivel, value=valor).grid(row=0, column=col)

        self.detalle = tk.BooleanVar(value=False)
        tk.Checkbutton(panel_info, text="Mostrar cargo", variable=self.detalle).grid(row=1, column=0, sticky="w", pady=5)

        tk.Button(panel_info, text="Ver Información", command=self.ver_informacion).grid(row=2, column=0, sticky="w")
        tk.Button(marco, text="Agregar Bono", command=self.agregar_bono, font=("Arial", 10)).grid(row=7, column=1, sticky="e", padx=10)
        tk.Button(marco, text="Calcular Nómina Total", command=self.calcular_nomina,
                  font=("Arial", 11, "bold"), bg="#4CAF50", fg="white").grid(row=8, column=0, columnspan=2, pady=8)

        self.salida = tk.Text(marco, width=65, height=8, font=("Arial", 10))
        self.salida.grid(row=9, column=0, columnspan=2)

        self.cambiar_etiqueta()

    def cambiar_etiqueta(self, evento=None):
        tipo = self.tipo.get()
        _, etiqueta_base, etiqueta_extra = TIPOS[tipo]
        self.etiqueta_extra.config(text=etiqueta_extra)
        self.etiqueta_base.config(text=etiqueta_base)

    def escribir(self, texto):
        self.salida.delete("1.0", tk.END)
        self.salida.insert(tk.END, texto)

    def agregar(self):
        try:
            nombre = self.entradas["Nombre"].get().strip()
            doc = self.entradas["Identificación"].get().strip()
            if not nombre or not doc:
                raise ValueError("Complete el nombre y la identificación.")
            if not self.entradas["Salario base"].get().strip():
                raise ValueError(f"Complete el campo: {self.etiqueta_base.cget('text')}")
            if not self.extra.get().strip():
                raise ValueError(f"Complete el campo: {self.etiqueta_extra.cget('text')}")

            base = numero_no_negativo(self.entradas["Salario base"].get())
            extra = numero_no_negativo(self.extra.get())
        except ValueError as error:
            messagebox.showerror("Error de Datos", str(error))
            return

        tipo = self.tipo.get()
        clase = TIPOS[tipo][0]
        emp = clase(nombre, doc, base, extra)

        # Actualizar si ya existe
        for idx, anterior in enumerate(self.empleados):
            if anterior.identificacion == doc and type(anterior) is type(emp):
                if isinstance(emp, Bonificable):
                    emp.bonos = anterior.bonos.copy()
                self.empleados[idx] = emp
                self.lista.item(str(idx), values=self.valores_fila(emp))
                self.escribir(f"✅ Datos actualizados: {emp.nombre}")
                return

        self.empleados.append(emp)
        self.lista.insert("", tk.END, iid=str(len(self.empleados)-1), values=self.valores_fila(emp))
        self.escribir(f"✅ Empleado agregado: {emp.nombre}")

        # Limpiar formulario
        for entrada in self.entradas.values():
            entrada.delete(0, tk.END)
        self.extra.delete(0, tk.END)

    def valores_fila(self, emp):
        return (emp.nombre, emp.identificacion, emp.nombre_cargo(), f"${emp.calcular_salario():,.0f}")

    def seleccionado(self):
        pos = self.lista.selection()
        if not pos:
            messagebox.showinfo("Aviso", "Seleccione un empleado de la lista")
            return None
        return self.empleados[int(pos[0])]

    def ver_informacion(self):
        emp = self.seleccionado()
        if emp:
            self.escribir(emp.mostrar_informacion(self.nivel.get(), self.detalle.get()))

    def agregar_bono(self):
        emp = self.seleccionado()
        if not emp:
            return
        if isinstance(emp, Bonificable):
            monto = simpledialog.askfloat("Agregar Bono", "Monto de la bonificación:", minvalue=0)
            if monto is not None:
                if monto < 0:
                    messagebox.showerror("Error", "El monto no puede ser negativo")
                    return
                emp.agregar_bonificacion(monto)
                self.lista.item(self.lista.selection()[0], values=self.valores_fila(emp))
                self.escribir(f"✅ Bono agregado a {emp.nombre}")
        else:
            messagebox.showinfo("Aviso", "Este tipo de empleado no admite bonificaciones")

    def calcular_nomina(self):
        total = 0
        texto = "📋 DETALLE DE NÓMINA:\n" + "-"*45 + "\n"
        for emp in self.empleados:
            pago = emp.calcular_salario()
            total += pago
            texto += f"{emp.nombre:20} | ${pago:12,.0f}\n"
        texto += "-"*45 + f"\n💰 TOTAL A PAGAR: ${total:,.0f}"
        self.escribir(texto)


if __name__ == "__main__":
    ventana = tk.Tk()
    Aplicacion(ventana)
    ventana.mainloop()

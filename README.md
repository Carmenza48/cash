import tkinter as tk
from tkinter import messagebox
import sqlite3
import getpass

# Conexión a la base de datos SQLite
conn = sqlite3.connect('cajero.db')
cursor = conn.cursor()

# Crear tabla si no existe
cursor.execute('''
CREATE TABLE IF NOT EXISTS usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT UNIQUE,
    contraseña TEXT,
    saldo REAL
)
''')
conn.commit()

def crear_cuenta():
    ventana_crear_cuenta = tk.Toplevel(root)
    ventana_crear_cuenta.title("Crear cuenta")

    lbl_nombre = tk.Label(ventana_crear_cuenta, text="Nombre de usuario:")
    lbl_nombre.grid(row=0, column=0, padx=10, pady=10)
    entry_nombre = tk.Entry(ventana_crear_cuenta)
    entry_nombre.grid(row=0, column=1, padx=10, pady=10)

    lbl_contraseña = tk.Label(ventana_crear_cuenta, text="Contraseña:")
    lbl_contraseña.grid(row=1, column=0, padx=10, pady=10)
    entry_contraseña = tk.Entry(ventana_crear_cuenta, show="*")
    entry_contraseña.grid(row=1, column=1, padx=10, pady=10)

    def guardar_cuenta():
        nombre = entry_nombre.get()
        contraseña = entry_contraseña.get()

        try:
            cursor.execute('INSERT INTO usuarios (nombre, contraseña, saldo) VALUES (?, ?, ?)', (nombre, contraseña, 0))
            conn.commit()
            messagebox.showinfo("Éxito", "Cuenta creada exitosamente.")
            ventana_crear_cuenta.destroy()
        except sqlite3.IntegrityError:
            messagebox.showerror("Error", "El nombre de usuario ya está en uso.")

    btn_guardar = tk.Button(ventana_crear_cuenta, text="Guardar", command=guardar_cuenta)
    btn_guardar.grid(row=2, column=0, columnspan=2, padx=10, pady=10)

def iniciar_sesion():
    ventana_inicio_sesion = tk.Toplevel(root)
    ventana_inicio_sesion.title("Iniciar sesión")

    lbl_nombre = tk.Label(ventana_inicio_sesion, text="Nombre de usuario:")
    lbl_nombre.grid(row=0, column=0, padx=10, pady=10)
    entry_nombre = tk.Entry(ventana_inicio_sesion)
    entry_nombre.grid(row=0, column=1, padx=10, pady=10)

    lbl_contraseña = tk.Label(ventana_inicio_sesion, text="Contraseña:")
    lbl_contraseña.grid(row=1, column=0, padx=10, pady=10)
    entry_contraseña = tk.Entry(ventana_inicio_sesion, show="*")
    entry_contraseña.grid(row=1, column=1, padx=10, pady=10)

    def verificar_credenciales():
        nombre = entry_nombre.get()
        contraseña = entry_contraseña.get()

        cursor.execute('SELECT * FROM usuarios WHERE nombre = ? AND contraseña = ?', (nombre, contraseña))
        usuario = cursor.fetchone()

        if usuario:
            messagebox.showinfo("Éxito", f"Bienvenido, {usuario[1]}")
            ventana_inicio_sesion.destroy()
            mostrar_operaciones(usuario)
        else:
            messagebox.showerror("Error", "Credenciales incorrectas.")

    btn_iniciar_sesion = tk.Button(ventana_inicio_sesion, text="Iniciar sesión", command=verificar_credenciales)
    btn_iniciar_sesion.grid(row=2, column=0, columnspan=2, padx=10, pady=10)

def mostrar_operaciones(usuario):
    ventana_operaciones = tk.Toplevel(root)
    ventana_operaciones.title("Operaciones en el cajero")

    lbl_saldo = tk.Label(ventana_operaciones, text=f"Saldo actual: {usuario[3]}")
    lbl_saldo.pack(padx=10, pady=10)

    def depositar():
        cantidad = float(entry_deposito.get())
        nuevo_saldo = usuario[3] + cantidad
        cursor.execute('UPDATE usuarios SET saldo = ? WHERE id = ?', (nuevo_saldo, usuario[0]))
        conn.commit()
        lbl_saldo.config(text=f"Saldo actual: {nuevo_saldo}")

    def retirar():
        cantidad = float(entry_retiro.get())
        if cantidad <= usuario[3]:
            nuevo_saldo = usuario[3] - cantidad
            cursor.execute('UPDATE usuarios SET saldo = ? WHERE id = ?', (nuevo_saldo, usuario[0]))
            conn.commit()
            lbl_saldo.config(text=f"Saldo actual: {nuevo_saldo}")
        else:
            messagebox.showerror("Error", "Saldo insuficiente.")

    lbl_deposito = tk.Label(ventana_operaciones, text="Cantidad a depositar:")
    lbl_deposito.pack(padx=10, pady=10)
    entry_deposito = tk.Entry(ventana_operaciones)
    entry_deposito.pack(padx=10, pady=10)
    btn_depositar = tk.Button(ventana_operaciones, text="Depositar", command=depositar)
    btn_depositar.pack(padx=10, pady=10)

    lbl_retiro = tk.Label(ventana_operaciones, text="Cantidad a retirar:")
    lbl_retiro.pack(padx=10, pady=10)
    entry_retiro = tk.Entry(ventana_operaciones)
    entry_retiro.pack(padx=10, pady=10)
    btn_retirar = tk.Button(ventana_operaciones, text="Retirar", command=retirar)
    btn_retirar.pack(padx=10, pady=10)

def main():
    global root
    root = tk.Tk()
    root.title("Cajero Automático CashEBAS")

    lbl_bienvenida = tk.Label(root, text="¡Bienvenidos al Cajero Automático CashEBAS!")
    lbl_bienvenida.pack(padx=10, pady=10)

    btn_crear_cuenta = tk.Button(root, text="Crear cuenta", command=crear_cuenta)
    btn_crear_cuenta.pack(padx=10, pady=10)

    btn_iniciar_sesion = tk.Button(root, text="Iniciar sesión", command=iniciar_sesion)
    btn_iniciar_sesion.pack(padx=10, pady=10)

    root.mainloop()

if _name_ == "_main_":
    main()

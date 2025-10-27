ProyectoCuentaBancaria/
│
├── src/
│   ├── Cliente.java
│   ├── Movimiento.java
│   ├── Cuenta.java
│   ├── ExportadorCuenta.java
│   └── Main.java
│
├── datos/                  // Carpeta para persistencia y exportaciones
│   ├── cuenta.dat
│   ├── cuenta.csv
│   ├── cuenta.xml
│   └── cuenta.json
│
└── README.md


- `src/` → Contiene todas las clases Java.
    
- `datos/` → Carpeta creada automáticamente para guardar el fichero de persistencia y las exportaciones.
    
- `cuenta.dat` → Fichero serializado con la cuenta bancaria.
    
- `cuenta.csv`, `cuenta.xml`, `cuenta.json` → Archivos generados al exportar los datos.


## **2️⃣ Código de las clases**

### **Cliente.java**

import java.io.Serializable;

public class Cliente implements Serializable {
    private String nombre;
    private String dni;

    public Cliente(String nombre, String dni) {
        this.nombre = nombre;
        this.dni = dni;
    }

    public String getNombre() {
        return nombre;
    }

    public String getDni() {
        return dni;
    }

    @Override
    public String toString() {
        return "Cliente: " + nombre + " (DNI: " + dni + ")";
    }
}


Movimiento.java

import java.io.Serializable;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Movimiento implements Serializable {
    private double cantidad;
    private String tipo; // "Ingreso" o "Retirada"
    private LocalDateTime fecha;

    public Movimiento(double cantidad, String tipo) {
        this.cantidad = cantidad;
        this.tipo = tipo;
        this.fecha = LocalDateTime.now();
    }

    public double getCantidad() { return cantidad; }
    public String getTipo() { return tipo; }
    public LocalDateTime getFecha() { return fecha; }

    @Override
    public String toString() {
        DateTimeFormatter formato = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
        return tipo + ": " + cantidad + " € | Fecha: " + fecha.format(formato);
    }
}

Cuenta.java

import java.io.Serializable;
import java.util.ArrayList;

public class Cuenta implements Serializable {
    private Cliente propietario;
    private ArrayList<Movimiento> movimientos;

    public Cuenta(Cliente propietario) {
        this.propietario = propietario;
        this.movimientos = new ArrayList<>();
    }

    public void ingresar(double cantidad) {
        if (cantidad > 0) {
            movimientos.add(new Movimiento(cantidad, "Ingreso"));
            System.out.println("Ingreso realizado correctamente.");
        } else {
            System.out.println("Cantidad inválida.");
        }
    }

    public void retirar(double cantidad) {
        if (cantidad > 0 && getSaldo() >= cantidad) {
            movimientos.add(new Movimiento(cantidad, "Retirada"));
            System.out.println("Retirada realizada correctamente.");
        } else {
            System.out.println("Saldo insuficiente o cantidad inválida.");
        }
    }

    public double getSaldo() {
        double saldo = 0;
        for (Movimiento m : movimientos) {
            saldo += m.getTipo().equals("Ingreso") ? m.getCantidad() : -m.getCantidad();
        }
        return saldo;
    }

    public void mostrarMovimientos() {
        if (movimientos.isEmpty()) {
            System.out.println("No hay movimientos registrados.");
        } else {
            for (Movimiento m : movimientos) {
                System.out.println(m);
            }
        }
    }

    public Cliente getPropietario() { return propietario; }
    public ArrayList<Movimiento> getMovimientos() { return movimientos; }
}


ExportadorCuenta.java

import java.io.*;
import java.time.format.DateTimeFormatter;

import org.json.JSONArray;
import org.json.JSONObject;

public class ExportadorCuenta {

    // Exportar a CSV
    public static void exportarCSV(Cuenta cuenta, String ruta) {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(ruta))) {
            bw.write("Tipo;Cantidad;Fecha\n");
            double saldo = 0;
            for (Movimiento m : cuenta.getMovimientos()) {
                bw.write(m.getTipo() + ";" + m.getCantidad() + ";" +
                        m.getFecha().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss")) + "\n");
                saldo += m.getTipo().equals("Ingreso") ? m.getCantidad() : -m.getCantidad();
            }
            bw.write("# Saldo final;" + saldo + "\n");
            System.out.println("CSV exportado correctamente a " + ruta);
        } catch (IOException e) {
            System.out.println("Error exportando CSV: " + e.getMessage());
        }
    }

    // Exportar a XML
    public static void exportarXML(Cuenta cuenta, String ruta) {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(ruta))) {
            bw.write("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
            bw.write("<cuenta>\n");
            bw.write("  <cliente>\n");
            bw.write("    <nombre>" + cuenta.getPropietario().getNombre() + "</nombre>\n");
            bw.write("    <dni>" + cuenta.getPropietario().getDni() + "</dni>\n");
            bw.write("  </cliente>\n");

            bw.write("  <movimientos>\n");
            double saldo = 0;
            for (Movimiento m : cuenta.getMovimientos()) {
                bw.write("    <movimiento tipo=\"" + m.getTipo() + "\">\n");
                bw.write("      <cantidad>" + m.getCantidad() + "</cantidad>\n");
                bw.write("      <fecha>" + m.getFecha().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss")) + "</fecha>\n");
                bw.write("    </movimiento>\n");
                saldo += m.getTipo().equals("Ingreso") ? m.getCantidad() : -m.getCantidad();
            }
            bw.write("  </movimientos>\n");
            bw.write("  <resumen>\n");
            bw.write("    <saldoFinal>" + saldo + "</saldoFinal>\n");
            bw.write("  </resumen>\n");
            bw.write("</cuenta>\n");

            System.out.println("XML exportado correctamente a " + ruta);
        } catch (IOException e) {
            System.out.println("Error exportando XML: " + e.getMessage());
        }
    }

    // Exportar a JSON
    public static void exportarJSON(Cuenta cuenta, String ruta) {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(ruta))) {
            JSONObject root = new JSONObject();
            JSONObject cliente = new JSONObject();
            cliente.put("nombre", cuenta.getPropietario().getNombre());
            cliente.put("dni", cuenta.getPropietario().getDni());
            root.put("cliente", cliente);

            JSONArray arrayMov = new JSONArray();
            double saldo = 0;
            for (Movimiento m : cuenta.getMovimientos()) {
                JSONObject mov = new JSONObject();
                mov.put("tipo", m.getTipo());
                mov.put("cantidad", m.getCantidad());
                mov.put("fecha", m.getFecha().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss")));
                arrayMov.put(mov);
                saldo += m.getTipo().equals("Ingreso") ? m.getCantidad() : -m.getCantidad();
            }
            root.put("movimientos", arrayMov);

            JSONObject resumen = new JSONObject();
            resumen.put("saldoFinal", saldo);
            root.put("resumen", resumen);

            bw.write(root.toString(4)); // pretty print
            System.out.println("JSON exportado correctamente a " + ruta);
        } catch (IOException e) {
            System.out.println("Error exportando JSON: " + e.getMessage());
        }
    }
}



Main.java


import java.io.*;
import java.util.Scanner;

public class Main {
    private static final String RUTA_DATOS = "datos/cuenta.dat";

    public static void main(String[] args) {
        Cuenta cuenta = null;
        Scanner sc = new Scanner(System.in);

        // Crear carpeta datos si no existe
        File carpeta = new File("datos");
        if (!carpeta.exists()) carpeta.mkdir();

        File fichero = new File(RUTA_DATOS);
        if (fichero.exists()) {
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(fichero))) {
                cuenta = (Cuenta) ois.readObject();
                System.out.println("Cuenta cargada correctamente.");
            } catch (IOException | ClassNotFoundException e) {
                System.out.println("Error al leer la cuenta: " + e.getMessage());
            }
        } else {
            System.out.print("Introduce el nombre del cliente: ");
            String nombre = sc.nextLine();
            System.out.print("Introduce el DNI del cliente: ");
            String dni = sc.nextLine();
            Cliente cliente = new Cliente(nombre, dni);
            cuenta = new Cuenta(cliente);
            System.out.println("Cuenta creada correctamente.");
        }

        int opcion;
        do {
            System.out.println("\n--- MENÚ ---");
            System.out.println("1. Ingresar dinero");
            System.out.println("2. Retirar dinero");
            System.out.println("3. Consultar saldo y movimientos");
            System.out.println("4. Exportar datos de la cuenta a CSV, XML y JSON");
            System.out.println("0. Salir");
            System.out.print("Elige una opción: ");
            opcion = sc.nextInt();

            switch (opcion) {
                case 1 -> {
                    System.out.print("Cantidad a ingresar: ");
                    double cantidad = sc.nextDouble();
                    cuenta.ingresar(cantidad);
                }
                case 2 -> {
                    System.out.print("Cantidad a retirar: ");
                    double cantidad = sc.nextDouble();
                    cuenta.retirar(cantidad);
                }
                case 3 -> {
                    System.out.println("Saldo actual: " + cuenta.getSaldo() + " €");
                    System.out.println("Movimientos:");
                    cuenta.mostrarMovimientos();
                }
                case 4 -> {
                    ExportadorCuenta.exportarCSV(cuenta, "datos/cuenta.csv");
                    ExportadorCuenta.exportarXML(cuenta, "datos/cuenta.xml");
                    ExportadorCuenta.exportarJSON(cuenta, "datos/cuenta.json");
                }
                case 0 -> System.out.println("Saliendo...");
                default -> System.out.println("Opción inválida.");
            }
        } while (opcion != 0);

        // Guardar cuenta
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fichero))) {
            oos.writeObject(cuenta);
            System.out.println("Cuenta guardada correctamente.");
        } catch (IOException e) {
            System.out.println("Error al guardar la cuenta: " + e.getMessage());
        }

        sc.close();
    }
}


SIN MEZCLA

ProyectoCuentaBancaria/
│
├── src/
│   ├── Cliente.java          // Clase que representa al cliente
│   ├── Movimiento.java       // Clase que representa cada movimiento
│   ├── Cuenta.java           // Clase que gestiona la cuenta bancaria
│   └── Main.java             // Programa principal con menú y persistencia
│
├── datos/                    // Carpeta donde se guardará el fichero de persistencia
│   └── cuenta.dat            // Fichero serializado (se crea al ejecutar el programa)
│
└── README.md                 // Opcional, descripción del proyecto


### **Estructura de clases**

1. **Cliente.java** – Representa al propietario de la cuenta.

import java.io.Serializable;

public class Cliente implements Serializable {
    private String nombre;
    private String dni;

    public Cliente(String nombre, String dni) {
        this.nombre = nombre;
        this.dni = dni;
    }

    public String getNombre() {
        return nombre;
    }

    public String getDni() {
        return dni;
    }

    @Override
    public String toString() {
        return "Cliente: " + nombre + " (DNI: " + dni + ")";
    }
}


---

2. **Movimiento.java** – Representa cada ingreso o retirada.
    
import java.io.Serializable;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Movimiento implements Serializable {
    private double cantidad;
    private String tipo; // "Ingreso" o "Retirada"
    private LocalDateTime fecha;

    public Movimiento(double cantidad, String tipo) {
        this.cantidad = cantidad;
        this.tipo = tipo;
        this.fecha = LocalDateTime.now();
    }

    public double getCantidad() {
        return cantidad;
    }

    public String getTipo() {
        return tipo;
    }

    public LocalDateTime getFecha() {
        return fecha;
    }

    @Override
    public String toString() {
        DateTimeFormatter formato = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
        return tipo + ": " + cantidad + " € | Fecha: " + fecha.format(formato);
    }
}

---

3. **Cuenta.java** – Gestiona la cuenta bancaria y sus movimientos.
    
import java.io.Serializable;
import java.util.ArrayList;

public class Cuenta implements Serializable {
    private Cliente propietario;
    private ArrayList<Movimiento> movimientos;

    public Cuenta(Cliente propietario) {
        this.propietario = propietario;
        this.movimientos = new ArrayList<>();
    }

    public void ingresar(double cantidad) {
        if (cantidad > 0) {
            movimientos.add(new Movimiento(cantidad, "Ingreso"));
            System.out.println("Ingreso realizado correctamente.");
        } else {
            System.out.println("Cantidad inválida.");
        }
    }

    public void retirar(double cantidad) {
        if (cantidad > 0 && getSaldo() >= cantidad) {
            movimientos.add(new Movimiento(cantidad, "Retirada"));
            System.out.println("Retirada realizada correctamente.");
        } else {
            System.out.println("Saldo insuficiente o cantidad inválida.");
        }
    }

    public double getSaldo() {
        double saldo = 0;
        for (Movimiento m : movimientos) {
            if (m.getTipo().equals("Ingreso")) {
                saldo += m.getCantidad();
            } else {
                saldo -= m.getCantidad();
            }
        }
        return saldo;
    }

    public void mostrarMovimientos() {
        if (movimientos.isEmpty()) {
            System.out.println("No hay movimientos registrados.");
        } else {
            for (Movimiento m : movimientos) {
                System.out.println(m);
            }
        }
    }

    public Cliente getPropietario() {
        return propietario;
    }
}


---

4. **Main.java** – Programa principal con persistencia y menú.
    

import java.io.*;
import java.util.Scanner;

public class Main {
    private static final String RUTA = "datos/cuenta.dat";

    public static void main(String[] args) {
        Cuenta cuenta = null;
        Scanner sc = new Scanner(System.in);

        // Comprobar si existe el directorio y el fichero
        File carpeta = new File("datos");
        if (!carpeta.exists()) {
            carpeta.mkdir();
        }

        File fichero = new File(RUTA);
        if (fichero.exists()) {
            // Cargar cuenta existente
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(fichero))) {
                cuenta = (Cuenta) ois.readObject();
                System.out.println("Cuenta cargada correctamente.");
            } catch (IOException | ClassNotFoundException e) {
                System.out.println("Error al leer la cuenta: " + e.getMessage());
            }
        } else {
            // Crear nueva cuenta
            System.out.print("Introduce el nombre del cliente: ");
            String nombre = sc.nextLine();
            System.out.print("Introduce el DNI del cliente: ");
            String dni = sc.nextLine();
            Cliente cliente = new Cliente(nombre, dni);
            cuenta = new Cuenta(cliente);
            System.out.println("Cuenta creada correctamente.");
        }

        // Menú de operaciones
        int opcion;
        do {
            System.out.println("\n--- MENÚ ---");
            System.out.println("1. Ingresar dinero");
            System.out.println("2. Retirar dinero");
            System.out.println("3. Consultar saldo y movimientos");
            System.out.println("0. Salir");
            System.out.print("Elige una opción: ");
            opcion = sc.nextInt();

            switch (opcion) {
                case 1 -> {
                    System.out.print("Cantidad a ingresar: ");
                    double cantidad = sc.nextDouble();
                    cuenta.ingresar(cantidad);
                }
                case 2 -> {
                    System.out.print("Cantidad a retirar: ");
                    double cantidad = sc.nextDouble();
                    cuenta.retirar(cantidad);
                }
                case 3 -> {
                    System.out.println("Saldo actual: " + cuenta.getSaldo() + " €");
                    System.out.println("Movimientos:");
                    cuenta.mostrarMovimientos();
                }
                case 0 -> System.out.println("Saliendo...");
                default -> System.out.println("Opción inválida.");
            }
        } while (opcion != 0);

        // Guardar cuenta al salir
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fichero))) {
            oos.writeObject(cuenta);
            System.out.println("Cuenta guardada correctamente.");
        } catch (IOException e) {
            System.out.println("Error al guardar la cuenta: " + e.getMessage());
        }

        sc.close();
    }
}

### ✅ **Características cumplidas**

- Uso de **ArrayList** para movimientos.
    
- Persistencia en disco con **ObjectInputStream/ObjectOutputStream**.
    
- Comprobación y creación de **directorio `datos/`** y fichero `cuenta.dat`.
    
- Gestión de **excepciones** en lectura/escritura.
    
- Menú interactivo con operaciones de ingreso, retirada, consulta de saldo y movimientos.
    
- Información clara para el usuario.
    

---

Si quieres, puedo hacer una **versión más “profesional”** con validación de entradas, formateo de cantidades y uso de **BigDecimal** para evitar errores de precisión en dinero. Esto dejaría el proyecto listo para una entrega excelente.

¿Quieres que haga esa versión mejorada?

##### Tú dijiste:

DAME LA ESTRUCTURA

###### ChatGPT Plus

Perfecto, aquí tienes la **estructura del proyecto** para la actividad de gestión de cuenta bancaria en Java, organizada de forma clara:

`ProyectoCuentaBancaria/ │ ├── src/ │   ├── Cliente.java          // Clase que representa al cliente │   ├── Movimiento.java       // Clase que representa cada movimiento │   ├── Cuenta.java           // Clase que gestiona la cuenta bancaria │   └── Main.java             // Programa principal con menú y persistencia │ ├── datos/                    // Carpeta donde se guardará el fichero de persistencia │   └── cuenta.dat            // Fichero serializado (se crea al ejecutar el programa) │ └── README.md                 // Opcional, descripción del proyecto`

- `src/` → Contiene todas las clases Java.
    
- `datos/` → Carpeta creada automáticamente para guardar `cuenta.dat`.
    
- `cuenta.dat` → Fichero donde se guarda y recupera la cuenta del cliente.
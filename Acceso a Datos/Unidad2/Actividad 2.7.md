```java

```
Versión para alumnos
Objetivo
Aplicar todos los conocimientos de JDBC avanzado en un caso integrado.
Tareas
1. Crea la BD con las tablas empleados , proyectos y asignaciones .
2. Inserta empleados y proyectos con PreparedStatement.
3. Implementa un procedimiento almacenado para asignar empleados a proyectos.
4. Crea un programa que:
Ejecute el procedimiento almacenado.
Realice una transacción que incremente el salario de un empleado y descuente
ese importe del presupuesto de un proyecto.
Use un pool de conexiones para obtener la conexión.
5. Muestra el estado final de las tablas en consola.
Entregables
Script SQL.
Código fuente en Java.
Capturas de pantalla con la ejecución.

RESPUESTA



DatabaseConfig
```java


package config;
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.io.InputStream;

import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;
public class DatabaseConfig {
private static HikariDataSource dataSource;
static {
try {
Properties props = new Properties();
try (InputStream input =

DatabaseConfig.class.getClassLoader().getResourceAsStream("db.proper
ties")) {

props.load(input);
}
HikariConfig config = new HikariConfig();
config.setJdbcUrl(props.getProperty("db.url"));
config.setUsername(props.getProperty("db.user"));
config.setPassword(props.getProperty("db.password"));
config.setMaximumPoolSize(5);
config.setMinimumIdle(2);
config.setIdleTimeout(10000);
config.setConnectionTimeout(10000);
config.setPoolName("TechDAMPool");
dataSource = new HikariDataSource(config);
System.out.println("Pool de conexiones HikariCP

inicializado correctamente.");
} catch (Exception e) {
throw new RuntimeException("Error al inicializar el pool

de conexiones", e);
}
}
public static Connection getConnection() throws SQLException {
return dataSource.getConnection();
}
public static void cerrarPool() {
if (dataSource != null && !dataSource.isClosed()) {
dataSource.close();
System.out.println("Pool de conexiones cerrado.");
}
}
}

AsignaccionDAO
package dao;
import config.DatabaseConfig;
import modelo.Asignacion;
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.SQLException;
public class AsignacionDAO {
public void asignarEmpleado(int empleadoId, int proyectoId) {
String sql = "{call asignar_empleado_a_proyecto(?, ?)}";
try (Connection conn = DatabaseConfig.getConnection();
CallableStatement cs = conn.prepareCall(sql)) {
cs.setInt(1, empleadoId);
cs.setInt(2, proyectoId);
cs.execute();
} catch (SQLException e) {
e.printStackTrace();
}
}
}

EmpleadoDAO
package dao;
import config.DatabaseConfig;
import modelo.Empleado;
import java.math.BigDecimal;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class EmpleadoDAO {
public int insertar(Empleado emp) {
String sql = "INSERT INTO empleados (nombre, departamento,
salario, activo) VALUES (?,?,?,?)";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql,

Statement.RETURN_GENERATED_KEYS)) {

ps.setString(1, emp.getNombre());
ps.setString(2, emp.getDepartamento());

ps.setBigDecimal(3, emp.getSalario());
ps.setBoolean(4, emp.isActivo());
ps.executeUpdate();
try (ResultSet rs = ps.getGeneratedKeys()) {
if (rs.next()) return rs.getInt(1);
}
} catch (SQLException e) {
e.printStackTrace();
}
return -1;
}
public List<Empleado> obtenerTodos() {
List<Empleado> lista = new ArrayList<>();
String sql = "SELECT * FROM empleados";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql);
ResultSet rs = ps.executeQuery()) {
while (rs.next()) {
lista.add(new Empleado(
rs.getInt("id"),
rs.getString("nombre"),
rs.getString("departamento"),
rs.getBigDecimal("salario"),
rs.getBoolean("activo")

));
}
} catch (SQLException e) {
e.printStackTrace();
}
return lista;
}
public boolean actualizarSalario(Connection conn, int id,
BigDecimal nuevoSalario) throws SQLException {
String sql = "UPDATE empleados SET salario = ? WHERE id =
?";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
ps.setBigDecimal(1, nuevoSalario);
ps.setInt(2, id);
return ps.executeUpdate() == 1;
}
}
}

ProyectoDAO

package dao;
import config.DatabaseConfig;
import modelo.Proyecto;
import java.math.BigDecimal;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class ProyectoDAO {
public int insertar(Proyecto proy) {
String sql = "INSERT INTO proyectos (nombre, presupuesto)
VALUES (?, ?)";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql,

Statement.RETURN_GENERATED_KEYS)) {

ps.setString(1, proy.getNombre());
ps.setBigDecimal(2, proy.getPresupuesto());
ps.executeUpdate();
try (ResultSet rs = ps.getGeneratedKeys()) {
if (rs.next()) return rs.getInt(1);
}
} catch (SQLException e) {
e.printStackTrace();
}
return -1;
}
public List<Proyecto> obtenerTodos() {
List<Proyecto> lista = new ArrayList<>();
String sql = "SELECT * FROM proyectos";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql);
ResultSet rs = ps.executeQuery()) {
while (rs.next()) {
lista.add(new Proyecto(
rs.getInt("id"),
rs.getString("nombre"),
rs.getBigDecimal("presupuesto")

));
}
} catch (SQLException e) {
e.printStackTrace();
}
return lista;

}
public boolean restarPresupuesto(Connection conn, int id,
BigDecimal monto) throws SQLException {
String sql = "UPDATE proyectos SET presupuesto =
presupuesto - ? WHERE id = ?";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
ps.setBigDecimal(1, monto);
ps.setInt(2, id);
return ps.executeUpdate() == 1;
}
}
}

Asignacion
package modelo;
public class Asignacion {
private int id;
private int empleadoId;
private int proyectoId;
public Asignacion() {}
public Asignacion(int id, int empleadoId, int proyectoId) {
this.id = id; this.empleadoId = empleadoId; this.proyectoId
= proyectoId;
}
public int getId() { return id; }
public void setId(int id) { this.id = id; }
public int getEmpleadoId() { return empleadoId; }
public void setEmpleadoId(int empleadoId) { this.empleadoId =
empleadoId; }
public int getProyectoId() { return proyectoId; }
public void setProyectoId(int proyectoId) { this.proyectoId =
proyectoId; }
@Override
public String toString() {
return "Empleado ID: " + empleadoId + " -> Proyecto ID: " +
proyectoId;
}
}

Empleado
package modelo;

import java.math.BigDecimal;
public class Empleado {
private int id;
private String nombre;
private String departamento;
private BigDecimal salario;
private boolean activo;
public Empleado() {}
public Empleado(int id, String nombre, String departamento,
BigDecimal salario, boolean activo) {
this.id = id; this.nombre = nombre; this.departamento =
departamento; this.salario = salario; this.activo = activo;
}
public int getId() { return id; }
public void setId(int id) { this.id = id; }
public String getNombre() { return nombre; }
public void setNombre(String nombre) { this.nombre = nombre; }
public String getDepartamento() { return departamento; }
public void setDepartamento(String departamento) {
this.departamento = departamento; }
public BigDecimal getSalario() { return salario; }
public void setSalario(BigDecimal salario) { this.salario =
salario; }
public boolean isActivo() { return activo; }
public void setActivo(boolean activo) { this.activo = activo; }
@Override
public String toString() {
return nombre + " - " + departamento + " - " + salario;
}
}

Proyecto
package modelo;
import java.math.BigDecimal;
public class Proyecto {
private int id;
private String nombre;
private BigDecimal presupuesto;
public Proyecto() {}

public Proyecto(int id, String nombre, BigDecimal presupuesto)
{
this.id = id; this.nombre = nombre; this.presupuesto =
presupuesto;
}
public int getId() { return id; }
public void setId(int id) { this.id = id; }
public String getNombre() { return nombre; }
public void setNombre(String nombre) { this.nombre = nombre; }
public BigDecimal getPresupuesto() { return presupuesto; }
public void setPresupuesto(BigDecimal presupuesto) {
this.presupuesto = presupuesto; }
@Override
public String toString() {
return nombre + " - " + presupuesto;
}
}

Main
import dao.EmpleadoDAO;
import dao.ProyectoDAO;
import modelo.Empleado;
import modelo.Proyecto;
import service.TransaccionesService;
import config.DatabaseConfig;
import java.math.BigDecimal;
import java.util.List;
public class Main {
public static void main(String[] args) {
EmpleadoDAO empleadoDAO = new EmpleadoDAO();
ProyectoDAO proyectoDAO = new ProyectoDAO();
AsignacionDAO asignacionDAO = new AsignacionDAO();
TransaccionesService service = new TransaccionesService();
// Mostrar empleados
System.out.println("Empleados antes:");
List<Empleado> empleados = empleadoDAO.obtenerTodos();
empleados.forEach(System.out::println);
// Mostrar proyectos
System.out.println("\nProyectos antes:");
List<Proyecto> proyectos = proyectoDAO.obtenerTodos();

proyectos.forEach(System.out::println);
// Asignar empleado al proyecto
asignacionDAO.asignarEmpleado(1, 1);
// Transacción: incrementar salario y descontar presupuesto
service.incrementarSalarioYDescontarPresupuesto(1, 1,
BigDecimal.valueOf(200));
// Mostrar resultados finales
System.out.println("\nEmpleados después:");
empleadoDAO.obtenerTodos().forEach(System.out::println);
System.out.println("\nProyectos después:");
proyectoDAO.obtenerTodos().forEach(System.out::println);
DatabaseConfig.cerrarPool();
}
}

TransaccionesServices
package service;
import dao.EmpleadoDAO;
import dao.ProyectoDAO;
import config.DatabaseConfig;
import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.SQLException;
public class TransaccionesService {
private final EmpleadoDAO empleadoDAO = new EmpleadoDAO();
private final ProyectoDAO proyectoDAO = new ProyectoDAO();
public boolean incrementarSalarioYDescontarPresupuesto(int
empleadoId, int proyectoId, BigDecimal monto) {
try (Connection conn = DatabaseConfig.getConnection()) {
conn.setAutoCommit(false);
empleadoDAO.actualizarSalario(conn, empleadoId,

empleadoDAO.obtenerTodos()
.stream()
.filter(e -> e.getId() == empleadoId)
.findFirst()
.get()
.getSalario()

.add(monto));

proyectoDAO.restarPresupuesto(conn, proyectoId, monto);
conn.commit();
return true;
} catch (SQLException e) {
e.printStackTrace();
return false;
}
}
}

db.properties
db.url=jdbc:mysql://localhost:3306/empresa
db.user=root
db.password=root123
db.maximumPoolSize=10

pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"

xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

<groupId>org.example</groupId>
<artifactId>3</artifactId>
<version>1.0-SNAPSHOT</version>

<properties>

<maven.compiler.source>23</maven.compiler.source>
<maven.compiler.target>23</maven.compiler.target>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<mysql.version>8.0.33</mysql.version>
<hikaricp.version>5.0.1</hikaricp.version>
<slf4j.version>2.0.9</slf4j.version>
</properties>

<dependencies>
<!-- Driver JDBC MySQL -->
<!--
https://mvnrepository.com/artifact/com.mysql/mysql-connector-j -->
<dependency>
<groupId>com.mysql</groupId>
<artifactId>mysql-connector-j</artifactId>
<version>${mysql.version}</version>
</dependency>

<!-- HikariCP -->
<dependency>
<groupId>com.zaxxer</groupId>
<artifactId>HikariCP</artifactId>
<version>${hikaricp.version}</version>
</dependency>

<!-- SLF4J - Logging -->
<dependency>
<groupId>org.slf4j</groupId>
<artifactId>slf4j-api</artifactId>
<version>${slf4j.version}</version>
</dependency>

<dependency>
<groupId>org.slf4j</groupId>
<artifactId>slf4j-simple</artifactId>
<version>${slf4j.version}</version>
</dependency>

</dependencies>

</project>

MYSQL

-- Crear base de datos
DROP DATABASE IF EXISTS empresa;
CREATE DATABASE empresa;
USE empresa;

-- Tabla empleados
CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    departamento VARCHAR(50),
    salario DECIMAL(10,2),
    activo BOOLEAN DEFAULT TRUE
);

-- Tabla proyectos
CREATE TABLE proyectos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    presupuesto DECIMAL(10,2)
);

-- Tabla asignaciones
CREATE TABLE asignaciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    empleado_id INT,
    proyecto_id INT,
    FOREIGN KEY (empleado_id) REFERENCES empleados(id),
    FOREIGN KEY (proyecto_id) REFERENCES proyectos(id)
);

-- Datos iniciales
INSERT INTO empleados (nombre, departamento, salario) VALUES
('Ana López','IT',2500.00),
('Luis Pérez','RRHH',2200.00),
('María Gómez','IT',2800.00);

INSERT INTO proyectos (nombre, presupuesto) VALUES
('Proyecto A',10000.00),
('Proyecto B',15000.00);

-- Procedimiento para asignar empleado a proyecto
DELIMITER $$
CREATE PROCEDURE asignar_empleado_a_proyecto(IN p_emp INT, IN p_proy INT)
BEGIN
    INSERT INTO asignaciones (empleado_id, proyecto_id)
    VALUES (p_emp, p_proy);
END$$
DELIMITER ;
```
``
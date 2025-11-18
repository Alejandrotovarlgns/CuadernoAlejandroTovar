
Versión para alumnos
Objetivo
Aprender a invocar procedimientos almacenados con parámetros de entrada y salida
desde Java usando CallableStatement .
Tareas
1. En la base de datos empresa , crea un procedimiento llamado
incrementar_salario que:
Reciba como parámetros de entrada el id de un empleado y un incremento
salarial ( IN ).
Devuelva el nuevo salario ( OUT ).
2. Crea una clase Java llamada PruebaCallable que:
Conecte con la BD empresa .
Llame al procedimiento incrementar_salario .
Muestre el nuevo salario por consola.
3. Comprueba que el salario del empleado se actualiza en la base de datos.
Entregables
Script SQL del procedimiento.
Código Java con la llamada al procedimiento.
Captura de pantalla mostrando el antes y el después en la tabla empleados .




RESULTADO

DatabaseConfig
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
config.setPoolName("EmpresaPool");
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
public static void closePool() {
if (dataSource != null && !dataSource.isClosed()) {
dataSource.close();
System.out.println("Pool de conexiones cerrado.");
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
import java.util.Optional;
public class EmpleadoDAO {
// CRUD mínimo (solo mostrar obtenerTodos para ejemplo)
public List<Empleado> obtenerTodos() {
List<Empleado> empleados = new ArrayList<>();
String sql = "SELECT * FROM empleados";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql);
ResultSet rs = ps.executeQuery()) {
while (rs.next()) {
Empleado e = new Empleado(
rs.getInt("id"),
rs.getString("nombre"),
rs.getBigDecimal("salario"),
rs.getString("departamento"),
rs.getBoolean("activo")

);
empleados.add(e);
}
} catch (SQLException e) {
e.printStackTrace();
}
return empleados;
}
// Llamada al procedimiento incrementar_salario
public BigDecimal incrementarSalario(int idEmpleado, BigDecimal
incremento) {
String sql = "{call incrementar_salario(?, ?, ?)}";
try (Connection conn = DatabaseConfig.getConnection();
CallableStatement cstmt = conn.prepareCall(sql)) {
cstmt.setInt(1, idEmpleado);
cstmt.setBigDecimal(2, incremento);
cstmt.registerOutParameter(3, Types.DECIMAL);
cstmt.execute();

return cstmt.getBigDecimal(3);
} catch (SQLException e) {
e.printStackTrace();
return BigDecimal.valueOf(-1);
}
}
}

Empleado
package modelo;
import java.math.BigDecimal;
public class Empleado {
private int id;
private String nombre;
private BigDecimal salario;
private String departamento;
private boolean activo;
public Empleado() {}
public Empleado(int id, String nombre, BigDecimal salario,
String departamento, boolean activo) {
this.id = id;
this.nombre = nombre;
this.salario = salario;
this.departamento = departamento;
this.activo = activo;
}
// Getters y setters
public int getId() { return id; }
public void setId(int id) { this.id = id; }
public String getNombre() { return nombre; }
public void setNombre(String nombre) { this.nombre = nombre; }
public BigDecimal getSalario() { return salario; }
public void setSalario(BigDecimal salario) { this.salario =
salario; }
public String getDepartamento() { return departamento; }
public void setDepartamento(String departamento) {
this.departamento = departamento; }

public boolean isActivo() { return activo; }
public void setActivo(boolean activo) { this.activo = activo; }
@Override
public String toString() {
return nombre + " - " + departamento + " - " + salario;
}
}
Main
package org.example;
import modelo.Empleado;
import dao.EmpleadoDAO;
import service.ProcedimientosService;
import java.math.BigDecimal;
import java.util.List;
public class Main {
public static void main(String[] args) {
EmpleadoDAO empleadoDAO = new EmpleadoDAO();
ProcedimientosService service = new
ProcedimientosService();
System.out.println("Lista de empleados antes del
incremento:");
List<Empleado> empleadosAntes = empleadoDAO.obtenerTodos();
empleadosAntes.forEach(System.out::println);
int idEmpleado = 1;
BigDecimal incremento = new BigDecimal("200.00");
BigDecimal nuevoSalario =
service.incrementarSalarioEmpleado(idEmpleado, incremento);
System.out.println("\nNuevo salario del empleado ID " +
idEmpleado + ": " + nuevoSalario);
System.out.println("\nLista de empleados después del
incremento:");
List<Empleado> empleadosDespues =
empleadoDAO.obtenerTodos();
empleadosDespues.forEach(System.out::println);
// Cerrar pool de conexiones al final
config.DatabaseConfig.closePool();

}
}
ProcedimientosServices
package service;
import dao.EmpleadoDAO;
import java.math.BigDecimal;
public class ProcedimientosService {
private EmpleadoDAO empleadoDAO = new EmpleadoDAO();
public BigDecimal incrementarSalarioEmpleado(int idEmpleado,
BigDecimal incremento) {
return empleadoDAO.incrementarSalario(idEmpleado,
incremento);
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
    nombre VARCHAR(50) NOT NULL,
    salario DECIMAL(10,2) NOT NULL,
    departamento VARCHAR(50),
    activo BOOLEAN DEFAULT TRUE
);

-- Insertar datos de prueba
INSERT INTO empleados (nombre, salario, departamento) VALUES
('Ana López', 2500.00, 'IT'),
('Luis Pérez', 2200.00, 'RRHH'),
('María Gómez', 2800.00, 'IT'),
('Jorge Díaz', 3000.00, 'Finanzas'),
('Laura Sánchez', 2000.00, 'Marketing');

-- Procedimiento incrementar_salario
DELIMITER $$

CREATE PROCEDURE incrementar_salario(
    IN p_idEmpleado INT,
    IN p_incremento DECIMAL(10,2),
    OUT p_nuevoSalario DECIMAL(10,2)
)
BEGIN
    -- Actualizar salario
    UPDATE empleados
    SET salario = salario + p_incremento
    WHERE id = p_idEmpleado;

    -- Devolver nuevo salario
    SELECT salario INTO p_nuevoSalario
    FROM empleados
    WHERE id = p_idEmpleado;
END$$

DELIMITER ;

Versión para alumnos
Objetivo
Aprender a manejar transacciones en JDBC, incluyendo commit , rollback y
savepoint .
Tareas
1. En la base de datos empresa , crea la tabla cuentas :
CREATE TABLE cuentas (
id INT AUTO_INCREMENT PRIMARY KEY,
titular VARCHAR(100),
saldo DECIMAL(10,2)
);
INSERT INTO cuentas (titular, saldo) VALUES
('Ana', 2000.00),
('Luis', 1500.00);
2. Implementa en Java una clase TransferenciaBancaria que:
Realice una transferencia de 500€ de la cuenta 1 a la cuenta 2.
Use transacciones manuales ( setAutoCommit(false) ).
Aplique commit() si la operación se completa correctamente.
Aplique rollback() si ocurre un error.
3. Extiende el ejercicio para que:
Se registre en una tabla logs cada paso de la transacción.
Si falla el registro del segundo paso, se use un savepoint para mantener el
primero.
Entregables
Script SQL con las tablas.
Código Java de la transferencia.
Capturas mostrando:
Estado de las cuentas antes y después de la operación.
Estado de los logs tras un rollback parcial.



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
config.setIdleTimeout(10000);
config.setConnectionTimeout(10000);
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
public static void cerrarPool() {
if (dataSource != null && !dataSource.isClosed()) {
dataSource.close();
System.out.println("Pool de conexiones cerrado.");
}
}
}
CuentaDAO
package dao;

import config.DatabaseConfig;
import modelo.Cuenta;
import java.math.BigDecimal;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class CuentaDAO {
public List<Cuenta> obtenerTodos() {
List<Cuenta> lista = new ArrayList<>();
String sql = "SELECT * FROM cuentas";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql);
ResultSet rs = ps.executeQuery()) {
while (rs.next()) {
lista.add(new Cuenta(
rs.getInt("id"),
rs.getString("titular"),
rs.getBigDecimal("saldo")

));
}
} catch (SQLException e) {
e.printStackTrace();
}
return lista;
}
public boolean actualizarSaldo(Connection conn, int id,
BigDecimal nuevoSaldo) throws SQLException {
String sql = "UPDATE cuentas SET saldo = ? WHERE id = ?";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
ps.setBigDecimal(1, nuevoSaldo);
ps.setInt(2, id);
return ps.executeUpdate() == 1;
}
}
public Cuenta obtenerPorId(int id) {
String sql = "SELECT * FROM cuentas WHERE id = ?";
try (Connection conn = DatabaseConfig.getConnection();
PreparedStatement ps = conn.prepareStatement(sql)) {
ps.setInt(1, id);
try (ResultSet rs = ps.executeQuery()) {
if (rs.next()) {
return new Cuenta(

rs.getInt("id"),
rs.getString("titular"),
rs.getBigDecimal("saldo")

);
}
}
} catch (SQLException e) {
e.printStackTrace();
}
return null;
}
}

LogDAO
package dao;
import config.DatabaseConfig;
import modelo.Log;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
public class LogDAO {
public boolean insertar(Connection conn, String mensaje) throws
SQLException {
String sql = "INSERT INTO logs (mensaje) VALUES (?)";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
ps.setString(1, mensaje);
return ps.executeUpdate() == 1;
}
}
}

Cuenta
package modelo;
import java.math.BigDecimal;
public class Cuenta {
private int id;
private String titular;
private BigDecimal saldo;
public Cuenta() {}

public Cuenta(int id, String titular, BigDecimal saldo) {
this.id = id;
this.titular = titular;
this.saldo = saldo;
}
public int getId() { return id; }
public void setId(int id) { this.id = id; }
public String getTitular() { return titular; }
public void setTitular(String titular) { this.titular = titular;
}
public BigDecimal getSaldo() { return saldo; }
public void setSaldo(BigDecimal saldo) { this.saldo = saldo; }
@Override
public String toString() {
return titular + " - " + saldo;
}
}
Log
package modelo;
import java.sql.Timestamp;
public class Log {
private int id;
private String mensaje;
private Timestamp fecha;
public Log() {}
public Log(String mensaje) {
this.mensaje = mensaje;
}
public int getId() { return id; }
public void setId(int id) { this.id = id; }
public String getMensaje() { return mensaje; }
public void setMensaje(String mensaje) { this.mensaje = mensaje;
}
public Timestamp getFecha() { return fecha; }
public void setFecha(Timestamp fecha) { this.fecha = fecha; }
@Override
public String toString() {
return fecha + " - " + mensaje;

}
}

Main
package org.example;
import dao.CuentaDAO;
import modelo.Cuenta;
import service.TransaccionesService;
import config.DatabaseConfig;
import java.math.BigDecimal;
import java.util.List;
public class Main {
public static void main(String[] args) {
CuentaDAO cuentaDAO = new CuentaDAO();
TransaccionesService service = new TransaccionesService();
System.out.println("Cuentas antes de la transferencia:");
List<Cuenta> cuentas = cuentaDAO.obtenerTodos();
cuentas.forEach(System.out::println);
// Transferencia 500€ de Ana (id 1) a Luis (id 2)
service.transferenciaConSavepoint(1, 2,
BigDecimal.valueOf(500));
System.out.println("\nCuentas después de la transferencia:");
cuentas = cuentaDAO.obtenerTodos();
cuentas.forEach(System.out::println);
DatabaseConfig.cerrarPool();
}
}

TransaccionesServices
package service;
import dao.CuentaDAO;
import dao.LogDAO;
import modelo.Cuenta;
import java.math.BigDecimal;
import java.sql.Connection;

import java.sql.SQLException;
import java.sql.Savepoint;
import config.DatabaseConfig;
public class TransaccionesService {
private final CuentaDAO cuentaDAO = new CuentaDAO();
private final LogDAO logDAO = new LogDAO();
public void transferenciaConSavepoint(int idOrigen, int
idDestino, BigDecimal monto) {
try (Connection conn = DatabaseConfig.getConnection()) {
conn.setAutoCommit(false);
Cuenta origen = cuentaDAO.obtenerPorId(idOrigen);
Cuenta destino = cuentaDAO.obtenerPorId(idDestino);
// Restar de origen
BigDecimal nuevoSaldoOrigen =
origen.getSaldo().subtract(monto);

cuentaDAO.actualizarSaldo(conn, idOrigen,

nuevoSaldoOrigen);

logDAO.insertar(conn, "Se restó " + monto + " de " +

origen.getTitular());

// Savepoint antes de actualizar destino
Savepoint spDestino = conn.setSavepoint("SP_DESTINO");
try {
BigDecimal nuevoSaldoDestino =

destino.getSaldo().add(monto);

cuentaDAO.actualizarSaldo(conn, idDestino,

nuevoSaldoDestino);

logDAO.insertar(conn, "Se sumó " + monto + " a " +

destino.getTitular());

} catch (SQLException e) {
conn.rollback(spDestino);
logDAO.insertar(conn, "Error al sumar a destino. Se

hace rollback parcial");

}
conn.commit();
System.out.println("Transferencia completada con commit y

savepoint.");
} catch (SQLException e) {
e.printStackTrace();
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

-- Tabla cuentas
CREATE TABLE cuentas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titular VARCHAR(100) NOT NULL,
    saldo DECIMAL(10,2) NOT NULL
);

-- Datos iniciales
INSERT INTO cuentas (titular, saldo) VALUES
('Ana', 2000.00),
('Luis', 1500.00);

-- Tabla logs
CREATE TABLE logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    mensaje VARCHAR(255),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Procedimiento para incrementar saldo (ejemplo adicional)
DELIMITER $$
CREATE PROCEDURE incrementar_saldo(IN p_id INT, IN p_monto DECIMAL(10,2), OUT p_saldo DECIMAL(10,2))
BEGIN
    UPDATE cuentas SET saldo = saldo + p_monto WHERE id = p_id;
    SELECT saldo INTO p_saldo FROM cuentas WHERE id = p_id;
END$$
DELIMITER ;
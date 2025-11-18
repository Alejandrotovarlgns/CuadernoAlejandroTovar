
Crear BD empresa .
Usar Statement para listar empleados.
Usar PreparedStatement para buscar empleados por id.
Crear procedimiento obtener_empleado y llamarlo con CallableStatement .
Entregar código Java, script SQL y capturas de pantalla.

estructura

codigo
pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"

xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>org.example</groupId>
<artifactId>EmpresaJDBC</artifactId>
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

db.properties
db.url=jdbc:mysql://localhost:3306/empresa
db.user=root
db.password=root123
db.maximumPoolSize=10

Conexion
package com.miempresa.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;
/**
* Utilidad para gestionar un DataSource basado en HikariCP mediante
inicialización
* estática. Esta clase expone métodos estáticos para obtener una
Connection y para
* cerrar el pool cuando la aplicación deja de necesitarlo.
* <p>
* Propósito académico:
* - Mostrar cómo inicializar un pool de conexiones al cargar la
clase.
* - Explicar la lectura de parámetros desde un fichero de
propiedades.
* - Enseñar la importancia de cerrar el pool al finalizar la
aplicación.
* <p>
* Observaciones didácticas:
* - La inicialización se realiza en un bloque static para disponer
del pool desde
* el arranque de la aplicación.
* - En entornos productivos, el fichero de propiedades no debe
contener credenciales
* en texto plano y su carga debe incluir comprobaciones más
exhaustivas.
*/
public class Conexion {
// DataSource compartido por toda la aplicación
private static HikariDataSource dataSource;
/*
* Bloque de inicialización estático:
* Se ejecuta al cargar la clase y configura el pool leyendo
propiedades desde
* el recurso 'db.properties' situado en el classpath.
*
* Nota académica: el uso de un bloque estático simplifica el
ejemplo; en aplicaciones
* complejas la configuración del pool suele centralizarse en
un módulo de inicialización
* o en la DI del contenedor.
*/

static {
try {
Properties props = new Properties();
// try-with-resources para cerrar automáticamente el

InputStream

// Se utiliza ConexionBD.class.getClassLoader() para

obtener el classloader.

try (InputStream input =

Conexion.class.getClassLoader().getResourceAsStream("db.properties
")) {

// Carga de las propiedades desde el fichero de

configuración.

// Se espera que 'db.url', 'db.user' y

'db.password' estén presentes.
props.load(input);
}
// Configuración mínima de HikariCP. Se explican los

parámetros relevantes.

HikariConfig config = new HikariConfig();
config.setJdbcUrl(props.getProperty("db.url"));

// URL JDBC, p.ej. jdbc:mysql://host:3306/schema

config.setUsername(props.getProperty("db.user"));

// Usuario de la BD

config.setPassword(props.getProperty("db.password"));

// Contraseña de la BD

// Parámetros de tuning básicos (ejemplo didáctico)
config.setMaximumPoolSize(5); // Número máximo de

conexiones activas en el pool

config.setMinimumIdle(2); // Número mínimo de

conexiones inactivas a mantener

config.setIdleTimeout(10000); // Tiempo (ms) tras el

cual una conexión inactiva puede cerrarse

config.setConnectionTimeout(10000); // Tiempo (ms)

máximo a esperar por una conexión libre

config.setPoolName("DAMPool"); // Nombre del pool

para facilitar diagnóstico

// Creación del DataSource (pool) a partir de la

configuración anterior.

dataSource = new HikariDataSource(config);
System.out.println("Pool de conexiones HikariCP

inicializado correctamente.");
} catch (Exception e) {
/*

* En este ejemplo didáctico se lanza una

RuntimeException para detener el

* arranque si el pool no puede inicializarse. En

entornos productivos se

* recomienda un manejo más fino (log, métricas,

reintento, fallback).
*/
throw new RuntimeException("Error al inicializar el

pool de conexiones", e);
}
}
/**
* Devuelve una Connection obtenida del pool.
* <p>
* Comportamiento pedagógico:
* - La Connection debe cerrarse por el consumidor cuando deje de
usarse.
* - En un pool, Connection.close() no cierra la conexión física,
sino que la devuelve al pool.
*
* @return Connection del pool HikariCP
* @throws SQLException si no es posible obtener la conexión
*/
public static Connection getConexion() throws SQLException {
return dataSource.getConnection();
}
/**
* Cierra el HikariDataSource liberando recursos asociados
(conexiones físicas, hilos, ...).
* <p>
* Uso recomendado:
* - Invocar al finalizar la aplicación, por ejemplo en un
shutdown hook o en el lifecycle
* del contenedor que despliegue la aplicación.
*/
public static void cerrarPool() {
if (dataSource != null && !dataSource.isClosed()) {
dataSource.close();
System.out.println("Pool de conexiones cerrado.");
}
}
}

empleadodao

package dao;
import java.sql.Connection;
import java.sql.Statement;
import java.sql.ResultSet;
import java.sql.PreparedStatement;
import java.sql.CallableStatement;
import static com.miempresa.config.Conexion.*;
public class empleadodao{
// 1️ Listar todos los empleados usando Statement
public void listarEmpleados() {
String sql = "SELECT * FROM empleados";
try (Connection con = getConexion();
Statement st = con.createStatement();
ResultSet rs = st.executeQuery(sql)) {
while (rs.next()) {
System.out.println("ID: " + rs.getInt("id")
+ " - Nombre: " + rs.getString("nombre")
+ " - Salario: " +

rs.getDouble("salario"));

}
} catch (Exception e) {
e.printStackTrace();
}
}
// 2️ Buscar empleado por ID usando PreparedStatement
public void empleadoPorId(int id) {
String sql = "SELECT * FROM empleados WHERE id = ?";
try (Connection con = getConexion();
PreparedStatement ps = con.prepareStatement(sql)) {
ps.setInt(1, id);
try (ResultSet rs = ps.executeQuery()) {
if (rs.next()) {
System.out.println("Empleado: " +

rs.getString("nombre")

+ " - Salario: " +

rs.getDouble("salario"));
} else {
System.out.println("Empleado no encontrado con

ID: " + id);
}
}

} catch (Exception e) {
e.printStackTrace();
}
}
// 3️ Llamar al procedimiento almacenado usando
CallableStatement
public void obtenerEmpleadoProcedimiento(int id) {
String sql = "{CALL obtener_empleado(?)}";
try (Connection con = getConexion();
CallableStatement cs = con.prepareCall(sql)) {
cs.setInt(1, id);
try (ResultSet rs = cs.executeQuery()) {
if (rs.next()) {
System.out.println("Empleado (proc): " +

rs.getString("nombre")

+ " - Salario: " +

rs.getDouble("salario"));
} else {
System.out.println("Empleado no encontrado con

ID: " + id);
}
}
} catch (Exception e) {
e.printStackTrace();
}
}
}

Main
package org.example;
import dao.empleadodao;
public class Main {
public static void main(String[] args) {
empleadodao dao = new empleadodao();
// Listar todos los empleados usando Statement
System.out.println("Listado de todos los empleados:");
dao.listarEmpleados();
// Buscar empleado por ID usando PreparedStatement
System.out.println("\nBuscar empleado con ID 2:");

dao.empleadoPorId(2);
// Llamar al procedimiento obtener_empleado usando
CallableStatement
System.out.println("\nLlamar al procedimiento
obtener_empleado con ID 3:");
dao.obtenerEmpleadoProcedimiento(3);
}
}

MYSQL


USE empresa;

-- Borrar tabla y procedimiento si existen
DROP TABLE IF EXISTS empleados;
DROP PROCEDURE IF EXISTS obtener_empleado;

-- Crear tabla
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    puesto VARCHAR(50),
    salario DECIMAL(10,2)
);

-- Insertar datos
INSERT INTO empleados (nombre, puesto, salario) VALUES
('Ana López', 'Administrativa', 1800),
('Juan Pérez', 'Programador', 2200),
('Lucía Gómez', 'Analista', 2500);

-- Crear procedimiento
DELIMITER $$
CREATE PROCEDURE obtener_empleado(IN emp_id INT)
BEGIN
    SELECT * FROM empleados WHERE id = emp_id;
END$$
DELIMITER ;

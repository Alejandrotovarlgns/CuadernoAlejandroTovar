
Versión para alumnos
Objetivo
Aprender a usar ResultSet, ResultSetMetaData y DatabaseMetaData en JDBC.
Tareas
1. Conéctate a la base de datos empresa .
2. Ejecuta una consulta SELECT * FROM empleados .
3. Muestra por consola los datos de todos los empleados.
4. Usa ResultSetMetaData para imprimir:
Número de columnas.
Nombre y tipo de cada columna.
5. Usa DatabaseMetaData para imprimir:
Nombre del producto de base de datos.
Versión del motor.
Nombre del driver JDBC.
Entregables
Código fuente en Java.
Capturas de pantalla con la ejecución y los resultados mostrados en consola.

ESTRUCTURA

conexion
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

EmpleadoDAO
package dao;
import java.sql.Connection;

import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.Statement;
import java.sql.DatabaseMetaData;
import static com.miempresa.config.Conexion.getConexion;
public class EmpleadoDAO {
// Listar todos los empleados y mostrar ResultSetMetaData
public void mostrarEmpleados() {
String sql = "SELECT * FROM empleados";
try (Connection con = getConexion();
Statement st = con.createStatement();
ResultSet rs = st.executeQuery(sql)) {
// 1️ Mostrar todos los empleados
System.out.println("Datos de empleados:");
while (rs.next()) {
System.out.println("ID: " + rs.getInt("id")
+ " - Nombre: " + rs.getString("nombre")
+ " - Salario: " +

rs.getDouble("salario"));

}
// 2️ ResultSetMetaData
ResultSetMetaData rsmd = rs.getMetaData();
int numColumnas = rsmd.getColumnCount();
System.out.println("\nNúmero de columnas: " +

numColumnas);

for (int i = 1; i <= numColumnas; i++) {
System.out.println("Columna " + i + ": " +

rsmd.getColumnName(i)

+ " - Tipo: " + rsmd.getColumnTypeName(i));

}
// 3️ DatabaseMetaData
DatabaseMetaData dbmd = con.getMetaData();
System.out.println("\nInformación de la base de

datos:");

System.out.println("Producto: " +

dbmd.getDatabaseProductName());

System.out.println("Versión: " +

dbmd.getDatabaseProductVersion());

System.out.println("Driver JDBC: " +

dbmd.getDriverName());
} catch (Exception e) {

e.printStackTrace();
}
}
}

Main
package org.example;
import dao.EmpleadoDAO;
public class Main {
public static void main(String[] args) {
EmpleadoDAO dao = new EmpleadoDAO();
dao.mostrarEmpleados();
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
-- Crear tabla proyectos
CREATE TABLE IF NOT EXISTS proyectos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    presupuesto DECIMAL(10,2)
);
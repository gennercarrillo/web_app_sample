# Guía para crear una aplicación web Java con Tomcat en GitHub Codespaces

Esta guía te ayudará a crear, configurar y ejecutar una aplicación web Java utilizando **Apache Tomcat** en **GitHub Codespaces**. La aplicación incluye un formulario HTML que envía datos a un Servlet, los guarda en una base de datos MySQL y muestra los registros en una tabla.

---

## Requisitos previos

1. **Cuenta de GitHub**: Necesitas una cuenta en GitHub.
2. **Acceso a GitHub Codespaces**: Asegúrate de tener acceso a Codespaces (puedes usar el plan gratuito).
3. **Conocimientos básicos**: Saber cómo clonar un repositorio y trabajar con GitHub.

---

## Pasos detallados

### 1. Crear un repositorio en GitHub

1. Ve a [GitHub](https://github.com) y crea un nuevo repositorio llamado `formulario-java-web`.
2. Inicializa el repositorio con un archivo `README.md`.

---

### 2. Configurar el entorno en GitHub Codespaces

1. Abre el repositorio en GitHub y haz clic en el botón `Code`.
2. Selecciona `Open with Codespaces` y luego `New Codespace`.
3. Espera a que se cree el Codespace. Esto puede tardar unos minutos.

---

### 3. Configurar el proyecto en el Codespace

1. **Abrir la terminal**:
   - En el Codespace, abre una terminal (`Ctrl + `` o `Terminal > New Terminal`).

2. **Crear la estructura del proyecto**:
   - Ejecuta los siguientes comandos para crear la estructura de un proyecto Java web:
     ```bash
     mkdir -p src/main/java src/main/webapp/WEB-INF
     touch src/main/webapp/index.jsp src/main/webapp/WEB-INF/web.xml
     ```

3. **Agregar un archivo `pom.xml`**:
   - Crea un archivo `pom.xml` en la raíz del proyecto con el siguiente contenido:
     ```xml
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <modelVersion>4.0.0</modelVersion>
         <groupId>com.example</groupId>
         <artifactId>formulario-java-web</artifactId>
         <version>1.0-SNAPSHOT</version>
         <packaging>war</packaging>

         <dependencies>
             <!-- Servlet API -->
             <dependency>
                 <groupId>javax.servlet</groupId>
                 <artifactId>javax.servlet-api</artifactId>
                 <version>4.0.1</version>
                 <scope>provided</scope>
             </dependency>
             <!-- MySQL Connector -->
             <dependency>
                 <groupId>mysql</groupId>
                 <artifactId>mysql-connector-java</artifactId>
                 <version>8.0.33</version>
             </dependency>
             <!-- JSTL -->
             <dependency>
                 <groupId>javax.servlet</groupId>
                 <artifactId>jstl</artifactId>
                 <version>1.2</version>
             </dependency>
         </dependencies>

         <build>
             <plugins>
                 <plugin>
                     <groupId>org.apache.maven.plugins</groupId>
                     <artifactId>maven-compiler-plugin</artifactId>
                     <version>3.8.1</version>
                     <configuration>
                         <source>17</source>
                         <target>17</target>
                     </configuration>
                 </plugin>
             </plugins>
         </build>
     </project>
     ```

4. **Compilar el proyecto**:
   - Ejecuta el siguiente comando para compilar el proyecto:
     ```bash
     mvn clean install
     ```

---

### 4. Configurar la base de datos MySQL

1. **Instalar MySQL en el Codespace**:
   - Ejecuta los siguientes comandos para instalar MySQL:
     ```bash
     sudo apt-get update
     sudo apt-get install mysql-server
     ```

2. **Iniciar el servicio MySQL**:
   - Inicia el servicio MySQL:
     ```bash
     sudo service mysql start
     ```

3. **Crear la base de datos y la tabla**:
   - Accede a MySQL:
     ```bash
     sudo mysql
     ```
   - Crea una base de datos y una tabla:
     ```sql
     CREATE DATABASE formulario_db;
     USE formulario_db;
     CREATE TABLE personas (
         id INT AUTO_INCREMENT PRIMARY KEY,
         nombre VARCHAR(50) NOT NULL,
         edad INT NOT NULL,
         documento VARCHAR(20) NOT NULL,
         direccion VARCHAR(100) NOT NULL
     );
     ```
   - Sal de MySQL:
     ```sql
     exit;
     ```

---

### 5. Crear el Servlet para manejar el formulario

1. **Crear la clase `FormularioServlet`**:
   - Crea un archivo `FormularioServlet.java` en `src/main/java/com/example`:
     ```java
     package com.example;

     import java.io.IOException;
     import java.sql.Connection;
     import java.sql.DriverManager;
     import java.sql.PreparedStatement;
     import javax.servlet.ServletException;
     import javax.servlet.annotation.WebServlet;
     import javax.servlet.http.HttpServlet;
     import javax.servlet.http.HttpServletRequest;
     import javax.servlet.http.HttpServletResponse;

     @WebServlet("/formulario")
     public class FormularioServlet extends HttpServlet {
         private static final String URL = "jdbc:mysql://localhost:3306/formulario_db";
         private static final String USER = "root";
         private static final String PASSWORD = "";

         @Override
         protected void doPost(HttpServletRequest request, HttpServletResponse response)
                 throws ServletException, IOException {
             String nombre = request.getParameter("nombre");
             int edad = Integer.parseInt(request.getParameter("edad"));
             String documento = request.getParameter("documento");
             String direccion = request.getParameter("direccion");

             try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD)) {
                 String sql = "INSERT INTO personas (nombre, edad, documento, direccion) VALUES (?, ?, ?, ?)";
                 PreparedStatement statement = conn.prepareStatement(sql);
                 statement.setString(1, nombre);
                 statement.setInt(2, edad);
                 statement.setString(3, documento);
                 statement.setString(4, direccion);
                 statement.executeUpdate();
             } catch (Exception e) {
                 e.printStackTrace();
             }

             response.sendRedirect("mostrarRegistros.jsp");
         }
     }
     ```

---

### 6. Crear la página JSP para mostrar los registros

1. **Crear `mostrarRegistros.jsp`**:
   - Crea un archivo `mostrarRegistros.jsp` en `src/main/webapp`:
     ```jsp
     <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
     <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
     <%@ page import="java.sql.*" %>
     <!DOCTYPE html>
     <html>
     <head>
         <title>Registros</title>
     </head>
     <body>
         <h1>Registros</h1>
         <table border="1">
             <tr>
                 <th>Nombre</th>
                 <th>Edad</th>
                 <th>Documento</th>
                 <th>Dirección</th>
             </tr>
             <%
                 Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/formulario_db", "root", "");
                 Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT * FROM personas");
                 while (rs.next()) {
             %>
             <tr>
                 <td><%= rs.getString("nombre") %></td>
                 <td><%= rs.getInt("edad") %></td>
                 <td><%= rs.getString("documento") %></td>
                 <td><%= rs.getString("direccion") %></td>
             </tr>
             <%
                 }
                 rs.close();
                 stmt.close();
                 conn.close();
             %>
         </table>
         <a href="index.jsp">Volver al formulario</a>
     </body>
     </html>
     ```

---

### 7. Crear el formulario HTML

1. **Crear `index.jsp`**:
   - Crea un archivo `index.jsp` en `src/main/webapp`:
     ```jsp
     <!DOCTYPE html>
     <html>
     <head>
         <title>Formulario</title>
     </head>
     <body>
         <h1>Formulario</h1>
         <form action="formulario" method="post">
             Nombre: <input type="text" name="nombre" required><br>
             Edad: <input type="number" name="edad" required><br>
             Documento: <input type="text" name="documento" required><br>
             Dirección: <input type="text" name="direccion" required><br>
             <input type="submit" value="Enviar">
         </form>
     </body>
     </html>
     ```

---

### 8. Configurar `web.xml`

1. **Editar `web.xml`**:
   - Abre el archivo `src/main/webapp/WEB-INF/web.xml` y agrega lo siguiente:
     ```xml
     <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
              http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
              version="3.1">
         <display-name>Formulario Java Web</display-name>
     </web-app>
     ```

---

### 9. Instalar y configurar Tomcat

1. **Descargar Tomcat**:
   - Ejecuta el siguiente comando para descargar Tomcat 10:
     ```bash
     wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.18/bin/apache-tomcat-10.1.18.tar.gz
     ```

2. **Descomprimir Tomcat**:
   - Descomprime el archivo descargado:
     ```bash
     tar -xvzf apache-tomcat-10.1.18.tar.gz
     ```

3. **Mover Tomcat a una ubicación conveniente**:
   - Mueve la carpeta de Tomcat a `/opt/tomcat`:
     ```bash
     sudo mv apache-tomcat-10.1.18 /opt/tomcat
     ```

4. **Configurar permisos**:
   - Asegúrate de que los archivos de Tomcat tengan los permisos correctos:
     ```bash
     sudo chmod -R 755 /opt/tomcat
     ```

5. **Configurar variables de entorno**:
   - Agrega la ruta de Tomcat a las variables de entorno:
     ```bash
     export CATALINA_HOME=/opt/tomcat
     export PATH=$PATH:$CATALINA_HOME/bin
     echo 'export CATALINA_HOME=/opt/tomcat' >> ~/.bashrc
     echo 'export PATH=$PATH:$CATALINA_HOME/bin' >> ~/.bashrc
     source ~/.bashrc
     ```

---

### 10. Desplegar la aplicación en Tomcat

1. **Compilar y empaquetar la aplicación**:
   - Ejecuta el siguiente comando para compilar y empaquetar la aplicación:
     ```bash
     mvn clean package
     ```

2. **Copiar el archivo `.war` a la carpeta `webapps` de Tomcat**:
   - Copia el archivo `.war` generado a la carpeta `webapps` de Tomcat:
     ```bash
     cp target/formulario-java-web.war $CATALINA_HOME/webapps/
     ```

3. **Iniciar Tomcat**:
   - Inicia Tomcat:
     ```bash
     $CATALINA_HOME/bin/startup.sh
     ```

---

### 11. Acceder a la aplicación

1. **Abrir la aplicación**:
   - Una vez que Tomcat esté en ejecución, abre un navegador y visita:
     ```
     https://<tu-codespace>-8080.app.github.dev/formulario-java-web
     ```
   - Esto abrirá la página `index.jsp` con el formulario.

2. **Probar la aplicación**:
   - Completa el formulario y envía los datos. Deberían guardarse en la base de datos MySQL y mostrarse en la página `mostrarRegistros.jsp`.

---

### 12. Detener Tomcat (opcional)

1. **Detener el servidor Tomcat**:
   - Si necesitas detener Tomcat, ejecuta:
     ```bash
     $CATALINA_HOME/bin/shutdown.sh
     ```

---

¡Listo! Ahora tienes una aplicación web Java funcionando con Tomcat en GitHub Codespaces. 😊

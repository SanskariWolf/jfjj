Okay, here is the complete code for the **Simple Contact Book Web App** using Servlets, JSP, and **Hibernate** with MySQL.

**Assumptions:**

*   You are using a build tool like Maven or Gradle (Maven `pom.xml` provided) OR you will manually place JARs in `WEB-INF/lib`.
*   You have MySQL installed and running.
*   You have Apache Tomcat (or another servlet container) installed.
*   You have a Java IDE (like Eclipse, IntelliJ).

---

**1. Project Structure (Maven Standard):**

```
SimpleContactBookHibernate/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── contactbook/
│   │   │               ├── bean/
│   │   │               │   └── Contact.java      (Entity)
│   │   │               ├── dao/
│   │   │               │   └── ContactDao.java     (Hibernate DAO)
│   │   │               ├── servlet/
│   │   │               │   └── ContactServlet.java (Controller)
│   │   │               └── util/
│   │   │                   └── HibernateUtil.java  (SessionFactory Helper)
│   │   ├── resources/
│   │   │   └── hibernate.cfg.xml     (Hibernate Config)
│   │   └── webapp/
│   │       ├── addContact.jsp        (View)
│   │       ├── viewContacts.jsp      (View)
│   │       └── WEB-INF/
│   │           ├── web.xml           (Deployment Descriptor)
│   │           └── lib/              (JARs go here if not using Maven/Gradle)
```

---

**2. `pom.xml` (Maven Dependencies)**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>SimpleContactBookHibernate</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging> <!-- Important: Set packaging to war -->

    <name>SimpleContactBookHibernate Maven Webapp</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source> <!-- Or newer like 11, 17 -->
        <maven.compiler.target>1.8</maven.compiler.target> <!-- Or newer like 11, 17 -->
        <hibernate.version>5.6.15.Final</hibernate.version> <!-- Use a stable Hibernate 5 version for simplicity with javax -->
        <!-- If using Hibernate 6+, use jakarta dependencies and adjust code imports -->
        <!-- <hibernate.version>6.4.4.Final</hibernate.version> -->
        <mysql.connector.version>8.0.33</mysql.connector.version>
        <servlet.api.version>4.0.1</servlet.api.version> <!-- Corresponds to javax.servlet -->
        <jsp.api.version>2.3.3</jsp.api.version>     <!-- Corresponds to javax.servlet.jsp -->
    </properties>

    <dependencies>
        <!-- Hibernate Core (using Hibernate 5 with javax.persistence) -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <!-- MySQL Connector -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>${mysql.connector.version}</version>
        </dependency>

        <!-- Servlet API (provided by server) -->
        <dependency>
            <groupId>javax.servlet</groupId> <!-- Use javax for Hibernate 5 / Servlet API 4 -->
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.api.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- JSP API (provided by server) -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId> <!-- Use javax for Hibernate 5 / JSP API 2.3 -->
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>${jsp.api.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- JSTL (Optional, if you use JSTL tags in JSP) -->
        <!--
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        -->

    </dependencies>

    <build>
        <finalName>SimpleContactBookHibernate</finalName>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-war-plugin</artifactId>
                    <version>3.2.2</version>
                     <configuration>
                        <failOnMissingWebXml>false</failOnMissingWebXml> <!-- Allow running without web.xml if using annotations -->
                    </configuration>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

*(**Note:** If you use Hibernate 6+, change dependencies to `org.hibernate.orm:hibernate-core`, `jakarta.servlet:jakarta.servlet-api`, `jakarta.servlet.jsp:jakarta.servlet.jsp-api` and update the imports in Java code from `javax.*` to `jakarta.*`)*

---

**3. `src/main/java/com/example/contactbook/bean/Contact.java`**

```java
package com.example.contactbook.bean;

// Using javax.persistence for Hibernate 5
import javax.persistence.Entity;
import javax.persistence.Table;
import javax.persistence.Id;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Column;
import java.io.Serializable;

@Entity
@Table(name = "contacts")
public class Contact implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "name", nullable = false, length = 100)
    private String name;

    @Column(name = "email", nullable = false, unique = true, length = 100)
    private String email;

    @Column(name = "phone", nullable = false, length = 20)
    private String phone;

    // Default Constructor (Required by Hibernate)
    public Contact() {}

    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }

    @Override
    public String toString() {
        return "Contact{" + "id=" + id + ", name='" + name + '\'' + ", email='" + email + '\'' + ", phone='" + phone + '\'' + '}';
    }
}
```

---

**4. `src/main/java/com/example/contactbook/util/HibernateUtil.java`**

```java
package com.example.contactbook.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class HibernateUtil {
    // Add SLF4J Logger (Optional but recommended)
    // You would need to add SLF4J API and a binding (like logback) to pom.xml
    // private static final Logger logger = LoggerFactory.getLogger(HibernateUtil.class);

    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            // Create the SessionFactory from hibernate.cfg.xml
             System.out.println("Attempting to configure Hibernate..."); // Simple console log
            Configuration configuration = new Configuration().configure(); // Looks for hibernate.cfg.xml in classpath
             System.out.println("Hibernate Configuration loaded.");
            SessionFactory factory = configuration.buildSessionFactory();
            System.out.println("SessionFactory created successfully.");
            return factory;
        } catch (Throwable ex) {
            // Make sure you log the exception, as it might be masked
            System.err.println("Initial SessionFactory creation failed." + ex);
            ex.printStackTrace(); // Print stack trace to console
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        if (sessionFactory != null && !sessionFactory.isClosed()) {
            System.out.println("Shutting down SessionFactory...");
            getSessionFactory().close();
            System.out.println("SessionFactory closed.");
        }
    }
}
```

---

**5. `src/main/java/com/example/contactbook/dao/ContactDao.java`**

```java
package com.example.contactbook.dao;

import com.example.contactbook.bean.Contact;
import com.example.contactbook.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query; // Use org.hibernate.query.Query for Hibernate 5+

import java.util.Collections;
import java.util.List;

public class ContactDao {

    public boolean addContact(Contact contact) {
        Transaction transaction = null;
        Session session = null;
        try {
            session = HibernateUtil.getSessionFactory().openSession();
            // Start a transaction
            transaction = session.beginTransaction();
            // Save the contact object
            session.save(contact);
            // Commit transaction
            transaction.commit();
            System.out.println("Contact added successfully: " + contact.getName());
            return true;
        } catch (Exception e) {
            if (transaction != null && transaction.isActive()) {
                try {
                    transaction.rollback();
                     System.err.println("Transaction rolled back for contact: " + contact.getName());
                } catch (Exception rbEx) {
                    System.err.println("Error rolling back transaction: " + rbEx.getMessage());
                }
            }
            System.err.println("Error adding contact: " + e.getMessage());
            e.printStackTrace(); // Log error
            return false;
        } finally {
            if (session != null && session.isOpen()) {
                session.close(); // Always close the session
            }
        }
    }

    public List<Contact> getAllContacts() {
         Session session = null;
        try {
             session = HibernateUtil.getSessionFactory().openSession();
            // Use HQL (Hibernate Query Language)
            // Make sure 'Contact' matches the class name, not table name
            Query<Contact> query = session.createQuery("FROM Contact c ORDER BY c.name", Contact.class);
            List<Contact> contacts = query.list();
            System.out.println("Retrieved " + contacts.size() + " contacts.");
            return contacts;
        } catch (Exception e) {
             System.err.println("Error retrieving contacts: " + e.getMessage());
            e.printStackTrace(); // Log error
            return Collections.emptyList(); // Return empty list on error
        } finally {
             if (session != null && session.isOpen()) {
                session.close(); // Always close the session
            }
        }
    }
}
```

---

**6. `src/main/java/com/example/contactbook/servlet/ContactServlet.java`**

```java
package com.example.contactbook.servlet;

import com.example.contactbook.bean.Contact;
import com.example.contactbook.dao.ContactDao;
import com.example.contactbook.util.HibernateUtil; // Import for shutdown

// Using javax.servlet for Servlet API 4 / Hibernate 5
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;
import java.util.List;

@WebServlet("/contacts") // Maps URL pattern /contacts to this servlet
public class ContactServlet extends HttpServlet {
    private ContactDao contactDao;

    @Override
    public void init() throws ServletException {
        System.out.println("ContactServlet initializing...");
        contactDao = new ContactDao();
         // Trigger SessionFactory creation on servlet init (optional, but good practice)
        HibernateUtil.getSessionFactory();
        System.out.println("ContactServlet initialized.");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        System.out.println("ContactServlet doPost called.");
        String name = request.getParameter("name");
        String email = request.getParameter("email");
        String phone = request.getParameter("phone");

        // Basic validation (add more robust validation as needed)
        if (name == null || name.trim().isEmpty() ||
            email == null || email.trim().isEmpty() ||
            phone == null || phone.trim().isEmpty()) {
            // Handle error - maybe forward back to form with an error message
            System.err.println("Validation failed: Missing contact details.");
             request.setAttribute("errorMessage", "All fields are required.");
             RequestDispatcher dispatcher = request.getRequestDispatcher("addContact.jsp");
             dispatcher.forward(request, response);
            return;
        }


        Contact newContact = new Contact();
        newContact.setName(name.trim());
        newContact.setEmail(email.trim());
        newContact.setPhone(phone.trim());

        boolean success = contactDao.addContact(newContact);

         if (!success) {
             System.err.println("Failed to add contact via DAO.");
             request.setAttribute("errorMessage", "Failed to save contact. Check logs.");
             RequestDispatcher dispatcher = request.getRequestDispatcher("addContact.jsp");
             dispatcher.forward(request, response);
             return;
         }

        System.out.println("Redirecting to GET /contacts after POST.");
        // Use context path for robust redirection
        response.sendRedirect(request.getContextPath() + "/contacts");
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
         System.out.println("ContactServlet doGet called.");
        List<Contact> contactList = contactDao.getAllContacts();
        request.setAttribute("contactList", contactList); // Store list in request scope
        RequestDispatcher dispatcher = request.getRequestDispatcher("/viewContacts.jsp"); // Use leading slash
        dispatcher.forward(request, response); // Forward to the JSP
         System.out.println("Forwarded to viewContacts.jsp");
    }

    @Override
    public void destroy() {
        System.out.println("ContactServlet destroying...");
        // Shutdown Hibernate SessionFactory when web application stops
        HibernateUtil.shutdown();
        System.out.println("ContactServlet destroyed.");
        super.destroy();
    }
}
```

---

**7. `src/main/resources/hibernate.cfg.xml`**

```xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <!-- == Database connection settings == -->
        <!-- Driver -->
        <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>

        <!-- URL -->
        <!-- !! IMPORTANT: Replace 'contact_db' if your DB name is different !! -->
        <!-- !! IMPORTANT: Add serverTimezone if needed by your driver/MySQL version !! -->
        <property name="connection.url">jdbc:mysql://localhost:3306/contact_db?useSSL=false&amp;serverTimezone=UTC</property>

        <!-- Credentials -->
        <!-- !! IMPORTANT: Replace with your actual MySQL username !! -->
        <property name="connection.username">root</property>
        <!-- !! IMPORTANT: Replace with your actual MySQL password !! -->
        <property name="connection.password">your_password</property>


        <!-- == Connection Pool (for testing only - use C3P0, HikariCP in production) == -->
        <property name="connection.pool_size">1</property>


        <!-- == SQL Dialect == -->
        <!-- Use MySQL8Dialect for MySQL 8.x, MySQL5Dialect for MySQL 5.x -->
        <property name="dialect">org.hibernate.dialect.MySQL8Dialect</property>


        <!-- == Optional Settings == -->
        <!-- Echo all executed SQL to stdout (useful for debugging) -->
        <property name="show_sql">true</property>
        <property name="format_sql">true</property>
        <property name="use_sql_comments">true</property>


        <!-- == Schema Management == -->
        <!-- Automatically validates or updates the schema -->
        <!-- Options: validate | update | create | create-drop | none -->
        <!-- 'update' is convenient for development but use with caution. -->
        <!-- 'validate' or 'none' are safer for production. -->
        <property name="hbm2ddl.auto">update</property>


        <!-- == Mention Annotated Entity Class == -->
        <mapping class="com.example.contactbook.bean.Contact"/>

    </session-factory>
</hibernate-configuration>
```
**---> REMEMBER TO CHANGE `connection.url`, `connection.username`, and `connection.password` ABOVE! <---**

---

**8. `src/main/webapp/addContact.jsp`**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Add Contact</title>
    <style>
        body { font-family: sans-serif; }
        .error { color: red; font-weight: bold; margin-bottom: 10px;}
        label { display: inline-block; width: 80px; margin-bottom: 5px;}
        input[type=text], input[type=email] { width: 200px; padding: 4px; margin-bottom: 10px;}
        input[type=submit] { padding: 5px 15px; cursor: pointer;}
        a { text-decoration: none; color: blue;}
        a:hover { text-decoration: underline;}
    </style>
</head>
<body>
    <h2>Add New Contact</h2>

    <%-- Display error message if present --%>
    <%
        String errorMessage = (String) request.getAttribute("errorMessage");
        if (errorMessage != null && !errorMessage.isEmpty()) {
    %>
        <p class="error"><%= errorMessage %></p>
    <%
        }
    %>

    <form action="<%= request.getContextPath() %>/contacts" method="post"> <%-- POST to ContactServlet --%>
        <div>
             <label for="name">Name:</label>
             <input type="text" id="name" name="name" required>
        </div>
        <div>
             <label for="email">Email:</label>
             <input type="email" id="email" name="email" required>
        </div>
         <div>
             <label for="phone">Phone:</label>
             <input type="text" id="phone" name="phone" required>
        </div>
        <div>
            <input type="submit" value="Add Contact">
        </div>
    </form>
    <br>
    <a href="<%= request.getContextPath() %>/contacts">View All Contacts</a> <%-- Link to GET handler --%>
</body>
</html>
```

---

**9. `src/main/webapp/viewContacts.jsp`**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.util.List" %>
<%@ page import="com.example.contactbook.bean.Contact" %>
<html>
<head>
    <title>Contact List</title>
    <style>
        body { font-family: sans-serif; }
        table { border-collapse: collapse; width: 80%; margin-top: 15px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        tr:nth-child(even) { background-color: #f9f9f9; }
        a { text-decoration: none; color: blue;}
        a:hover { text-decoration: underline;}
    </style>
</head>
<body>
    <h2>Contact List</h2>

    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>Phone</th>
            </tr>
        </thead>
        <tbody>
        <%
            // Retrieve the list passed from the servlet
            Object attribute = request.getAttribute("contactList");
            List<Contact> contactList = null;
            if (attribute instanceof List<?>) {
                 // Basic check to prevent ClassCastException if attribute is wrong type
                 contactList = (List<Contact>) attribute;
            }


            if (contactList != null && !contactList.isEmpty()) {
                for (Contact contact : contactList) {
        %>
            <tr>
                <td><%= contact.getId() %></td>
                <td><%= contact.getName() %></td>
                <td><%= contact.getEmail() %></td>
                <td><%= contact.getPhone() %></td>
            </tr>
        <%
                } // end for loop
            } else {
        %>
            <tr>
                <td colspan="4" style="text-align: center;">No contacts found.</td>
            </tr>
        <%
            } // end if/else
        %>
        </tbody>
    </table>
    <br>
    <a href="<%= request.getContextPath() %>/addContact.jsp">Add New Contact</a>
</body>
</html>
```

---

**10. `src/main/webapp/WEB-INF/web.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <display-name>SimpleContactBookHibernate</display-name>

    <!-- Optional: If not using @WebServlet in ContactServlet.java -->
    <!--
    <servlet>
        <servlet-name>ContactServlet</servlet-name>
        <servlet-class>com.example.contactbook.servlet.ContactServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>ContactServlet</servlet-name>
        <url-pattern>/contacts</url-pattern>
    </servlet-mapping>
    -->

    <!-- Define the first page to load -->
    <welcome-file-list>
        <welcome-file>addContact.jsp</welcome-file>
        <!-- You could also redirect to /contacts servlet here if you prefer -->
        <!-- <welcome-file>contacts</welcome-file> -->
    </welcome-file-list>

</web-app>
```

---

**11. MySQL Database Setup SQL**

Execute this in your MySQL client:

```sql
-- Create the database (if it doesn't exist)
-- !! Use the SAME database name as in hibernate.cfg.xml !!
CREATE DATABASE IF NOT EXISTS contact_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Switch to the database
USE contact_db;

-- Create the contacts table (Hibernate's hbm2ddl.auto=update might do this too, but good practice)
CREATE TABLE IF NOT EXISTS contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20) NOT NULL,
    -- Hibernate automatically manages tables based on Entity and hbm2ddl.auto
    -- No need for created_at unless you map it in the Entity
);

-- Optional: Verify table creation
-- DESCRIBE contacts;
```

---

**How to Run:**

1.  **Prerequisites:** JDK, IDE (Eclipse/IntelliJ), Apache Tomcat, MySQL Server.
2.  **Database:** Run the SQL script above. **CRITICAL:** Update the database URL, username, and password in `src/main/resources/hibernate.cfg.xml`.
3.  **Project Setup:**
    *   Create a new Maven Project in your IDE. Choose `maven-archetype-webapp`.
    *   Replace the generated `pom.xml` with the one provided above.
    *   Create the necessary package structure under `src/main/java`.
    *   Copy/Paste all the Java files (`Contact.java`, `HibernateUtil.java`, `ContactDao.java`, `ContactServlet.java`) into their respective packages.
    *   Create the `src/main/resources` folder and place `hibernate.cfg.xml` inside it.
    *   Place `addContact.jsp` and `viewContacts.jsp` directly under `src/main/webapp`.
    *   Place `web.xml` under `src/main/webapp/WEB-INF`.
4.  **Maven Update:** Right-click on your project in the IDE -> Maven -> Update Project (or Reimport). This downloads the dependencies defined in `pom.xml`. The JARs should now appear under Maven Dependencies (you don't need to manually put them in `WEB-INF/lib`).
5.  **Server Configuration:** Add your Apache Tomcat server to your IDE's Servers view.
6.  **Deploy & Run:**
    *   Right-click your project.
    *   Select "Run As" -> "Run on Server".
    *   Choose your configured Tomcat server.
    *   Click Finish.
7.  **Access:** Your IDE should start Tomcat and open a browser to `http://localhost:8080/SimpleContactBookHibernate/` (adjust port and context root if needed). This will show `addContact.jsp`. Use the form and links to navigate. Check your IDE console and Tomcat logs for output and errors.

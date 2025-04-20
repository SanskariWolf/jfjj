Okay, let's create the **Simple Contact Book Web App** using **Hibernate** instead of plain JDBC.

This version still covers:

*   **OOP:** `Contact` entity bean.
*   **JSP:** Pages for adding and viewing contacts.
*   **Servlet:** Handles adding contacts and retrieving them.
*   **Hibernate:** ORM layer for database interaction.
*   **MySQL:** The database.

**Key Differences from JDBC Version:**

*   We'll use Hibernate annotations on `Contact.java`.
*   `ContactDao.java` will use Hibernate's `Session` and HQL/Criteria API.
*   We need Hibernate configuration (`hibernate.cfg.xml` or similar).
*   We need Hibernate JARs and their dependencies.

---

**Files Needed:**

1.  `Contact.java` (Hibernate Entity)
2.  `HibernateUtil.java` (SessionFactory helper)
3.  `ContactDao.java` (Data Access Object using Hibernate)
4.  `ContactServlet.java` (Servlet Controller - *minor change to instantiate Hibernate DAO*)
5.  `addContact.jsp` (Input Form View - *No changes needed*)
6.  `viewContacts.jsp` (Display List View - *No changes needed*)
7.  `hibernate.cfg.xml` (Hibernate Configuration)
8.  `web.xml` (Servlet Mapping - *No changes needed if using `@WebServlet`*)
9.  MySQL Database Setup (*Schema remains the same*)
10. Project Dependencies (Hibernate JARs, MySQL Driver)

---

**Code & Configuration:**

**1. `Contact.java` (Hibernate Entity)**
*   Add Hibernate annotations (`@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`).

```java
package com.example.contactbook.bean;

import javax.persistence.Entity;
import javax.persistence.Table;
import javax.persistence.Id;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Column;
import java.io.Serializable; // Still good practice

@Entity // Mark class as a Hibernate entity
@Table(name = "contacts") // Map to the 'contacts' table
public class Contact implements Serializable {

    @Id // Mark as primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Auto-increment
    @Column(name = "id") // Map to the 'id' column
    private int id;

    @Column(name = "name", nullable = false, length = 100) // Map to 'name' column
    private String name;

    @Column(name = "email", nullable = false, unique = true, length = 100) // Map to 'email'
    private String email;

    @Column(name = "phone", nullable = false, length = 20) // Map to 'phone'
    private String phone;

    // Default Constructor (Required by Hibernate)
    public Contact() {}

    // Getters and Setters (Required by Hibernate/JSP EL)
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }

    // toString (Optional, for debugging)
    @Override
    public String toString() {
        return "Contact{" + "id=" + id + ", name='" + name + '\'' + ", email='" + email + '\'' + ", phone='" + phone + '\'' + '}';
    }
}
```

**2. `HibernateUtil.java` (SessionFactory Helper)**
*   Manages the single `SessionFactory` instance.

```java
package com.example.contactbook.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            // Create the SessionFactory from hibernate.cfg.xml
            return new Configuration().configure().buildSessionFactory();
        } catch (Throwable ex) {
            // Make sure you log the exception, as it might be masked
            System.err.println("Initial SessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        // Close caches and connection pools
        getSessionFactory().close();
    }
}
```

**3. `ContactDao.java` (Hibernate DAO)**
*   Uses Hibernate `Session` for CRUD.

```java
package com.example.contactbook.dao;

import com.example.contactbook.bean.Contact;
import com.example.contactbook.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query; // Use org.hibernate.query.Query

import java.util.Collections;
import java.util.List;

public class ContactDao {

    public boolean addContact(Contact contact) {
        Transaction transaction = null;
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            // Start a transaction
            transaction = session.beginTransaction();
            // Save the contact object
            session.save(contact);
            // Commit transaction
            transaction.commit();
            return true;
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace(); // Log error
            return false;
        }
    }

    public List<Contact> getAllContacts() {
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            // Use HQL (Hibernate Query Language)
            // Make sure 'Contact' matches the class name, not table name
            Query<Contact> query = session.createQuery("FROM Contact c ORDER BY c.name", Contact.class);
            return query.list();
        } catch (Exception e) {
            e.printStackTrace(); // Log error
            return Collections.emptyList(); // Return empty list on error
        }
    }
}
```

**4. `ContactServlet.java` (Servlet)**
*   Only the `init` method changes slightly to instantiate the Hibernate DAO.

```java
package com.example.contactbook.servlet;

import com.example.contactbook.bean.Contact;
import com.example.contactbook.dao.ContactDao; // Make sure this is the Hibernate DAO

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;
import java.util.List;

@WebServlet("/contacts")
public class ContactServlet extends HttpServlet {
    private ContactDao contactDao; // This will now be the Hibernate DAO

    public void init() {
        contactDao = new ContactDao(); // Instantiate the Hibernate DAO
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String name = request.getParameter("name");
        String email = request.getParameter("email");
        String phone = request.getParameter("phone");

        Contact newContact = new Contact();
        newContact.setName(name);
        newContact.setEmail(email);
        newContact.setPhone(phone);

        contactDao.addContact(newContact);
        response.sendRedirect("contacts");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        List<Contact> contactList = contactDao.getAllContacts();
        request.setAttribute("contactList", contactList);
        RequestDispatcher dispatcher = request.getRequestDispatcher("viewContacts.jsp");
        dispatcher.forward(request, response);
    }

    @Override
    public void destroy() {
         // Optional: Shutdown Hibernate SessionFactory when servlet is destroyed
         // com.example.contactbook.util.HibernateUtil.shutdown(); // Be careful with this in shared environments
        super.destroy();
    }
}
```

**5. `addContact.jsp` - No Changes Needed**

**6. `viewContacts.jsp` - No Changes Needed**

**7. `hibernate.cfg.xml` (Hibernate Configuration)**
*   Place this file in `src/main/resources` (if using Maven/Gradle structure) or directly under the `src` folder otherwise (so it ends up on the classpath).

```xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://localhost:3306/contact_db?useSSL=false&amp;serverTimezone=UTC</property> <!-- Your DB Name & settings -->
        <property name="connection.username">root</property> <!-- Your DB username -->
        <property name="connection.password">your_password</property> <!-- Your DB password -->

        <!-- JDBC connection pool (use a real pool in production!) -->
        <property name="connection.pool_size">1</property>

        <!-- SQL dialect -->
        <property name="dialect">org.hibernate.dialect.MySQL8Dialect</property> <!-- Or MySQL5Dialect if using MySQL 5.x -->

        <!-- Echo all executed SQL to stdout -->
        <property name="show_sql">true</property>
        <property name="format_sql">true</property>

        <!-- Drop and re-create the database schema on startup ( DANGEROUS in production!) -->
        <!-- Use 'validate' or 'update' for development, 'none' for production -->
        <property name="hbm2ddl.auto">update</property>

        <!-- Mention annotated class -->
        <mapping class="com.example.contactbook.bean.Contact"/>

    </session-factory>
</hibernate-configuration>
```
**Important:**
*   Update `connection.url`, `connection.username`, `connection.password`, and `dialect` according to your MySQL setup.
*   `hbm2ddl.auto` = `update` will try to update the schema. `validate` checks if the schema matches the entities. `create-drop` creates on start, drops on close. `create` creates on start, leaves it. `none` does nothing. Use `update` or `validate` carefully during development, and usually `none` or `validate` in production.

**8. `web.xml` - No Changes Needed (if using `@WebServlet`)**

**9. MySQL Database Setup - No Changes Needed**
*   The same SQL schema creation script works. Hibernate will map to the existing `contact_db` database and `contacts` table.

**10. Project Dependencies**
*   You need the Hibernate Core JARs and its dependencies (like JAXB, Byte Buddy, ANTLR, etc.) and the MySQL JDBC Driver JAR.
*   **If using Maven:** Add these to your `pom.xml`:
    ```xml
    <dependencies>
        <!-- Hibernate Core -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <!-- Use a recent stable version -->
            <version>6.4.4.Final</version> 
        </dependency>

        <!-- MySQL Connector -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>8.0.33</version> <!-- Use appropriate version -->
        </dependency>

        <!-- Servlet API (provided by server, scope=provided) -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>5.0.0</version> <!-- Adjust based on your Tomcat version -->
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>jakarta.servlet.jsp</groupId>
            <artifactId>jakarta.servlet.jsp-api</artifactId>
            <version>3.0.0</version> <!-- Adjust based on your Tomcat version -->
            <scope>provided</scope>
        </dependency>

        <!-- JSTL for JSP Expression Language (if you use it in JSPs) -->
       <!-- <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>jakarta.servlet.jsp.jstl</artifactId>
            <version>2.0.0</version> 
        </dependency> -->
    </dependencies>
    ```
*   **If not using Maven/Gradle:** Download the required JARs (Hibernate Core distribution bundle, MySQL JDBC driver) and place them all in your `src/main/webapp/WEB-INF/lib` folder.

---

**How to Run:**

1.  **Prerequisites:** Same as before (JDK, IDE, Tomcat, MySQL) **plus** Hibernate JARs (or Maven/Gradle setup).
2.  **Database:** Ensure MySQL is running and the `contact_db` database and `contacts` table exist (or let Hibernate create/update it based on `hbm2ddl.auto`). Update `hibernate.cfg.xml` with your DB credentials.
3.  **Create Project:** Create a Dynamic Web Project.
4.  **Add Code & Config:**
    *   Create packages and add the Java files (`Contact.java`, `HibernateUtil.java`, `ContactDao.java`, `ContactServlet.java`).
    *   Add the JSPs (`addContact.jsp`, `viewContacts.jsp`) to `webapp`.
    *   Add `hibernate.cfg.xml` to `src/main/resources` (Maven) or `src` (otherwise).
    *   Add `web.xml` to `webapp/WEB-INF` (if needed).
5.  **Add Dependencies:**
    *   **Maven/Gradle:** Let the build tool download dependencies.
    *   **Manual:** Copy all required Hibernate JARs and the MySQL JDBC Driver JAR into `webapp/WEB-INF/lib`.
6.  **Configure Server:** Ensure Tomcat is configured in your IDE.
7.  **Deploy and Run:** Right-click project -> Run As -> Run on Server -> Choose Tomcat.
8.  **Access the App:** Navigate to `http://localhost:8080/YourProjectName/` (or `/addContact.jsp` or `/contacts`).

This version achieves the same functionality but uses the more powerful Hibernate ORM for database operations.

Okay, here's a short and quick project idea focusing on the core web and database concepts you've learned: **A Simple Contact Book Web App**.

This covers:

*   **OOP:** A `Contact` JavaBean.
*   **JSP:** Pages for adding and viewing contacts.
*   **Servlet:** Handles adding contacts and retrieving them for display.
*   **JDBC:** Connects to MySQL to save and load contacts.
*   **MySQL:** The database to store the contacts.

**Project: Simple Contact Book**

**Features:**

1.  Add a new contact (Name, Email, Phone).
2.  View all saved contacts.

**Files Needed:**

1.  `Contact.java` (JavaBean)
2.  `ContactDao.java` (Data Access Object using JDBC)
3.  `ContactServlet.java` (Servlet Controller)
4.  `addContact.jsp` (Input Form View)
5.  `viewContacts.jsp` (Display List View)
6.  `web.xml` (Servlet Mapping)
7.  MySQL Database Setup

---

**Code Snippets (Illustrative - you'll need to fill them out):**

**1. `Contact.java` (JavaBean)**

```java
package com.example.contactbook.bean;

import java.io.Serializable;

public class Contact implements Serializable {
    private int id;
    private String name;
    private String email;
    private String phone;

    // Default Constructor
    public Contact() {}

    // Getters and Setters for id, name, email, phone
    // ... (generate these using your IDE)
}
```

**2. `ContactDao.java` (JDBC DAO)**

```java
package com.example.contactbook.dao;

import com.example.contactbook.bean.Contact;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ContactDao {
    // --- IMPORTANT: Replace with your actual DB details ---
    private String jdbcURL = "jdbc:mysql://localhost:3306/contact_db"; // Your DB name
    private String jdbcUsername = "root"; // Your DB username
    private String jdbcPassword = "your_password"; // Your DB password
    private String jdbcDriver = "com.mysql.cj.jdbc.Driver";
    // --- ---

    private Connection getConnection() throws SQLException, ClassNotFoundException {
        Class.forName(jdbcDriver);
        return DriverManager.getConnection(jdbcURL, jdbcUsername, jdbcPassword);
    }

    public boolean addContact(Contact contact) {
        String sql = "INSERT INTO contacts (name, email, phone) VALUES (?, ?, ?)";
        boolean rowInserted = false;
        try (Connection conn = getConnection();
             PreparedStatement statement = conn.prepareStatement(sql)) {

            statement.setString(1, contact.getName());
            statement.setString(2, contact.getEmail());
            statement.setString(3, contact.getPhone());
            rowInserted = statement.executeUpdate() > 0;

        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace(); // Handle appropriately
        }
        return rowInserted;
    }

    public List<Contact> getAllContacts() {
        List<Contact> contacts = new ArrayList<>();
        String sql = "SELECT * FROM contacts ORDER BY name";
        try (Connection conn = getConnection();
             Statement statement = conn.createStatement();
             ResultSet resultSet = statement.executeQuery(sql)) {

            while (resultSet.next()) {
                Contact contact = new Contact();
                contact.setId(resultSet.getInt("id"));
                contact.setName(resultSet.getString("name"));
                contact.setEmail(resultSet.getString("email"));
                contact.setPhone(resultSet.getString("phone"));
                contacts.add(contact);
            }
        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace(); // Handle appropriately
        }
        return contacts;
    }
}
```

**3. `ContactServlet.java` (Servlet)**

```java
package com.example.contactbook.servlet;

import com.example.contactbook.bean.Contact;
import com.example.contactbook.dao.ContactDao;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.WebServlet; // Or use web.xml mapping
import java.io.IOException;
import java.util.List;

@WebServlet("/contacts") // Map this servlet to the URL /contacts
public class ContactServlet extends HttpServlet {
    private ContactDao contactDao;

    public void init() {
        contactDao = new ContactDao();
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // Handle adding a new contact
        String name = request.getParameter("name");
        String email = request.getParameter("email");
        String phone = request.getParameter("phone");

        Contact newContact = new Contact();
        newContact.setName(name);
        newContact.setEmail(email);
        newContact.setPhone(phone);

        contactDao.addContact(newContact);
        response.sendRedirect("contacts"); // Redirect to the GET handler to view all
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // Handle displaying contacts
        List<Contact> contactList = contactDao.getAllContacts();
        request.setAttribute("contactList", contactList); // Store list in request scope
        RequestDispatcher dispatcher = request.getRequestDispatcher("viewContacts.jsp");
        dispatcher.forward(request, response); // Forward to the JSP
    }
}
```

**4. `addContact.jsp` (Input Form)**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Add Contact</title>
</head>
<body>
    <h2>Add New Contact</h2>
    <form action="contacts" method="post"> <%-- POST to ContactServlet --%>
        Name: <input type="text" name="name" required><br><br>
        Email: <input type="email" name="email" required><br><br>
        Phone: <input type="text" name="phone" required><br><br>
        <input type="submit" value="Add Contact">
    </form>
    <br>
    <a href="contacts">View All Contacts</a> <%-- Link to GET handler --%>
</body>
</html>
```

**5. `viewContacts.jsp` (Display List)**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.util.List" %>
<%@ page import="com.example.contactbook.bean.Contact" %>
<html>
<head>
    <title>Contact List</title>
    <style> table, th, td { border: 1px solid black; border-collapse: collapse; padding: 5px;} </style>
</head>
<body>
    <h2>Contact List</h2>
    <table>
        <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Phone</th>
        </tr>
        <%
            List<Contact> contactList = (List<Contact>) request.getAttribute("contactList");
            if (contactList != null && !contactList.isEmpty()) {
                for (Contact contact : contactList) {
        %>
            <tr>
                <td><%= contact.getName() %></td>
                <td><%= contact.getEmail() %></td>
                <td><%= contact.getPhone() %></td>
            </tr>
        <%
                }
            } else {
        %>
            <tr>
                <td colspan="3">No contacts found.</td>
            </tr>
        <%
            }
        %>
    </table>
    <br>
    <a href="addContact.jsp">Add New Contact</a>
</body>
</html>
```

**6. `web.xml` (Deployment Descriptor - if not using `@WebServlet`)**

Place this inside `src/main/webapp/WEB-INF/`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>ContactServlet</servlet-name>
        <servlet-class>com.example.contactbook.servlet.ContactServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>ContactServlet</servlet-name>
        <url-pattern>/contacts</url-pattern> <%-- Map URLs ending in /contacts to this servlet --%>
    </servlet-mapping>

    <welcome-file-list>
        <welcome-file>addContact.jsp</welcome-file> <%-- Start page --%>
    </welcome-file-list>
</web-app>
```

*(Note: If you use `@WebServlet("/contacts")` in `ContactServlet.java`, you might not need the `<servlet>` and `<servlet-mapping>` parts in `web.xml`, but the `welcome-file-list` is still useful).*

**7. MySQL Database Setup**

Execute this SQL in your MySQL client (like MySQL Workbench, phpMyAdmin, or command line):

```sql
-- Create the database (if it doesn't exist)
CREATE DATABASE IF NOT EXISTS contact_db;

-- Use the database
USE contact_db;

-- Create the contacts table
CREATE TABLE IF NOT EXISTS contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

**How to Run:**

1.  **Prerequisites:**
    *   Java Development Kit (JDK) installed.
    *   An IDE like Eclipse (for Web Developers), IntelliJ IDEA Ultimate, or VS Code with Java extensions.
    *   Apache Tomcat server installed and configured in your IDE.
    *   MySQL Server running.
    *   MySQL JDBC Driver JAR file downloaded. ([Download Link](https://dev.mysql.com/downloads/connector/j/))

2.  **Database:** Run the SQL script above to create the `contact_db` database and `contacts` table. **Remember** to update the `jdbcURL`, `jdbcUsername`, and `jdbcPassword` in `ContactDao.java` with your actual MySQL details.

3.  **Create Project:**
    *   In your IDE, create a new "Dynamic Web Project" (Eclipse) or equivalent Java EE/Web Application project.
    *   Name it something like `SimpleContactBook`.

4.  **Add Code:**
    *   Create the package structure (`com.example.contactbook.bean`, `.dao`, `.servlet`).
    *   Copy/paste the Java code into the respective files (`Contact.java`, `ContactDao.java`, `ContactServlet.java`).
    *   Place `addContact.jsp` and `viewContacts.jsp` directly under the `webapp` (or `WebContent`) folder.
    *   Place `web.xml` inside the `webapp/WEB-INF` folder.

5.  **Add JDBC Driver:**
    *   Copy the downloaded MySQL JDBC Driver JAR file (e.g., `mysql-connector-j-8.x.x.jar`) into the `src/main/webapp/WEB-INF/lib` folder of your project. (If the `lib` folder doesn't exist, create it).

6.  **Configure Server:**
    *   In your IDE, add your Apache Tomcat server if you haven't already.

7.  **Deploy and Run:**
    *   Right-click on your project in the IDE.
    *   Select "Run As" -> "Run on Server".
    *   Choose your configured Tomcat server.
    *   Your IDE should build, deploy the project to Tomcat, start Tomcat, and open a web browser.

8.  **Access the App:**
    *   The browser should open to `http://localhost:8080/SimpleContactBook/` (or similar, depending on your Tomcat port and project name). This should display `addContact.jsp` because of the `welcome-file` setting.
    *   You can directly navigate to `http://localhost:8080/SimpleContactBook/addContact.jsp` to add contacts.
    *   You can navigate to `http://localhost:8080/SimpleContactBook/contacts` to view the list (this triggers the `doGet` method of the servlet).

Now you have a basic web application demonstrating core concepts!

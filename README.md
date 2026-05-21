# Exp-04-Spring-Boot-with-REST-API-and-Hibernate-Integration
## AIM:
To develop a Spring Boot application to store and retrieve data from a Student database using Object Relational Mapping (ORM) with Hibernate and expose it via REST APIs.

## ALGORITHM:
Create a Spring Boot project using start.spring.io with the following dependencies:

Spring Web: For creating RESTful web services.

Spring Data JPA: To persist data in SQL stores with Java Persistence API using Spring Data and Hibernate.

H2 Database: An in-memory database, good for development and testing.

Configure application.properties to set up the H2 database connection, enable the H2 console, and configure JPA/Hibernate settings (like ddl-auto and show-sql).

Create the Student entity (POJO) in an entity package. Annotate it with @Entity and define fields like id, name, department, and email. Use @Id and @GeneratedValue for the primary key.

Create the StudentRepository interface in a repository package. Make it extend JpaRepository<Student, Long>. Spring Data JPA will automatically implement the basic CRUD methods.

Create a StudentService class in a service package. Annotate it with @Service and inject the StudentRepository. This class will contain the business logic for all CRUD operations.

Create the StudentController class in a controller package. Annotate it with @RestController and @RequestMapping("/students") to define the base URL.

Inject the StudentService into the controller.

Define REST endpoints for all CRUD operations, mapping them to the service methods:

POST /students: Create a new student.

GET /students: Retrieve all students.

GET /students/{id}: Retrieve a single student by their ID.

PUT /students/{id}: Update an existing student's details.

DELETE /students/{id}: Delete a student by their ID.

Run the application and test the endpoints using a tool like Postman.

### PROGRAM CODE (Main Files):
application.properties
Located in src/main/resources/application.properties

## Properties

### H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:studentdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password

### Enable H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

### JPA/Hibernate Configuration
### 'update' will create/update the schema based on entities without dropping data
```
spring.jpa.hibernate.ddl-auto=update
'true' will log all executed SQL statements to the console
spring.jpa.show-sql=true
```
### Student.java (Entity)
Located in src/main/java/com/example/jpa_project/entity/Student.java

Java
```
package com.example.jpa_project.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String department;
    private String email;

    public Student() {
    }

    public Student(String name, String department, String email) {
        this.name = name;
        this.department = department;
        this.email = email;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public String getEmail() {
        return email;
    }

}
```
### StudentRepository.java (Repository)
Located in src/main/java/com/example/jpa_project/repository/StudentRepository.java
```Java

package com.example.jpa_project.repository;

import com.example.jpa_project.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
}
StudentService.java (Service)
Located in src/main/java/com/example/jpa_project/service/StudentService.java

Java

package com.example.jpa_project.service;

import com.example.jpa_project.entity.Student;
import com.example.jpa_project.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class StudentService {

    @Autowired
    private StudentRepository studentRepository;

    public Student createStudent(Student student) {
        return studentRepository.save(student);
    }

    public List<Student> getAllStudents() {
        return studentRepository.findAll();
    }

    public Optional<Student> getStudentById(Long id) {
        return studentRepository.findById(id);
    }

    public Optional<Student> updateStudent(Long id, Student studentDetails) {
        return studentRepository.findById(id).map(existingStudent -> {
            existingStudent.setName(studentDetails.getName());
            existingStudent.setDepartment(studentDetails.getDepartment());
            existingStudent.setEmail(studentDetails.getEmail());
            return studentRepository.save(existingStudent);
        });
    }

    public boolean deleteStudent(Long id) {
        return studentRepository.findById(id).map(student -> {
            studentRepository.delete(student);
            return true;
        }).orElse(false); 
    }
}
```

### StudentController.java (Controller)
Located in src/main/java/com/example/jpa_project/controller/StudentController.java

Java
```
package com.example.jpa_project.controller;

import com.example.jpa_project.entity.Student;
import com.example.jpa_project.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/students") // Base path for all endpoints in this controller
public class StudentController {

    @Autowired
    private StudentService studentService;


    @PostMapping
    public Student createStudent(@RequestBody Student student) {
        return studentService.createStudent(student);
    }

    @GetMapping
    public List<Student> getAllStudents() {
        return studentService.getAllStudents();
    }


    @GetMapping("/{id}")
    public ResponseEntity<Student> getStudentById(@PathVariable Long id) {
        return studentService.getStudentById(id)
                .map(ResponseEntity::ok) // If found, return 200 OK with student
                .orElse(ResponseEntity.notFound().build()); // If not found, return 404 Not Found
    }

    @PutMapping("/{id}")
    public ResponseEntity<Student> updateStudent(@PathVariable Long id, @RequestBody Student studentDetails) {
        return studentService.updateStudent(id, studentDetails)
                .map(ResponseEntity::ok) // If updated, return 200 OK with updated student
                .orElse(ResponseEntity.notFound().build()); // If not found, return 404 Not Found
    }


    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteStudent(@PathVariable Long id) {
        // Use <Void> for an empty response body
        return studentService.deleteStudent(id)
                ? ResponseEntity.ok().<Void>build() // If deleted, return 200 OK
                : ResponseEntity.notFound().build(); // If not found, return 404 Not Found
    }
}
```
### OUTPUT (Testing with Postman):
Run the Spring Boot application.

Open Postman (or a similar API client).
//post
<img width="1429" height="675" alt="image" src="https://github.com/user-attachments/assets/4c26cbed-2c6d-40df-850d-70b6f7e7a354" />

//GET
<img width="1585" height="902" alt="image" src="https://github.com/user-attachments/assets/1f0f4896-76bf-4d6c-8984-b4bdcd3f3c79" />
//PUT
<img width="1592" height="933" alt="image" src="https://github.com/user-attachments/assets/42015804-3874-46c2-8c72-28da8eb31942" />
//DELETE
<img width="1586" height="917" alt="image" src="https://github.com/user-attachments/assets/1fe0b8b9-368b-4610-9938-80650cf87383" />







### RESULT:
A Spring Boot application was successfully developed to perform all CRUD operations on a Student database using Spring Data JPA (Hibernate) and exposed the functionality via RESTful APIs.

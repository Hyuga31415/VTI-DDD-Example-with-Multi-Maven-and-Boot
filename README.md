# MultiMavenDemo: DDD + Clean Architecture Demo Project

---

## Giới Thiệu Dự Án

### Mô Tả Chung
**MultiMavenDemo** là một dự án demo nhỏ gọn xây dựng trên nền tảng:
- **Spring Boot 2.3.1** - Framework web Java hiện đại
- **Domain-Driven Design (DDD)** - Phương pháp thiết kế hướng miền
- **Clean Architecture** - Kiến trúc sạch với tách biệt rõ ràng giữa các lớp
- **Multi-Module Maven** - Cấu trúc module hóa để quản lý mã nguồn

### Chức Năng Chính
Dự án cung cấp một hệ thống **quản lý khách hàng (Customer Management)** với các tính năng:
- Lấy danh sách tất cả khách hàng
- Tạo khách hàng mới
- Lấy thông tin khách hàng theo ID
- Xóa khách hàng
- ác thực người dùng qua JWT Token (AuthServer)

### Stack Công Nghệ
| Thành Phần | Công Nghệ |
|-----------|----------|
| Runtime | Java 11 |
| Framework | Spring Boot 2.3.1 |
| Build Tool | Apache Maven 3.x |
| Database | H2 (In-Memory) |
| ORM | Spring Data JPA |
| Authentication | JWT (JSON Web Token) |

---

## Vấn Đề Mà Project Giải Quyết

### 1. **Sự Phức Tạp của Các Ứng Dụng Lớn**
Khi xây dựng ứng dụng lớn, mã nguồn có thể trở nên vô tổ chức, khó bảo trì, và khó kiểm tra.

**Giải Pháp:** Dự án sử dụng **Clean Architecture** để tách biệt mối quan tâm (Separation of Concerns):
```
Presentation Layer (Controller) 
    ↓ (phụ thuộc)
Application Layer (Service)
    ↓ (phụ thuộc)
Domain Layer (Business Logic)
    ↓ (phụ thuộc)
Infrastructure Layer (Database)
```

### 2. **Khẳng Định Lại Các Quy Tắc Kinh Doanh**
Các quy tắc kinh doanh trong ứng dụng cần được bảo vệ từ những thay đổi bên ngoài.

**Giải Pháp:** **Domain-Driven Design (DDD)** đóng gói logic kinh doanh vào Entities, Aggregates, và Domain 

### 3. **Khó Kiểm Tra & Tái Sử Dụng**
Các module bị ràng buộc chặt với nhau khiến việc kiểm tra và tái sử dụng trở nên khó khăn.

**Giải Pháp:** **Dependency Inversion Principle** - Lớp cao phụ thuộc vào abstraction, không phụ thuộc vào chi tiết:
```java
// Service (Application Layer) phụ thuộc vào interface, không chi tiết
public class CustomerServiceImpl implements CustomerService {
    @Autowired
    private CustomerRepository customerRepo;  // Dependency Injection
}
```

---

## Kiến Trúc Hệ Thống

### 1. Domain-Driven Design (DDD) Là Gì?

**Domain-Driven Design** là một phương pháp thiết kế phần mềm tập trung vào:
- **Hiểu rõ lĩnh vực kinh doanh (Domain)**
- **Mô hình hóa Domain vào Code** - Domain Model
- **Tách biệt Domain Logic từ Infrastructure** - Domain Layer độc lập

#### Các Khái Niệm Chính DDD:

| Khái Niệm | Mô Tả | Ví Dụ |
|-----------|------|-------|
| **Entity** | Đối tượng có danh tính duy nhất, có vòng đời | Customer với ID |
| **Aggregate** | Nhóm các Entities/Value Objects được xem như một đơn vị | Customer và địa chỉ của nó |
| **Repository** | Cách để lấy/lưu Aggregates ra/vào persistent storage | CustomerRepository |
| **Domain Service** | Xử lý logic kinh doanh không thuộc về Entity nào cụ thể | Validation, Calculation |
| **Value Object** | Đối tượng không có danh tính, định nghĩa bằng giá trị | Address, Money |

#### Áp Dụng DDD Trong Project:
```
Domain Module (de.jenshardt.multimavendemo.customer.domain)
├── aggregate/
│   └── Customer.java         ← Aggregate Root (Entity)
├── repository/
│   └── CustomerRepository.java ← Repository Interface
└── service/
    └── CustomerService.java    ← Domain Service Interface
```

**Tại sao cấu trúc này quan trọng?**
- Domain Layer **độc lập** với Spring, Database, Web
- Logic kinh doanh được **bảo vệ** tại một nơi
- Dễ dàng kiểm tra Domain Logic mà không cần Database hoặc HTTP

---

### 2. Clean Architecture Là Gì?

**Clean Architecture** định nghĩa cách tổ chức mã nguồn theo **các vòng tròn lồng nhau**, với quy tắc cơ bản:

**"Các phụ thuộc luôn hướng vào trong (từ bên ngoài vào bên trong)"**

```
┌─────────────────────────────────────────┐
│      Outermost: Web, DB, Frameworks     │ ← Delivery (Controller)
├─────────────────────────────────────────┤
│           Application Layer             │ ← Use Cases (Service)
├─────────────────────────────────────────┤
│             Domain/Entities             │ ← Business Rules
├─────────────────────────────────────────┤
│      Most Inner: Enterprise Logic       │
└─────────────────────────────────────────┘
```

#### Các Nguyên Tắc Chính:

1. **Độc Lập với Framework**
   - Domain Layer không phụ thuộc Spring, JPA, hoặc XML
   
2. **Có Thể Kiểm Tra**
   - Unit test có thể chạy mà không cần Database, Web Server
   
3. **Không Phụ Thuộc UI**
   - Logic kinh doanh không thay đổi khi UI thay đổi
   
4. **Không Phụ Thuộc Database**
   - Dễ dàng chuyển từ H2 sang PostgreSQL mà không ảnh hưởng Domain
   
5. **Độc Lập Framework**
   - Có thể chạy trên Spring, Micronaut, hoặc framework khác

---

### 3. Cách Project Áp Dụng DDD + Clean Architecture

#### **Mô Hình 3 Lớp (3-Tier Architecture)**

```
┌────────────────────────────────────────────────────────┐
│ PRESENTATION LAYER - Controller Module                 │
│ - HTTP Request/Response Handling                        │
│ - REST Endpoints                                       │
│ - Input Validation (basic)                             │
└────────────────────────────────────────────────────────┘
                       ↓ depends on
┌────────────────────────────────────────────────────────┐
│ APPLICATION LAYER - Service Module                     │
│ - Business Logic Implementation                        │
│ - Transaction Management                              │
│ - Use Case Coordination                                │
│ - Calls Domain Services & Repositories                │
└────────────────────────────────────────────────────────┘
                       ↓ depends on
┌────────────────────────────────────────────────────────┐
│ DOMAIN LAYER - Domain Module                           │
│ - Entities (Customer)                                  │
│ - Domain Services                                      │
│ - Repository Interfaces                                │
│ - Business Rules & Validations                         │
│ NO Spring, JPA annotations (only necessary ones)   │
└────────────────────────────────────────────────────────┘
                       ↓ depends on
┌────────────────────────────────────────────────────────┐
│ INFRASTRUCTURE LAYER - (Implicit)                      │
│ - Database (H2, PostgreSQL)                            │
│ - External Services                                    │
│ - File System                                          │
└────────────────────────────────────────────────────────┘
```

---

## Cấu Trúc Dự Án

### Tổng Quan Module

```
MultiMavenDemo (Parent POM)
├── Domain/                    ← Core Domain Logic (No Framework!)
├── Service/                   ← Application Services
├── Controller/                ← REST API Layer
├── Application/               ← Spring Boot Entry Point
├── AuthServer/                ← JWT Authentication Service
└── pom.xml                    ← Parent POM
```

### Chi Tiết Mỗi Module

#### **1. Domain Module** (Lõi)
```
Domain/
├── src/main/java/.../customer/domain/
│   ├── aggregate/
│   │   └── Customer.java          ← Aggregate Root Entity
│   ├── repository/
│   │   └── CustomerRepository.java ← Repository Interface (JpaRepository)
│   └── service/
│       └── CustomerService.java    ← Domain Service Interface
└── pom.xml
```

**Trách Nhiệm:**
- Định nghĩa Entities (Aggregates)
- Khai báo Repository interfaces
- Định nghĩa Domain Services
- Chứa Business Rules

**Đặc Điểm:**
- Không chứa Spring `@Service`, `@Autowired`
- Chỉ chứa JPA annotations cần thiết (`@Entity`, `@Id`, `@Column`)
- Không phụ thuộc vào bất kì module nào khác
- Có thể kiểm tra mà không cần Runtime container

**Lợi Ích:**
- Core business model không bị contaminate bởi framework annotations
- Dễ hiểu: Đây là cốt lõi, không có "noise" từ infrastructure
- Testable: Có thể khởi tạo và test Entities mà không cần Spring

---

#### **2. Service Module** (Application Layer)
```
Service/
├── src/main/java/.../customer/service/
│   └── implementation/
│       └── CustomerServiceImpl.java ← Implementation của Domain Service
└── pom.xml
```

**Trách Nhiệm:**
- Thực hiện các Business Use Cases
- Điều phối giữa Domain Logic và Infrastructure
- Quản lý Transactions
- Xử lý Cross-Cutting Concerns (Logging, Security)

**Đặc Điểm:**
- Phụ thuộc vào Domain Module
- Chứa `@Service`, `@Transactional` annotations
- Inject Repository và Domain Services
- Thực hiện Use Cases (get customer, create customer, etc.)

**Lợi Ích:**
- Tách biệt Business Logic khỏi Web Framework
- Dễ test - kiểm tra logic mà không cần Database, HTTP
- Có thể thay đổi Web Framework mà không ảnh hưởng Service
- Tái sử dụng Service cho CLI, gRPC, hay REST

---

#### **3. Controller Module** (Presentation Layer)
```
Controller/
├── src/main/java/.../customer/controller/
│   └── CustomerController.java ← REST Endpoints
└── pom.xml
```

**Trách Nhiệm:**
- Xử lý HTTP Requests/Responses
- Định nghĩa REST Endpoints
- Validation Input (basic)
- Response Formatting

**Đặc Điểm:**
- Phụ thuộc vào Domain Module
- Chứa `@RestController`, `@GetMapping`, `@PostMapping`
- Inject Domain Service Interface
- **NỀN CHỈ gọi Domain Services, không gọi Repository trực tiếp**

**Lợi Ích:**
- Controller mỏng (thin controller)
- Dễ test thông qua mocking Service
- Không chứa business logic, chỉ orchestrate requests

---

#### **4. Application Module** (Bootstrapping)
```
Application/
├── src/main/java/.../customer/
│   ├── CustomerApplication.java  ← Main Spring Boot Class
│   └── config/
│       └── LoadDatabase.java      ← Database initialization
├── src/main/resources/
│   └── application.properties
└── pom.xml
```

**Trách Nhiệm:**
- Spring Boot Application Entry Point
- Bean Configuration & Component Scanning
- Database Initialization (seeding)
- Application Properties

**Đặc Điểm:**
- Phụ thuộc vào Controller & Service (để Component Scan)
- Chứa `@SpringBootApplication`
- `main()` method để chạy application

---

#### **5. AuthServer Module** (Optional)
```
AuthServer/
├── src/main/java/.../authentication/
│   ├── AuthApplication.java
│   ├── JwtAuthenticationController.java
│   ├── JwtTokenUtil.java
│   ├── JwtUserDetailsService.java
│   ├── JwtRequestFilter.java
│   └── WebSecurityConfig.java
└── pom.xml
```

**Trách Nhiệm:**
- JWT Token Authentication
- User Credentials Validation
- Security Configuration

---

## Quy Tắc Phụ Thuộc (Dependency Rules)

### Quy Tắc Vàng: **Inner Circle Never Depends on Outer Circle**

```
Application Layer (Service Implementation)
    ↑↓ (Service depends on Domain, không ngược lại)
Domain Layer (Core Business)
    ↑↓ (Domain maybe uses Repository interface)
Infrastructure (Database drivers, external APIs)
```

### Chi Tiết Quy Tắc

| Layer | Có Thể Phụ Thuộc | Không Thể Phụ Thuộc |
|-------|-----------------|-------------------|
| **Domain** | Không phụ thuộc layer nào | Spring, Web, Database drivers |
| **Service** | Domain, JPA, Database | Controller (reverse dependency!) |
| **Controller** | Domain, Service | Service Implementation directly |
| **Application** | Toàn bộ | Framework/Library (except Spring in config) |

**Tại Sao Điều Này Quan Trọng?**

**Độc lập từng Layer:** Thay đổi Database không ảnh hưởng Domain Logic
**Tái sử dụng:** Service có thể dùng cho REST, CLI, gRPC
**Kiểm tra:** Unit test Domain không cần Database
**Rõ ràng:** Ngay lập tức biết layer nào có business logic
---

## Cách Hoạt Động

### Request Flow Diagram

```
┌────────────────────────────────────────────────────────────┐
│ 1. HTTP Request từ Client                                   │
│    GET /customers/123                                       │
└────────┬─────────────────────────────────────────────────────┘
         │
         ↓
┌────────────────────────────────────────────────────────────┐
│ 2. CONTROLLER LAYER (Presentation)                          │
│    CustomerController.getCustomer(UUID customerId)          │
│    - Nhận HTTP request                                      │
│    - Validate PathVariable                                  │
│    - Gọi Service Interface                                  │
└────────┬─────────────────────────────────────────────────────┘
         │
         ↓ customerService.getCustomerById(customerId)
┌────────────────────────────────────────────────────────────┐
│ 3. SERVICE LAYER (Application)                              │
│    CustomerServiceImpl.getCustomerById(UUID customerId)      │
│    - Thực hiện use case logic                               │
│    - Điều phối giữa Domain & Infrastructure                │
│    - Quản lý transactions                                   │
└────────┬─────────────────────────────────────────────────────┘
         │
         ↓ customerRepo.findById(customerId)
┌────────────────────────────────────────────────────────────┐
│ 4. DOMAIN LAYER (Core Business)                             │
│    - CustomerRepository (Interface)                          │
│    - Customer (Entity/Aggregate)                             │
│    - CustomerService (Interface)                             │
│    - Business Rules & Validations                            │
└────────┬─────────────────────────────────────────────────────┘
         │
         ↓
┌────────────────────────────────────────────────────────────┐
│ 5. INFRASTRUCTURE LAYER (Implicit)                          │
│    - JpaRepository Implementation (Spring Data)             │
│    - Database Queries                                       │
│    - H2 Database / PostgreSQL                               │
└────────┬─────────────────────────────────────────────────────┘
         │
         ↓ Optional<Customer>
┌────────────────────────────────────────────────────────────┐
│ 6. Return Path (Response)                                   │
│    Customer object → JSON → HTTP 200/404                    │
└────────────────────────────────────────────────────────────┘
```

### Ví Dụ Chi Tiết: Tạo Khách Hàng Mới

#### **Step 1: Controller nhận request**
```http
POST /customers?name=John&job=Engineer
Content-Type: application/json
```

```java
@PostMapping("customers")
public ResponseEntity<Customer> postCustomer(
    @RequestParam String name,
    @RequestParam String job
) {
    LOG.info("Create new customer with name {} and job {}", name, job);
    // Gọi Service Interface (not implementation)
    return new ResponseEntity<Customer>(
        customerService.createCustomer(name, job),
        HttpStatus.CREATED
    );
}
```

#### **Step 2: Service thực hiện business logic**
```java
@Service
public class CustomerServiceImpl implements CustomerService {
    
    @Autowired
    private CustomerRepository customerRepo;
    
    @Override
    public Customer createCustomer(String name, String job) {
        // Business logic: tạo instance mới
        Customer customer = new Customer(name, job);
        
        // Có thể thêm validations, business rules:
        // - Validate name không rỗng
        // - Validate job tồn tại trong danh sách valid jobs
        // - Apply business rules, pricing, etc.
        
        // Lưu vào database qua Repository
        return customerRepo.save(customer);
    }
}
```

#### **Step 3: Domain chứa business entity**
```java
@Entity
@Table(name = "Person")
public class Customer {
    @Id
    @GeneratedValue
    private UUID id;
    
    @Column(name = "name")
    private String name;
    
    @Column(name = "job")
    private String job;
    
    // Constructor tạo instance mới
    public Customer(String name, String job) {
        this.name = name;
        this.job = job;
    }
    
    // Getters/Setters
    // Business methods (nếu có)
}

// Repository Interface
public interface CustomerRepository extends JpaRepository<Customer, UUID> {
}
```

#### **Step 4: Database execute & return**
```
INSERT INTO Person (id, name, job) VALUES (generated-uuid, 'John', 'Engineer')
↓
SELECT * FROM Person WHERE id = generated-uuid
↓
Customer object returned
```

#### **Step 5: Return JSON response**
```json
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "John",
  "job": "Engineer"
}
```

---

## API Endpoints

### Base URL
```
http://localhost:8080
```

### Endpoint Reference

#### **1. Lấy Danh Sách Tất Cả Khách Hàng**
```http
GET /customers
```

**Response (200 OK):**
```json
[
  {
    "id": "8fe580dd-b6ca-4681-96dd-a9478a33123a",
    "name": "Bilbo Baggins",
    "job": "burglar"
  },
  {
    "id": "5ea2556b-3db6-48f8-88fa-b09ec2fe0d41",
    "name": "Frodo Baggins",
    "job": "thief"
  }
]
```

---

#### **2. Lấy Thông Tin Khách Hàng Theo ID**
```http
GET /customers/{customerId}
```

**Path Parameters:**
- `customerId` (UUID): ID của khách hàng

**Example:**
```http
GET /customers/8fe580dd-b6ca-4681-96dd-a9478a33123a
```

**Response (200 OK):**
```json
{
  "id": "8fe580dd-b6ca-4681-96dd-a9478a33123a",
  "name": "Bilbo Baggins",
  "job": "burglar"
}
```

**Response (404 Not Found):**
```json
```
(Empty response)

---

#### **3. Tạo Khách Hàng Mới**
```http
POST /customers
Content-Type: application/x-www-form-urlencoded
```

**Query Parameters:**
- `name` (String): Tên khách hàng (required)
- `job` (String): Công việc khách hàng (required)

**Example:**
```http
POST /customers?name=Gandalf&job=wizard
```

**Response (201 Created):**
```json
{
  "id": "cb52a679-fd4d-432f-95c7-96f91ee3227a",
  "name": "Gandalf",
  "job": "wizard"
}
```

---

#### **4. Xóa Khách Hàng**
```http
DELETE /customers/{customerId}
```

**Path Parameters:**
- `customerId` (UUID): ID của khách hàng

**Example:**
```http
DELETE /customers/cb52a679-fd4d-432f-95c7-96f91ee3227a
```

**Response (200 OK):**
```json
true
```

**Response (404 Not Found):**
```json
false
```

---

### API Testing

#### **Sử dụng cURL:**
```bash
# Get all customers
curl -X GET http://localhost:8080/customers

# Get customer by ID
curl -X GET http://localhost:8080/customers/8fe580dd-b6ca-4681-96dd-a9478a33123a

# Create new customer
curl -X POST "http://localhost:8080/customers?name=Aragorn&job=king"

# Delete customer
curl -X DELETE http://localhost:8080/customers/8fe580dd-b6ca-4681-96dd-a9478a33123a
```

#### **Sử dụng Postman:**
1. Import endpoints trên
2. Set Method: GET, POST, DELETE
3. Set Headers: `Content-Type: application/json` (nếu cần)
4. Send request

#### **H2 Database Console:**
```
URL: http://localhost:8080/h2-console

```

---

#### **5. Test API**
```bash
# Terminal 1: Application đang chạy
# Terminal 2: Test API

# Get all customers
curl http://localhost:8080/customers

# Create new customer
curl -X POST "http://localhost:8080/customers?name=Legolas&job=archer"

# Get customer by ID
curl http://localhost:8080/customers/{customerId}

# Delete customer
curl -X DELETE http://localhost:8080/customers/{customerId}
```

---

## Lợi Ích Của Kiến Trúc Này

### 1. **Separation of Concerns (SoC)**
```
Mỗi layer có một trách nhiệm duy nhất:
- Controller: HTTP handling
- Service: Business logic coordination
- Domain: Core business rules
- Infrastructure: Data access
```
Dễ hiểu, dễ bảo trì, dễ mở rộng

### 2. **Testability (Khả Năng Kiểm Tra)**
```java
// Unit test Domain logic mà không cần Database
// Unit test Service bằng mocking Repository
@Test
void testCreateCustomer() {
    CustomerService service = new CustomerServiceImpl();
    Customer customer = service.createCustomer("Test", "test-job");
    Assert.assertNotNull(customer.getId());
}
```

### 3. **Reusability (Tái Sử Dụng)**
```
Có thể dùng CustomerService cho:
- REST API (Controller)
- gRPC service
- GraphQL endpoint
- CLI command
- Message queue consumer
```

### 4. **Scalability (Khả Năng Mở Rộng)**
```
Thêm module mới mà không ảnh hưởng existing code
- Thêm Order module
- Thêm Payment module
- Thêm Shipping module
```

### 5. **Flexibility (Linh Hoạt)**
```
Dễ dàng thay đổi implementation:
- Thay H2 bằng PostgreSQL → Chỉ thay pom.xml
- Thay Spring MVC bằng Spring WebFlux → Chỉ sửa Controller
- Thay JPA bằng MongoDB → Chỉ thay Service impl
```

### 6. **Framework Independence (Độc Lập Framework)**
```
Domain logic không phụ thuộc Spring:
- Có thể port sang framework khác (Quarkus, Micronaut)
- Có thể reuse logic cho batch processing
- Có thể test offline
```
---

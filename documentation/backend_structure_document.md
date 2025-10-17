# Cafe_system Backend Structure Document

This document outlines the backend setup for the Cafe_system project. It explains the overall architecture, database design, APIs, hosting, infrastructure, and security measures in everyday language so anyone can understand how the backend works.

## 1. Backend Architecture

### Overview
- We use a **layered architecture**, where each layer has a clear responsibility:
  - **API Layer** handles incoming HTTP requests and routes them.
  - **Service Layer** contains the business logic (order processing, inventory updates, etc.).
  - **Data Access Layer** interacts with the database through an ORM (Object-Relational Mapper).
- The backend is built with **Node.js** and **Express**, providing a lightweight, fast environment for handling requests.

### Design Patterns and Frameworks
- **Model-View-Controller (MVC)** pattern to keep code organized:
  - Models define the data structure.
  - Controllers manage requests and responses.
  - Services encapsulate business rules.
- **Repository Pattern** in the data access layer for clean separation between database queries and the rest of the code.
- **Express middleware** for logging, error handling, and authentication.
- **TypeScript** for type safety and easier code maintenance.

### Scalability, Maintainability, Performance
- **Scalability**:
  - Each component runs in its own container (Docker), making it easy to scale horizontally.
  - Stateless API servers can be replicated behind a load balancer.
- **Maintainability**:
  - Clear module boundaries (orders, inventory, users, payments).
  - Consistent code style enforced by **CodeGuide** (linting, formatting).
  - Automated tests (unit and integration) ensure changes don’t break existing features.
- **Performance**:
  - Caching frequently accessed data (e.g., menu items) in **Redis**.
  - Database indexing on key fields (e.g., order status, customer ID).
  - Connection pooling to manage database connections efficiently.

## 2. Database Management

### Technologies Used
- **Primary Database**: PostgreSQL (relational SQL database)
- **Cache**: Redis (in-memory key-value store)

### Data Organization and Practices
- We store structured data (orders, menu items, customers) in PostgreSQL for strong consistency.
- Redis holds temporary data like session tokens, rate-limiting counters, and cached query results.
- Backups:
  - Automated daily snapshots of PostgreSQL.
  - Redis uses AOF (Append Only File) for persistence.
- Migrations managed by **Sequelize** (ORM) to ensure database schema changes are applied consistently across environments.

## 3. Database Schema

### Human-Readable Overview
- **Customers**: Profiles with name, contact info, order history, loyalty points.
- **Menu_Items**: Dishes or drinks offered, including name, category, price, and available modifiers.
- **Orders**: Records of customer orders, linked to customers, containing items, quantities, status, timestamps.
- **Order_Items**: Line items in each order, referencing a menu item, selected modifiers, and quantity.
- **Inventory**: Tracks current stock levels for each ingredient, reorder thresholds, supplier info.
- **Payments**: Records payment transactions, method (cash, card, digital wallet), amount, timestamp.
- **Users**: Staff accounts with roles (staff, manager, admin) and permissions.
- **Configurations**: System-wide settings like tax rate, operational hours, feature toggles.

### SQL Schema (PostgreSQL)
```sql
-- Customers
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(150) UNIQUE NOT NULL,
  phone VARCHAR(20),
  loyalty_points INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Menu Items
CREATE TABLE menu_items (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  category VARCHAR(50),
  price DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Orders
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INT REFERENCES customers(id),
  status VARCHAR(20) NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Order Items
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INT REFERENCES orders(id) ON DELETE CASCADE,
  menu_item_id INT REFERENCES menu_items(id),
  modifiers JSONB,
  quantity INT NOT NULL,
  unit_price DECIMAL(10,2) NOT NULL
);

-- Inventory
CREATE TABLE inventory (
  id SERIAL PRIMARY KEY,
  ingredient_name VARCHAR(100) NOT NULL,
  quantity_in_stock INT NOT NULL,
  reorder_threshold INT NOT NULL,
  supplier VARCHAR(100),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Payments
CREATE TABLE payments (
  id SERIAL PRIMARY KEY,
  order_id INT REFERENCES orders(id),
  method VARCHAR(20) NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  paid_at TIMESTAMP DEFAULT NOW()
);

-- Users
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Configurations
CREATE TABLE configurations (
  key VARCHAR(50) PRIMARY KEY,
  value VARCHAR(255) NOT NULL
);
```

## 4. API Design and Endpoints

### Approach
- We use **RESTful APIs** over HTTPS.
- Each resource (customers, orders, menu_items, inventory, payments, users, configurations) has its own set of endpoints.
- Controllers validate inputs and call services; services interact with the data layer.

### Key Endpoints
- **Customers**
  - `GET /api/customers` – list all customers
  - `GET /api/customers/{id}` – get a single customer
  - `POST /api/customers` – create a new customer
  - `PUT /api/customers/{id}` – update customer data
- **Orders**
  - `GET /api/orders` – list orders, filter by status or date
  - `POST /api/orders` – create a new order
  - `PUT /api/orders/{id}/status` – update order status (e.g., preparing, ready)
- **Menu Items**
  - `GET /api/menu` – fetch menu items and categories
  - `POST /api/menu` – add a new menu item (admin only)
- **Inventory**
  - `GET /api/inventory` – view stock levels
  - `POST /api/inventory` – add or adjust stock
- **Payments**
  - `POST /api/payments` – record a payment for an order
- **Authentication & Users**
  - `POST /api/auth/login` – authenticate and receive a token
  - `POST /api/auth/logout` – invalidate the session
  - `GET /api/users` – list users (admin only)
- **Configurations**
  - `GET /api/config` – list settings
  - `PUT /api/config/{key}` – update a setting (admin only)

## 5. Hosting Solutions

### Cloud Provider
- We host the backend on **Amazon Web Services (AWS)**.
  - **Elastic Container Service (ECS)** or **EKS** for running Docker containers.
  - **RDS (PostgreSQL)** for managed database instances.
  - **ElastiCache (Redis)** for caching.

### Benefits
- **Reliability**: Managed services with automatic failover, regular backups.
- **Scalability**: Auto scaling of containers and database resources.
- **Cost-Effectiveness**: Pay only for what you use; reserved instances for predictable workloads.

## 6. Infrastructure Components

- **Load Balancer** (AWS Application Load Balancer): Distributes incoming traffic across multiple container instances.
- **Caching** (Redis via ElastiCache): Speeds up repeated reads (e.g., menu data, sessions).
- **Content Delivery Network** (CloudFront): Serves static assets (images, receipts) close to users.
- **Message Queue** (AWS SQS): Handles background tasks like sending notifications or batch inventory updates.
- **CI/CD Pipeline** (GitHub Actions or AWS CodePipeline): Automates testing, linting (via CodeGuide), and deployments.

## 7. Security Measures

- **Authentication**: JSON Web Tokens (JWT) for stateless, secure sessions.
- **Authorization**: Role-based access control (RBAC) enforced in middleware.
- **Data Encryption**:
  - TLS for all data in transit.
  - AES-256 encryption at rest (RDS encryption, S3 for backups).
- **Password Management**: Bcrypt hashing.
- **Vulnerability Scanning**: Automated scans in CI pipeline (CodeGuide, Dependabot).
- **Rate Limiting**: Throttles suspicious traffic to prevent DOS attacks.
- **Audit Logging**: Records critical actions (logins, configuration changes) for compliance.

## 8. Monitoring and Maintenance

- **Monitoring Tools**:
  - **AWS CloudWatch** for metrics (CPU, memory, database connections).
  - **Datadog** or **New Relic** for application performance monitoring (APM).
  - **Sentry** for error tracking and alerting.
- **Maintenance Strategies**:
  - **Automated Backups**: Daily DB snapshots, weekly full backups.
  - **Dependency Updates**: Monthly review and updates via Dependabot.
  - **Health Checks**: ECS health checks restart unhealthy containers automatically.
  - **Scheduled Maintenance Windows**: Communicated in advance for version upgrades.

## 9. Conclusion and Overall Backend Summary

The Cafe_system backend is built to support a full suite of cafe operations—from order management and inventory tracking to payments and reporting—using a clear layered architecture in Node.js/Express. Data lives in PostgreSQL with Redis for speed. We host on AWS for reliability and scalability, with robust security, monitoring, and automated workflows. All these components work together to ensure a fast, reliable, and maintainable system ready to grow with cafe needs.
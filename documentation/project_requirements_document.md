# Project Requirements Document (PRD) for Cafe_system

## 1. Project Overview
Cafe_system is a web-based application designed to manage everyday operations of a cafe—covering order handling, inventory control, customer profiles, point-of-sale transactions, and analytics. Its core purpose is to streamline workflows for staff, provide transparency to customers, reduce manual errors, and give managers actionable insights. Rather than juggling paper tickets, spreadsheets, and separate payment tools, Cafe_system centralizes everything under one roof.

We’re building this system because small and medium-sized cafes often lack affordable, integrated software tailored to their unique day-to-day needs. The key objectives for version 1.0 are: (1) reliable order management from creation through fulfillment, (2) real-time inventory tracking with low-stock alerts, (3) basic POS checkout with multiple payment methods, (4) customer profiles for order history and loyalty, and (5) high-level sales reports. Success will be measured by reduced order errors, faster checkout times, and positive feedback from cafe staff during beta testing.

## 2. In-Scope vs. Out-of-Scope

### In-Scope (Version 1.0)
- Order Management: Create, modify, track status, and assign staff.
- Inventory Control: Automatic stock decrement, threshold alerts, and manual adjustments.
- Customer Profile Management: Register, view order history, and loyalty status.
- Point-of-Sale Transactions: Cash/card/wallet payments via a single checkout interface; generate digital or printed receipts.
- Reporting & Analytics: Dashboards for daily sales, popular items, and basic staff performance metrics.
- User Authentication & Authorization: Role-based access for staff, managers, administrators.
- CodeGuide Integration: Static code analysis, linting, formatting rules enforced via CI pipeline.
- Basic Configuration Module: Adjustable tax rates, menu categories, and operation hours.

### Out-of-Scope (Planned for Later Phases)
- Mobile-native applications (iOS/Android).
- Advanced promotions engine (e.g., time-based discounts, coupons).
- Third-party delivery platform integrations.
- Multi-location support (only single-cafe setup in v1).
- Multi-currency or advanced tax jurisdictions.
- In-depth custom report builder (limited to preset dashboards in v1).

## 3. User Flow

When a staff member logs in, they land on a dashboard showing current open orders and low-stock alerts in a sidebar. The main panel displays a list of in-progress orders with filters (e.g., by status or assigned staff). To create a new order, the user clicks “New Order,” selects menu items (with modifiers like size or add-ons), assigns it to themselves or a barista, and submits. Real-time updates move the order from “Pending” through “Preparing” to “Ready” as staff members update its status.

At checkout, the user selects a completed order, clicks “Checkout,” picks a payment method (cash, card, digital wallet), and processes payment via an integrated payment gateway. A receipt is generated automatically. Managers access a separate “Reports” section where they can view charts of daily revenue, top-selling items, and simple staff performance metrics. All settings—like tax rate or menu categories—are adjusted in an “Admin Settings” area accessible only to administrators.

## 4. Core Features
- Authentication & Authorization: Secure login, password hashing, session management, and role-based permissions.
- Order Management Module: CRUD for orders, status workflows, item modifiers, staff assignment, and timestamped status updates.
- Inventory Control Module: Ingredient stock tracking, automatic decrement on fulfillment, low-stock thresholds, notifications, and manual stock adjustments.
- Customer Profiles Module: Store personal details, order history, loyalty points, and basic preferences.
- POS Checkout Module: Unified interface for processing payments, integrating with payment gateway APIs, handling refunds, and generating receipts.
- Reporting & Analytics Module: Predefined dashboards for sales trends, item popularity, and staff metrics; export to CSV.
- Configuration Module: Admin interface for global settings—tax rates, operation hours, menu categories, and feature toggles.
- CodeGuide Integration: Automated linting, formatting, and static analysis enforced in CI/CD pipelines.

## 5. Tech Stack & Tools
- Frontend: React with TypeScript, React Router for navigation, Material-UI for UI components, Redux or Context API for state management.
- Backend: Node.js with Express (or NestJS) in TypeScript, RESTful API endpoints.
- Database: PostgreSQL for relational data, Sequelize or TypeORM as ORM.
- Authentication: JSON Web Tokens (JWT) for session handling.
- Payment Gateway: Stripe or PayPal SDK.
- DevOps & CI/CD: GitHub Actions or GitLab CI for builds; Docker for containerization; CodeGuide for linting & static analysis.
- Hosting: Cloud provider such as AWS (EC2/RDS) or Heroku.

## 6. Non-Functional Requirements
- Performance: API response time under 200ms for 95% of requests; frontend page load under 1s on fast connections.
- Security: OWASP Top 10 compliance, HTTPS enforced, password hashing with bcrypt, rate limiting on sensitive endpoints.
- Scalability: Support up to 50 concurrent users without degradation; stateless backend services to allow horizontal scaling.
- Reliability: 99.5% uptime SLA; database daily backups.
- Usability: Intuitive UI with consistent styling; follow WCAG 2.1 AA for basic accessibility.

## 7. Constraints & Assumptions
- CodeGuide must remain operational; builds will fail if static analysis rules breach.
- Single-cafe context assumed; no multi-tenant requirements in v1.
- Payment gateway account credentials available at launch.
- Managers and admins have desktop or tablet devices—mobile support is secondary.
- Internet connectivity is stable; offline mode is not required in v1.

## 8. Known Issues & Potential Pitfalls
- API Rate Limits: Payment gateways may impose rate limits—implement retry logic and exponential backoff.
- Inventory Race Conditions: Simultaneous order fulfillment could cause stock underflow—use database transactions and row locking.
- Data Consistency: Ensure transactional integrity between order creation and inventory decrement.
- CodeGuide Overhead: Strict linting may slow developers—balance rules with practical development speed.
- Browser Compatibility: Test on Chrome, Firefox, and Safari; polyfill as needed.

---

This PRD lays out all core requirements and boundaries for Cafe_system version 1.0. It’s designed to be the single source of truth for AI-driven generation of technical docs, design specs, and implementation guidelines without ambiguity.
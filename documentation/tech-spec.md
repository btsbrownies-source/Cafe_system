# Tech Specification: Multi-Tenant E-Ordering Cafe System

## Project Overview

A comprehensive multi-tenant e-ordering platform designed for cafes and restaurants with AI-powered customer recommendations, subscription-based pricing, and complete restaurant management capabilities.

## System Requirements

### Business Requirements
- Multi-tenant architecture supporting unlimited restaurants
- AI-powered menu recommendations with conversational interface
- Subscription model: Free (3 menu limit) and Premium (50,000 IDR/month)
- QRIS payment integration with proof upload system
- Complete restaurant management: Order, Cashier, Kitchen modules
- Admin dashboard for tenant and payment management

### Technical Stack
- **Backend**: PHP 8.1+ (with Composer)
- **Frontend**: Bootstrap 5, JavaScript, HTML5
- **Database**: MySQL 8.0+
- **Architecture**: Multi-tenant SaaS with schema isolation
- **Payment**: QRIS integration
- **AI**: JavaScript-based recommendation engine with NLP

## System Architecture

### Multi-Tenancy Model
**Schema-per-Tenant Approach** for complete data isolation:

```
Master Database:
├── tenants (tenant information)
├── subscriptions
├── plans
├── payments
└── admin_users

Tenant Database (per restaurant):
├── tenant_{id}_menus
├── tenant_{id}_orders
├── tenant_{id}_users
├── tenant_{id}_categories
└── tenant_{id}_settings
```

### Subdomain Routing
- `{restaurant_name}.cafeorder.com` format
- Automatic tenant identification via subdomain
- Fallback to path-based routing: `cafeorder.com/{restaurant_name}`

## Database Schema

### Master Database Tables

#### tenants
```sql
CREATE TABLE tenants (
    id INT PRIMARY KEY AUTO_INCREMENT,
    restaurant_name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    address TEXT,
    subscription_status ENUM('free', 'premium', 'suspended') DEFAULT 'free',
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### plans
```sql
CREATE TABLE plans (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    menu_limit INT DEFAULT 3,
    features JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### subscriptions
```sql
CREATE TABLE subscriptions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    tenant_id INT NOT NULL,
    plan_id INT NOT NULL,
    status ENUM('active', 'expired', 'pending', 'cancelled') DEFAULT 'pending',
    start_date DATE,
    end_date DATE,
    auto_renew BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (tenant_id) REFERENCES tenants(id),
    FOREIGN KEY (plan_id) REFERENCES plans(id)
);
```

#### payments
```sql
CREATE TABLE payments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    tenant_id INT NOT NULL,
    subscription_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_method ENUM('qris') DEFAULT 'qris',
    qris_code VARCHAR(255),
    proof_of_payment VARCHAR(255),
    status ENUM('pending', 'verified', 'rejected') DEFAULT 'pending',
    payment_date TIMESTAMP NULL,
    verified_by INT NULL,
    verified_at TIMESTAMP NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (tenant_id) REFERENCES tenants(id),
    FOREIGN KEY (subscription_id) REFERENCES subscriptions(id)
);
```

### Tenant Database Schema

#### categories
```sql
CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    image_url VARCHAR(255),
    sort_order INT DEFAULT 0,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### menu_items
```sql
CREATE TABLE menu_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    category_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    image_url VARCHAR(255),
    preparation_time INT DEFAULT 15, -- in minutes
    status ENUM('available', 'unavailable') DEFAULT 'available',
    is_featured BOOLEAN DEFAULT FALSE,
    sort_order INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

#### orders
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    customer_name VARCHAR(255),
    customer_phone VARCHAR(20),
    table_number VARCHAR(10),
    order_type ENUM('dine_in', 'takeaway', 'delivery') DEFAULT 'dine_in',
    status ENUM('pending', 'confirmed', 'preparing', 'ready', 'completed', 'cancelled') DEFAULT 'pending',
    subtotal DECIMAL(10,2) NOT NULL,
    tax DECIMAL(10,2) DEFAULT 0,
    service_charge DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    payment_method ENUM('cash', 'qris', 'transfer') DEFAULT 'cash',
    payment_status ENUM('pending', 'paid', 'refunded') DEFAULT 'pending',
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP NULL
);
```

#### order_items
```sql
CREATE TABLE order_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    menu_item_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (menu_item_id) REFERENCES menu_items(id)
);
```

## System Modules

### 1. Multi-Tenant Core System

#### Tenant Registration Flow
1. Restaurant fills registration form
2. System validates subdomain availability
3. Create tenant database schema
4. Set up default categories and admin user
5. Send welcome email with credentials

#### Tenant Identification Middleware
```php
// app/Middleware/TenantMiddleware.php
class TenantMiddleware {
    public function handle($request, $next) {
        $subdomain = $this->extractSubdomain($request);
        $tenant = Tenant::where('subdomain', $subdomain)->first();
        
        if (!$tenant || $tenant->status !== 'active') {
            abort(404, 'Restaurant not found');
        }
        
        // Set tenant database connection
        config(['database.connections.tenant' => [
            'driver' => 'mysql',
            'database' => "tenant_{$tenant->id}",
            // ... other config
        ]]);
        
        return $next($request);
    }
}
```

### 2. Subscription Management System

#### Plan Features
- **Free Plan**: 3 menu items, basic features
- **Premium Plan**: Unlimited menus, AI recommendations, analytics

#### QRIS Payment Integration
```php
// app/Services/PaymentService.php
class PaymentService {
    public function generateQRIS($amount, $orderId) {
        // Integration with QRIS provider (e.g., Midtrans, Xendit)
        return [
            'qris_code' => $this->qrisProvider->generateQR($amount, $orderId),
            'expires_at' => now()->addHours(24)
        ];
    }
    
    public function uploadProof($paymentId, $file) {
        // Validate and store proof of payment
        // Send notification to admin
    }
}
```

#### Subscription Enforcement
```php
// app/Http/Middleware/SubscriptionMiddleware.php
class SubscriptionMiddleware {
    public function handle($request, $next) {
        $tenant = $this->getCurrentTenant();
        $plan = $tenant->subscription->plan;
        
        // Check menu limit for free tier
        if ($plan->menu_limit > 0) {
            $menuCount = MenuItem::count();
            if ($menuCount >= $plan->menu_limit) {
                return redirect()->route('subscription.upgrade');
            }
        }
        
        return $next($request);
    }
}
```

### 3. Restaurant Management System

#### Menu Management
- CRUD operations for menu items
- Image upload with optimization
- Category organization
- Availability management
- Pricing control

#### Order Processing
```javascript
// Real-time order updates using Server-Sent Events
const eventSource = new EventSource('/orders/stream');
eventSource.onmessage = function(event) {
    const data = JSON.parse(event.data);
    updateOrderDisplay(data);
};
```

#### Kitchen Display System
- Real-time order status
- Preparation timer
- Order priority management
- Staff assignment

### 4. Customer Ordering Interface

#### AI Recommendation Engine
```javascript
// app/Assets/js/ai-recommender.js
class AIRecommender {
    constructor() {
        this.chatHistory = [];
        this.menuData = [];
    }
    
    async getRecommendations(userInput, orderHistory = []) {
        const response = await fetch('/api/ai/recommend', {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({
                message: userInput,
                history: orderHistory,
                preferences: this.userPreferences
            })
        });
        
        return response.json();
    }
    
    generateResponse(input, menuItems) {
        // Simple NLP for food recommendations
        const keywords = this.extractKeywords(input);
        return this.recommendMenu(keywords, menuItems);
    }
}
```

#### Conversational AI Interface
```html
<!-- Chat Interface -->
<div id="ai-assistant" class="chat-widget">
    <div class="chat-header">
        <h4>AI Assistant</h4>
        <span class="status online">🟢 Online</span>
    </div>
    <div class="chat-messages" id="chat-messages">
        <div class="message bot">
            Hi! I'm here to help you choose from our menu. What are you in the mood for?
        </div>
    </div>
    <div class="chat-input">
        <input type="text" id="user-input" placeholder="Ask me about our menu...">
        <button onclick="sendMessage()">Send</button>
    </div>
</div>
```

### 5. Admin Super Panel

#### Dashboard Features
- Tenant overview and statistics
- Revenue analytics
- Payment verification queue
- System health monitoring
- User management

#### Payment Verification Workflow
```php
// app/Http/Controllers/Admin/PaymentController.php
class PaymentController extends Controller {
    public function verifyPayment($paymentId) {
        $payment = Payment::findOrFail($paymentId);
        
        if ($request->action === 'approve') {
            $payment->status = 'verified';
            $payment->verified_at = now();
            $payment->verified_by = auth()->id();
            
            // Update subscription
            $payment->subscription->status = 'active';
            $payment->subscription->end_date = now()->addMonth();
            
            // Send confirmation email
            Mail::to($payment->tenant->email)->send(new PaymentVerified($payment));
        }
        
        return redirect()->back()->with('success', 'Payment verified successfully');
    }
}
```

## Security Implementation

### Multi-Tenant Security
- Database isolation per tenant
- Input validation and sanitization
- CSRF protection
- SQL injection prevention
- File upload security

### Payment Security
- Secure file storage for payment proofs
- Admin-only payment verification
- Audit trail for all payment actions
- Encrypted sensitive data

## Deployment Architecture

### Recommended Setup
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │────│   Web Server    │────│   MySQL Cluster │
│   (Nginx)       │    │   (PHP-FPM)     │    │   (Primary)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │                       │
                              │              ┌─────────────────┐
                              └──────────────│   MySQL Backup  │
                                             │   (Replica)     │
                                             └─────────────────┘
```

### File Structure
```
/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   └── Middleware/
│   ├── Models/
│   ├── Services/
│   └── Providers/
├── database/
│   ├── migrations/
│   └── seeds/
├── public/
│   ├── assets/
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   └── uploads/
├── resources/
│   ├── views/
│   │   ├── tenant/
│   │   ├── admin/
│   │   └── auth/
│   └── lang/
├── routes/
│   ├── web.php
│   ├── api.php
│   └── admin.php
└── config/
```

## Development Phases

### Phase 1: Multi-Tenant Foundation (Week 1-2)
- Tenant registration system
- Database schema management
- Basic authentication
- Subdomain routing

### Phase 2: Subscription & Payment (Week 3-4)
- Subscription plans implementation
- QRIS payment integration
- Payment proof upload system
- Admin verification workflow

### Phase 3: Restaurant Management (Week 5-6)
- Menu management system
- Order processing
- Cashier interface
- Kitchen display system

### Phase 4: Customer Interface & AI (Week 7-8)
- Customer ordering UI
- AI recommendation engine
- Chat interface
- Real-time updates

### Phase 5: Admin Panel & Analytics (Week 9-10)
- Admin dashboard
- Reporting tools
- System monitoring
- Performance optimization

## Technology Specifications

### PHP Requirements
- Version 8.1 or higher
- Extensions: PDO, MySQLi, GD, cURL, JSON, MBString
- Memory limit: 256MB minimum
- Composer for dependency management

### Database Requirements
- MySQL 8.0+ or MariaDB 10.5+
- InnoDB storage engine
- UTF8MB4 character set
- Optimized indexing for tenant queries

### Frontend Requirements
- Bootstrap 5.3+
- jQuery 3.6+
- Chart.js for analytics
- Pusher/WebSockets for real-time updates

## API Endpoints

### Public APIs
```
GET  /api/{tenant}/menu          # Get restaurant menu
POST /api/{tenant}/orders        # Create order
GET  /api/{tenant}/orders/{id}   # Track order status
POST /api/ai/recommend           # AI recommendations
```

### Admin APIs
```
GET  /api/admin/tenants          # List all tenants
POST /api/admin/payments/verify  # Verify payment
GET  /api/admin/analytics        # Platform analytics
```

## Performance Considerations

### Database Optimization
- Indexing strategy for tenant queries
- Connection pooling for multi-tenant access
- Query optimization for large datasets

### Caching Strategy
- Redis for session storage
- Menu caching per tenant
- API response caching

### Scalability
- Horizontal scaling for web servers
- Database read replicas
- CDN for static assets

## Testing Strategy

### Unit Tests
- Model relationships and validation
- Business logic testing
- AI recommendation accuracy

### Integration Tests
- Multi-tenant functionality
- Payment processing
- Real-time features

### Load Testing
- Concurrent tenant operations
- Order processing under load
- Database performance

## Monitoring & Analytics

### Application Monitoring
- Error tracking and logging
- Performance metrics
- User behavior analytics

### Business Intelligence
- Revenue tracking per tenant
- Popular menu items
- Peak hour analysis
- Customer satisfaction metrics

This technical specification provides a comprehensive foundation for building your multi-tenant e-ordering cafe system with AI recommendations and subscription management.
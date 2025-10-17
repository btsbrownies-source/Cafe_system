# App Flow Document for Cafe System

## Onboarding and Sign-In/Sign-Up

When a new user arrives at the Cafe System, they see a simple landing page with a clear call to action to create an account or sign in. Users begin by entering their email address and choosing a password, or they can opt to sign in with a linked Google account if they prefer. After submitting their details, the system sends a confirmation email to verify the account. Once verified, the user is redirected to the sign-in page where they use their credentials to log in. If a user forgets their password, they can click the “Forgot Password” link, enter their email, and receive a password reset link. Following the link allows them to set a new password and return to the sign-in page. At any time while they are signed in, users can sign out via a logout button in the header.

## Main Dashboard or Home Page

After signing in, users land on the main dashboard, which features a sidebar on the left, a top navigation header, and a content area in the center. The sidebar lists the primary sections labeled Orders, Inventory, Customers, Point of Sale, Reports, and Settings. The header displays the user’s name, a notification icon for alerts such as low stock warnings or new orders, and a logout button. The content area shows a high-level overview widget summarizing today’s order count, current inventory alerts, top-selling items, and a quick link to start a new order. From this dashboard, clicking any sidebar item takes the user directly to the corresponding module.

## Detailed Feature Flows and Page Transitions

### Order Management Flow

In the Orders section, users see a table of active and past orders with quick filters for status such as pending, in preparation, and completed. To create a new order, the user clicks the New Order button, which brings up a menu of cafe items. As items are added, the user can choose sizes or modifiers like extra syrup or substitute milk. When selection is complete, pressing the Assign Staff button allows the user to select a team member to handle the order. The order then appears under In Preparation and updates automatically when the assigned staff marks it as Ready or Completed. Users can click any order row to view or edit details and add notes about special requests. A Back button returns users to the order list.

### Inventory Control Flow

In the Inventory section, users see a list of ingredients and supplies with current quantities and threshold markers. If stock drops below a set threshold, the system highlights the item in red. Clicking any ingredient row opens a detail page where the user can manually adjust quantity, view usage history, and create a purchase order. The Create Purchase Order button leads to a form where users enter supplier information and expected delivery date before submitting. Once supplies arrive, the user returns to that purchase order, marks items as received, and the inventory updates automatically. A Back to Inventory link navigates back to the main list.

### Customer Profile Management Flow

The Customers section presents a searchable list of customer profiles. Users type a name or email to filter results. Selecting a profile reveals order history, loyalty points, and saved preferences such as favorite drinks. From the profile page, staff can edit contact information, add notes about preferences, and assign promotional offers. A Save Changes button commits updates and returns the staff member to the filtered customer list.

### Point-of-Sale Transaction Flow

The Point of Sale section provides a compact interface optimized for quick checkouts. Staff start by selecting items and modifiers, then choose a payment method such as cash, credit card, or a digital wallet integration. After tapping Process Payment, the system contacts the payment gateway. If the transaction succeeds, a receipt prints automatically and a digital copy is offered via email. Users then click Done to return to the POS screen to serve the next customer.

### Reporting and Analytics Flow

In the Reports section, users begin by choosing a date range in a calendar picker at the top. The page then displays charts summarizing total sales, sales by item, peak hours, and staff performance metrics. Users can switch between chart types or export data into CSV or PDF formats by clicking an Export button. After exporting, a notification confirms success, and users remain on the report page unless they click Back to Dashboard.

### User Authentication and Authorization Flow

Administrators have access to a User Management page under Settings where they can view all staff accounts. From there, admins can create new accounts by entering name, email, and role, or modify roles and deactivate accounts. Changes apply immediately, and affected users see their new permissions the next time they sign in.

## Settings and Account Management

The Settings section contains a submenu for Profile, Notifications, System Configuration, and User Management. In the Profile page, users update their name, email, and password. The Notifications page allows users to toggle email alerts for low inventory, new orders, or system updates. System Configuration is reserved for administrators who set tax rates, menu categories, operational hours, and feature toggles for upcoming functionality. Every Settings page includes a Save button to apply changes and a Cancel link that returns the user to the main dashboard.

## Error States and Alternate Paths

If users enter incorrect login credentials, an error message appears above the form prompting them to retry or reset their password. When creating an order and a selected item is out of stock, a warning banner notifies the user and prevents adding that item. In the POS flow, if payment processing fails, the system displays an error with the gateway’s message and offers the option to retry or choose a different payment method. Network interruptions trigger a banner at the top of the screen indicating connectivity issues; the app automatically tries to reconnect and informs the user once service is restored.

## Conclusion and Overall App Journey

From the moment a user signs up or logs in, they arrive at a unified dashboard that guides them seamlessly into order creation, inventory tracking, customer management, checkout, and reporting. Administrators benefit from extra controls in settings for user roles and system parameters. At every step, clear navigation between sections and helpful error messages keep users moving forward. Day in and day out, staff rely on the Cafe System to process orders accurately, maintain stock levels, serve customers quickly, and analyze performance to improve operations over time.
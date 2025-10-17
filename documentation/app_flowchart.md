flowchart TD
    Start[Start] --> Login[User Login]
    Login --> AuthCheck{Valid User}
    AuthCheck -- Yes --> Dashboard[Dashboard]
    AuthCheck -- No --> Login[User Login]
    Dashboard --> ManageOrders[Order Management]
    Dashboard --> Inventory[Inventory Control]
    Dashboard --> Customers[Customer Profile Management]
    Dashboard --> POS[Point of Sale Transactions]
    Dashboard --> Reports[Reporting and Analytics]
    Dashboard --> Settings[Configuration and Settings]
    ManageOrders --> NewOrder[Create New Order]
    NewOrder --> TrackOrder[Track or Update Order]
    TrackOrder --> Dashboard
    Inventory --> ViewStock[View Current Stock]
    ViewStock --> Reorder[Generate Purchase Order]
    Reorder --> Dashboard
    Customers --> ViewProfile[View Customer Profiles]
    ViewProfile --> ManageLoyalty[Manage Loyalty Program]
    ManageLoyalty --> Dashboard
    POS --> ProcessPayment[Process Payment]
    ProcessPayment --> GenerateReceipt[Generate Receipt]
    GenerateReceipt --> Dashboard
    Reports --> ViewReports[View Sales and Analytics]
    ViewReports --> Dashboard
    Settings --> UpdateSettings[Update System Settings]
    UpdateSettings --> Dashboard
    Dashboard --> Logout[Logout]
    Logout --> End[End]
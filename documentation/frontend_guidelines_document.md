# Frontend Guideline Document for Cafe_system

This document lays out clear, step-by-step guidelines for building and maintaining the frontend of the Cafe_system application. It covers architecture, design rules, styling, components, data flow, navigation, performance, and testing. With these guidelines, any developer—technical or not—can understand and contribute to the project without confusion.

## 1. Frontend Architecture

### 1.1 Overview
- **Framework**: We use **React** (with **TypeScript**), a popular library for building user interfaces. It breaks the UI into reusable pieces called components.
- **Build Tools**: **Webpack** (for bundling code) and **Babel** (for translating modern JavaScript/TypeScript into browser-compatible code).
- **Code Quality**: **CodeGuide** controls linting (ESLint) and formatting (Prettier) automatically, so our code stays consistent and easy to read.

### 1.2 Scalability, Maintainability, Performance
- **Scalability**: Components live in folders by feature (e.g., `orders/`, `inventory/`), so we can add new pages without breaking existing code.
- **Maintainability**: TypeScript types and clear folder structure help catch mistakes early and make it easy to onboard new developers.
- **Performance**: We enable code splitting and lazy loading at route-level. Unused code doesn’t get shipped to the browser, keeping load times fast.

## 2. Design Principles

### 2.1 Key Principles
1. **Usability**: Interfaces should be simple, with clear labels and straightforward actions.
2. **Accessibility**: All interactive elements must work with keyboard navigation and screen readers (use ARIA roles, `alt` text, proper color contrast).
3. **Responsiveness**: Layouts adapt to different devices (desktop, tablet, mobile) using flexible grids and media queries.
4. **Consistency**: Reuse the same components and styles to give users a predictable experience.

### 2.2 Applying the Principles
- Buttons and links always look and behave the same across the app.
- Forms provide inline validation messages and focus states for keyboard users.
- Breakpoints at 320px, 768px, and 1024px ensure the UI scales gracefully.

## 3. Styling and Theming

### 3.1 Styling Approach
- **SASS + CSS Modules**: We write styles in `.module.scss` files. This keeps CSS scoped to each component and avoids naming conflicts.
- **BEM Naming**: Classes follow Block__Element--Modifier (e.g., `card__header--highlighted`) for clarity.

### 3.2 Theming and Look-and-Feel
- **Design Style**: Modern flat design—clean, minimal use of shadows, sharp edges.
- **Color Palette**:
  - Primary: #5A2D82 (deep purple)
  - Secondary: #FFB800 (warm yellow)
  - Neutral Dark: #333333
  - Neutral Light: #F5F5F5
  - Success: #28A745 (green)
  - Error: #DC3545 (red)

### 3.3 Typography
- **Font Family**: Roboto (weights: 400, 500, 700)
- **Headings**: Use `700` weight; body text uses `400`.
- **Line Height**: 1.5 for readability.

## 4. Component Structure

### 4.1 Organization
- **Atomic Design**:
  - **Atoms**: Buttons, inputs, labels (e.g., `components/atoms/Button/`)
  - **Molecules**: Small combinations, like `SearchBar` or `OrderSummary`
  - **Organisms**: Full sections, such as `OrderList` or `InventoryOverview`
  - **Templates & Pages**: Layouts that combine organisms into screens like `OrdersPage`.

### 4.2 Reusability and Maintenance
- Every component lives in its own folder with `index.tsx`, `styles.module.scss`, and test files.
- Props and types are clearly defined to avoid unexpected behavior.
- Components do one thing and do it well—this makes testing and updates easier.

## 5. State Management

### 5.1 Approach
- **Redux Toolkit**: Central store for global data (e.g., current user, inventory levels, active orders).
- **RTK Query**: For fetching and caching data from APIs.
- **Local State**: Component-level state (e.g., form inputs) uses React’s `useState` or `useReducer` when needed.

### 5.2 Sharing State
- Use `useSelector` to read from Redux and `useDispatch` to update.
- Keep API logic in RTK Query slices; UI components call hooks like `useGetOrdersQuery()`.
- Avoid deeply nested props by leveraging the store for widely used data.

## 6. Routing and Navigation

### 6.1 Library
- **React Router v6** handles client-side routing.

### 6.2 Structure
- **Nested Routes**: e.g., `/orders`, `/orders/:orderId`, `/inventory`, `/customers`, `/settings`.
- **Navigation Component**: A sidebar or top bar with links (using `<NavLink>` for active styling).
- **Protected Routes**: Wrap manager/admin pages with a `RequireAuth` component that checks user role.

## 7. Performance Optimization

- **Lazy Loading**: Use `React.lazy` and `Suspense` for major pages.
- **Code Splitting**: Webpack splits bundles per route.
- **Asset Optimization**: Compress images (WebP or optimized PNG), inline small SVG icons, and minify CSS/JS.
- **Caching**: Leverage browser caching headers for static assets.
- **Bundle Analysis**: Run tools like `webpack-bundle-analyzer` to keep an eye on large dependencies.

## 8. Testing and Quality Assurance

### 8.1 Unit and Integration Tests
- **Jest** for running tests.
- **React Testing Library** for rendering components and simulating user events (clicks, typing).
- **Coverage**: Aim for at least 80% coverage on critical modules (orders, inventory).

### 8.2 End-to-End Tests
- **Cypress**: Automate flows like placing an order, updating inventory, logging in as admin.
- Maintain a small, fast suite that runs on every pull request.

### 8.3 Linting and Formatting
- **ESLint** rules enforced by CodeGuide catch common mistakes.
- **Prettier** handles code formatting automatically.
- **CI Pipeline**: Fails the build if lint or tests do not pass.

## 9. Conclusion and Overall Frontend Summary

This guide covers everything from project structure to testing. By following these rules, the Cafe_system frontend will be:
- **Consistent**: Uniform look and feel via theming and components.
- **Maintainable**: Clear folder structure, typed code, and reusable pieces.
- **Performant**: Optimized loading, caching, and code splitting.
- **Reliable**: Solid test coverage and automated quality checks.

Together, these guidelines ensure a smooth developer experience and a fast, accessible interface for cafe staff and customers alike.
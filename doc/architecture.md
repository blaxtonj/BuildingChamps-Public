## Architecture

This document outlines the architecture of **BuildingChamps**, focusing on how the system coordinates scheduling, payments, and user interactions within a unified platform.

Unlike a traditional marketing site, this application operates as a multi-system solution, integrating RESTful APIs, third-party services (Stripe and Google Calendar), and a persistent data layer to support real-world user flows.

The architecture is designed to ensure data consistency, secure transactions, and reliable communication between client-side interactions and server-side logic, while remaining scalable and maintainable.

### System Overview
The application is built as a full-stack Next.js platform that combines
client-side interactions with server-side API routes to handle
authentication, scheduling, and payments.

Core systems include:

- RESTful API routes for server-side logic
- MongoDB for persistent data storage
- Stripe for payment processing
- Google Calendar API for scheduling and availability
- Auth0 for authentication and protected user flows

These systems work together to support a complete user journey from browsing to booking and payment.

### API Structure & Responsibilities

The API layer is organized around feature-specific routes, each responsible for a distinct part of the system. This structure follows a domain-driven approach, where related logic is grouped by functionality rather than technical type.

Key API domains include:

- **Authentication (`/api/auth`)**
  - Handles user authentication and protected route access.

- **Payments (`/api/payment_intent`, `/api/paymentEmail`)**
  - Manages Stripe payment intents and post-payment communication.
  - Ensures transactions are validated before completing bookings.

- **Orders & Products (`/api/order`, `/api/products`, `/api/customer`)**
  - Handles purchasable sessions and customer-related data.
  - Supports the e-commerce portion of the platform.

- **Scheduling (`/api/scheduleSession`, `/api/session`, `/api/addAttendees`, `/api/popUp`)**
  - Manages booking logic, session creation, and user intake for scheduling.
  - Collects required user information (name, email, phone, etc.) before initiating bookings.
  - Integrates with Google Calendar for availability and event creation.

- **Forms & Communication (`/api/submit`)**
  - Handles general contact form submissions and non-transactional user inquiries.

Each route is responsible for validating input, executing business logic, interacting with external services or the database, and returning a structured response to the client.

### Google Calendar Integration

### Payment Processing

### Data Persistence

### Client–Server Interaction

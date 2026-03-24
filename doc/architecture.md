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

Scheduling functionality is powered by the Google Calendar API, enabling the application to manage real-time availability and automate session bookings.

The system uses Google APIs to:

- check availability for selected time slots  
- prevent scheduling conflicts  
- create calendar events upon successful booking  

This ensures that sessions are only confirmed when a valid time slot is available, keeping the scheduling system consistent with the business’s actual calendar.

To support this functionality, the application requires **Google Workspace domain-wide authorization**, allowing it to securely create and manage calendar events on behalf of the business.

This integration introduced additional architectural considerations, including:

- secure authentication and credential management  
- synchronization between client-side state and external calendar data  
- handling edge cases such as overlapping bookings or failed event creation  

By integrating directly with Google Calendar, the platform ensures that scheduling remains accurate, automated, and tightly coupled with real-world availability.

### Payment Processing

Payment functionality is handled through Stripe, enabling secure and reliable transactions for purchasing training sessions.

The system uses Stripe to:

- create and manage payment intents  
- securely process transactions  
- confirm successful payments before completing bookings  

Rather than treating payments as a separate feature, the application integrates payment processing directly into the booking lifecycle. A session is only scheduled after a payment has been successfully confirmed.

This ensures:

- users cannot reserve time slots without payment  
- bookings are tied to verified transactions  
- financial data remains secure and handled externally by Stripe  

From an architectural perspective, payment handling is implemented through dedicated API routes that communicate with Stripe’s servers. Sensitive operations, such as creating payment intents and confirming transactions, are performed server-side to maintain security.

This integration required careful coordination between:

- client-side user interactions  
- server-side validation and processing  
- external payment confirmation  

By structuring payments as a required step in the booking flow, the system maintains consistency between financial transactions and scheduled sessions.

### Data Persistence

All critical application data—including user profiles, bookings, payment confirmations, and session schedules—is stored in **MongoDB** via **Mongoose**.

The system maintains persistence by:

- **Customers**: Profiles, contact info, and booking history  
- **Sessions**: Available time slots, booked slots, capacity  
- **Transactions**: Payment intents, confirmations, receipts  
- **Pop-ups / Forms**: Submission data for inquiries or class registrations  

Mongoose schemas enforce data integrity and validation, ensuring that:

- only valid bookings and transactions are stored  
- required fields (like session time, customer ID, payment status) are always present  
- relationships between customers, sessions, and payments remain consistent  

The persistence layer interacts closely with API routes:

- **API** endpoints write to and read from MongoDB  
- Transactions are saved only after external verification (Stripe for payments, Google Calendar for scheduling)  
- This guarantees consistency between stored data and external systems  

By centralizing data in MongoDB, the platform can reliably track bookings, manage session availability, and audit transactions while keeping the frontend decoupled from business logic.

### Client–Server Interaction

BuildingChamps follows a **modern client-server architecture**, with the frontend built in **Next.js / React** and the backend providing **RESTful API endpoints** for all transactional and data operations.

Key interaction flows include:

- **User Authentication**
  - Users authenticate via **Auth0**, with the frontend sending login requests to the Auth0 endpoints.
  - Authenticated sessions provide a JWT token used for subsequent API requests.

- **Browsing Classes**
  - The frontend fetches available sessions and class details from the **/api/session** and **/api/products** endpoints.
  - Data is cached on the client using **SWR** for efficient revalidation.

- **Booking & Scheduling**
  - When a user schedules a class:
    1. The frontend sends booking details to **/api/scheduleSession**.
    2. The backend validates data, saves it in **MongoDB**, and creates a Google Calendar event via the **Google Calendar API**.
    3. Confirmation is returned to the client, updating UI state.

- **Payment Flow**
  - Users initiate payment through Stripe Checkout via **/api/payment_intent**.
  - Once Stripe confirms payment, the backend stores the transaction in MongoDB and triggers a confirmation email via **/api/paymentEmail**.

- **Form Submissions**
  - General inquiries or contact forms are submitted through **/api/submit**.
  - These are validated and persisted in client email for later follow-up.

- **Error Handling & Feedback**
  - All API responses include structured success/error messages.
  - Frontend provides real-time feedback for errors (e.g., failed payment, scheduling conflict).

By clearly separating **client-side UI**, **API endpoints**, and **external services** (Stripe, Google Calendar), the application ensures data consistency, reliable booking, and a smooth user experience.

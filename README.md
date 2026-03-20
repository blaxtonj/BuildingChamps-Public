# BuildingChamps-Techincal Case Study

## Overview 
This repository documents the development of a professional web platform built for **BuildingChamps**, a business in the training and fitness industry. The primary goal of the application was to support class **scheduling** and online **session purchases** within a single, cohesive system.

The platform functions as a hybrid between a marketing site, scheduling system, and lightweight e-commerce solution. It allows users to:

- browse available classes
- purchase training sessions
- schedule and manage bookings

Unlike previous projects focused primarily on UI and performance, this application introduces more complex state management and transactional flows, where data consistency and user interactions are critical.

The core project resides in a private repository due to proprietary business logic and payment-related integrations. This public version focuses on architectural decisions, user flow design, and the engineering challenges involved in building a reliable scheduling and purchasing system

## Project Status

This project is currently functionally complete, but not deployed to production due to external client dependencies.

Final deployment requires confirmation from the client to proceed with Google Workspace domain-wide authorization, which is needed to support scheduling and calendar integrations.

While development is complete from an engineering standpoint—including core scheduling logic, purchasing flows, and state management—the project remains on hold pending client approval and communication.




## What This Repo Demonstrates

- Full-stack application architecture
- Authentication and protected user flows
- Payment processing and transactional systems (Stripe)
- Scheduling logic and calendar integration (Google APIs)
- Data consistency and state management across user interactions
- Integration of multiple third-party services
- Real-world client constraints and deployment dependencies

## Documentation
- Architecture
- Engineering Challenges
- Performance Strategy
- Key Engineering Decisions
- User Flows 

## Tech Stack

The platform integrates multiple systems—including authentication, payments, scheduling, and database persistence—requiring a full-stack architecture that balances user experience with data consistency and security.

| Category | Tools & Technologies |
| :--- | :--- |
| **Framework** | Next.js, React, TypeScript |
| **Authentication** | Auth0 |
| **Styling & UI** | Tailwind CSS, Headless UI, Heroicons |
| **Animation** | Framer Motion |
| **Forms & Validation** | React Hook Form, Yup |
| **Data Fetching** | SWR, Axios |
| **Database** | MongoDB, Mongoose |
| **Payments** | Stripe, use-shopping-cart |
| **Scheduling & External Integrations** | Google APIs (Calendar), gapi-script |
| **Backend Utilities** | Node.js, Nodemailer |
| **Security & Utilities** | crypto-js, nextjs-cors |
| **Testing** | Jest |
| **Tooling** | ESLint, Prettier, PostCSS |
| **Deployment & Ops** | GitHub, Vercel |

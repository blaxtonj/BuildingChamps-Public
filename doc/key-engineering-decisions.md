## Key Engineering Decisions


### 1. Hash-Based Session Entitlement System

#### Decision
Instead of tracking session usage with a simple counter, the system generates unique hash identifiers for each purchased session.

#### Context
Users purchase session packages (e.g., 4 or 8 sessions), and each session must only be used once. A traditional counter-based approach introduces risks around race conditions and inconsistent state.

#### Alternatives Considered

- decrementing a session count per booking
- storing usage flags on a single record
- tying sessions directly to Stripe transactions

#### Why This Approach
By generating unique hashes:

- each session becomes a consumable token
- usage is immutable and traceable
- scheduling is decoupled from payment logic

#### Trade-offs

- increased complexity in session management
- requires additional logic for validation and lookup

#### Impact

- prevents overbooking at a system level
- ensures strong consistency between purchases and bookings
- enables flexible scheduling logic without modifying payment data

---

### 2. Using Google Calendar as Source of Truth

#### Decision
Instead of building a custom scheduling backend or using a third-party booking platform, the system uses **Google Calendar as the single source of truth** for all class availability and scheduling.

#### Context
The client was already using Google services for day-to-day operations, making Google Calendar a natural fit.

By leveraging this existing workflow:

- the client can create and manage classes directly in Google Calendar
- all changes automatically sync with the platform
- no additional tools or training are required

This allowed the platform to integrate seamlessly into the client’s existing ecosystem while reducing operational complexity.

#### Alternatives Considered
Several third-party scheduling platforms were evaluated, including:

- [Cal.com](https://cal.com/)
- [Setmore](https://www.setmore.com/)

While both provide out-of-the-box scheduling features, they introduced trade-offs:

- limited control over data and customization
- difficulty integrating deeply with custom payment + session logic
- additional overhead for the client to learn and manage another tool

Ultimately, these options did not align with the goal of maintaining a fully integrated and customizable system.

#### Why This Approach
Using Google Calendar as the source of truth provided several advantages:

- **Real-time synchronization**:
  All scheduling data reflects instantly without manual updates
- **Client-friendly workflow**:
  The client manages sessions in a familiar interface
- **Reduced backend complexity**:
  No need to build and maintain a dedicated scheduling system
- **Full control over frontend experience**:
  A custom calendar UI enables tailored UX and deeper integration with booking logic 

#### Trade-offs

This decision introduced a few technical challenges:

- Requires **domain-wide delegation** via Google Workspace for write operations
- Increased reliance on external APIs (Google Calendar API)
- Additional complexity in handling:
    - recurring event expansion
    - data normalization
    - API rate limits and performance consideration

#### Impact

- Established a single, reliable source of truth for all scheduling data
- Significantly reduced client-side complexity and training requirements
- Enabled real-time availability without manual synchronization
- Provided a scalable foundation that integrates cleanly with:
    - booking logic
    - payment processing
    - session entitlement tracking

### 3. Client-Orchestrated Checkout Flow

#### Decision

#### Context

#### Alternatives Considered

#### Why This Approach

#### Trade-offs

#### Impact


---

### 4. Parallel Session Provisioning with Promise.all

#### Decision

#### Context

#### Alternatives Considered

#### Why This Approach

#### Trade-offs

#### Impact


---

### 5. Custom Calendar UI vs Third-Party Component

#### Decision

#### Context

#### Alternatives Considered

#### Why This Approach

#### Trade-offs

#### Impact


---

### 6. Separation of API Responsibilities (REST Design)

#### Decision

#### Context

#### Alternatives Considered

#### Why This Approach

#### Trade-offs

#### Impact


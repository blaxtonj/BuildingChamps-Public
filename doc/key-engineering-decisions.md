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

---

### 3. Client-Orchestrated Checkout Flow

#### Decision
The checkout flow was designed to be **client-orchestrated**, where the frontend coordinates multiple API calls (customer creation, order creation, session provisioning, and payment confirmation) rather than relying on a single backend endpoint.

#### Context

The checkout process in BuildingChamps is not a simple payment transaction. Each purchase must:

- create or retrieve a customer
- persist an order
- generate session entitlements (hash-based tokens)
- confirm payment through Stripe

These steps involve both internal APIs and external services, and must remain synchronized to avoid inconsistent states (e.g., payment without sessions).

#### Alternatives Considered

1. Fully Backend-Orchestrated Flow (Single Endpoint)
    - One API route handles the entire checkout lifecycle
    - Pros: centralized logic, easier atomic control
    - Cons: harder to debug, less flexible, more complex backend coupling

2. Webhook-Driven Architecture (Stripe-first approach)
    - Use Stripe webhooks to create orders and sessions after payment confirmation
    - Pros: strong payment consistency guarantees
    - Cons: delayed processing, more complex state reconciliation, harder local testing

#### Why This Approach

A client-orchestrated flow was chosen because it provides:

- fine-grained control over sequencing of each step
- immediate feedback to the user during checkout
- simpler debugging and observability across each operation
- flexibility to evolve individual steps independently

By explicitly coordinating each step on the client, the system ensures that:

- required data exists before moving forward
- failures are caught early
- dependent operations (like session creation) happen in the correct order

#### Trade-offs

- increased complexity in the frontend
- lack of true atomic transactions across systems
- requires careful error handling to avoid partial failures
- more network requests compared to a single backend call

#### Impact

- enabled a flexible and modular checkout pipeline
- simplified integration with multiple APIs (Stripe + internal services)
- improved visibility into each step of the transaction process
- provided a scalable foundation for adding features like retries, logging, or analytics


---

### 4. Parallel Session Provisioning with Promise.all

#### Decision
Session entitlements are generated in parallel using `Promise.all` during checkout, rather than sequentially.

#### Context
When a user completes a purchase, they may buy multiple session packages in a single transaction. Each package requires generating one or more session entitlements (hashed identifiers).

This means a single checkout could trigger multiple independent session creation operations.

A naïve implementation would process these sequentially, increasing latency and degrading the user experience.

#### Alternatives Considered

- Sequential Processing (for-loop / await chain)
    - Simpler to implement
    - Easier to debug
    - Slower as operations block each other
- Backend Batch Processing
    - Move session creation entirely to the server
    - Added complexity and tighter backend coupling
    - Less flexibility in handling partial failures on the client

#### Why This Approach
Using `Promise.all` allows all session creation operations to execute concurrently:

```ts
const sessions = await Promise.all(
  products.map((product) =>
    createSession({
      stripe_id: product.stripe_id,
      customer: { email: user?.email },
      product: `${product.name} ${product.description}`,
    })
  )
);
```
This approach provides:

- improved performance through parallel execution
- reduced checkout latency
- clean, declarative orchestration of async operations

It also enables a centralized validation step:

```ts
const hasError = sessions.some((session) => session.error);
```

#### Trade-offs
- If one operation fails, all results must still be evaluated manually
- No built-in rollback mechanism (not transactional)
- Requires explicit error handling for partial failures

#### Impact
- Faster and more responsive checkout experience
- Scales efficiently as more products/session types are added
- Keeps session provisioning logic simple and composable
- Provides a clear pattern for handling concurrent async workflows



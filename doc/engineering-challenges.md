## Engineering Challenges

BuildingChamps introduced a different set of engineering challenges compared to a traditional frontend-focused application.

Rather than focusing purely on UI and layout, the platform required coordinating multiple systems—including payment processing, scheduling, and data persistence—into a single, reliable user experience.

Each user action, such as booking a session, involves interactions between the client, server, and external services like Stripe and Google Calendar. This introduced challenges around data consistency, system synchronization, and failure handling.

The following sections highlight the key challenges encountered during development and the approaches used to address them.

---

### 1. Coordinating Payments with Scheduling

One of the most complex challenges in BuildingChamps was ensuring that payments and scheduling remained fully synchronized.

Unlike a typical booking system, users do not schedule arbitrary sessions. Instead, they purchase session packages, and each purchase generates a finite number of usable sessions.

To support this, the system introduces a **hash-based session allocation model**

This model acts as the bridge between payment and scheduling, ensuring that every booking is backed by a verified purchase.

- each purchase creates a set of unique session identifiers (hashes)  
- each hash represents a single usable booking  
- once used, a hash cannot be reused  

During checkout:

1. A customer is created or retrieved  
2. An order is generated  
3. A session package is created  
4. A set of unique hashes is generated and stored  

```ts
const sessionHash = crypto.createHash("sha256");
sessionHash.update(stripe_id + Date.now().toString());
```

During scheduling:

1. The system retrieves all purchased sessions
2. Previously used hashes are identified
3. The next available (unused) hash is selected
4. The booking is created and persisted
5. The hash is marked as consumed

```ts
const hash = await checkHash({
  scheduledHash,
  matchSession
});

await axios.post("/api/scheduleSession", {
  eventId,
  eventDate,
  stripe_id,
  hash
});
```

This approach ensures that:

- users cannot schedule more sessions than they have purchased
- each booking is tied directly to a valid payment
- session usage is tracked with full consistency across the system

By treating purchased sessions as a finite resource and enforcing controlled consumption, the platform maintains strong data integrity between payments and scheduled bookings.

---

### 2. Google Calendar Integration

To handle real-time class availability and scheduling, the platform integrates directly with the Google Calendar API as the system of record for all scheduling data.

#### Architecture Overview
- A dedicated Google Calendar was created to manage all class sessions
- The frontend fetches live event data using the Google API (`gapi-script`)
- Recurring events are expanded into individual bookable time slots
- Users interact with a custom-built calendar UI to select sessions

#### Fetching & Normalizing Events

Instead of relying on static data, the system dynamically retrieves events from the Google Calendar API.

A key challenge was handling recurring events, which are not returned as individual bookable sessions by default. To ensure each class could be scheduled independently, recurring events are expanded into discrete instances:

  
```ts
let res = await gapi.client.request({
  path: `https://www.googleapis.com/calendar/v3/calendars/${calendarID}/events/${id}/instances`,
  params: {
    timeMin: startOfDay.toISOString(),
    singleEvents: true,
  },
});
```

This normalization step allows the system to:

- treat each class session as a unique booking slot
- filter events based on real-time availability
- eliminate the need for manual schedule management

As a result:

- scheduling remains fully dynamic
- availability is always up-to-date
- all users see consistent booking data

#### Custom Calendar UI

A fully custom calendar component was built using `date-fns` to:
- render monthly views
- handle date selection
- display available sessions per day
- highlight days with available classes

Users can:
- navigate months
- view available time slots
- select specific sessions for booking

This approach provided:
- full control over UX
- tighter integration with scheduling logic
- better performance compared to third-party calendar embeds

#### Event Selection Flow

1. User selects a date
2. Available sessions for that day are displayed
3. User selects a time slot
4. Selected event is passed into the scheduling system

This event data becomes the foundation for:
- booking validation
- payment coordination
- attendee registration

#### Integration Challenges

Working with Google Calendar introduced several challenges:
- Recurring events required additional API calls to expand instances
- Client-side API usage required careful handling of API keys
- Data normalization was necessary to align Google event structure with internal models

These were solved by:
- building a normalization layer for event data
- isolating API logic into reusable utilities
- limiting fetched data to only relevant time ranges

#### Result
- Real-time scheduling powered by an external system
- No need for a custom-built scheduling backend
- Scalable and easy for the client to manage
- Seamless integration with booking and payment flows

--- 

### 3. Checkout & Transaction Coordination

The checkout process required coordinating multiple asynchronous systems, including internal APIs, and session provisioning logic.

Unlike a simple payment flow, this system needed to ensure that:

- customer data is created or retrieved
- orders are persisted before payment confirmation
- session entitlements are generated correctly
- all systems remain consistent even in the event of failure

This introduced challenges around sequencing, reliability, and maintaining data integrity across distributed operations.

#### Checkout Flow

The checkout process follows a multi-step pipeline:

1. Validate form input using React Hook Form and Yup  
2. Initialize Stripe payment elements on the client  
3. Create a payment intent via `/api/payment_intent`  
4. Create or retrieve a customer (`/api/customer`)  
5. Create an order record (`/api/order`)  
6. Generate session entitlements for each purchased product  
7. Confirm payment using Stripe


#### Session Provisioning

A critical part of the checkout flow involves provisioning session entitlements for each purchased product.

Because users can purchase multiple session packages in a single transaction, the system must generate these sessions concurrently while ensuring all operations complete successfully before advancing the transaction.

```ts
// Parallel session provisioning for purchased products
const sessions = await Promise.all(
  products.map((product) =>
    createSession({
      stripe_id: product.stripe_id,
      customer: { email: user?.email },
      product: `${product.name} ${product.description}`,
    })
  )
);

// Validate all session creations succeeded
if (sessions.some((session) => session.error)) {
  console.error("Session provisioning failed", sessions);
} else {
  console.log("Sessions created successfully", sessions);
}
```
This approach ensures:

- efficient parallel processing using Promise.all
- consistent session creation across purchased items
- centralized error handling for partial failures


#### Core Challenge: Transaction Consistency

A key challenge was ensuring consistency across multiple independent operations:

- payment processing (Stripe)
- database writes (customer, order, session)
- client-side confirmation

Because these operations are not inherently atomic, failures at any step could result in:

- payments without session access
- sessions created without successful payment
- partial or inconsistent data across systems
  

#### Implementation Strategy

To address this, the checkout flow was structured as a controlled sequence:

- Customer creation and order persistence occur before payment confirmation  
- Session entitlements are generated immediately after order creation  
- Each step validates responses before proceeding  
- Errors are captured and surfaced to prevent silent failures  

Session generation uses a hash-based system:
- each purchased package generates unique session identifiers  
- these identifiers are later consumed during scheduling

#### Session Entitlement System

Instead of relying on simple counters, the system generates unique hashed session identifiers for each purchase.

This enables:
- secure tracking of session usage  
- prevention of duplicate bookings  
- decoupling of payment data from scheduling logic  

Each session effectively becomes a consumable token tied to a purchase.

#### Tradeoffs & Future Improvements

Because session provisioning occurs before final payment confirmation, there is a potential risk of orphaned session records if a payment fails.

In a production-grade system, this could be improved by:

- moving session creation to a Stripe webhook triggered after successful payment
- ensuring session provisioning is tied directly to confirmed transactions
- implementing cleanup or rollback mechanisms for failed operations

#### Result

- Fully functional checkout pipeline integrating payments and scheduling  
- Consistent data flow across customer, order, and session systems  
- Flexible session model supporting complex scheduling logic  
- Scalable foundation for future e-commerce expansion  















































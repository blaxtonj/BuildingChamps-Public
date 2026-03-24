## Engineering Challenges

BuildingChamps introduced a different set of engineering challenges compared to a traditional frontend-focused application.

Rather than focusing purely on UI and layout, the platform required coordinating multiple systems—including payment processing, scheduling, and data persistence—into a single, reliable user experience.

Each user action, such as booking a session, involves interactions between the client, server, and external services like Stripe and Google Calendar. This introduced challenges around data consistency, system synchronization, and failure handling.

The following sections highlight the key challenges encountered during development and the approaches used to address them.

---

### 1. Coordinating Payments with Scheduling

One of the most complex challenges in BuildingChamps was ensuring that payments and scheduling remained fully synchronized.

Unlike a typical booking system, users do not schedule arbitrary sessions. Instead, they purchase session packages, and each purchase generates a finite number of usable sessions.

To support this, the system introduces a **hash-based session allocation model**:

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

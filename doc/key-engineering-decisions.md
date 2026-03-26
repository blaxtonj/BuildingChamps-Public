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

## User Flow

This document outlines the end-to-end user journey within BuildingChamps, from purchasing a session package to scheduling and attending a class.

### High-Level Flow Diagram

```
User → Checkout → Payment (Stripe)
     → Order Created → Session Hashes Generated
     → Calendar View → Select Event
     → Validate Hash → Schedule Session
     → Google Calendar Updated
```

---

### 1. Purchase Flow (Checkout)

#### Steps
1. User selects a session package
2. Adds product to cart
3. Proceeds to checkout
4. Enters payment details
5. Submits payment

#### System Actions
- Create payment intent (`/api/payment_intent`)
- Create or retrieve customer (`/api/customer`)
- Create order (`/api/order`)
- Generate session entitlements (`/api/session`)
- Confirm payment via Stripe

#### Outcome
- Payment is processed
- User receives session entitlements (hashes)

---

### 2. View Available Sessions

#### Steps
1. User navigates to scheduling page
2. System loads available sessions

#### System Actions
- Fetch purchased sessions (`/api/session/:email`)
- Fetch scheduled sessions (`/api/scheduleSession/:email`)
- Filter out used hashes
- Display remaining available sessions

#### Outcome
- User sees how many sessions remain per package

---

### 3. Browse Calendar & Select Session

#### Steps
1. User opens calendar
2. Navigates between months
3. Selects a date
4. Selects a time slot

#### System Actions
- Fetch events from Google Calendar API
- Expand recurring events into instances
- Filter events by selected date

#### Outcome
- User selects a valid session slot

---

### 4. Schedule a Session

#### Steps
1. User confirms selected session
2. Submits scheduling form

#### System Actions
- Validate available session hash
- Assign next unused hash
- Create booking (`/api/scheduleSession`)
- Add attendee to Google Calendar (`/api/addAttendees`)
- Store scheduling metadata (`/api/popUp`)

#### Outcome
- Session is successfully booked
- Hash is marked as used
- User is added to calendar event

---

### 5. Post-Scheduling State

#### System Behavior
- Scheduled session appears in user records
- Available session count decreases
- Calendar reflects updated attendance

---

### Key System Guarantees

- Users cannot exceed purchased sessions
- Every booking is tied to a valid payment
- Scheduling data stays consistent across:
  - internal database
  - Google Calendar
- Real-time availability prevents double booking

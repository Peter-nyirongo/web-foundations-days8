# TicketHub System Design

## 1. Requirements
### Functional Requirements
- Users can browse upcoming events and concerts.
- Users can view the seat map and availability for a specific event.
- Users can temporarily hold seats (e.g., for 10 minutes) while proceeding to checkout.
- Users can pay for their held seats to finalize the purchase.
- Users can view their purchased tickets in their profile.

### Non-Functional Requirements
- **Speed**: Static pages and event details must load in under 2 seconds. Checkout processing should take less than 3 seconds.
- **Correctness**: Absolute accuracy in ticket sales. No double-bookings; exactly one ticket per seat.
- **Fairness**: During high-demand sales, the system must treat all users equally, preventing bots or users with faster connections from monopolizing tickets.
- **Availability**: The system must remain responsive and not crash during massive traffic spikes.

## 2. Traffic Estimates
### Normal Traffic
- 50,000 visitors/day, viewing 10 pages each = 500,000 page views/day.
- Assuming a 16-hour active window: ~31,250 page views/hour, or ~8.7 requests per second (RPS).
- 5,000 tickets sold/day = ~0.05 tickets per second.
- *Comparison*: Normal traffic is very low and easily handled by a basic monolithic setup.

### "Big Sale" Peak Traffic
- 200,000 users attempt to buy 20,000 seats within 10 minutes (600 seconds).
- Peak user arrival rate: ~333 users/second. If each user makes 5 API requests, that's ~1,600 RPS.
- Ticket purchase rate: 20,000 / 600 = ~33 tickets/second.
- *Comparison*: While the overall RPS is manageable, the database write contention for holding and purchasing the same 20,000 seats will be the primary bottleneck compared to normal days.

## 3. API Design
1. `GET /api/events`: Browse events. Supports pagination and filtering by date/genre.
2. `GET /api/events/{eventId}/seats`: View the seat map and current availability for an event.
3. `POST /api/events/{eventId}/seats/hold`: Attempt to hold specific seats for the authenticated user. Returns a hold token and expiration time.
4. `POST /api/orders/pay`: Process payment for held seats using the hold token. Converts the hold into a confirmed order.
5. `GET /api/users/{userId}/tickets`: Retrieve a list of purchased tickets for the user.

## 4. Data Model
- **Users**: `id`, `name`, `email`, `created_at`
- **Events**: `id`, `name`, `date`, `venue_id`, `total_seats`
- **Seats**: `id`, `event_id`, `row`, `seat_number`, `status` (available, held, sold), `held_by_user_id`, `hold_expires_at`, `order_id`
- **Orders**: `id`, `user_id`, `event_id`, `total_amount`, `status` (pending, paid, cancelled), `created_at`

**Relationships**:
- One `Event` has many `Seats` (1:N).
- One `User` has many `Orders` (1:N).
- One `Order` can have many `Seats` (1:N).
- A `Seat` can be temporarily associated with a `User` via `held_by_user_id`.

## 5. Preventing Double-Booking
To prevent two people from buying the same seat, we rely on **Database Transactions** and **Row-Level Locking**. 

When a user attempts to hold a seat, the system executes a transaction:
```sql
BEGIN TRANSACTION;
UPDATE Seats 
SET status = 'held', held_by_user_id = ?, hold_expires_at = ?
WHERE id = ? AND status = 'available';
-- The application checks if exactly 1 row was updated. 
-- If 0 rows were updated, the seat was already taken.
COMMIT;
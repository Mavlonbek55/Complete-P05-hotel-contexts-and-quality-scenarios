# Design Notes

## P03 notes

The original app searches rooms, accepts a valid booking, rejects overlapping stays, permits a new guest to check in on the previous guest's check-out date, rejects invalid dates, cancels reservations, and includes a race demonstration. A stay includes its check-in date and excludes its check-out date.

# P04 design notes

## 1. Refactoring

Refactoring means improving the internal structure of code without intentionally changing its externally observable behaviour. The P03 demo output and booking rules should stay the same. P04 adds a new requirement, R6, separately: a notification failure must not remove or hide an already-saved booking.

## 2. What the original design makes difficult

- `BookingService` was coupled to concrete storage (`ReservationStore` implemented as a list) and a concrete email sender.
- Adding SMS or another delivery method would require changing booking logic.
- Replacing the list with PostgreSQL would require changing booking logic instead of adding a new storage implementation.
- Tests using only the real email sender cannot reliably simulate a delivery failure.

## 3. SOLID review

- **S — Single Responsibility Principle:** Booking rules, persistence details and notification delivery should have separate responsibilities. `BookingService` coordinates the booking use case; store classes handle persistence; sender classes handle notifications.
- **O — Open/Closed Principle:** The service should be open to extension but closed to modification. New senders (such as SMS) and stores can implement the protocols without editing booking rules.
- **L — Liskov Substitution Principle:** Any implementation that fulfils `ReservationStore` or `NotificationSender` must be usable in place of another implementation while respecting the same contract.
- **I — Interface Segregation Principle:** The service depends on small, purpose-specific protocols instead of a large class containing unrelated operations.
- **D — Dependency Inversion Principle:** High-level booking policy depends on abstractions (`ReservationStore` and `NotificationSender` protocols), not on concrete in-memory storage or email classes.

The tightly coupled P03 design mainly conflicts with O and D, and mixes responsibilities in ways that weaken S and I. Substitutable implementations must still follow the contracts for L.

## 4. Target design

`BookingService` receives a `ReservationStore` and a `NotificationSender` through its constructor. `InMemoryReservationStore` implements the storage protocol. `EmailNotificationSender`, `SmsNotificationSender`, and test fakes implement the notification protocol. `main.py` is the composition root: it chooses the concrete implementations. Domain objects (`Guest`, `Room`, `DateRange`, `Reservation`) live in `domain.py`.

## 5. Proof that behaviour is preserved

Run from the project root:

```bash
python -m unittest -v
python main.py
python main.py --race
```

Verification performed for this project:

- All 8 unit tests pass.
- The standard demo prints the same 9 lines documented in `README.md`.
- The race demo prints `Double booking! Both requests checked before either one saved.` This is intentionally preserved for P04; concurrency protection is outside the refactoring scope.
- A fake sender that raises an exception verifies R6: the reservation remains saved and the result remains `Accepted` with `notification_failed=True`.
- An SMS sender can be supplied to `BookingService` without changing `booking.py`.

## 6. Limitation

The race demo demonstrates a genuine check-then-save race in this teaching implementation. A production system would need an atomic storage operation/transaction or database constraint to prevent concurrent double bookings. P04 is a structure/refactoring exercise, not a concurrency fix.

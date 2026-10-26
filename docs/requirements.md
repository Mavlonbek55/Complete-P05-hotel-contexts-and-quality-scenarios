# Hotel Booking App — Requirements

## Existing requirements (P03)

- **R1 — Search:** The app lists rooms that are free for a requested stay.
- **R2 — Book:** A guest can reserve an available room and receives a reservation ID.
- **R3 — Prevent clashes:** A reservation is refused when the same room overlaps an existing confirmed reservation by at least one night.
- **R4 — Cancel:** A confirmed reservation can be cancelled; a cancelled reservation no longer blocks the room. Repeated cancellation does not cancel it twice.
- **R5 — Valid dates and race demonstration:** Check-out must be after check-in. The race demo illustrates the check-then-save race in the in-memory example; this refactoring does not fix that concurrency defect because the goal is to preserve P03 behaviour.

## New requirement (P04)

- **R6 — Notification failure must not lose a booking:** Save the reservation before attempting to send a confirmation. If the notification sender raises an exception, keep the booking and return an accepted result that indicates the notification failed. The booking must not be deleted or reported as rejected merely because email/SMS failed.

## Design constraint

Refactoring must preserve the observable P03 booking behaviour. Storage and notification implementations must be replaceable through protocols without changing `BookingService`.

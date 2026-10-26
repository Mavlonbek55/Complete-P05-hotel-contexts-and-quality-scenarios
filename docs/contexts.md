# Hotel contexts

## Words

| Word | Meaning here | Name in our code |
|---|---|---|
| Stay | Check-in to check-out; 12–14 Nov means the nights of 12 and 13 Nov | DateRange (check_in, check_out) |
| Reservation | Guest + room + stay + status, identified by an id such as R101 | Reservation (id, guest, room, stay, status) |
| Confirmed | The room is promised to this guest for these nights | ReservationStatus.CONFIRMED |
| Guest | The person making the booking | Guest |
| Room | A hotel room identified by its number and type | Room (number, type) |
| Room type | The category of a room, such as Standard or Deluxe | Room.type |
| Nightly price | The price charged for one night in a room | not in code yet |
| Out of service | A room is unavailable for sale because of repairs or another issue | not in code yet |
| Cancelled | A reservation's promise is withdrawn and its nights become available again | ReservationStatus.CANCELLED |
| Book | Request a room for a guest and stay | BookingService.reserve(guest, room, stay) |
| Cancel | Withdraw an existing reservation | BookingService.cancel(reservation_id) |
| Free rooms | Rooms without a conflicting confirmed reservation for the requested stay | BookingService.find_free_rooms(stay) |
| Deposit | 30% of the stay's price, taken when the guest books | not in code yet |
| Paid | The deposit has been successfully paid | not in code yet |
| Declined | The deposit payment was not successful | not in code yet |
| Confirmation message | A message telling the guest that the reservation was confirmed | NotificationSender.send_confirmation(reservation) |
| Sent | A notification was successfully sent | not in code yet |
| Failed | A notification attempt did not succeed | not in code yet |

## Rooms

Job: knows which rooms the hotel sells, including their type, nightly price and whether they are out of service.

| Word | Meaning here |
|---|---|
| Room | A room's inventory record, including its number, type, nightly price and service status |
| Room type | The category used to describe a room, such as Standard or Deluxe |
| Nightly price | The price for one night for this room or room type |
| Out of service | The room is temporarily not offered to guests |

Rules this context owns:
- Card 4: A room that is out of service is not offered to guests.
- Card 5: The nightly price depends on the room type; Deluxe costs more than Standard.

Needs from other contexts: nothing.
Tells other contexts: Rooms answers Reservations' question asking which rooms exist, returning each room's number and type.

## Reservations

Job: decides whether a guest gets a room for some nights, and keeps every reservation.

| Word | Meaning here |
|---|---|
| Stay | Check-in to check-out; 12–14 Nov means the nights of 12 and 13 Nov |
| Reservation | Guest + room number + stay + status, with an id such as R101 |
| Confirmed | The room is promised to this guest for these nights |
| Cancelled | The promise is withdrawn; the nights are free again |
| Guest | The person making the booking |
| Book | Request a room for a guest and stay |
| Cancel | Withdraw a reservation and release its nights |
| Free rooms | Rooms with no conflicting Confirmed reservation for the requested stay |

Rules this context owns:
- Card 1: A room never has two Confirmed reservations that share a night.
- Card 2: Check-out must be after check-in.
- Card 3: Cancelling a reservation frees its nights for new bookings.

Needs from other contexts: Rooms (the list of rooms and their room numbers and types).
Tells other contexts: announces ReservationConfirmed to Notifications when a reservation is saved as Confirmed.

## Payments

Job: takes deposits and records whether they were paid.

| Word | Meaning here |
|---|---|
| Deposit | 30% of the stay's price, taken when the guest books |
| Paid | The deposit was successfully paid |
| Declined | The deposit payment was not successful |

Rules this context owns:
- Card 6: The deposit is 30% of the stay's price.
- Card 7: Only a Paid deposit can be refunded.

Needs from other contexts: Rooms' nightly price information when calculating the stay's price.
Tells other contexts: answers a request to take a deposit with Paid or Declined.

## Notifications

Job: tells guests what happened, by email or SMS, and records whether each message arrived.

| Word | Meaning here |
|---|---|
| Confirmation message | A message telling the guest that the reservation was confirmed |
| Sent | The notification was successfully sent |
| Failed | The notification attempt did not succeed |

Rules this context owns:
- Card 8: A failed message is recorded as Failed and never changes the reservation.

Needs from other contexts: the ReservationConfirmed event and its message fields.
Tells other contexts: nothing; a failed message is recorded locally and does not change the reservation.

## Same word, different meaning

| Word | Rooms | Reservations | Payments | Notifications |
|---|---|---|---|---|
| Room | R8: Deluxe, nightly price, out-of-service status | The room number R8 attached to a reservation | Not used | Text in the message, such as “Room R8” |
| Status | In service / out of service | Confirmed / Cancelled | Paid / Declined | Sent / Failed |
| Cancel | Take a room off sale | Withdraw a reservation | Refund a deposit | Not used |

If Notifications could change a reservation's status to Failed, a failed message could overwrite Confirmed and Maya could lose room R7, breaking requirement R6 that a failed message must never change the reservation.

## Rule owners

| Card | Rule | Owner | Why this owner |
|---|---|---|---|
| 1 | A room never has two Confirmed reservations that share a night. | Reservations | It keeps every reservation for the room and can check them together. |
| 2 | Check-out must be after check-in. | Reservations | It owns the stay dates attached to each reservation. |
| 3 | Cancelling a reservation frees its nights for new bookings. | Reservations | It owns reservation status and the dates that become available. |
| 4 | A room that is out of service is not offered to guests. | Rooms | It owns each room's service status and inventory list. |
| 5 | The nightly price depends on the room type (Deluxe costs more than Standard). | Rooms | It owns room types and their nightly prices. |
| 6 | The deposit is 30% of the stay's price. | Payments | It owns deposit amounts and payment rules; it asks Rooms for nightly prices. |
| 7 | Only a Paid deposit can be refunded. | Payments | It owns deposit payment status and refund eligibility. |
| 8 | A failed message is recorded as Failed and never changes the reservation. | Notifications | It owns message delivery status; reservation status belongs to Reservations. |

## Between contexts

### Reservations → Notifications: ReservationConfirmed (event)

Sent when: a reservation is saved as Confirmed.

Fields:
- reservationId: R101
- guestEmail: maya@example.com
- roomNumber: R7
- checkIn: 2026-11-12
- checkOut: 2026-11-14

Reservations does not wait for an answer. A failed message never changes R101 (requirement R6).

Never crosses: Reservations' list of reservations. Notifications never reads it.

### Reservations → Rooms: which rooms exist? (question)

Sent when: Reservations needs the inventory list to search for free rooms or validate a requested room.

What Reservations sends:
- No reservation data; it asks for the current list of rooms.

The answer and its fields:
- A list of rooms.
- roomNumber: R7
- roomType: Standard
- roomNumber: R8
- roomType: Deluxe

Never crosses: Reservations' list of reservations or guest details. Rooms does not read reservation records.

### Reservations → Payments: take a deposit (question, from P17)

Sent when: a booking workflow needs to take the deposit.

What Reservations sends:
- reservationId: R101
- amount: the deposit amount (30% of the stay's price)

The answer and its fields:
- status: Paid or Declined

Never crosses: Reservations' list of reservations or the guest's reservation status. Payments owns payment status and does not update a reservation directly.

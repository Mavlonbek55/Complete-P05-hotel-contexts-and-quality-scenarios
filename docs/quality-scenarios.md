# Quality scenarios

## Stakeholders

| Who | What they care about |
|---|---|
| Guest | Finds a free room quickly; never loses a booking |
| Receptionist | Never promises one room twice; can cancel at the desk |
| Hotel manager (domain expert) | Knows the rules; wants deposits and SMS soon |
| Development team | Can change one area without breaking the others |
| Email/SMS and payment providers | Outside systems that can be slow or down |

## S1 · Performance

| Part | S1 |
|---|---|
| Source | 200 guests |
| Stimulus | They search free rooms for 12–14 Nov |
| Environment | During normal operation, within one minute |
| Artifact | The free-room search in BookingService |
| Response | Return the correct free rooms |
| Measure | p95 response time ≤ 1 second |
| Checked in | P08 (cache), P23 (load test) |

## S2 · Data integrity

| Part | S2 |
|---|---|
| Source | 100 pairs of guests |
| Stimulus | Each pair tries to book the same room for the same nights at the same moment |
| Environment | Concurrent booking requests |
| Artifact | Reservation creation and the booking rule |
| Response | Exactly one reservation in each pair is Confirmed |
| Measure | 0 double bookings; exactly 1 Confirmed reservation per pair |
| Checked in | P09 |

## S3 · Availability: a failed message keeps the booking

| Part | S3 |
|---|---|
| Source | The email provider |
| Stimulus | It stops answering |
| Environment | For 10 minutes, during normal bookings |
| Artifact | BookingService.reserve and the notification sender |
| Response | Every valid booking is still Accepted and saved; failed messages are reported |
| Measure | 0 lost bookings; every failed message is counted |
| Checked in | P04 (R6 test, done) and P10 |

## S4 · Security

| Part | S4 |
|---|---|
| Source | A guest |
| Stimulus | Tries 100 times to read another guest's reservation |
| Environment | While using the reservation service |
| Artifact | Reservation access and authorization checks |
| Response | Do not show another guest's reservation |
| Measure | 0 reservations shown to the wrong guest |
| Checked in | P21 |

## S5 · Modifiability

| Part | S5 |
|---|---|
| Source | Hotel manager |
| Stimulus | Requests SMS confirmations in addition to email |
| Environment | During a normal change to the application |
| Artifact | Notification sender and BookingService |
| Response | Add SMS without editing BookingService |
| Measure | BookingService is not edited; at most 2 files change |
| Checked in | P04 (SMS sender, done) |

## S6 · Sustainability

| Part | S6 |
|---|---|
| Source | Development team |
| Stimulus | Runs the same 1,000 searches before and after a change |
| Environment | Under comparable test conditions |
| Artifact | Free-room search implementation |
| Response | Reduce energy used while completing the searches |
| Measure | At least 20% fewer watt-hours per 1,000 completed searches |
| Checked in | P08, P24 |

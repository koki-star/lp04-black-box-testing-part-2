# North Vacation Booking Test Design

## 1. Conditions and actions

Conditions: check-in is before checkout; stay is 1 to 31 nights, there are 1 to 12 guests; all reserved nights are available; there are more than five guests.

Actions: create the reservation; calculate 200 dollars per night plus a 50 dollar cleaning fee; add 25 dollars per extra guest per night; raise `ValueError`; raise `UnavailableError`.

## 2. Decision table

`T` = true, `F` = false, `-` = does not matter, `X` = action occurs.

| Conditions and actions | R1 | R2 | R3 | R4 | R5 | R6 |
| --- | :---: | :---: | :---: | :---: | :---: | :---: |
| Check-in before checkout | T | T | F | T | T | T |
| 1 to 31 nights | T | T | - | F | T | T |
| 1 to 12 guests | T | T | - | - | F | T |
| All nights available | T | T | - | - | - | F |
| More than five guests | F | T | - | - | - | - |
| **Create reservation** | X | X |  |  |  |  |
| **Calculate standard price** | X | X |  |  |  |  |
| **Add extra guest fee** |  | X |  |  |  |  |
| **Raise `ValueError`** |  |  | X | X | X |  |
| **Raise `UnavailableError`** |  |  |  |  |  | X |

## 3. Decision table test cases

All dates are in 2026. Checkout is not a reserved night.

| Test ID | Conditions and inputs | Expected action | Expected price or exception |
| --- | --- | --- | --- |
| DT01 / R1 | Oct 1 to Oct 2; 1 night; 5 guests; all nights available | Create reservation in `Pending`; calculate standard price | 250 dollars = 200 + 50 |
| DT02 / R2 | Oct 1 to Nov 1; 31 nights; 12 guests; all nights available | Create reservation in `Pending`; calculate standard price and extra guest fee | 11,675 dollars = (31 × 200) + 50 + (7 × 31 × 25) |
| DT03 / R3 | Oct 2 to Oct 2; 5 guests | Reject reservation | `ValueError`; no price |
| DT04 / R4 | Oct 1 to Nov 2; 32 nights; 5 guests | Reject reservation | `ValueError`; no price |
| DT05 / R5 | Oct 1 to Oct 2; 1 night; 13 guests | Reject reservation | `ValueError`; no price |
| DT06 / R6 | Oct 1 to Oct 3; 2 nights; 3 guests; Oct 1 available, Oct 2 unavailable | Reject reservation | `UnavailableError`; no price |

## 4. States and events

States: `Pending`, `Confirmed`, `Checked In`, `Completed`, `Cancelled`.

Events: payment succeeds, payment fails, guest cancels, guest checks in, stay ends after checkout.

## 5. State transition table

The reservation is Oct 1 to Oct 3, 2026. Invalid events raise `InvalidStateError` and leave the state unchanged.

| ID | Current state | Event | Expected state | Expected result |
| --- | --- | --- | --- | --- |
| V1 | Pending | Payment succeeds | Confirmed | Confirm reservation |
| V2 | Pending | Payment fails | Cancelled | Cancel reservation |
| V3 | Pending | Guest cancels | Cancelled | Cancel reservation |
| V4 | Confirmed | Guest cancels | Cancelled | Cancel reservation |
| V5 | Confirmed | Guest checks in on Oct 1 | Checked In | Check in guest |
| V6 | Checked In | Stay ends on Oct 4 | Completed | Complete reservation |
| I1 | Pending | Guest tries to check in | Pending | `InvalidStateError` |
| I2 | Cancelled | Payment succeeds | Cancelled | `InvalidStateError` |
| I3 | Completed | Guest tries to cancel | Completed | `InvalidStateError` |

## 6. State transition test cases

Each test starts from the state shown. The reservation is Oct 1 to Oct 3, 2026.

| Test ID | Starting state | Event | Expected state | Expected result | Type |
| --- | --- | --- | --- | --- | --- |
| ST01 / V1 | Pending | Payment succeeds | Confirmed | Reservation confirmed | Valid |
| ST02 / V2 | Pending | Payment fails | Cancelled | Reservation cancelled | Valid |
| ST03 / V3 | Pending | Guest cancels | Cancelled | Reservation cancelled | Valid |
| ST04 / V4 | Confirmed | Guest cancels | Cancelled | Reservation cancelled | Valid |
| ST05 / V5 | Confirmed | Guest checks in on Oct 1 | Checked In | Guest checked in | Valid |
| ST06 / V6 | Checked In | Stay ends on Oct 4 | Completed | Reservation completed | Valid |
| ST07 / I1 | Pending | Guest tries to check in on Oct 1 | Pending | `InvalidStateError` | Invalid |
| ST08 / I2 | Cancelled | Payment succeeds | Cancelled | `InvalidStateError` | Invalid |
| ST09 / I3 | Completed | Guest tries to cancel | Completed | `InvalidStateError` | Invalid |
| ST10 / full stay | Pending | Payment succeeds; check in Oct 1; stay ends Oct 4 | Completed | Pending → Confirmed → Checked In → Completed | Valid sequence |
| ST11 / cancellation | Pending | Payment succeeds; guest cancels before check-in | Cancelled | Pending → Confirmed → Cancelled | Valid sequence |

## AI Transparency

AI was used in the creation of this file.
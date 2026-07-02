# Restaurant Reservation Bot — Bot specification

**Archetype:** booking

**Voice:** friendly and professional — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for restaurant table reservations with real-time availability checks and management. Guests book tables by selecting date, time, and party size; the bot shows only available slots and creates a reservation with a short code. Guests can modify or cancel reservations via inline buttons. Restaurant owners have a separate interface to view and manage all reservations, mark no-shows, and receive real-time notifications about changes.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant guests
- restaurant managers

## Success criteria

- Guests can successfully book, modify, and cancel reservations
- Restaurant owners can view and manage all reservations in real-time
- Real-time notifications are sent for all reservation changes
- Availability is accurately calculated and shown to guests

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu for guests
- **/owner** (command, actor: user, command: /owner) — Open the owner interface for managing reservations
- **Book a table** (button, actor: user, callback: booking:start) — Start the reservation process for a table
  - inputs: date, time, party size
  - outputs: reservation confirmation with code
- **Modify reservation** (button, actor: user, callback: booking:modify) — Modify an existing reservation
  - inputs: reservation code, new date/time
  - outputs: updated reservation confirmation
- **Cancel reservation** (button, actor: user, callback: booking:cancel) — Cancel an existing reservation
  - inputs: reservation code
  - outputs: cancellation confirmation

## Flows

### Guest reservation flow
_Trigger:_ /start

1. Greet guest
2. Show date selection calendar
3. Show available time slots for selected date
4. Request party size
5. Show summary and request contact information
6. Confirm reservation
7. Send confirmation with code and instructions

_Data touched:_ Reservation

### Reservation modification flow
_Trigger:_ booking:modify

1. Request reservation code
2. Show current reservation details
3. Show available time slots for new date
4. Update reservation with new time
5. Notify owner and guest of changes

_Data touched:_ Reservation

### Reservation cancellation flow
_Trigger:_ booking:cancel

1. Request reservation code
2. Confirm cancellation
3. Update reservation status
4. Notify owner and guest of cancellation

_Data touched:_ Reservation

### Owner management flow
_Trigger:_ /owner

1. Authenticate owner
2. Show list of upcoming reservations
3. Show availability summary
4. Show reservation details
5. Allow owner to confirm, cancel, or reschedule reservations
6. Allow owner to mark no-shows

_Data touched:_ Reservation, Restaurant

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Restaurant** _(retention: persistent)_ — Restaurant configuration including operating hours, seating duration, and table types
  - fields: operating_hours, seating_duration, table_types
- **Table** _(retention: persistent)_ — Table type with capacity and quantity
  - fields: type, capacity, quantity
- **Reservation** _(retention: persistent)_ — Reservation details including date, time, party size, assigned tables, code, and status
  - fields: date, start_time, seating_duration, party_size, assigned_tables, code, status
- **Guest** _(retention: persistent)_ — Guest information including name, Telegram contact, and phone number (optional)
  - fields: name, telegram_contact, phone_number

## Integrations

- **Telegram** (required) — Bot API messaging for guests and owners
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View all upcoming reservations
- Mark no-shows
- Confirm, cancel, or reschedule reservations
- View availability summary
- Configure restaurant settings (operating hours, seating duration, table types)
- Set reminder time before reservation

## Notifications

- Real-time notifications to owner for reservation creation/modification/cancellation
- Reminder notifications to guests before their reservation time

## Permissions & privacy

- Guest data is stored confidentially and only accessible to the owner and guest via reservation code
- Owner has access to all reservation data
- Guests can choose to provide or withhold contact information

## Edge cases

- Guest provides incomplete or strange data during reservation process
- Multiple guests trying to book the same table simultaneously
- Owner tries to modify a reservation that has already started
- Guest tries to modify/cancel a reservation with insufficient notice

## Required tests

- End-to-end reservation flow from date selection to confirmation
- Real-time notification system for owners
- Availability calculation with different table types and party sizes
- Error handling for incomplete or invalid input

## Assumptions

- Restaurant has a single configuration for operating hours, seating duration, and table types
- Guests will provide accurate information when booking
- Owner will have access to the /owner interface
- Notifications will be delivered through Telegram

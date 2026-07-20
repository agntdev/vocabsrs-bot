# Language SRS Bot — Bot specification

**Archetype:** education

**Voice:** warm and encouraging — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for language vocabulary learning using spaced repetition. Users manage decks/cards, schedule daily reviews with customizable new-card limits, complete quick SRS sessions with progress tracking, receive local-time reminders, and maintain private data with resumable sessions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual language learners
- self-paced vocabulary students

## Success criteria

- users complete daily SRS review sessions with progress tracking
- users maintain active streaks through scheduled reminders
- users create and manage private decks with custom vocabulary

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with deck selection and onboarding
- **Start Study Session** (button, actor: user, callback: session:start) — Begin daily review queue with due cards and new cards
  - inputs: deck selection, review preferences
  - outputs: card front display, progress summary
- **Manage Decks** (button, actor: user, callback: deck:manage) — Create, edit, delete, or import decks
  - inputs: deck metadata, card content
  - outputs: deck list, card preview
- **View Progress** (button, actor: user) — Create, edit, delete, or import decks
  - inputs: deck metadata, card content
  - outputs: deck list, card preview
- **View Progress** (button, actor: user, callback: progress:view) — Show daily due count, streaks, and per-deck statistics

## Flows

### Study Session
_Trigger:_ session:start or scheduled reminder

1. show card front (word)
2. show answer on button tap
3. collect SRS rating
4. update card interval
5. show next card or session summary

_Data touched:_ Review state, Session state

### Deck Management
_Trigger:_ /deck or button

1. list decks
2. select deck action
3. modify deck/cards

_Data touched:_ Deck, Card

### Reminder Handling
_Trigger:_ daily scheduled notification

1. check due cards
2. send reminder message
3. open session on tap

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with settings and preferences
  - fields: telegram_id, daily_new_cards, reminder_time, current_deck_id
- **Deck** _(retention: persistent)_ — Private vocabulary collection
  - fields: title, description, cards, visibility
- **Card** _(retention: persistent)_ — Word/translation pair with metadata
  - fields: front, back, example, srs_metadata
- **Session State** _(retention: session)_ — Paused review progress
  - fields: current_deck_id, card_index, paused_at
- **Starter Deck** _(retention: persistent)_ — Read-only template for importing
  - fields: title, cards

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- deck creation/editing/deletion
- session resuming
- daily settings (new cards, reminder time)

## Notifications

- daily reminder at user-specified time when reviews are due

## Permissions & privacy

- all data private to user's Telegram account
- no sharing between users
- session state expires after 7 days of inactivity

## Edge cases

- session expiration after 7 days
- no due cards on reminder time
- deck deletion during active session

## Required tests

- end-to-end study session with SRS rating flow
- session resuming after interruption
- deck import/export validation

## Assumptions

- SM-2 style SRS algorithm with default parameters
- default new-cards-per-day set to 20
- default reminder at 20:00 local time

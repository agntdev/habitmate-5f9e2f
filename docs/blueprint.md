# Private Habit Tracker — Bot specification

**Archetype:** workflow

**Voice:** warm and encouraging — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that helps users build habits through gentle single-button reminders, streak tracking, and milestone celebrations in a private, no-friction environment.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual users
- habit builders
- productivity-focused individuals

## Success criteria

- Users maintain consistent habit completion through daily check-ins
- Streak tracking motivates long-term habit formation
- Milestone celebrations increase engagement

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with habit overview and controls
- **Create New Habit** (button, actor: user, callback: habit:create) — Start habit creation flow with default schedule options
  - inputs: habit name, schedule type
  - outputs: new habit object
- **View Stats** (button, actor: user, callback: stats:weekly) — Show weekly recap with completion grid and streak info
- **Check In Now** (button, actor: user, callback: reminder:checkin) — Manually trigger immediate check-in for active habits

## Flows

### habit_creation
_Trigger:_ habit:create

1. Enter habit name
2. Select schedule type (daily/weekdays/weekly)
3. Confirm default reminder time

_Data touched:_ Habit

### daily_reminder
_Trigger:_ scheduled:local_time

1. Send single-button reminder message
2. Handle Check In/Snooze response
3. Lock day record after completion

_Data touched:_ HabitInstance, Streaks

### milestone_celebration
_Trigger:_ streak_milestone:7/21/60

1. Send milestone message with sticker
2. Update streak tracking

_Data touched:_ Streaks

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user profile with timezone and preferences
  - fields: telegram_id, timezone, default_reminder_time
- **Habit** _(retention: persistent)_ — User-created habit definition with schedule rules
  - fields: title, schedule_type, active_status, notes
- **HabitInstance** _(retention: persistent)_ — Daily habit completion records
  - fields: date, status, completion_time
- **Streaks** _(retention: persistent)_ — Tracking of habit consistency metrics
  - fields: current_streak, longest_streak, completion_rate

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete habits
- Pause/resume habits
- View habit history
- Configure reminder time

## Notifications

- Daily check-in reminders at local time
- Milestone celebration messages
- Weekly recap summaries

## Permissions & privacy

- All user data is private and only accessible to the user
- No cross-account sharing or social features
- Data stored securely with user consent

## Edge cases

- Timezone changes during active streak tracking
- Multiple habit check-ins attempted for same day
- Missed reminders during Telegram inactivity

## Required tests

- End-to-end check-in flow with snooze handling
- Streak tracking across 7/21/60 day milestones
- Weekly recap generation with visual grid

## Assumptions

- Users prefer 08:00 default reminder time
- Single-button check-in is sufficient for most users
- Milestone thresholds at 7/21/60 days are motivational

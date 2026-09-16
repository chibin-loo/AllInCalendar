# AllInCalendar

A desktop calendar for students that pulls every deadline and class into one place, then finds time in your week to actually get the work done.

AllInCalendar merges your school calendar feeds (Brightspace, Canvas, Google Calendar, or any iCal/ICS link) with your Gradescope assignments and your own tasks. It then looks at the free gaps between your commitments and automatically schedules work sessions for each task ahead of its due date.

Built with Java 21 and Swing.

---

## Features

**One view for everything**
- Import up to 10 iCal/ICS calendar feeds, including recurring events
- Log in to Gradescope and pull in every dated assignment across your courses
- Add your own tasks (with a deadline) and events (with a start and end time)
- List, Day, Week, and Month views, plus a mini calendar for quick navigation
- Consistent colours per course or recurring series
- Double-click an imported item to open its link (e.g. the assignment page)

**Automatic work scheduling**
- Finds free time between your events, within your working hours
- Places work blocks for each task, earliest deadline first
- Spreads big tasks across several days (one session per day) before falling back to packing them in
- Respects a daily work cap, maximum block length, breaks between blocks, and days you never want to work
- Warns you with a banner when a task can't fully fit before its deadline, and shows how much time is unscheduled

**Hands-on control**
- Drag a work block to move it — it gets "pinned" there and the scheduler plans around it
- Drag on empty grid space to create an event; double-click to create a task
- Drag your own tasks and events to reschedule them
- Mark anything done, including imported events (tracked locally)

**Smart estimates**
- Guesses whether something you type is a task or an event (e.g. "Study for midterm" vs. "Midterm exam")
- Suggests how long a task will take from keywords (essay, problem set, final project…) and from how long you sized similar past tasks ("CS 180 Project 3" learns from "CS 180 Project 2")
- Each kind of work gets its own lead time and session length — studying is split into shorter spaced sessions, projects get longer runways
- Optional AI estimate: with an API key, click **Estimate from notes** to have a language model size a task from its description

**Export**
- **File → Export .ics…** writes your own tasks, events, and scheduled work blocks to a standard `.ics` file you can import into Google Calendar, Apple Calendar, Outlook, etc. Imported feeds are left out, since you can subscribe to those directly.

---

## Requirements

- Java 21 or newer
- Maven 3.x

Dependencies (downloaded automatically by Maven): [ical4j](https://github.com/ical4j/ical4j), [jsoup](https://jsoup.org/), [FlatLaf](https://www.formdev.com/flatlaf/), and [Gson](https://github.com/google/gson).

## Getting started

```bash
git clone https://github.com/chibin-loo/AllInCalendar.git
cd AllInCalendar
mvn compile exec:java
```

The app stores its data in the directory you launch it from, so run it from the same folder each time.

## Setup

Open **Settings** from the List tab. Everything is optional — the app works as a plain task calendar with nothing configured.

| Tab | What to set |
| --- | --- |
| **Calendars** | Paste iCal/ICS subscription links from Brightspace, Canvas, Google Calendar, etc. |
| **Gradescope** | Your Gradescope email and password. Leave blank to skip. |
| **Scheduling** | Auto-scheduling on/off, working hours, default task length, daily work cap, longest block, break length, minimum gap, lead days, planning horizon, and days off. |
| **AI** | An API key for task-length estimates. Defaults to Groq (a free key from [console.groq.com](https://console.groq.com)), but any OpenAI-compatible chat endpoint works — just change the model and URL. |
| **Display** | How many months of history and how many months ahead to load. |

After saving, the app reloads all feeds. Use **Full Refresh** any time to re-download them.

> The Settings window shows five calendar link fields. Up to ten are supported — add `calendar6` through `calendar10` to `settings.txt` by hand if you need more.

## How scheduling works

1. Imported events and your own events with start and end times count as busy time. Deadlines on their own don't block time.
2. For each day in the planning window, the gaps between busy periods (inside your working hours, and longer than the minimum gap) become free blocks.
3. Pinned work blocks are placed first.
4. Each unfinished task, in deadline order, gets a profile: total minutes, how many days before the deadline to start, and the longest session. Your own duration beats the estimate, your **max block** setting caps session length, and your **lead days** setting acts as a minimum runway.
5. Work is placed into free blocks in the window before the deadline — one session per day first, then without that limit if needed.
6. Anything that still doesn't fit is reported as unscheduled.

## Data files

All data is stored as plain text in the working directory. These are already listed in `.gitignore`.

| File | Contents |
| --- | --- |
| `settings.txt` | All settings as `key=value` lines |
| `tasks.txt` | Your tasks and events |
| `done-overrides.txt` | Imported events you've marked done |
| `work-pins.txt` | Work blocks you've dragged into place |

> **Security note:** your Gradescope password and AI API key are saved unencrypted in `settings.txt`. Keep that file private and never commit it.

## Project structure

```
src/main/java/com/artlu/
├── Window.java          # Entry point: main window, tabs, list view
├── Main.java            # Feed import, task storage, free-time finder, scheduler
├── CalendarUI.java      # Shared calendar grid, drag handling, context menus
├── DayWindow.java       # Day view
├── WeekWindow.java      # Week view
├── MonthWindow.java     # Month view
├── MiniCalendar.java    # Sidebar month picker
├── TaskDialog.java      # New/edit task and event form
├── SettingsWindow.java  # Settings dialog
├── Settings.java        # settings.txt reader/writer
├── Gradescope.java      # Gradescope login and assignment scraping
├── Estimator.java       # Task length / lead time / session estimates
├── Classify.java        # Task-vs-event guessing
├── AiDuration.java      # Optional LLM duration estimate
└── IcsExport.java       # .ics export
```

## Limitations

- Gradescope support works by scraping the website, so it may break if Gradescope changes its pages. It also won't work with accounts that sign in through school SSO only, since it needs an email and password.
- Imported calendar events are read-only; edit them at the source.
- Work blocks are matched to tasks by name, so two tasks with the same name will share pins.

## License

No license has been specified yet.

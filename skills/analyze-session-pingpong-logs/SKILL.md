---
name: analyze-session-pingpong-logs
description: Use when analyzing omp/OpenCode session data for ping-pong, micromanagement, pauses, and cost patterns. Not for live debugging or history edits.
---

# Analyze Session Ping-Pong Logs

Identify repeated manual micro-management in stored agent session logs. Keep all sources read-only and derive metrics from detailed records rather than incomplete rollups or duplicated histories.

## Data sources

1. **omp statistics database:** `~/.omp/stats.db` (default location; verify on the current system).
   - `messages`: assistant steps, including `session_file`, `folder`, `timestamp`, `model`, token fields, and `cost_total`.
   - `user_messages`: timestamp, `chars`, `prose`, and `session`.
   - `session_rollup` and `message_rollup` may be incomplete; calculate metrics from the detailed tables.
   - Use `folder` or project metadata for study-versus-work classification, with an explicit mapping appropriate to the analyzed paths.
2. **omp session export:** JSON containing `sessions[]`.
   - `metadata`: `id`, `file`, `project`, `title`, `created`, and `modified`.
   - `raw_jsonl_lines`: events; messages have `type=message` and `role=user` or `role=assistant`; assistant token totals are in `assistant.message.usage.totalTokens`.
   - Forked sessions can duplicate history; deduplicate them and never sum copies.
3. **OpenCode database:** `~/.local/share/opencode/opencode.db` (default location; verify on the current system).
   - Canonical path: `session_v2` (`parent_id IS NULL` selects main sessions) → `session_message` (`seq`, `type`, `time_created`, JSON `data`).
   - Read user text from `data.text` and assistant steps from `data.content[]`.
   - Legacy `session`, `message`, and `part` tables have already been copied into `session_v2`; do not add both representations.

Open SQLite databases read-only through Python `sqlite3` with the URI `file:<path>?mode=ro`; WAL files may be present. Never modify source logs.

## Analysis procedure

1. Inventory the available sources and select main, deduplicated sessions. Do not combine legacy and canonical OpenCode records or forked omp histories.
2. Build a chronological event stream for each session. Preserve user text, timestamps, assistant steps, tokens, cost, and project or folder metadata.
3. Construct the **user-turn chronology** by removing assistant, tool, and system events from consideration. Those events do not break a user series, even though the harness normally inserts assistant records between user messages.
4. Detect a ping-pong series using all of these rules:
   - Contain at least 3 consecutive user messages within one session.
   - Require every message to contain fewer than 300 characters after trimming.
   - Require each adjacent-message interval to be less than 5 minutes.
   - Break the series at a message of 300 or more trimmed characters or a gap of 5 minutes or more.
5. Flag messages containing these marker words or phrases: `proceed`, `progress?`, `check state`, `also`, `wait also`, and `no no`.
6. Detect cross-session monitoring: session A has no activity for at least 15 minutes while session B has activity.
7. For pauses longer than 1 hour, inspect the last assistant response before the pause and classify it as a completed stage or an unfinished pipeline.

Do not apply a literal requirement for physically adjacent user rows; assistant records between them do not invalidate a series.

## Report format

1. Provide a session table with project, duration, user-message count, assistant-step count, tokens, and cost.
2. For sessions containing series, report series count, mean and maximum series length, and verbatim examples of 2–4 consecutive user messages.
3. Report cross-session pauses that indicate parallel monitoring.
4. Compare study and work using the explicit `folder` or project mapping.
5. Summarize long pauses by whether the preceding assistant response reached a natural completion point.
6. Summarize marker-word occurrences alongside the detected series; do not treat a marker alone as proof of a series.

Aggregate large exports and databases with Python instead of reading them whole. Use SQLite read selectors or Python `sqlite3` for queries, and keep analysis outputs separate from source files.

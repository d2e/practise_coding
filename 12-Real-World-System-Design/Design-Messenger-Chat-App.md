---
title: "Design Messenger/Chat App (like WhatsApp) — Build the APIs"
aliases: ["Design WhatsApp"]
tags: [real-world, system-design, api-design, hard]
source: "Glassdoor (Sr SWE), reported Nov 2025"
leetcode_number: unknown
leetcode_difficulty: "Hard (system design, not on LeetCode)"
pattern: "System Design"
status: not-started
---

# Design Messenger/Chat App (like WhatsApp) — Build the APIs

> [!warning] Reconstructed from a crowd-sourced interview report
> The source says only "Design Messenger/Chat app like WhatsApp — build the APIs." No further detail survived. The scope below (1:1 + group messaging, delivery/read receipts, presence, offline delivery) is the standard shape of this classic system-design prompt, deliberately anchored to "build the APIs" per the report rather than deep infra (sharding strategy, load balancer topology) — but skim the *Scaling Considerations* section too, since a senior-level interviewer will likely push there once the API contract is solid.

## Problem Prompt
Design the backend APIs for a messaging app like WhatsApp: users can send 1:1 and group text messages, see delivery and read receipts, see who's online, and receive messages sent while they were offline once they reconnect. Focus on the API contract and data model; discuss scaling only as a follow-up.

## Requirements Gathering

### Functional Requirements
- Send a message (1:1 or group) and have it delivered to all recipients, online or not.
- Fetch conversation history with pagination.
- Delivery receipts (message reached recipient's device) and read receipts (recipient opened it).
- Presence (online/offline/last-seen).
- Create/manage group conversations (add/remove members).
- Typing indicators (nice-to-have — mention it, don't over-invest).

### Non-Functional Requirements
- **Low latency** for online-to-online delivery (sub-second, feels "instant").
- **Durability** — a message must never be silently lost, even if the recipient is offline for days.
- **Ordering** — messages within one conversation must display in a consistent order across all participants' devices.
- **At-least-once delivery** with client-side dedup (via a client-generated message ID) is an acceptable, standard trade-off over exactly-once, which is much harder to guarantee end-to-end.

## Clarifying Questions to Ask
- Text-only, or also media (images/video)? (Assume text-only unless told otherwise — media adds a whole separate blob-storage/CDN subsystem.)
- End-to-end encryption in scope, or just transport security? (Assume out of scope unless asked — it changes where read receipts and server-side search can even be implemented.)
- Scale target — is this a 10-person team chat or a billion-user global app? (Materially changes the storage/sharding answer; ask before designing for the wrong scale.)

## API Design

| Method | Path | Description |
|---|---|---|
| `POST` | `/conversations` | Create a 1:1 or group conversation; body includes participant user IDs and, for groups, a name. |
| `GET` | `/conversations` | List the current user's conversations, most-recently-active first. |
| `POST` | `/conversations/{id}/messages` | Send a message; body includes `client_message_id` (for dedup), `text`, `sent_at`. Returns the server-assigned `message_id` and `sequence_number`. |
| `GET` | `/conversations/{id}/messages?before={cursor}&limit=50` | Paginated message history, cursor-based (not offset-based — see *Data Model*). |
| `POST` | `/conversations/{id}/messages/{message_id}/receipts` | Report a delivery or read receipt for a message. |
| `PATCH` | `/conversations/{id}/members` | Add/remove group members. |
| `WS` | `/ws/connect` | Persistent WebSocket for real-time inbound message push, presence updates, and typing indicators. |
| `GET` | `/users/{id}/presence` | Fallback poll for a user's online/last-seen status (primary delivery is via the WebSocket presence channel). |

## Data Model
- **User**: `user_id`, `display_name`, `last_seen_at`.
- **Conversation**: `conversation_id`, `type` (`direct`/`group`), `name` (groups only), `member_ids`.
- **Message**: `message_id` (server-assigned, globally unique), `client_message_id` (client-generated, for dedup on retry), `conversation_id`, `sender_id`, `text`, `sequence_number` (monotonic **per conversation** — this is what actually defines display order, not wall-clock `sent_at`, since clocks skew across devices), `sent_at`.
- **Receipt**: `message_id`, `user_id`, `status` (`delivered`/`read`), `timestamp`. Modeled as its own table/stream rather than a field on `Message`, since a group message has one receipt *per recipient*, not one global status.

Pagination should be **cursor-based** on `sequence_number` (`before=<sequence_number>`), not page-number-based — conversations are append-only logs, and offset pagination silently skips/duplicates messages as new ones arrive between page fetches.

## High-Level Architecture
- **Connection/Gateway layer**: holds the persistent WebSocket connections, maps `user_id -> connection` (typically in-memory per gateway node, backed by a shared registry like Redis so any node can find where a user is currently connected).
- **Messaging service**: owns the API above; on `POST .../messages`, it (a) durably persists the message first, (b) assigns the sequence number, then (c) pushes it to any currently-connected recipients via the gateway layer, and (d) enqueues it for offline recipients.
- **Message store**: an append-only, partitioned-by-conversation log (a Kafka-like log or a wide-column store keyed by `conversation_id` + `sequence_number` is a natural fit) — optimized for "append + range-scan," which is exactly the read/write pattern here.
- **Offline delivery queue**: per-user queue of undelivered message references; on reconnect, the client fetches everything since its last-acknowledged `sequence_number` per conversation.
- **Presence service**: a fast, eventually-consistent store (in-memory + short TTL) — presence is inherently approximate (a user can vanish mid-TCP-timeout), so don't over-engineer strong consistency here.

## Real-Time Delivery & Scaling
- WebSocket (or a push-notification fallback like FCM/APNs when the app is backgrounded) is the standard choice over long-polling for a chat app — it minimizes both latency and the connection-churn overhead of repeated polling at scale.
- **Fan-out on write vs. fan-out on read**: for large groups, don't push directly to every member's connection synchronously on the request path — publish the message to a per-conversation topic and let each member's gateway connection subscribe/pull. This decouples "how many recipients" from "how long the sender's request takes."
- **Sharding**: shard the message store and offline-queue by `conversation_id` (or `user_id` for the offline queue) so a single conversation's write load doesn't require touching every node.
- **Ordering guarantee**: because `sequence_number` is assigned by a single authority per conversation (not by clients), you avoid the classic distributed-clock-skew ordering bug — this is worth calling out explicitly, it's usually the detail that separates a strong answer from a mediocre one on this question.

## Trade-offs & Follow-ups
- At-least-once delivery + client-side dedup via `client_message_id` (simple, standard) vs. exactly-once (requires distributed transactions/idempotency keys at every hop — usually not worth the complexity for chat).
- Storing full message history forever vs. a retention/archival policy (move old conversations to cold storage) — ask about retention requirements before assuming "forever."
- A common interviewer follow-up: "now the user has 3 devices logged in simultaneously — how does delivery/read-receipt state stay consistent across devices?" (Answer shape: receipts are per-`(message_id, user_id)`, not per-device, and each device independently tracks its own last-synced `sequence_number` per conversation.)

## Related Problems
- [[Merge-k-Sorted-Lists]]
- [[Accounts-Merge]]
- [[Process-Stock-Transactions-to-Generate-Portfolio]]

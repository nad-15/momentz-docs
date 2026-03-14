# Momentz – Architecture Notes

## Goal

Design a scalable schema that supports:

* Moments
* Memories (textual reflections)
* Media (photos/videos)
* User feedback to developer
* Content reporting / moderation

The design should allow future expansion without large database migrations.

---

# Core Concept

Real-world memory behavior works in two directions:

1. Multiple photos can represent **one memory**
2. One photo can trigger **multiple memories**

Therefore we should avoid strict relationships like:

Photo → Memory

Instead we anchor everything to **Moment**.

Correct structure:

Moment

* Media[]
* Memories[]

---

# Final Recommended Database Structure

## Moment

The main container object.

```
Moment
 ├─ Media[]
 ├─ Memories[]
 ├─ Reactions
 ├─ Bookmarks
 └─ Notifications
```

A Moment represents an event, experience, or situation.

---

## Media

Stores photos or videos uploaded to Cloudinary.

Even if the app currently allows **only one media**, the schema should support **multiple media in the future**.

```
model Media {
  id          String @id @default(uuid())
  momentId    String
  moment      Moment @relation(fields: [momentId], references: [id], onDelete: Cascade)

  mediaUrl          String?
  mediaPublicId     String?
  mediaAssetId      String?
  mediaFormat       String?
  mediaWidth        Int?
  mediaHeight       Int?
  mediaBytes        BigInt?
  mediaVersion      String?
  mediaResourceType String?
  mediaType         String? @default("image")
  mediaDuration     Float?
  mediaThumbnailUrl String?
  mediaAltText      String?

  order      Int      @default(0)
  createdAt  DateTime @default(now())
}
```

Current rule (app logic):

```
max upload = 1 media
```

Future rule:

```
max upload = multiple media
```

No database migration required.

---

## Memory

A textual reflection connected to a Moment.

A single photo may trigger multiple memories.

```
model Memory {
  id          String   @id @default(uuid())
  title       String
  description String
  emotions    String[] @default([])

  momentId    String
  moment      Moment @relation(fields: [momentId], references: [id], onDelete: Cascade)

  order       Int      @default(0)
  createdAt   DateTime @default(now())
}
```

Example:

Moment: "Trip to Japan"

Memories:

* "First time seeing Tokyo nightlife"
* "Best sushi I ever ate"
* "Kyoto temple felt peaceful"

---

# Developer Feedback System

Users should be able to send feedback directly to the developer.

This should **NOT use the Notification table**.

Instead create a dedicated Feedback table.

```
model Feedback {
  id        String   @id @default(uuid())

  userId    String?
  user      User? @relation(fields: [userId], references: [id], onDelete: SetNull)

  message   String
  type      FeedbackType @default(GENERAL)
  status    FeedbackStatus @default(PENDING)

  createdAt DateTime @default(now())
}
```

Enums:

```
enum FeedbackType {
  BUG
  FEATURE
  GENERAL
}

enum FeedbackStatus {
  PENDING
  REVIEWED
  RESOLVED
}
```

Purpose:

Developer inbox for bug reports and feature suggestions.

---

# Content Reporting System

Users must be able to report inappropriate memories.

This requires a **Report table**.

```
model Report {
  id         String @id @default(uuid())

  reporterId String
  reporter   User @relation(fields: [reporterId], references: [id], onDelete: Cascade)

  memoryId   String
  memory     Memory @relation(fields: [memoryId], references: [id], onDelete: Cascade)

  reason     ReportReason
  message    String?

  createdAt  DateTime @default(now())

  @@unique([reporterId, memoryId])
}
```

This prevents spam reporting (one user can report only once).

---

## Report Reasons

```
enum ReportReason {
  SPAM
  HARASSMENT
  VIOLENCE
  NSFW
  MISINFORMATION
  OTHER
}
```

---

# Moderation Logic

Content should **not be auto-deleted immediately**.

Recommended workflow:

```
Reports >= 5
→ mark memory as hidden

Admin review
→ delete or restore
```

Add a flag to Memory:

```
isHidden Boolean @default(false)
```

Backend logic example:

```
reportCount >= 5 → hide memory
```

This prevents malicious users from mass-reporting legitimate content.

---

# Final Architecture Summary

```
Moment
 ├─ Media[]
 ├─ Memory[]
 ├─ Reaction
 ├─ Bookmark
 └─ Notification

Memory
 └─ Report[]

Feedback
 └─ Developer inbox
```

Each system has a separate responsibility.

---

# Key Design Principles

1. Anchor everything to **Moment**
2. Keep **Media separate from Moment**
3. Allow **multiple memories**
4. Future-proof media uploads
5. Separate **feedback** from **notifications**
6. Use a **report table for moderation**

This architecture prevents large schema migrations later and supports future product growth.

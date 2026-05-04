# 🎟️ Building a Seat Reservation System — Explained Simply

> **The Problem:** Imagine Beyoncé tickets go on sale. 300,000 fans hit "Reserve" at the same second for 80,000 seats. How do you make sure no two people get the same seat?

---

## 📐 The Schema (Database Table Design)

Think of this like designing a spreadsheet for seats. Each row represents one physical seat.

```sql
CREATE TABLE seats (
  id          BIGSERIAL PRIMARY KEY,       -- unique seat ID
  event_id    BIGINT NOT NULL,             -- which concert
  section     VARCHAR(10) NOT NULL,        -- e.g., "A", "VIP"
  row_label   VARCHAR(5) NOT NULL,         -- e.g., "R1", "R2"
  number      INTEGER NOT NULL,            -- seat number in that row
  status      VARCHAR(20) NOT NULL DEFAULT 'available',
  -- status can be: 'available', 'reserved', 'booked'
  held_by     BIGINT,                      -- which user is holding it
  held_until  TIMESTAMPTZ,                 -- hold expires at this time
  version     INTEGER NOT NULL DEFAULT 0,  -- for optimistic locking
  UNIQUE (event_id, section, row_label, number)
);

-- Fast lookup: find all seats for an event by status
CREATE INDEX idx_seats_event_status ON seats(event_id, status);

-- Fast expiry sweep: only index rows that are currently reserved
CREATE INDEX idx_seats_held_until ON seats(held_until) WHERE status = 'reserved';
```

**Why `version`?** It's a counter. Every time a seat is updated, version goes up (0 → 1 → 2...). This helps detect if someone else changed the seat while you were looking at it.

**Why the partial index on `held_until`?** Instead of scanning all 80,000 rows to find expired holds, this index only tracks the "reserved" ones — much faster.

---

## 🚨 The Core Problem: Race Conditions

### What is a Race Condition?

Imagine two people at a store, both picking up the last item at the exact same moment. The cashier lets both buy it. That's a race condition — two operations happening so fast they don't see each other.

### The Broken Version

```typescript
// ❌ BROKEN — This looks safe but is NOT under high traffic
async function reserveSeats(seatIds: number[], userId: number): Promise<void> {
  await db.transaction(async (trx) => {
    // Step 1: Read all seats
    const seats = await trx("seats").whereIn("id", seatIds);

    // Step 2: Check if all are available
    for (const seat of seats) {
      if (seat.status !== "available") {
        throw new Error(`Seat ${seat.id} is not available`);
      }
    }

    // ⚠️ GAP HERE: Another user can sneak in between Step 1 and Step 3!

    // Step 3: Mark them as reserved
    await trx("seats").whereIn("id", seatIds).update({
      status: "reserved",
      held_by: userId,
      held_until: new Date(Date.now() + 10 * 60 * 1000), // 10 min from now
    });
  });
}
```

### Why Does This Fail?

PostgreSQL uses something called **MVCC (Multi-Version Concurrency Control)**. When you do a `SELECT`, it takes a **snapshot** — a photo of the data at that moment. The photo becomes stale instantly.

```
User A reads seats [102, 103] → both "available" ✅
User B reads seats [102, 103] → both "available" ✅  (same snapshot!)

User A writes → seats 102, 103 = "reserved for A" 
User B writes → seats 102, 103 = "reserved for B" 💥 DOUBLE BOOKING!
```

The transaction block only guarantees your writes go in together or not at all — **it does NOT stop others from reading and writing at the same time**.

---

## ✅ Fix 1: Pessimistic Locking (`SELECT FOR UPDATE`)

### The Idea

"I'm about to update these rows — **nobody else can touch them until I'm done.**"

This is like putting your hand on the last item in a store and saying "This is mine while I pay."

```typescript
// ✅ CORRECT — Pessimistic locking with ordered IDs
async function reserveSeats(seatIds: number[], userId: number): Promise<void> {
  // 🔑 CRITICAL: Always sort IDs before locking to prevent deadlocks
  const sortedIds = [...new Set(seatIds)].map(Number).sort((a, b) => a - b);

  await db.transaction(async (trx) => {
    // Lock rows in ascending order — nobody else can touch these rows now
    const seats = await trx("seats")
      .whereIn("id", sortedIds)
      .orderBy("id", "asc")
      .forUpdate()               // ← This is SELECT ... FOR NO KEY UPDATE
      .select("*");

    // Check all seats are available
    const unavailable = seats.filter((s) => s.status !== "available");
    if (unavailable.length > 0) {
      throw new Error(
        `Seats not available: ${unavailable.map((s) => s.id).join(", ")}`
      );
    }

    // All good — update them in one batch
    await trx("seats").whereIn("id", sortedIds).update({
      status: "reserved",
      held_by: userId,
      held_until: new Date(Date.now() + 10 * 60 * 1000),
    });
  });
  // 🔓 Locks are released automatically when transaction ends
}
```

```cpp
// C++ pseudocode — same concept using a database library
void reserveSeats(std::vector<int> seatIds, int userId, DBConnection& db) {
    // Sort IDs to prevent deadlocks
    std::sort(seatIds.begin(), seatIds.end());
    seatIds.erase(std::unique(seatIds.begin(), seatIds.end()), seatIds.end());

    auto txn = db.beginTransaction();

    try {
        // Lock rows in sorted order
        std::string placeholders = buildPlaceholders(seatIds);
        auto seats = txn.query(
            "SELECT * FROM seats WHERE id IN (" + placeholders + ")"
            " ORDER BY id FOR NO KEY UPDATE",
            seatIds
        );

        // Check availability
        for (const auto& seat : seats) {
            if (seat["status"] != "available") {
                throw std::runtime_error("Seat " + seat["id"] + " unavailable");
            }
        }

        // Update all in one shot
        auto expiryTime = currentTime() + 10 * 60; // 10 minutes
        txn.execute(
            "UPDATE seats SET status='reserved', held_by=?, held_until=?"
            " WHERE id IN (" + placeholders + ")",
            {userId, expiryTime, seatIds}
        );

        txn.commit();
    } catch (...) {
        txn.rollback();
        throw;
    }
}
```

### Why Sort the IDs? 🔑

This prevents **deadlocks**. More on that below, but the short answer:

- User A wants seats [201, 205] → locks 201 first, then 205
- User B wants seats [205, 201] → also locks 201 first (because sorted!), then waits

Without sorting, A locks 201 and B locks 205 at the same time → **neither can proceed → deadlock**.

### Pros and Cons

| Pros | Cons |
|------|------|
| Simple to understand | Requests wait in a queue |
| No retries needed | Slow operations inside = everything backs up |
| Works with default isolation | Need consistent lock ordering |

---

## ✅ Fix 2: Optimistic Locking — Fail Fast Instead of Waiting

### The Idea

"I'll assume nobody changed this. Let me write, and if I'm wrong, the database will tell me."

Like filling out a form — you assume the last name you read is still correct. If someone else changed it while you wrote, you get an error and try again.

```typescript
// ✅ Optimistic Locking using version column
async function reserveSeatsOptimistic(
  seatIds: number[],
  userId: number
): Promise<void> {
  const seats = await db("seats")
    .whereIn("id", seatIds)
    .orderBy("id")
    .select("*");

  // Check availability upfront (no lock yet)
  const unavailable = seats.filter((s) => s.status !== "available");
  if (unavailable.length > 0) {
    throw new Error("Some seats are not available");
  }

  await db.transaction(async (trx) => {
    for (const seat of seats) {
      const updatedRows = await trx("seats")
        .where({ id: seat.id, version: seat.version }) // ← The magic check
        .update({
          status: "reserved",
          held_by: userId,
          held_until: new Date(Date.now() + 10 * 60 * 1000),
          version: seat.version + 1, // ← Increment version
        });

      if (updatedRows === 0) {
        // 0 rows updated = someone else changed this seat first
        throw new Error(`Seat ${seat.id} was modified by another user`);
      }
    }
  });
}

// ✅ Even simpler: Single atomic UPDATE (best for single seats)
async function claimSeat(seatId: number, userId: number): Promise<boolean> {
  const updatedRows = await db("seats")
    .where({ id: seatId, status: "available" }) // ← Only update if still available
    .update({
      status: "reserved",
      held_by: userId,
      held_until: new Date(Date.now() + 10 * 60 * 1000),
    });

  return updatedRows === 1; // true = got it, false = someone beat us
}
```

```cpp
// C++ version — atomic conditional UPDATE
bool claimSeat(int seatId, int userId, DBConnection& db) {
    auto expiryTime = currentTime() + 10 * 60;

    // This single SQL statement is atomic — no race condition possible!
    // The WHERE status='available' check and the UPDATE happen atomically
    int rowsAffected = db.execute(
        "UPDATE seats SET status='reserved', held_by=?, held_until=?"
        " WHERE id=? AND status='available'",
        {userId, expiryTime, seatId}
    );

    return rowsAffected == 1;
    // true  → we got the seat
    // false → someone else already reserved it
}
```

### Why Does Single UPDATE Work Without a Transaction?

PostgreSQL treats every single SQL statement as its own mini-transaction. The `WHERE status = 'available'` check and the write happen **in the same instant** — there's no gap for another user to sneak in.

### Pros and Cons

| Pros | Cons |
|------|------|
| Non-blocking — fails immediately | Multi-seat = complex rollback logic |
| Very high throughput when conflicts are rare | Under high traffic, everyone fails at once |
| Single UPDATE = fastest possible | Doesn't help with aggregate rule violations |

---

## ✅ Fix 3: Transaction Isolation Levels

Think of isolation levels as "how much does your transaction care about what others are doing?"

### Level 1: READ COMMITTED (Default)

Each SQL statement sees a **fresh snapshot**. Stale data between statements.

```typescript
// ❌ PROBLEM: Write skew — user ends up with too many seats
async function reserveWithLimit(
  seatIds: number[],
  userId: number,
  eventId: number
): Promise<void> {
  await db.transaction(async (trx) => {
    // Count how many seats this user already has
    const [{ count }] = await trx("seats")
      .where({ event_id: eventId, held_by: userId })
      .whereIn("status", ["reserved", "booked"])
      .count("* as count");

    const existing = Number(count); // e.g., 2

    if (existing + seatIds.length <= 6) {
      // ⚠️ RIGHT HERE: another request commits 4 seats for this user!
      // When we reach the UPDATE below, user will have 2 + 4 + 3 = 9 seats
      await trx("seats").whereIn("id", seatIds).update({ status: "reserved" });
    }
  });
}
```

### Level 2: REPEATABLE READ

The entire transaction uses **one frozen snapshot** from the start. Better, but still has a blind spot.

```typescript
// Better — but still has a gap for disjoint rows
await db.transaction({ isolationLevel: "repeatable read" }, async (trx) => {
  const [{ count }] = await trx("seats")
    .where({ event_id: eventId, held_by: userId })
    .whereIn("status", ["reserved", "booked"])
    .count("* as count");

  // Snapshot is frozen from first statement
  // BUT: if another transaction reserved DIFFERENT seats for this user,
  // PostgreSQL won't detect a conflict (different rows = no overlap)
  // Result: user still ends up with 9 seats!
});
```

### Level 3: SERIALIZABLE (Full Protection)

PostgreSQL tracks **what you read** (the WHERE clause), not just what you wrote. If another transaction's writes would have changed your reads, one of you gets aborted.

```typescript
// ✅ Full protection — catches write skew on disjoint rows
async function reserveWithLimitSafe(
  seatIds: number[],
  userId: number,
  eventId: number,
  maxRetries = 3
): Promise<void> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      await db.transaction({ isolationLevel: "serializable" }, async (trx) => {
        const [{ count }] = await trx("seats")
          .where({ event_id: eventId, held_by: userId })
          .whereIn("status", ["reserved", "booked"])
          .count("* as count");

        if (Number(count) + seatIds.length > 6) {
          throw new Error("Seat limit exceeded");
        }

        await trx("seats").whereIn("id", seatIds).update({
          status: "reserved",
          held_by: userId,
          held_until: new Date(Date.now() + 10 * 60 * 1000),
        });
      });

      return; // success!
    } catch (err: any) {
      const isSerializationError =
        err.code === "40001"; // PostgreSQL serialization failure code
      if (!isSerializationError || attempt === maxRetries - 1) throw err;

      // Wait a bit before retrying
      await sleep(50 * (attempt + 1));
    }
  }
}

function sleep(ms: number) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

```cpp
// C++ retry wrapper for serializable transactions
void reserveWithSerializable(std::vector<int> seatIds, int userId, int eventId,
                              DBConnection& db, int maxRetries = 3) {
    for (int attempt = 0; attempt < maxRetries; attempt++) {
        try {
            auto txn = db.beginTransaction("SERIALIZABLE");

            // Count existing seats for user
            auto result = txn.queryOne(
                "SELECT COUNT(*) as cnt FROM seats "
                "WHERE event_id=? AND held_by=? AND status IN ('reserved','booked')",
                {eventId, userId}
            );
            int existing = std::stoi(result["cnt"]);

            if (existing + (int)seatIds.size() > 6) {
                txn.rollback();
                throw std::runtime_error("Seat limit exceeded");
            }

            txn.execute("UPDATE seats SET status='reserved', held_by=?, held_until=? "
                        "WHERE id IN (?)",
                        {userId, currentTime() + 600, seatIds});
            txn.commit();
            return; // done!

        } catch (const SerializationError&) {
            if (attempt == maxRetries - 1) throw;
            std::this_thread::sleep_for(std::chrono::milliseconds(50 * (attempt + 1)));
        }
    }
}
```

### Isolation Level Comparison

| Level | Prevents Double-Booking? | Prevents Write Skew? | Performance |
|-------|--------------------------|----------------------|-------------|
| READ COMMITTED | ❌ (needs locks) | ❌ | ⚡ Fastest |
| REPEATABLE READ | ✅ (same rows) | ❌ (different rows) | 🔶 Medium |
| SERIALIZABLE | ✅ | ✅ | 🐢 Slowest |

---

## 💀 Fix 4: Understanding and Preventing Deadlocks

### What is a Deadlock?

Two people are stuck waiting for each other — forever.

```
User A holds lock on seat 200, wants lock on seat 300
User B holds lock on seat 300, wants lock on seat 200
→ Both wait forever → Deadlock!
```

PostgreSQL detects this after ~1 second and kills one transaction with `ERROR: deadlock detected`.

### How Deadlocks Happen in Production

```typescript
// ❌ DANGEROUS — No guaranteed lock order
async function reserveUnsafe(seatIds: number[], userId: number): Promise<void> {
  await db.transaction(async (trx) => {
    // Without ORDER BY, PostgreSQL picks order based on internal storage
    // After a VACUUM, this order can CHANGE — causing deadlocks in prod!
    const seats = await trx("seats")
      .whereIn("id", seatIds)
      .forUpdate() // locks in unpredictable order
      .select("*");

    // ...
  });
}
```

```
User A wants [201, 205] → locks 201 first (heap order)
User B wants [205, 201] → locks 205 first (different heap order after VACUUM)

A is waiting for 205 (held by B)
B is waiting for 201 (held by A)
→ DEADLOCK 💥
```

### The Fix: Always Lock in Sorted Order

```typescript
// ✅ SAFE — Deterministic lock order prevents deadlocks
async function reserveSafe(seatIds: number[], userId: number): Promise<void> {
  // Sort IDs ascending — EVERY transaction locks in the same order
  const sortedIds = [...new Set(seatIds.map(Number))].sort((a, b) => a - b);

  await db.transaction(async (trx) => {
    const seats = await trx("seats")
      .whereIn("id", sortedIds)
      .orderBy("id", "asc") // ← Guarantees lock order
      .forUpdate()
      .select("*");

    // Now User A: locks 201 → locks 205
    //     User B: locks 201 → waits for A → locks 205 when A releases
    // No circular wait → No deadlock ✅
  });
}
```

```cpp
// C++ — same pattern
void reserveSafe(std::vector<int> seatIds, int userId, DBConnection& db) {
    // Remove duplicates and sort
    std::sort(seatIds.begin(), seatIds.end());
    seatIds.erase(std::unique(seatIds.begin(), seatIds.end()), seatIds.end());

    auto txn = db.beginTransaction();

    try {
        // ORDER BY id ensures locks always acquired in same sequence
        auto seats = txn.query(
            "SELECT * FROM seats WHERE id = ANY(?) "
            "ORDER BY id FOR NO KEY UPDATE",
            {seatIds}
        );

        for (const auto& seat : seats) {
            if (seat["status"] != "available") {
                throw std::runtime_error("Seat unavailable: " + seat["id"]);
            }
        }

        auto expiry = currentTime() + 600; // 10 minutes
        txn.execute(
            "UPDATE seats SET status='reserved', held_by=?, held_until=? "
            "WHERE id = ANY(?)",
            {userId, expiry, seatIds}
        );

        txn.commit();
    } catch (...) {
        txn.rollback();
        throw;
    }
}
```

### FOR NO KEY UPDATE vs FOR UPDATE

This is a subtle but important detail:

```typescript
// FOR UPDATE (stronger) — blocks even foreign key INSERTs on child tables
await trx("seats").whereIn("id", ids).orderBy("id").forUpdate();

// FOR NO KEY UPDATE (weaker, but usually correct) — allows FK inserts
// Use this when you're NOT modifying the primary key column
await trx("seats")
  .whereIn("id", ids)
  .orderBy("id")
  .forUpdate({ skipLocked: false, noKeyUpdate: true }); 
  // Raw SQL: SELECT ... FOR NO KEY UPDATE
```

**Why does it matter?** When you insert into a `booking_seats` table that references `seats.id`, PostgreSQL silently acquires a `FOR KEY SHARE` lock on the seat row. `FOR UPDATE` conflicts with this, causing unexpected blocking. `FOR NO KEY UPDATE` does not — so inserts into child tables remain fast.

---

## 🏭 Fix 5: Production-Ready Complete Solution

```typescript
// ✅ Full production implementation
const HOLD_DURATION_MS = 10 * 60 * 1000; // 10 minutes
const MAX_SEATS = 6;

interface ReservationResult {
  seatIds: number[];
  userId: number;
  status: "reserved";
  expiresAt: string;
}

// ── RESERVE: Two-phase hold ──────────────────────────────────────────────────
async function reserve(
  seatIds: number[],
  userId: number
): Promise<ReservationResult> {
  // Validate input
  if (seatIds.length === 0) throw new Error("Select at least one seat");
  if (seatIds.length > MAX_SEATS)
    throw new Error(`Max ${MAX_SEATS} seats per reservation`);

  const sortedIds = [...new Set(seatIds.map(Number))].sort((a, b) => a - b);

  return await db.transaction(async (trx) => {
    // 1. Lock rows in sorted order (prevents deadlocks)
    const seats = await trx("seats")
      .whereIn("id", sortedIds)
      .orderBy("id", "asc")
      .forUpdate() // SELECT ... FOR NO KEY UPDATE in raw SQL
      .select("id", "status");

    // 2. Ensure all requested seats exist
    if (seats.length !== sortedIds.length) {
      const foundIds = seats.map((s: any) => s.id);
      const missing = sortedIds.filter((id) => !foundIds.includes(id));
      throw new Error(`Seats not found: ${missing.join(", ")}`);
    }

    // 3. All-or-nothing availability check
    const unavailable = seats.filter((s: any) => s.status !== "available");
    if (unavailable.length > 0) {
      throw new Error(
        `Seats taken: ${unavailable.map((s: any) => s.id).join(", ")}`
      );
    }

    // 4. Single batch update — lock held for minimum time
    const expiresAt = new Date(Date.now() + HOLD_DURATION_MS);
    await trx("seats").whereIn("id", sortedIds).update({
      status: "reserved",
      held_by: userId,
      held_until: expiresAt,
    });

    return { seatIds: sortedIds, userId, status: "reserved", expiresAt: expiresAt.toISOString() };
  });
}

// ── CONFIRM: Convert hold to booking after payment ───────────────────────────
async function confirm(
  seatIds: number[],
  userId: number,
  paymentReference: string
): Promise<{ bookingId: number }> {
  const sortedIds = [...new Set(seatIds.map(Number))].sort((a, b) => a - b);

  return await db.transaction(async (trx) => {
    // Lock the user's specific reserved seats
    const seats = await trx("seats")
      .whereIn("id", sortedIds)
      .where({ status: "reserved", held_by: userId })
      .orderBy("id", "asc")
      .forUpdate()
      .select("id", "held_until", "event_id");

    if (seats.length !== sortedIds.length) {
      throw new Error("Some seats are no longer reserved by you");
    }

    // Check for expiry
    const now = new Date();
    const expired = seats.filter((s: any) => new Date(s.held_until) < now);
    if (expired.length > 0) {
      throw new Error(`Reservation expired for seats: ${expired.map((s: any) => s.id).join(", ")}`);
    }

    // Create booking record
    const [booking] = await trx("bookings")
      .insert({
        user_id: userId,
        event_id: seats[0].event_id,
        payment_reference: paymentReference,
        confirmed_at: now,
      })
      .returning("id");

    // Mark seats as permanently booked
    await trx("seats").whereIn("id", sortedIds).update({
      status: "booked",
      held_until: null,
    });

    // Link seats to booking
    await trx("booking_seats").insert(
      sortedIds.map((seatId) => ({ booking_id: booking.id, seat_id: seatId }))
    );

    return { bookingId: booking.id };
  });
}

// ── EXPIRE: Background job — runs every 30 seconds ───────────────────────────
async function expireStaleReservations(): Promise<number> {
  // No locking needed — just a WHERE clause update
  // If a confirm() runs simultaneously, its pessimistic lock serializes correctly
  const count = await db("seats")
    .where({ status: "reserved" })
    .where("held_until", "<", new Date())
    .update({
      status: "available",
      held_by: null,
      held_until: null,
    });

  console.log(`[Sweeper] Released ${count} expired seat holds`);
  return count;
}
```

```cpp
// C++ production-ready structure
class ReservationService {
public:
    struct ReservationResult {
        std::vector<int> seatIds;
        int userId;
        std::string status;
        std::string expiresAt;
    };

    ReservationResult reserve(std::vector<int> seatIds, int userId) {
        if (seatIds.empty()) throw std::invalid_argument("Select at least one seat");
        if (seatIds.size() > MAX_SEATS) throw std::invalid_argument("Too many seats");

        // Sort and deduplicate
        std::sort(seatIds.begin(), seatIds.end());
        seatIds.erase(std::unique(seatIds.begin(), seatIds.end()), seatIds.end());

        auto txn = db_.beginTransaction();
        try {
            // Lock in sorted order
            auto seats = txn.query(
                "SELECT id, status FROM seats WHERE id = ANY(?) "
                "ORDER BY id FOR NO KEY UPDATE",
                {seatIds}
            );

            if ((int)seats.size() != (int)seatIds.size()) {
                throw std::runtime_error("Some seats not found");
            }

            for (const auto& seat : seats) {
                if (seat.at("status") != "available") {
                    throw std::runtime_error("Seat " + seat.at("id") + " is unavailable");
                }
            }

            auto expiresAt = currentTime() + HOLD_DURATION_SECONDS;
            txn.execute(
                "UPDATE seats SET status='reserved', held_by=?, held_until=? "
                "WHERE id = ANY(?)",
                {userId, expiresAt, seatIds}
            );

            txn.commit();
            return {seatIds, userId, "reserved", toISOString(expiresAt)};

        } catch (...) {
            txn.rollback();
            throw;
        }
    }

    int expireStaleReservations() {
        // No locking needed — simple conditional UPDATE
        return db_.execute(
            "UPDATE seats SET status='available', held_by=NULL, held_until=NULL "
            "WHERE status='reserved' AND held_until < NOW()"
        );
    }

private:
    DBConnection db_;
    static constexpr int MAX_SEATS = 6;
    static constexpr int HOLD_DURATION_SECONDS = 600; // 10 minutes
};
```

---

## 🗺️ Seat State Machine

Every seat moves through exactly these states — never backwards, never skipping:

```
available ──reserve()──▶ reserved ──confirm()──▶ booked
    ▲                        │
    └──── expiry job ────────┘
          (held_until < now)
```

- **available** → nobody has it
- **reserved** → someone is holding it for 10 minutes while they pay
- **booked** → payment done, seat is permanently taken

---

## 🏗️ System Architecture (Big Picture)

```
300,000 users
      │
      ▼
[Rate Limiter]          ← Redis: 2 tries / 10s per user
      │
      ▼
[Load Balancer]
      │
      ▼
[API Servers]
      │
      ├──── GET /seats ────▶ [Read Replica] (stale by ~100ms, that's OK for browsing)
      │
      └──── POST /reserve ─▶ [PgBouncer] ─▶ [Primary PostgreSQL]
                                                    │
                                              [Seat Locks]
                                                    │
                                            [Background Job]
                                          (expire every 30s)
```

**Key decisions:**

| Decision | Why |
|----------|-----|
| Reads go to replica | 90% less load on primary; slight staleness is fine for seat map browsing |
| Rate limiting in Redis | Stop bots before they reach the database |
| PgBouncer connection pool | 500 shared DB connections instead of 3,000 direct ones |
| No external calls inside transaction | Payment happens BEFORE confirm() — keeps lock time under 10ms |
| Background sweeper every 30s | Releases expired holds using the partial index — fast even at scale |

---

## 📋 Key Takeaways — Cheat Sheet

| Concept | Simple Explanation |
|---------|-------------------|
| `transaction do` | Groups writes together — does NOT prevent others from reading |
| `SELECT FOR UPDATE` | "I'm editing this — hands off" until transaction ends |
| `ORDER BY id` before locking | Makes every transaction lock in same order → no deadlocks |
| `FOR NO KEY UPDATE` | Weaker lock that doesn't block child table inserts |
| Optimistic locking | "I'll try, and fail fast if someone beat me" |
| READ COMMITTED | Default — each statement sees fresh data (risky for multi-step logic) |
| REPEATABLE READ | One frozen snapshot — still misses write skew on different rows |
| SERIALIZABLE | Full protection — detects all conflicts but needs retry logic |
| Atomic `UPDATE WHERE` | Fastest pattern — read + write happen in one indivisible step |
| Keep transactions short | Every millisecond you hold a lock, others are waiting |

---

> 💡 **Golden Rule:** The lock window is your throughput ceiling. Lock late, check fast, write once, release immediately.



> Reference : https://freedium-mirror.cfd/https://medium.com/womenintechnology/building-a-seat-reservation-system-deadlock-avoidance-and-transaction-isolation-levels-cad7186eb589

# Riders (Actor)

| Constraint | Type | Justification |
|---|---|---|
| RiderID PRIMARY KEY | PK | Surrogate key. name is neither unique nor stable, so it cannot identify a rider. |
| RiderID NOT NULL | Domain | Primary keys cannot be null. |
| name NOT NULL | Domain | A rider without a name is meaningless. |
| name VARCHAR(80) | Domain | Open text domain; 80 characters is ample for a personal name. |

# Drivers (Producer)

| Constraint | Type | Justification |
|---|---|---|
| DriverID PRIMARY KEY | PK | Surrogate key. no natural attribute is guaranteed unique and stable. |
| DriverID NOT NULL | Domain | Primary keys cannot be null. |
| name NOT NULL | Domain | A driver without a name is meaningless. |
| name VARCHAR(80) | Domain | Open text domain, same as riders. |
| isActive NOT NULL | Domain | Every driver must have a definite dispatch state. |
| isActive DEFAULT TRUE | Domain | New drivers are dispatch-eligible by default. |
| isActive BOOLEAN | Domain | Closed two-value domain: {true, false}. |
| rating NUMERIC(2,1) | Domain | One decimal place, sufficient precision for a 1–5 rating. |

# Trips (Event)

| Constraint | Type | Justification |
|---|---|---|
| TripID PRIMARY KEY | PK | Surrogate key, no natural key exists, and the same rider/driver pair can recur. |
| TripID NOT NULL | Domain | Primary keys cannot be null. |
| TripID BIGINT | Domain | The fact table is expected to exceed the INTEGER range over the platform's lifetime. |
| RiderID NOT NULL | Domain | Every trip must have a rider. |
| DriverID NOT NULL | Domain | Every completed trip must have a driver. |
| RiderID FOREIGN KEY → Riders.RiderID | FK | A trip's rider must be a real rider. |
| RiderID ON DELETE RESTRICT | FK | Trip history is a financial record; deleting a rider must not silently erase their trips. |
| RiderID ON UPDATE NO ACTION | FK | Surrogate keys never change, so this never fires. |
| DriverID FOREIGN KEY → Drivers.DriverID | FK | A trip's driver must be a real driver. |
| DriverID ON DELETE RESTRICT | FK | Same reasoning as rider; drivers are soft-deleted via isActive, so RESTRICT is the safety net. |
| DriverID ON UPDATE NO ACTION | FK | Surrogate keys never change. |
| fareAmount NOT NULL | Domain | Every trip must record what was charged. |
| fareAmount NUMERIC(10,2) | Domain | Exact decimal for money — never FLOAT. Eight digits before the decimal, two after. |
| timeBooked NOT NULL | Domain | Every trip has a booking time. |
| timeBooked TIMESTAMPTZ | Domain | Timezone-aware, so cross-region trips compare correctly. |

# DriverBadges (Catalog)

| Constraint | Type | Justification |
|---|---|---|
| DriverBadgeID PRIMARY KEY | PK | Surrogate key,protects against a badge being renamed without breaking references. |
| DriverBadgeID NOT NULL | Domain | Primary keys cannot be null. |
| name NOT NULL | Domain | A badge without a name is meaningless. |
| name UNIQUE | Unique | Badge names are a controlled vocabulary, two badges cannot share a name. |
| name VARCHAR(50) | Domain | Short text domain, sufficient for badge labels such as five_star. |

# DriverBadgeAwards (Junction)

| Constraint | Type | Justification |
|---|---|---|
| (DriverID, DriverBadgeID) PRIMARY KEY | PK | Composite key, the pair is the natural identity of an award and prevents the same badge being awarded twice to the same driver. |
| DriverID NOT NULL | Domain | Part of the primary key; cannot be null. |
| DriverBadgeID NOT NULL | Domain | Part of the primary key; cannot be null. |
| DriverID FOREIGN KEY → Drivers.DriverID | FK | An award must reference a real driver. |
| DriverID ON DELETE RESTRICT | FK | Badge awards are historical evidence of driver quality; they must survive any driver deletion. |
| DriverBadgeID FOREIGN KEY → DriverBadges.DriverBadgeID | FK | An award must reference a real badge. |
| DriverBadgeID ON DELETE RESTRICT | FK | RESTRICT preserves award history when a badge is retired. |
| awardedAt NOT NULL | Domain | An award with no date cannot support time-based analysis. |
| awardedAt TIMESTAMPTZ | Domain | Timezone-aware timestamp. |
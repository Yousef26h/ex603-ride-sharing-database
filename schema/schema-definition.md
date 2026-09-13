The schema follows the five-role structure.


| Role | Table | Purpose |
|---|---|---|
| **Actor** | `riders` | The user requesting and paying for a trip. |
| **Producer** | `drivers` | The supply-side entity being acted upon; carries an activity flag and a rating used for filtering. |
| **Event** | `trips` | The high-volume fact table. One row per trip, FK to `riders` and `drivers`, a timestamp, and `fare_amount`. |
| **Catalog** | `driver_badges` | Descriptive dimension classifying drivers (e.g. `five_star`, `ev_fleet`). |
| **Junction** | `driver_badge_awards` | Many-to-many link between `drivers` and `driver_badges`, with a composite primary key. |



1. Riders (Actor): The user requesting and paying for a trip. attributes:
   1. RiderID: Surrogate integer key. INTEGER (not expecting huge number of riders), NOT NULL, UNIQUE, auto-generated. Never reused.
   2. name: VARCHAR(80), NOT NULL. No format constraint, names are culturally variable.
2. Drivers (Producer): The supply-side entity being acted upon; carries an activity flag and a rating used for filtering. attributes:
   1. DriverID: Surrogate integer key. INTEGER, NOT NULL, UNIQUE, auto-generated.
   2. name: VARCHAR(80), NOT NULL. No format constraint, names are culturally variable.
   3. isActive: Boolean. BOOLEAN, NOT NULL, default TRUE. Closed two-value domain: {true, false}.
   4. rating: NUMERIC(2,1). Closed numeric range: [1.0, 5.0] with one decimal place, new drivers have null rating value.
3. Trips (Event): The high-volume fact table. One row per trip, FK to riders and drivers, a timestamp, and fare_amount. attributes:
   1. TripID: Surrogate key. BIGINT, NOT NULL, UNIQUE, auto-generated. Chosen over INTEGER because the fact table is expected to have large number of rows over the platform's lifetime.
   2. RiderID: Foreign key → Riders.RiderID. Same type as parent (INTEGER), NOT NULL (every trip must have a rider).
   3. DriverID: Foreign key → Drivers.DriverID. Same type as parent (INTEGER), NOT NULL (every completed trip has a driver).
   4. fareAmount: NUMERIC(10,2), NOT NULL. Closed numeric range: >= 0.00 (a fare cannot be negative). Upper bound is open but precision is fixed at 2 decimal places.
   5. timeBooked: TIMESTAMP, NOT NULL. Domain: valid timestamps.
4. DriverBadges (Catalog): Descriptive dimension classifying drivers (e.g. five_star, ev_fleet). attributes:
   1. DriverBadgeID: Surrogate integer key. INTEGER, NOT NULL, UNIQUE, auto-generated.
   2. name: VARCHAR(50), NOT NULL, UNIQUE. This one is a candidate key, the catalog is a controlled vocabulary, so two badges can't share a name.
5. DriverBadgeAwards (Junction): Many-to-many link between drivers and driver_badges, with a composite primary key. attributes:
   1. DriverID: Foreign key → Drivers.DriverID. NOT NULL.
   2. DriverBadgeID: Foreign key → DriverBadges.DriverBadgeID. NOT NULL.
   
   PRIMARY KEY (DriverID, DriverBadgeID) — the pair is the key, so a driver can't be awarded the same badge twice. Each column alone is not unique.
   3. awardedAt: TIMESTAMP, NOT NULL. Domain: valid timestamps. When the badge was awarded.

# 🏎️ Parking System Workflow
> **This README provides a step-by-step breakdown of the Comic-Con India Multi-Zone Parking System design. It explains the flow of data from vehicle entry to exit, the reasoning behind each structural choice, and a practical example to illustrate how the system functions.**

### Step 1: Vehicle Registration & Identification
When a vehicle arrives at the gate, the system first checks if this vehicle (License Plate) exists in the database.

- Reasoning: Separating `Vehicles` from `parking_sessions` allows the system to track repeat visitors. A car that comes on Day 1 for an anime screening can return on Day 3 for a creator meetup without duplicating static data (color, model).

- Example: A "Tesla Model 3" (License: `RJ-01-CV-1234`) enters. The system checks the Vehicles table; if it's new, it creates a record and flags it as `EV` under `vechicle_categorie`.

### Step 2: Access Category & Spot Matching
The system checks the user's " Access Category" (e.g., VIP, Cosplayer with Props, Staff).

- Reasoning: Not all spots are equal. A Cosplayer with large props needs a wider spot; an EV needs a charging point. The `Packing_spots_categories` table acts as a filter to ensure the right person gets the right spot.

- Example: The driver is a "VIP Guest." The system filters the `parking_spots` table for `is_available = TRUE` and `packing_category = 'VIP'.`

### Step 3: Session Initiation & Ticket Generation
Once a spot is found (e.g., Level 2, Zone B, Spot 42), a `parking_sessions` begins and a `parking_tickets` is issued.

- Reasoning: The `parking_tickets` is the physical proof for the user, while the `parking_sessions` is the internal log. Splitting them allows for "Digital Tickets" or "Lost Ticket" recovery by linking back to the `parking_session_id`.

- Example: `Session_889` is created with `Entry_Time: 10:00 AM`. `Ticket_4552` is printed with a QR code linking to this session.

### Step 4: Real-Time Availability Update
The assigned `parking_spots` is marked as `Occupied`.

Reasoning: To prevent double-booking in a high-traffic event, the `is_occupied` flag in the `parking_spots` table must be toggled immediately upon session start.

Example: Spot `L2-B-42` status changes from `1` (Available) to `0` (Occupied).

### Step 5: Session Termination & Fee Calculation
When the vehicle leaves, the system records the `Exit_Time`.

- Reasoning: Different vehicle types have different rates (e.g., SUVs pay more than Bikes). By joining `parking_sessions` → `Vehicle` → `vechicles_categories`, the system pulls the correct hourly rate and calculates the duration.

- Example: The Tesla leaves at 2:00 PM. Duration = 4 hours. Rate for `EV` = ₹50/hr. Total = ₹200.

### Step 6: Payment Recording
The final step is the creation of a `Payment` record linked to the session.

- Reasoning: Separating `Payment` from `parking_sessions` allows the system to track payment methods (Cash, UPI, Card) and handle "Pending" vs "Completed" statuses for accounting audits.

- Example: `Payment_992` is recorded for ₹200 via `UPI`, and `Session_889` is marked as `Closed`.

### 📂 Entity-Relationship Logic (The "Why")
| Feature | Design Choice | Reasoning |
|----------|----------------|-----------|
| Multi-Day Visits | `Vehicle` 1:N `parking_sessions` | One vehicle ID can appear in many session records over the event duration. |
| Zone Tracking | `pzone_zones` 1:N `parking_spots` | Organizes the venue into levels and sections for easier navigation and management. |
| Reserved Areas | `parking_spots_Categories` table | Allows the venue to lock specific spots for VIPs or Staff without hard-coding rules into the Spot table. |
| Pricing | `vechicles_categories` table | Centralizes pricing logic. If the venue decides to increase SUV rates on Day 2, they only change one row. |
| Current Occupancy | `NULL` Exit Timestamps | Any session where `exit_time IS NULL` tells management exactly who is still inside. |

### 📝 Example Scenario: The Cosplayer Arrival
1. Entry: A bike arrives. The rider is a Cosplayer with a giant sword prop.
2. Lookup: System identifies the vehicle as a Bike.
3. Filtering: System looks for a spot that is both Bike-friendly AND in the Cosplayer Reserved zone (near the prop-check station).
4.  Assignment: Assigned to Zone A, Level 1, Spot 05.
5. Tracking: ParkingSession starts at 09:30 AM. 
Exit: Rider leaves at 06:30 PM. System sees the Bike rate is ₹20/hr.
6. Calculation: 9 hours × ₹20 = ₹180.
7. Completion: Payment is cleared; Spot 05 becomes Available for the next visitor.

## 🚀 How to Use

1. Open the provided ER Diagram image alongside this documentation.
2. Follow the Primary Key (PK) and Foreign Key (FK) lines to see how data flows from a customer's DM to a completed delivery.
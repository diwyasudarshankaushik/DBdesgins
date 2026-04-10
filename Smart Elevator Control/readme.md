# Smart Elevator Control Platform
### Database Design Philosophy
The ER architecture follows **three core principles**:

1. **Decoupling Configuration from State:** Static data (Building/Floor/Elevator specs) is separated from dynamic data (Current Position/Status).
2. **Auditability:** Every ride and maintenance event is logged in historical tables rather than overwriting current state.
3. **Many-to-Many Flexibility:** The design supports complex routing where multiple elevators can serve overlapping sets of floors.

### Entity Relationship Summary

1. Infrastructure (Static)
- Buildings: The root entity representing a physical site.
- Floors: Individual levels within a building.
- Elevator_Shafts: Physical voids where elevators reside.
- Elevators: The specific lift units with capacity and model details.

2. Operational (Dynamic)
- Elevator_Status: Tracks real-time telemetry (current floor, direction, and operational state like Moving, Idle, or Out of Service).
- Floor_Requests: Captures user intent (source floor, destination, and time).
- Ride_Assignments: The "Brain" of the system—links a specific request to the most efficient elevator.

3. Historical (Logs)
- Ride_Logs: Stores completed trips for traffic analysis and energy optimization.
- Maintenance_Logs: Records the service history, technician notes, and downtime for every elevator unit.
### Example Workflow: A Day in the Life of a Request

1. The Request:
- A user on Floor 10 of "Tower A" presses the "Down" button.
- Table Update: A new entry in Floor_Requests is created with source_floor_id: 10 and direction: 'Down'.

2. The Assignment:
- The system identifies Elevator #4 is the closest idle unit.

- Table Update: A record is created in Ride_Assignments linking Request_ID to Elevator_ID: 4.

3. The Status Update:
- As the elevator moves from Floor 15 down to 10.

- Table Update: Elevator_Status is updated in real-time as the lift passes floors.

4. The Completion:
- The user is dropped off at the Ground Floor.

- Table Update: The request status is set to Completed. A new row is added to Ride_Logs capturing the total travel time and distance for analytics.

### How to View the Diagram
1. Open the provided image file (LiftGrid_ERD.png) or PDF.
2. Follow the Crow’s Foot notation to understand the cardinality ($1:N$ or $M:N$) between entities.
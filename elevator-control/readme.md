# Elevator Management System
This is an elevator management system for malls, offices, hospitals and high rise buildings. 
It handles:
- Multiple elevator per shaft
- Zone based (low level, mid level, high level) elevator dispatching
- Service area or wind based routing
- Different elevator types (public, staff, goods, private, VIP, emergency)
- Real time tracking of elevator
- RIde lifecycle
- Maintainance 
---

## Decisions Made During Design

- Elevator requests are centralized, user doesn’t choose lift manually
- Elevator selection is handled by a dispatcher system
- Zones + service areas prevent inefficient long-distance elevator assignment
- Elevators can belong to multiple zones and service areas
- Request lifecycle is tracked separately from ride execution
- Real-time state (elevator_status) is separated from historical logs (ride_log)
- Maintenance blocks elevator availability
- Many-to-many relationships handled via junction tables
- Composite unique constraints prevent duplicate mappings
---

## ER Diagram Tables

- **building**
    - Represents a physical building

- **floor**
    - Represents floors inside a building
    - Each floor belongs to one building

- **zone**
    - vertical division according to floor
    - Defines logical grouping of floors (e.g., Low Rise, High Rise)

- **zone_floor_map**
    - Junction table mapping floors to zones
    - Supports many-to-many:
        - A zone can span multiple floors
        - A floor can belong to multiple zones

- **service_area**
    - horizontal division ie. same floor
    - Represents physical sections of building (e.g., Left Wing, Right Wing, Atrium)

- **floor_service_area_map**
    - Maps floors to service areas

- **elevator_service_area_map**
    - Maps elevators to service areas

This enables:
- Preventing elevators from far wings responding unnecessarily
- Better locality-based dispatch

- **shaft**
    - Represents a vertical shaft in a building
    - One shaft can contain multiple elevators

- **elevator**
    - Represents individual lift
    - ENUM (  public, staff, goods, private, vip, emergency)

- **elevator_floor_map**
    - Defines which floors an elevator serves

- **elevator_zone_map**
    - Defines which zones an elevator belongs to

Enables:
- Zone-based filtering
- Floor reachability validation

- **floor_request**
    - Represents a user pressing a lift button
    - ENUM (pending, assigned, in_progress, completed, cancelled, timed_out)

User elevator gets assigned by system

- **elevator_assignment**
    - Maps request to elevator (1:1).
    - One request = one elevator
    - Ensures controlled dispatch

- **ride_log**
    - Tracks actual ride execution
    - ENUM (completed, failed, interrupted, emergency_stop)

- **elevator_status**
    - Stores live state of elevator
    - ENUM (idle, moving, loading, unloading, maintainance, out_of_service, emergency)
    - Used for diapatch decision

- **maintenance**
    - Tracks repain and servicing
    - ENUM maintainance type(scheduled, breakdown,inspection, emergency repair)
    - ENUM maintainance_status (schedule, ongoing, completed, cancelled)
    - prevent faulty elevator from getting assigned 

- **maintainance_log**
    - Tracks maintainance activity
    - Enables audit
    - ENUM action (inspection_started, repair_started, part_replaced, testing, paused, resumed, completed)
    - Enum status(schedule, ongoing, completed, cancelled)

- **technician**
    - Stores technician details responsible for maintenance


### Relationships

1-M
- building -> floor
- building -> zone
- building -> shaft
- building -> elevator
- building -> service_area
- shaft -> elevator
- floor -> floor_request (source & destination)
- floor -> ride_log
- floor -> elevator_status
- elevator -> elevator_status
- elevator -> maintenance
- elevator -> ride_log
- elevator -> elevator_assignment
- floor_request -> elevator_assignment
- floor_request -> ride_log
- maintenance -> maintenance_log
- elevator -> maintenance_log

⸻

M-M (via junction tables)
- zone <-> floor -> zone_floor_map
- elevator <-> zone -> elevator_zone_map
- elevator <-> floor -> elevator_floor_map
- elevator <-> service_area -> elevator_service_area_map
- floor <-> service_area -> floor_service_area_map
---
## Flow

Request
```
user presses button
↓
system creates floor_request
↓
status = pending
```

Elevator assignment
```
dispatcher finds best elevator based on:
- same service_area
- same zone
- closest distance
- idle/moving state
- usage_type compatibility
↓
create elevator_assignment
↓
status = assigned
```

Ride
```
elevator reaches user current floor
↓
status = in_progress
↓
passenger boards
↓
elevator moves to destination
↓
ride_log created
↓
status = completed
```


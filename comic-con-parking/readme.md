# Comic Con Parking

This is a parking model dedicatedly designed for the comic con event as it as sports according to the type of guest the events host. It caters:
- Multiple vehicle type
- Reserved spots for VIP, cosplayer, etc
- Allocate spot dynamically
- Session based parking 
- Ticket and payment handling

## Decisions Made During Design

- vehicle can visit multiple times; handles using sessions
- parking slot reusable; session based allocation
- active vehicles = exit_time IS NULL
- dynamic pricing via pricing_rule
- ticket and session separation
- junction table used for many to many relations
- EV is modeled as vehicle category EV charging as spot

## ER Diagram Tables

1. Vehicle and Vehicle Category

- **vehicle_category**
Defines vehicle type and size (small: for 2 wheeler, compact for hatchback or mini suv, large for suv or mini van) allowing to add metadata.
The vehicle can be Bike, Car, SUV or **EV**

- **vehicle**
Stores data about vehicle accessing parking lot. 
One vehicle will have one record as licence_numver is unique

  - vehicle ca visit multiple times which is handles with parking_session
  - licence number is also stored

2. Access Control System

- **access_category**
Determine is the user is VIP, Staff, Exhibitor, Cosplayer or Visitor

- **vehicle_access** 
This is the junction table. A vehicle can have multiple purpose. Example: It can be for a exhibitor and other time can bring in cosplayer. 
This creates many-to-many relationship

  - this helps in flexibility of roles
  - enforce permission based parking

3. Parking Model

- **parking_zone**
This represents the layout of the lot determining zone or level. One zone has multiple parking spots

- **spot_category**
This denotes if the spot is meant for a VIP, exhibitor, staff, EV charging, cosplayer
It includes metadata for reserved and priority 

- **parking_spot** 
Represent the parking slot where:
  - zone_id : zone it belongs to
  - spot_category_id : what type of spot it is
  - spot_size ENUM : compatibility with vehicle
  - is_active : availability control

  This give session based control to the spots

4. Spot Access Control

- **spot_category_access**
This is a junction table which determine the sports a user can access
This prevent invalid allocations

5. Ticket System

- **parking_ticket**
The ticket that gets issued once a user enters the lot
 - ticket_id : internal ID
 - ticket_number : user-facing ID
 - vehicle_number : snapshot of vehicle number
 - issued_at

6. Parking Session

- **parking_session**
The vehicle may enter multiple times, spots are reused, and tracking of entry and exit is a key aspect.
This helps to calculate the duration a vehicle use parking slot, and make the slot reusable 

7. Payments System

- **payment**
Tracks payment which is linked to session via parking_session and contains metadata for payment

8. Dynamic Parking Pricing

- **pricing_rule**
This enables flexible pricing depending on vehicle type, spot (VIP or EV charging), and time based pricing (can change on weekends)
  - vehicle_category_id : type of vehicle
  - spot_category_id : type of parking spot
  - base_price : fixed entry charge
  - price_per_hour : variable cost
  - valid_from, valid_to : time range for rule

Relationship:

1-M
  - vehicle_category -> vehicle
  - vehicle -> parking_session
  - parking_zone -> parking_spot
  - spot_category -> parking_spot
  - parking_spot -> parking_session
  - parking_session -> payment
  - vehicle_category -> pricing_rule
  - spot_category -> pricing_rule

M-M
  - vehicle <-> access_category -> vehicle_access
  - spot_category <-> access_category -> spot_category_access

### Flow:

Entry:
```
vehicle enters
↓
identify category and access
↓
assign valid parking spot
↓
generate ticket
↓
create session with entry time
```
Exitt:
```
fetch session
↓
calculate duration
↓
apply pricing rule
↓
generate payment
↓
session is completed
```


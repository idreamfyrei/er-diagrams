# Clinic Appointment and Diagnosis System

This is a clinic appointment booking system which delegates:
- 	Doctors & Patients
- 	Appointments & Consultations
- 	Doctor Availability & Time Slots
- 	Diagnostic Tests & Reports
-   Payments & Reviews

## Tables and Their Purpose

1. **users**: store all platform users with their basic information along with role
    
    Relationship:
-   1–1 with doctor
-   1–1 with patient

2. **doctor**: doctor specific details

    Relationships:
- 1–1 with users
- 1–M with doctor_specialisation
- 1–M with doctor_clinic
- 1–M with doctor_availbility
- 1–M with appointment
- 1–M with consultation
- 1–M with time_slot

3. **patient**: patient specific details

    Relationships:
- 1–1 with users
- 1–M with appointment
- 1–M with consultation
- 1–M with review

4. **specialisation**: list of medical specialization like orthopedic, cardiology.

    Relationships: M–N with doctor via doctor_specialisation

5. **doctor_specialisation**: mapping table for doctors and their specialties

6. **clinic**: stores slinic details

    Relationships:
- 1–M with doctor_clinic
- 1–M with doctor_availbility
- 1–M with time_slot
- 1–M with appointment
- 1–M with consultation
- 1–M with review

7. **review**: patient reviews

    Relationships:
- M–1 to patient
- M–1 to clinic

8. **doctor_clinic**: map doctors to clinic (M:M)

9. **doctor_availbility**: defines doctor's availability through days of week and available hours

    Relationships:
- M–1 to doctor
- M–1 to clinic
- 1–M with time_slot

10. **time_slot**: generate available slots for patients along with capacity of patients per solt

    Relationship:
- M–1 to doctor
- M–1 to clinic
- M–1 to doctor_availbility
- 1–M with appointment

11. **appointment**: booking of a patient

    Relationship:
- M–1 to patient
- M–1 to doctor
- M–1 to clinic
- M–1 to time_slot
- 1–M with appointment_status_log
- 1–1 with consultation
- 1–M with payment

12. **appointment_status_log**: tracks appointment status changes

    Relationships: M–1 to appointment

13. **consultation**: doctor visit

    Relationships:
- 1–1 with appointment
- M–1 to patient
- M–1 to doctor
- M–1 to clinic
- 1–M with prescribed_test
- 1–M with payment

14. **test**: list of diagnostic test

    Relationships: 1–M with prescribed_test

15. **prescribed_test**: test recommended during consultation

    Relationships:
- M–1 to consultation
- M–1 to test
- 1–1 with report
- 1–1 with lab_test_assignment
- 1–M with payment

16. **lab**: diagnostic labs

    Relationship: 1–M with lab_test_assignment

17. **lab_test_assignment**: assigns tests to labs

    Relationship: 
- M–1 to lab
- 1–1 to prescribed_test

18. **report**: store diagnostic results

    Relationship: 1–1 with prescribed_test

19. **payment**: handles all financial transactions

    Relationship: 	
- M–1 to appointment
- M–1 to consultation
- M–1 to prescribed_test
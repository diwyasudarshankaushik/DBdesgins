
# Clinic Operational Flow

> **This documentation serves as a guide to the database architecture for the Clinic Appointment and Diagnostics Platform. It explains the logic behind the workflow—from a patient’s first booking to receiving their diagnostic reports—and why certain design choices were made to ensure scalability.**

## Step 1: Patient Registration & Doctor Discovery
- **Action:** A new patient registers, and the clinic lists doctors categorized by their specialties.
- **Example:** Diwya signs up. He looks for a "Cardiologist" and finds Dr. Manav Sudarshan kaushik.
- **Reasoning:** *Specialization as a separate entity*- Instead of a text field in the `Doctors` table, a separate `Specializations` table allows the clinic to add metadata (like department floor) and prevents typos (e.g., "Cardio" vs "Cardiology").

## Step 2: The Appointment (Intent) 
- **Action:** The patient selects a time slot for a specific doctor.
- **Example:** Diwya books an appointment for Oct 12th at 10:00 AM. The status is set to `Confirmed`.
- **Reasoning:** * Appointment vs. Consultation: Not every appointment happens (no-shows). By keeping `Appointments` separate, we can track "Cancellation Rates" and "No-Show" statistics without cluttering the medical record table.

## Step 3: The Consultation (The Visit)
- **Action:** The doctor meets the patient. This creates a "Visit" or "Consultation" record.
- **Example:** Diwya arrives. Dr.Manav records symptoms (chest pain) and charges a $50 fee.
- **Reasoning:** * Direct Links: The `Consultation/Vists` table links to both `Patient` and `Doctor`. While it can link to an `Appointment`, making that link optional allows for "Walk-in" patients who don't have a prior booking.

## Step 4: Diagnostics Prescription
- **Action:** During the visit, the doctor prescribes one or more tests.
- **Example:** Dr.Manav prescribes an ECG and a Blood Test for diwya.
- **Reasoning:** * Bridge Table (`Prescribed_Tests`): A consultation can have multiple tests. We use a bridge table to link `Consultations/Vists` to the master `Tests` list. This allows us to track the status of each test (e.g., ECG is done, but Blood Test is still pending).

## Step 5: Reporting & Completion
- **Action:** The lab uploads results, which are linked back to the prescription.
- **Example:** The lab uploads a PDF for the ECG. diwya receives a notification.
- **Reasoning:** * Report Linking: Reports are linked to the `Prescribed_Tests_ID`, not just the patient. This ensures that if a patient has had five ECGs in a year, the system knows exactly which doctor visit and which specific order this report belongs to.

## Step 6: Billing & Payment
- **Action:** A bill is generated based on the consultation fees and test costs.
- **Example:** A total bill of $150 ($50 visit + $100 tests) is paid via Credit Card.
- **Reasoning:** * Payment Connection: Payments are tied to the `Consultation`. This covers everything that happened during that specific visit, making it easier for the patient to see a single "Visit Invoice."

## 💡 Key Design Decisions
1. **Scalability:** By normalizing `Specializations` and `Tests`, the clinic can expand to 100+ departments and 1000+ different test types without changing the database structure.

2. **Data Integrity:** Foreign Keys (FK) ensure that a report cannot exist without a valid prescription, and a prescription cannot exist without a doctor's consultation.

3. **Flexibility:** The optional relationship between `Appointment` and `Consultation` handles both scheduled patients and emergency walk-ins seamlessly.
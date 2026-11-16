# Exo 1 – Hospital Management System 🏥

## Task 1 – Basic class diagram

Each actor of the platform becomes a class. The Patient submits problems to the Organizer, and the Organizer consults one or more Doctors.

```mermaid
classDiagram
    class Patient {
        -int patient_id
        +str name
        -str password
        +str email
        +submit_problem(description)
        +pay(amount, method)
        +view_prescription()
    }
    class Organizer {
        -int organizer_id
        +str name
        -str password
        +read_problem()
        +consult_doctor(problem)
        +send_prescription(patient)
        +forward_payment(doctor)
        +issue_credentials(member)
    }
    class Doctor {
        -int doctor_id
        +str name
        -str password
        +str specialty
        +provide_prescription(problem)
        +receive_payment(amount)
    }
    Patient "0..*" --> "1" Organizer : submits problems to
    Organizer "1" --> "1..*" Doctor : consults
```

## Task 2 – Complete class diagram (6 classes)

- A Patient submits 0 or more HealthProblems; a HealthProblem belongs to exactly 1 Patient.
- A Doctor writes Prescriptions; each Prescription answers exactly 1 HealthProblem.
- A Payment pays exactly 1 Prescription and is made by 1 Patient.
- Payment types use **inheritance**: Payment is abstract and each type implements `process()` in its own way.

```mermaid
classDiagram
    class Patient {
        -int patient_id
        +str name
        -str password
        +submit_problem(description)
        +pay(amount, method)
    }
    class Organizer {
        -int organizer_id
        +str name
        -str password
        +consult_doctor(problem)
        +send_prescription(patient)
        +forward_payment(doctor)
        +issue_credentials(member)
    }
    class Doctor {
        -int doctor_id
        +str name
        -str password
        +str specialty
        +provide_prescription(problem)
    }
    class HealthProblem {
        +int problem_id
        +str description
        +date submitted_at
    }
    class Prescription {
        +int prescription_id
        +str content
        +date issued_at
    }
    class Payment {
        <<abstract>>
        +float amount
        +date paid_at
        +process()*
    }
    class Check {
        +str check_number
        +process()
    }
    class Cash {
        +process()
    }
    class CreditCard {
        -str card_number
        +process()
    }
    Patient "1" --> "0..*" HealthProblem : submits
    Organizer "1" --> "0..*" HealthProblem : handles
    Doctor "1" --> "0..*" Prescription : writes
    HealthProblem "1" --> "0..1" Prescription : answered by
    Patient "1" --> "0..*" Payment : makes
    Prescription "1" --> "0..1" Payment : paid by
    Payment "0..*" --> "1" Doctor : forwarded to
    Payment <|-- Check
    Payment <|-- Cash
    Payment <|-- CreditCard
```

### Sequence diagram – one patient submits a problem, receives a prescription, and pays

```mermaid
sequenceDiagram
    actor P as Patient
    participant O as Organizer
    participant D as Doctor
    P->>O: submit_problem(description)
    O->>D: consult_doctor(problem)
    D-->>O: prescription
    O->>P: send_prescription(prescription)
    P->>O: pay(amount, method)
    O->>D: forward_payment(amount)
    O-->>P: confirmation
```

## Task 3 – Advanced

Additions compared with Task 2:
- a **User** superclass for the shared attributes (id, name, password) and `login()` / `logout()`;
- the payment strategy modelled with **polymorphism** (abstract `Payment.process()`);
- **status tracking** for HealthProblem with a `Status` enumeration (pending, in-review, resolved);
- **timestamps** (`submitted_at`, `issued_at`, `paid_at`) and a reference to a **past prescription**.

```mermaid
classDiagram
    class User {
        <<abstract>>
        -int user_id
        +str name
        -str password
        +login()
        +logout()
    }
    class Patient {
        +str email
        +submit_problem(description)
        +pay(amount, method)
        +view_history()
    }
    class Organizer {
        +create_member(name)
        +consult_doctor(problem)
        +send_prescription(patient)
        +forward_payment(doctor)
        +maintain_database()
    }
    class Doctor {
        +str specialty
        +provide_prescription(problem)
    }
    class Status {
        <<enumeration>>
        PENDING
        IN_REVIEW
        RESOLVED
    }
    class HealthProblem {
        +int problem_id
        +str description
        +Status status
        +datetime submitted_at
        +update_status(new_status)
    }
    class Prescription {
        +int prescription_id
        +str content
        +datetime issued_at
    }
    class Payment {
        <<abstract>>
        +float amount
        +datetime paid_at
        +process()*
    }
    class Check {
        +str check_number
        +process()
    }
    class Cash {
        +process()
    }
    class CreditCard {
        -str card_number
        +process()
    }
    User <|-- Patient
    User <|-- Organizer
    User <|-- Doctor
    Patient "1" --> "0..*" HealthProblem : submits
    Organizer "1" --> "0..*" HealthProblem : handles
    HealthProblem --> Status
    Doctor "1" --> "0..*" Prescription : writes
    HealthProblem "1" --> "0..1" Prescription : answered by
    Prescription "0..1" --> "0..1" Prescription : previous prescription
    Patient "1" --> "0..*" Payment : makes
    Prescription "1" --> "0..1" Payment : paid by
    Payment "0..*" --> "1" Doctor : forwarded to
    Payment <|-- Check
    Payment <|-- Cash
    Payment <|-- CreditCard
```

### System sequence diagram 1 – Patient registration (the Organizer creates a new member)

```mermaid
sequenceDiagram
    actor O as Organizer
    participant S as System
    actor P as Patient
    O->>S: create_member(name, email)
    S->>S: generate_id_and_password()
    S-->>O: member_id, password
    O->>P: send_credentials(member_id, password)
    P->>S: login(member_id, password)
    S-->>P: login successful
```

### System sequence diagram 2 – Consultation flow (from problem submission to prescription delivery)

```mermaid
sequenceDiagram
    actor P as Patient
    participant HP as HealthProblem
    actor O as Organizer
    actor D as Doctor
    participant PR as Prescription
    P->>HP: create(description)
    Note over HP: status = PENDING
    O->>HP: update_status(IN_REVIEW)
    O->>D: consult_doctor(problem)
    D->>PR: create(content)
    D-->>O: prescription
    O->>HP: update_status(RESOLVED)
    O->>P: send_prescription(prescription)
```

### System sequence diagram 3 – Payment processing (the patient pays, the organizer forwards to the doctor)

```mermaid
sequenceDiagram
    actor P as Patient
    participant PAY as Payment
    actor O as Organizer
    actor D as Doctor
    P->>PAY: create(amount, method)
    PAY->>PAY: process()
    alt payment accepted
        PAY-->>O: payment confirmed
        O->>D: forward_payment(amount)
        O-->>P: receipt
    else payment refused
        PAY-->>P: error, please try another method
    end
```

## Questions to consider

**How would you handle a patient consulting multiple doctors?**
A patient can have several HealthProblems (Patient 1 → 0..* HealthProblem), and each problem gets a Prescription written by a Doctor. So a patient with several problems can see several doctors. If one single problem needs several doctors, the multiplicity becomes HealthProblem 1 → 0..* Prescription.

**Should prescriptions be reusable? How would you model that?**
Past prescriptions are kept with their date, linked to the patient's previous health problems, so they form a history. A new prescription can reference a past one with the self-association `previous prescription` (0..1), for example to renew a treatment.

**What design pattern fits the payment system?**
The **Strategy** pattern: `Payment` is an abstract strategy with `process()`, and `Check`, `Cash` and `CreditCard` are interchangeable concrete strategies. A new payment method can be added without modifying the existing classes (Open/Closed principle).

**How do you ensure data integrity when payments are processed?**
Validate the payment before processing it (amount greater than 0, valid card or check), keep sensitive data private (encapsulation: `-card_number`, `-password`), only forward the money to the doctor once the payment is confirmed, and record every payment with a timestamp so it can be traced.

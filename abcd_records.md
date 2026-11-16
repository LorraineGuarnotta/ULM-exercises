# Exo 2 – ABCD Records 📼

> Note: Mermaid has no dedicated use case diagram, so use case diagrams are drawn with a flowchart:
> actors are circles, use cases are rounded boxes inside the system boundary, and `include` / `extend` are dotted arrows.

## Level 1

### Task 1.1 – Simple use case diagram

```mermaid
flowchart LR
    M((Member))
    C((Order Processing Clerk))
    subgraph ABCD Records
        UC1([Place Order])
        UC2([Verify Membership])
        UC3([Process Order])
        UC4([Make Payment])
    end
    M --- UC1
    M --- UC4
    C --- UC2
    C --- UC3
```

### Task 1.2 – Basic class diagram

```mermaid
classDiagram
    class Member {
        +int member_id
        +str name
        +str address
        +place_order(items)
        +make_payment(amount)
    }
    class Order {
        +int order_id
        +date order_date
        +str status
        +add_item(item, quantity)
        +total()
    }
    class OrderProcessingClerk {
        +int employee_id
        +str name
        +verify_membership(member)
        +process_order(order)
    }
    Member "1" --> "0..*" Order : places
    OrderProcessingClerk "1" --> "0..*" Order : processes
```

## Level 2

### Task 2.1 – Complete use case diagram

- **include** (always happens): placing an order always verifies membership; processing an order always checks availability and prints the invoice and the shipping list.
- **extend** (only sometimes): a discount applies only to Royal members; ordering unavailable items is only for Royal members; the membership application is only sent when the person is not a member.
- A Royal Member is a kind of Member (actor generalization).

```mermaid
flowchart LR
    M((Member))
    RM((Royal Member))
    NM((Non-Member))
    C((Order Processing Clerk))
    CD((Collection Department Clerk))
    subgraph ABCD Records
        PO([Place Order])
        VM([Verify Membership])
        SMA([Send Membership Application])
        OUI([Order Unavailable Items])
        PR([Process Order])
        VIA([Verify Item Availability])
        AD([Apply Discount])
        PI([Print Invoice])
        PSL([Print Shipping List])
        MP([Make Payment])
    end
    RM -->|is a| M
    M --- PO
    M --- MP
    NM --- PO
    RM --- OUI
    C --- VM
    C --- PR
    CD --- MP
    PO -. include .-> VM
    SMA -. extend .-> VM
    OUI -. extend .-> PO
    PR -. include .-> VIA
    PR -. include .-> PI
    PR -. include .-> PSL
    AD -. extend .-> PR
```

### Task 2.2 – Complete class diagram

- **Member types**: `Member` is abstract, `RoyalMember` (discounts, priority, can order unavailable items) and `RegularMember` inherit from it.
- **Payment types**: `Payment` is abstract with `process()`, implemented by `Cash`, `Check` and `BankDraft`.
- **Item types**: `Item` is abstract, `CD` and `Tape` inherit from it.
- An `Order` is **composed of** one or more `OrderLine`s (filled diamond: the lines don't exist without their order); each line refers to one `Item`.
- Each order produces exactly one `Invoice` and one `ShippingList` (composition), and is saved in the `DailyRecordsFile`.

```mermaid
classDiagram
    class Member {
        <<abstract>>
        +int member_id
        +str name
        +str address
        +place_order(items)
        +discount_rate()*
    }
    class RoyalMember {
        +bool priority
        +discount_rate()
        +order_unavailable_item(item)
    }
    class RegularMember {
        +discount_rate()
    }
    class NonMember {
        +str name
        +str address
        +request_membership()
    }
    class MembershipApplication {
        +int application_id
        +date sent_at
        +str status
    }
    class Order {
        +int order_id
        +date order_date
        +str status
        +add_line(item, quantity)
        +total()
    }
    class OrderLine {
        +int quantity
        +subtotal()
    }
    class Item {
        <<abstract>>
        +int item_id
        +str title
        +float price
        +int stock
        +is_available()
    }
    class CD
    class Tape
    class Invoice {
        +int invoice_id
        +float amount
        +print_invoice()
    }
    class ShippingList {
        +int list_id
        +str address
        +print_list()
    }
    class Payment {
        <<abstract>>
        +float amount
        +date paid_at
        +process()*
    }
    class Cash {
        +process()
    }
    class Check {
        +str check_number
        +process()
    }
    class BankDraft {
        +str draft_number
        +process()
    }
    class OrderProcessingClerk {
        +int employee_id
        +verify_membership(member)
        +process_order(order)
    }
    class CollectionDepartmentClerk {
        +int employee_id
        +collect_payment(payment)
    }
    class DailyRecordsFile {
        +date day
        +save(order)
    }
    Member <|-- RoyalMember
    Member <|-- RegularMember
    Item <|-- CD
    Item <|-- Tape
    Payment <|-- Cash
    Payment <|-- Check
    Payment <|-- BankDraft
    Member "1" --> "0..*" Order : places
    Order "1" *-- "1..*" OrderLine : contains
    OrderLine "0..*" --> "1" Item : refers to
    Order "1" *-- "1" Invoice
    Order "1" *-- "1" ShippingList
    Order "1" --> "0..1" Payment : paid by
    NonMember "1" --> "0..1" MembershipApplication : receives
    OrderProcessingClerk "1" --> "0..*" Order : processes
    CollectionDepartmentClerk "1" --> "0..*" Payment : handles
    DailyRecordsFile "1" o-- "0..*" Order : records
```

### Scenario – a Royal member orders 2 CDs (one unavailable) and pays by check

```mermaid
sequenceDiagram
    actor RM as Royal Member
    actor C as Order Processing Clerk
    participant O as Order
    participant I as Item (CD)
    actor CD as Collection Department Clerk
    RM->>C: place_order([CD1, CD2])
    C->>C: verify_membership(member)
    C->>O: create order
    C->>I: is_available(CD1)
    I-->>C: available
    C->>I: is_available(CD2)
    I-->>C: unavailable
    C->>O: order_unavailable_item(CD2) (Royal members only)
    C->>O: apply discount (Royal member)
    C-->>RM: invoice
    RM->>CD: pay by check
    CD->>CD: process() check payment
    CD-->>RM: payment confirmed
```

## Level 3 – Advanced sequence diagrams

### Complete order processing flow

```mermaid
sequenceDiagram
    actor M as Member
    actor C as Order Processing Clerk
    participant O as Order
    participant I as Item
    participant F as DailyRecordsFile
    M->>C: place_order(items)
    C->>C: verify_membership(member)
    C->>O: create(order)
    loop for each item
        C->>I: is_available()
        I-->>C: yes / no
        C->>O: add_line(item, quantity)
    end
    C->>O: apply discount if Royal member
    C->>O: print_invoice()
    C->>O: print_list() (shipping list)
    C->>F: save(order)
    C-->>M: order confirmed + invoice
```

### Membership verification (success and failure)

```mermaid
sequenceDiagram
    actor P as Person
    actor C as Order Processing Clerk
    participant S as Members database
    P->>C: place_order(items)
    C->>S: find_member(id)
    alt is a member
        S-->>C: member found
        C-->>P: order accepted
    else not a member
        S-->>C: not found
        C->>P: send membership application form
        C-->>P: order refused until membership
    end
```

### Payment processing (all types)

```mermaid
sequenceDiagram
    actor M as Member
    actor CD as Collection Department Clerk
    participant PAY as Payment
    participant O as Order
    M->>CD: pay(amount, method)
    alt Cash
        CD->>PAY: Cash.process()
    else Check
        CD->>PAY: Check.process()
    else Bank Draft
        CD->>PAY: BankDraft.process()
    end
    PAY-->>CD: payment confirmed
    CD->>O: mark_as_paid()
    CD-->>M: receipt
```

### Item reordering for Royal members

```mermaid
sequenceDiagram
    actor RM as Royal Member
    actor C as Order Processing Clerk
    participant I as Item
    participant SUP as Supplier
    RM->>C: order_unavailable_item(item)
    C->>I: is_available()
    I-->>C: unavailable
    C->>SUP: reorder(item)
    SUP-->>C: delivery date
    C-->>RM: item reserved, priority delivery
    SUP->>I: restock(item)
    C->>RM: ship item
```

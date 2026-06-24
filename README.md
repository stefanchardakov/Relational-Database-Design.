# 📚 Relational Database Design — Library Loans System 

> Database design for a mid-sized public library, covering full normalisation (UNF → 3NF), a  Logical Model, and a production-ready Physical Model with justified data types and constraints.

---

## 📌 Project Overview

This project delivers a relational database design for the **Library Loans System** — a community library that offers loans of books, journals, magazines, DVDs, and vinyl records, including inter-library loans with partner institutions.

The design was developed in response to a real operational brief from the head librarian, who required a streamlined, web-accessible database to replace manual loan tracking. The solution covers two formal deliverables:

- **Logical Model:** A fully normalised ERD in Crow's Foot notation (Barker/UML style), progressing from raw data through UNF, 1NF, 2NF, and 3NF, with documented assumptions and decisions at every stage.
- **Physical Model:** A concrete database schema derived from the Logical Model, with named conventions, data types, and integrity constraints ready for implementation in any standard RDBMS.

---
## 🎯 Key Skills Demonstrated

- **Database Normalisation** — Rigorous progression from UNF to 3NF with annotated assumptions at each stage
- **Entity-Relationship Modelling** — Crow's Foot (Barker/UML) notation; Chen style explicitly avoided per brief
- **Relational Schema Design** — Correct use of primary keys, foreign keys, composite keys, and associative entities
- **Requirements Analysis** — Business brief translated into formal data model with justified design decisions
- **Physical Data Modelling** — Data type selection (VARCHAR, INT, DECIMAL) with rationale
- **Constraint Design** — NOT NULL, nullable, and referential integrity constraints applied deliberately
- **Technical Documentation** — Written to a professional standard suitable for a development handover
---

## 🧩 Business Requirements

The following requirements were extracted from the client brief and informed all design decisions:

| # | Requirement |
|---|---|
| 1 | Members can borrow any number of items in a single loan transaction |
| 2 | Loans are processed at a staff desk; staff identity must be recorded |
| 3 | All items must be categorised (books, journals, magazines, DVDs, vinyl) |
| 4 | Publisher records must be maintained for purchased books |
| 5 | Fines must be recorded for items returned past their due date |
| 6 | Inter-library loans with partner libraries must be supported |
| 7 | The system must be accessible via a web application |

---


## 🗂️ Core Entities

| Entity | Role in the System |
|---|---|
| **Member** | Community members who borrow library items |
| **Item** | Borrowable stock (books, journals, magazines, DVDs, vinyl records) |
| **Loan** | A borrowing transaction; one loan may contain multiple items |
| **LoanItem** | Associative entity resolving the many-to-many Loan ↔ Item relationship |
| **Staff** | Library personnel who process loan transactions |
| **ItemType** | Classification of items, with associated category — satisfies the categorisation requirement |
| **Publisher** | Publishers from whom the library purchases books and journals |
| **Library** | Partner library institutions supporting inter-library loans |

---

## 🔄 Normalisation Process

### UNF — Unnormalised Form

The starting point was a flat loan record containing repeating groups: because a single loan can include multiple items, item-related attributes (title, due date, return date, fine amount) appeared multiple times within one record. Member and staff details appeared once per loan and were treated as non-repeating.

**Key decisions at this stage:**

- Artificial primary keys (e.g., `LoanID`, `MemberID`, `ItemID`) were introduced to guarantee entity integrity. Natural keys (e.g., member names) were ruled out as they cannot be guaranteed unique.
- A `Library` entity was included from the outset to support inter-library loan tracking, as required by the brief.
- Each item was given its own `DueDate` and `FineAmount` because items within the same loan may be returned at different times.



---

### 1NF — First Normal Form

**Rules applied:** All attribute values must be atomic; no repeating groups; every record must be uniquely identifiable.

The UNF was decomposed into six relations to eliminate repeating groups:

<img width="448" height="243" alt="1nf" src="https://github.com/user-attachments/assets/33cc17e5-a9fc-4ce2-822c-4181e07a9237" />



**Key decisions:**
- The many-to-many relationship between `Loan` and `Item` was resolved through the **`LoanItem`** associative entity, allowing each item in a loan to be tracked individually — essential for recording per-item due dates, return dates, and fines.
- `DueDate`, `ReturnDate`, and `FineAmount` were placed in `LoanItem` because they are facts about a specific item within a specific loan, not about the loan as a whole.
- `MemberID` and `StaffID` were introduced as foreign keys in `Loan`, capturing which member borrowed and which staff member processed the transaction.
- `LibraryID` was introduced as a foreign key in `Item` to associate items with their owning library, enabling inter-library loan support.
- `Publisher` was retained in `Item` at this stage but flagged as nullable — not all item types (e.g., DVDs, vinyl) have a publisher.
- `MemberName` was treated as a single atomic attribute; first/last name separation was not required by the brief.

---

### 2NF — Second Normal Form

**Rules applied:** Must satisfy 1NF; every non-key attribute must depend on the *whole* primary key, not a subset of it (partial dependency must be eliminated).

The only composite primary key in the schema is `(LoanID, ItemID)` in `LoanItem`. All non-key attributes in that table — `DueDate`, `ReturnDate`, `FineAmount` — depend on the full composite key: the due date of an item is specific to a particular item *on* a particular loan. No attribute depends on `LoanID` or `ItemID` alone, so no partial dependency exists.

All other tables use single-attribute primary keys, meaning partial dependency is structurally impossible.

**Result:** 2NF is satisfied without requiring any further decomposition.

---

### 3NF — Third Normal Form

**Rules applied:** Must satisfy 2NF; no transitive dependencies (a non-key attribute must not be functionally dependent on another non-key attribute).

Two transitive dependencies were identified and resolved:

<img width="352" height="350" alt="3nf" src="https://github.com/user-attachments/assets/678577d9-3f21-4ef4-8336-22fe1b3d039f" />


**1. Category → ItemType → ItemID (in `Item`)**
`Category` described the broad classification of an item (e.g., "Non-fiction"), but it depended on `ItemType` (e.g., "Book") rather than on the item's primary key directly. This is a transitive dependency. Resolution: a new **`ItemType`** table was created, holding `ItemTypeID`, `ItemType`, and `Category`. `Item` now references `ItemType` via a foreign key, satisfying the library's requirement that all items be categorised.

**2. Publisher attributes in `Item`**
Publisher details (name, contact, etc.) were transitively dependent on `PublisherID` rather than `ItemID`. Resolution: a new **`Publisher`** table was extracted. `Item` now holds a nullable `PublisherID` foreign key — nullable because item types such as DVDs and vinyl records do not have a publisher, which is a legitimate business state rather than missing data.

**Result:** All transitive dependencies eliminated; 3NF achieved across the full schema.

---

## 🗺️ Logical Model

The Logical Model uses **Crow's Foot (Barker/UML) notation** to express cardinality and participation constraints, as specified in the brief. Chen-style notation was explicitly not used.

<img width="480" height="338" alt="logical model" src="https://github.com/user-attachments/assets/87e5a053-27b4-422f-bf1a-9c00adbddf50" />

| Relationship | Cardinality & Participation |
|---|---|
| Member → Loan | A member may have zero or many loans; each loan belongs to exactly one member |
| Staff → Loan | A staff member may process zero or many loans; each loan is processed by exactly one staff member |
| Loan → LoanItem | A loan must contain at least one LoanItem (mandatory participation) |
| Item → LoanItem | An item may appear in zero or many LoanItems; each LoanItem refers to exactly one item |
| ItemType → Item | One item type can classify many items; each item must have exactly one type |
| Publisher → Item | A publisher may supply many items; an item may have zero or one publisher (nullable) |
| Library → Item | A library may own many items; each item must belong to exactly one library |

---

## 🏗️ Physical Model

The Physical Model translates the Logical Model into a concrete, implementation-ready schema.
<img width="445" height="354" alt="Physical Model" src="https://github.com/user-attachments/assets/176e2274-5622-4e61-aeb0-d55bf06556c0" />


### Data Type Decisions

| Data Type | Applied To | Rationale |
|---|---|---|
| `VARCHAR(100)` | Names, MemberName, StaffName | Variable-length; 100 characters is sufficient for personal names |
| `VARCHAR(150)` | ItemTitle | Longer allowance for book and journal titles |
| `VARCHAR(50)` | ItemType, Category | Short, bounded classification strings |
| `INT` | All ID / primary key columns | Provides a wide range of unique identifiers without needing a length cap |
| `DECIMAL(6,2)` | FineAmount | Precise two-decimal-place currency storage, supporting values up to `9,999.99` |
| `DATE` | DueDate, ReturnDate | Standard date type; no time component required for loan tracking |

### Constraint Decisions

| Constraint | Applied To | Rationale |
|---|---|---|
| `NOT NULL` | All mandatory attributes (e.g., LoanDate, MemberID) | Enforces entity and referential integrity at the database level |
| `NULL` | PublisherID in Item | Legitimately absent for non-published item types; represents valid business state |
| `PRIMARY KEY` | All ID columns | Guarantees uniqueness and entity integrity |
| `FOREIGN KEY` | All relationship columns | Enforces referential integrity across the schema |
| `COMPOSITE PK` | (LoanID, ItemID) in LoanItem | Ensures each item can only appear once per loan transaction |

---
## 🛠️ Tools Used

- Draw.io 
- MySQL Workbench
- Relational Database Design
- Crow's Foot ER Modelling
- Database Normalisation (UNF → 3NF)

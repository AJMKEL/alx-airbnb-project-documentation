# Airbnb Clone - Data Flow Diagram (DFD)

## Overview
This document provides comprehensive documentation for the Data Flow Diagram (DFD) of the Airbnb Clone backend system. A DFD is a visual representation that shows how data moves through a system, illustrating the flow of information from input to processing to output across different system components.

## Purpose
The Data Flow Diagram serves to:
- **Visualize Data Movement**: Show how information flows between system components
- **Identify Data Stores**: Document where data is stored and retrieved
- **Map Processes**: Illustrate data transformations and processing
- **Clarify System Boundaries**: Define what's inside and outside the system
- **Support Design**: Guide database and API design decisions
- **Communication**: Help stakeholders understand data handling

---

## What is a Data Flow Diagram?

A Data Flow Diagram is a graphical representation of the flow of data through an information system. It models:
- **Sources/Destinations**: External entities that send or receive data
- **Processes**: Actions that transform data
- **Data Stores**: Where information is held
- **Data Flows**: Movement of data between components

### DFD Notation

#### 1. External Entities (Rectangles)
Represent sources or destinations of data outside the system:
- **Guest User**
- **Registered User**
- **Host**
- **Administrator**
- **Payment Gateway** (Stripe, PayPal)
- **Email Service**
- **SMS Service**

#### 2. Processes (Circles/Rounded Rectangles)
Represent actions that transform or process data:
- **1.0 Authenticate User**
- **2.0 Manage Properties**
- **3.0 Process Booking**
- **4.0 Handle Payment**
- **5.0 Manage Reviews**
- **6.0 Send Notifications**

#### 3. Data Stores (Parallel Lines/Open Rectangles)
Represent where data is stored:
- **D1: User Database**
- **D2: Property Database**
- **D3: Booking Database**
- **D4: Payment Database**
- **D5: Review Database**
- **D6: Message Database**

#### 4. Data Flows (Arrows)
Represent movement of data between components:
- Labeled with the data being transferred
- Arrow direction shows flow direction
- Can split and merge

---

## System Context Diagram (Level 0 DFD)

The highest level view showing the entire system as a single process with external entities.

### External Entities:
1. **Guest User**: Unauthenticated visitors
2. **Registered User**: Authenticated guests
3. **Host**: Property owners
4. **Administrator**: Platform managers
5. **Payment Gateway**: External payment processor
6. **Email Service**: External notification service

### Data Flows (Level 0):

**From Guest User:**
- Search criteria → System
- Registration data → System
- Login credentials → System

**To Guest User:**
- Property listings ← System
- Search results ← System
- Registration confirmation ← System

**From Registered User:**
- Booking request → System
- Payment information → System
- Review data → System

**To Registered User:**
- Booking confirmation ← System
- Property details ← System
- Notifications ← System

**From Host:**
- Property data → System
- Availability updates → System
- Booking responses → System

**To Host:**
- Booking requests ← System
- Earnings reports ← System
- Guest information ← System

**From Administrator:**
- Moderation actions → System
- Configuration changes → System

**To Administrator:**
- Platform analytics ← System
- User reports ← System
- Audit logs ← System

**System ↔ Payment Gateway:**
- Payment requests → Gateway
- Payment status ← Gateway
- Refund requests → Gateway

**System ↔ Email Service:**
- Email data → Email Service
- Delivery status ← Email Service

---

## Level 1 DFD - Core Processes

Breaks down the system into major functional processes.

### Process 1.0: User Authentication and Management

**Inputs:**
- Registration data (email, password, name, phone)
- Login credentials (email, password)
- Profile update data
- Password reset request

**Processing:**
1. Validate input data format
2. Check for existing user (registration)
3. Hash password using bcrypt
4. Verify credentials against database
5. Generate JWT token
6. Create/update user record

**Outputs:**
- JWT authentication token
- User profile data
- Success/error messages
- Email verification link

**Data Stores Accessed:**
- D1: User Database (read/write)

**Data Flows:**
```
Guest User → [Registration Data] → 1.1 Validate Registration
1.1 → [Validated Data] → 1.2 Create User Record
1.2 → [User Record] → D1: User Database
1.2 → [Confirmation] → Guest User
1.2 → [Verification Email] → Email Service

User → [Login Credentials] → 1.3 Authenticate User
1.3 → [Query] → D1: User Database
D1 → [User Record] → 1.3
1.3 → [JWT Token] → User
```

---

### Process 2.0: Property Management

**Inputs:**
- Property details (name, description, location, price)
- Property images
- Amenities list
- Availability dates
- Pricing updates

**Processing:**
1. Validate property data
2. Verify host authorization
3. Process and optimize images
4. Calculate pricing with fees
5. Update availability calendar
6. Generate property ID

**Outputs:**
- Property listing
- Property ID
- Confirmation messages
- Updated property data

**Data Stores Accessed:**
- D2: Property Database (read/write)
- D1: User Database (read - verify host)

**Data Flows:**
```
Host → [Property Data] → 2.1 Validate Property
2.1 → [Validated Data] → 2.2 Process Images
2.2 → [Property with Images] → 2.3 Create Listing
2.3 → [Host ID Query] → D1: User Database
D1 → [Host Verification] → 2.3
2.3 → [Property Record] → D2: Property Database
2.3 → [Confirmation] → Host
2.3 → [Notification] → Email Service

Host → [Availability Update] → 2.4 Update Calendar
2.4 → [Calendar Data] → D2: Property Database
```

---

### Process 3.0: Booking Management

**Inputs:**
- Search criteria (location, dates, guests)
- Booking request (property ID, dates, guests)
- Cancellation request
- Booking modification

**Processing:**
1. Search properties by criteria
2. Check availability for dates
3. Calculate total price
4. Verify no booking conflicts
5. Create booking record
6. Update property availability
7. Process cancellation
8. Calculate refund amount

**Outputs:**
- Search results
- Booking confirmation
- Booking ID
- Cancellation confirmation
- Updated availability

**Data Stores Accessed:**
- D2: Property Database (read)
- D3: Booking Database (read/write)
- D1: User Database (read)

**Data Flows:**
```
User → [Search Criteria] → 3.1 Search Properties
3.1 → [Query] → D2: Property Database
D2 → [Matching Properties] → 3.1
3.1 → [Search Results] → User

User → [Booking Request] → 3.2 Check Availability
3.2 → [Availability Query] → D3: Booking Database
3.2 → [Property Query] → D2: Property Database
D3 → [Existing Bookings] → 3.2
D2 → [Property Details] → 3.2
3.2 → [Available] → 3.3 Calculate Price

3.3 → [Price Breakdown] → 3.4 Create Booking
3.4 → [User Query] → D1: User Database
D1 → [User Details] → 3.4
3.4 → [Booking Record] → D3: Booking Database
3.4 → [Payment Request] → Process 4.0
3.4 → [Confirmation Data] → Email Service
3.4 → [Booking Confirmation] → User

User → [Cancellation Request] → 3.5 Cancel Booking
3.5 → [Booking Query] → D3: Booking Database
D3 → [Booking Details] → 3.5
3.5 → [Refund Amount] → Process 4.0
3.5 → [Updated Status] → D3: Booking Database
3.5 → [Cancellation Notice] → Email Service
```

---

### Process 4.0: Payment Processing

**Inputs:**
- Payment method (card, PayPal)
- Booking amount
- Refund request
- Payout request

**Processing:**
1. Validate payment information
2. Tokenize card details
3. Process payment via gateway
4. Record transaction
5. Calculate refund amount
6. Process refund
7. Schedule host payout
8. Process payout

**Outputs:**
- Payment confirmation
- Transaction ID
- Receipt
- Refund confirmation
- Payout confirmation

**Data Stores Accessed:**
- D4: Payment Database (read/write)
- D3: Booking Database (read)
- D1: User Database (read)

**Data Flows:**
```
Process 3.0 → [Payment Request] → 4.1 Validate Payment
4.1 → [Payment Details] → Payment Gateway
Payment Gateway → [Authorization] → 4.1
4.1 → [Confirmed] → 4.2 Record Transaction

4.2 → [Booking Query] → D3: Booking Database
D3 → [Booking Details] → 4.2
4.2 → [Transaction Record] → D4: Payment Database
4.2 → [Payment Confirmation] → User
4.2 → [Receipt Data] → Email Service
4.2 → [Update Status] → D3: Booking Database

Process 3.0 → [Refund Request] → 4.3 Calculate Refund
4.3 → [Cancellation Policy Query] → D2: Property Database
4.3 → [Payment Query] → D4: Payment Database
D4 → [Original Payment] → 4.3
4.3 → [Refund Amount] → 4.4 Process Refund

4.4 → [Refund Request] → Payment Gateway
Payment Gateway → [Refund Status] → 4.4
4.4 → [Refund Record] → D4: Payment Database
4.4 → [Refund Notice] → Email Service

Scheduler → [Payout Time] → 4.5 Process Payout
4.5 → [Completed Bookings Query] → D3: Booking Database
4.5 → [Host Details Query] → D1: User Database
4.5 → [Payout Request] → Payment Gateway
Payment Gateway → [Payout Status] → 4.5
4.5 → [Payout Record] → D4: Payment Database
4.5 → [Payout Notice] → Email Service → Host
```

---

### Process 5.0: Review and Rating Management

**Inputs:**
- Review data (rating, comment)
- Review edit
- Review deletion request

**Processing:**
1. Verify booking completion
2. Validate review data
3. Check review eligibility
4. Store review (hidden initially)
5. Make visible after deadline/both reviews
6. Calculate average ratings
7. Update property rating

**Outputs:**
- Review confirmation
- Updated property rating
- Review visibility status

**Data Stores Accessed:**
- D5: Review Database (read/write)
- D3: Booking Database (read)
- D2: Property Database (update)
- D1: User Database (read)

**Data Flows:**
```
User → [Review Data] → 5.1 Validate Review
5.1 → [Booking Query] → D3: Booking Database
D3 → [Booking Status] → 5.1
5.1 → [Eligibility Check] → 5.2 Create Review

5.2 → [User Query] → D1: User Database
D1 → [User Details] → 5.2
5.2 → [Review Record] → D5: Review Database
5.2 → [Confirmation] → User
5.2 → [Review Notice] → Email Service

Scheduler → [Review Check] → 5.3 Check Visibility
5.3 → [Review Status Query] → D5: Review Database
D5 → [Review Pairs] → 5.3
5.3 → [Make Visible] → D5: Review Database
5.3 → [Notification] → Email Service

5.3 → [Trigger] → 5.4 Calculate Ratings
5.4 → [All Reviews Query] → D5: Review Database
D5 → [Property Reviews] → 5.4
5.4 → [Average Rating] → D2: Property Database
```

---

### Process 6.0: Messaging and Communication

**Inputs:**
- Message content
- Recipient information
- File attachments

**Processing:**
1. Validate sender authorization
2. Check recipient exists
3. Store message
4. Create notification
5. Send real-time notification

**Outputs:**
- Message delivery confirmation
- Message notifications
- Message history

**Data Stores Accessed:**
- D6: Message Database (read/write)
- D1: User Database (read)

**Data Flows:**
```
User → [Message Data] → 6.1 Validate Message
6.1 → [Sender Query] → D1: User Database
6.1 → [Recipient Query] → D1: User Database
D1 → [User Verification] → 6.1
6.1 → [Validated Message] → 6.2 Store Message

6.2 → [Message Record] → D6: Message Database
6.2 → [Delivery Confirmation] → Sender
6.2 → [Notification Data] → Process 7.0
6.2 → [Real-time Message] → Recipient

User → [Message History Request] → 6.3 Retrieve Messages
6.3 → [User Query] → D1: User Database
6.3 → [Message Query] → D6: Message Database
D6 → [Message Thread] → 6.3
6.3 → [Message History] → User
```

---

### Process 7.0: Notification Management

**Inputs:**
- Notification triggers from other processes
- User preferences
- Template data

**Processing:**
1. Determine notification type
2. Check user preferences
3. Select appropriate template
4. Populate template with data
5. Send via appropriate channel(s)
6. Track delivery status

**Outputs:**
- Email notifications
- SMS notifications
- In-app notifications
- Push notifications

**Data Stores Accessed:**
- D1: User Database (read - preferences)
- Notification Log (write)

**Data Flows:**
```
Process 3.0 → [Booking Event] → 7.1 Determine Notification Type
Process 4.0 → [Payment Event] → 7.1
Process 5.0 → [Review Event] → 7.1
Process 6.0 → [Message Event] → 7.1

7.1 → [User Preferences Query] → D1: User Database
D1 → [Notification Settings] → 7.1
7.1 → [Notification Type] → 7.2 Select Template

7.2 → [Template + Data] → 7.3 Send Notification
7.3 → [Email Notification] → Email Service
7.3 → [SMS Notification] → SMS Service
7.3 → [In-App Notification] → User
7.3 → [Delivery Log] → Notification Log

Email Service → [Delivery Status] → 7.4 Track Delivery
7.4 → [Status Update] → Notification Log
```

---

### Process 8.0: Administrative Operations

**Inputs:**
- Admin commands
- Moderation requests
- Configuration changes
- Report parameters

**Processing:**
1. Verify admin authorization
2. Execute admin actions
3. Log admin activities
4. Generate reports
5. Update configurations

**Outputs:**
- Moderation decisions
- System configurations
- Analytics reports
- Audit logs

**Data Stores Accessed:**
- All data stores (read/write)
- Audit Log (write)

**Data Flows:**
```
Admin → [Moderation Request] → 8.1 Verify Admin
8.1 → [Admin Query] → D1: User Database
D1 → [Admin Verification] → 8.1
8.1 → [Authorized] → 8.2 Execute Action

8.2 → [Content Query] → D2/D5 Databases
8.2 → [Moderation Decision] → D2/D5 Databases
8.2 → [Action Log] → Audit Log
8.2 → [Notification] → Email Service

Admin → [Report Request] → 8.3 Generate Report
8.3 → [Data Query] → All Data Stores
All Data Stores → [Data] → 8.3
8.3 → [Aggregate] → 8.4 Format Report
8.4 → [Report] → Admin

Admin → [Config Change] → 8.5 Update Configuration
8.5 → [Validation] → 8.6 Apply Settings
8.6 → [New Settings] → System Configuration
8.6 → [Change Log] → Audit Log
```

---

## Data Stores Details

### D1: User Database
**Contents:**
- User ID (Primary Key)
- Email (Unique)
- Password Hash
- Name (First, Last)
- Phone Number
- Profile Picture URL
- Role (Guest, Host, Admin)
- Verification Status
- Created Date
- Last Login

**Accessed By:**
- Process 1.0 (Authentication)
- Process 2.0 (Host verification)
- Process 3.0 (Booking user details)
- Process 4.0 (Payment user info)
- Process 5.0 (Review author)
- Process 6.0 (Messaging participants)
- Process 7.0 (Notification preferences)
- Process 8.0 (Admin operations)

---

### D2: Property Database
**Contents:**
- Property ID (Primary Key)
- Host ID (Foreign Key)
- Title
- Description
- Location (Address, Coordinates)
- Property Type
- Number of Guests/Bedrooms/Bathrooms
- Amenities (JSON)
- Pricing (Nightly Rate)
- Images (URLs)
- House Rules
- Cancellation Policy
- Average Rating
- Status (Active, Inactive)
- Created/Updated Dates

**Accessed By:**
- Process 2.0 (Property management)
- Process 3.0 (Search and booking)
- Process 4.0 (Cancellation policy)
- Process 5.0 (Rating updates)
- Process 8.0 (Admin moderation)

---

### D3: Booking Database
**Contents:**
- Booking ID (Primary Key)
- Property ID (Foreign Key)
- User ID (Foreign Key)
- Check-in Date
- Check-out Date
- Number of Guests
- Total Price
- Status (Pending, Confirmed, Cancelled, Completed)
- Booking Date
- Cancellation Date
- Special Requests

**Accessed By:**
- Process 3.0 (Booking operations)
- Process 4.0 (Payment processing)
- Process 5.0 (Review eligibility)
- Process 7.0 (Booking notifications)
- Process 8.0 (Admin reports)

---

### D4: Payment Database
**Contents:**
- Payment ID (Primary Key)
- Booking ID (Foreign Key)
- Amount
- Payment Method
- Transaction ID (Gateway)
- Payment Status
- Payment Date
- Refund Amount
- Refund Date
- Payout Status
- Payout Date

**Accessed By:**
- Process 4.0 (All payment operations)
- Process 8.0 (Financial reports)

---

### D5: Review Database
**Contents:**
- Review ID (Primary Key)
- Property ID (Foreign Key)
- User ID (Reviewer)
- Booking ID (Foreign Key)
- Overall Rating (1-5)
- Category Ratings (Cleanliness, Accuracy, etc.)
- Comment
- Visibility Status (Hidden, Public)
- Review Date
- Edit Date

**Accessed By:**
- Process 5.0 (Review management)
- Process 3.0 (Display on search results)
- Process 8.0 (Moderation)

---

### D6: Message Database
**Contents:**
- Message ID (Primary Key)
- Sender ID (Foreign Key)
- Recipient ID (Foreign Key)
- Property ID (Foreign Key - optional)
- Booking ID (Foreign Key - optional)
- Message Content
- Attachments (URLs)
- Read Status
- Sent Date

**Accessed By:**
- Process 6.0 (Messaging)
- Process 7.0 (Message notifications)
- Process 8.0 (Moderation)

---

## Key Data Flows

### 1. User Registration Flow
```
Guest → Registration Form → Validate Input → Check Duplicate Email →
Hash Password → Create User Record → D1: User DB → Send Verification Email →
Email Service → User Receives Email → Clicks Link → Verify Account →
Update D1: User DB → Login User → Generate JWT → Return Token
```

### 2. Property Listing Creation Flow
```
Host → Property Details → Validate Host → Validate Data → Process Images →
Generate Property ID → Create Property Record → D2: Property DB →
Update Host Listings → Send Confirmation → Email Service → Host Notified
```

### 3. Booking Creation Flow
```
User → Search Properties → Query D2: Property DB → Return Results →
User Selects Property → Select Dates → Check Availability → Query D3: Booking DB →
Calculate Price → Create Booking Record → Request Payment → Payment Gateway →
Process Payment → Update D4: Payment DB → Confirm Booking → Update D3: Booking DB →
Send Confirmations → Email Service → User & Host Notified
```

### 4. Review Submission Flow
```
User → Review Form → Verify Booking Completed → Query D3: Booking DB →
Validate Review Data → Create Review Record → D5: Review DB (Hidden) →
Check Other Party Review → If Both Submitted → Make Visible →
Calculate New Average → Update D2: Property DB → Notify Host → Email Service
```

### 5. Payment Processing Flow
```
Booking Confirmed → Payment Request → Validate Payment Method →
Send to Payment Gateway → Gateway Processes → Return Status →
Record Transaction → D4: Payment DB → Generate Receipt →
Email Service → User Notified → Schedule Host Payout →
24 Hours After Check-in → Process Payout → Gateway Transfers →
Update D4: Payment DB → Notify Host → Email Service
```

---

## Creating the DFD in Visily

### Step 1: Set Up Your Canvas
1. Go to [Visily.ai](https://visily.ai)
2. Create a new project
3. Select blank canvas
4. Choose landscape orientation
5. Set canvas to large size

### Step 2: Add External Entities
1. Draw rectangles for external entities:
   - Guest User (top left)
   - Registered User (left)
   - Host (left)
   - Administrator (top right)
   - Payment Gateway (right)
   - Email Service (bottom right)
   - SMS Service (bottom right)
2. Label each entity clearly
3. Position them around the perimeter

### Step 3: Add Processes
1. Draw circles or rounded rectangles for main processes:
   - 1.0 User Authentication (top center)
   - 2.0 Property Management (center left)
   - 3.0 Booking Management (center)
   - 4.0 Payment Processing (center right)
   - 5.0 Review Management (bottom center)
   - 6.0 Messaging (bottom left)
   - 7.0 Notifications (bottom right)
   - 8.0 Admin Operations (top right)
2. Number each process
3. Use consistent shape and size

### Step 4: Add Data Stores
1. Draw parallel lines or open rectangles:
   - D1: User Database
   - D2: Property Database
   - D3: Booking Database
   - D4: Payment Database
   - D5: Review Database
   - D6: Message Database
2. Position them centrally or along the bottom
3. Use consistent styling

### Step 5: Draw Data Flows
1. Connect entities to processes with arrows
2. Label each arrow with data being transferred
3. Connect processes to data stores
4. Connect processes to other processes
5. Ensure arrow direction is correct

**Key Flows to Include:**
- Guest → 1.0: Registration Data, Login Credentials
- 1.0 → D1: User Records
- Host → 2.0: Property Data
- 2.0 → D2: Property Records
- User → 3.0: Booking Request
- 3.0 → D3: Booking Records
- 3.0 → 4.0: Payment Request
- 4.0 ↔ Payment Gateway: Payment Data
- 3.0 → D2: Availability Query
- User → 5.0: Review Data
- 5.0 → D5: Review Records
- All Processes → 7.0: Notification Triggers
- 7.0 → Email Service: Email Data

### Step 6: Add Color Coding
1. Use different colors for different types:
   - Blue for external entities
   - Green for processes
   - Yellow for data stores
   - Red for critical data flows (payment)
2. Keep colors consistent and professional

### Step 7: Refine and Label
1. Ensure all arrows are labeled
2. Check that data flow direction is correct
3. Verify all processes are numbered
4. Confirm all data stores are prefixed with "D"
5. Add a legend explaining symbols
6. Add title: "Airbnb Clone - Data Flow Diagram"

### Step 8: Export
1. Review the complete diagram
2. Click Export or Download
3. Select PNG format
4. Choose high resolution (300 DPI minimum)
5. Save as "data-flow.png"

---

## DFD Best Practices Applied

1. **Clear Naming**: All components have descriptive names
2. **Consistent Notation**: Standard DFD symbols used throughout
3. **Balanced Layout**: Entities and processes well-distributed
4. **Minimal Crossing**: Data flows cross minimally
5. **Hierarchical**: Can be decomposed to lower levels
6. **Complete**: All major system functions included
7. **Accurate**: Reflects actual system behavior

---

## Visual Documentation

![Data Flow Diagram](./data-flow.png)

*The diagram above illustrates the complete data flow through the Airbnb Clone system, showing how information moves between external entities, processes, and data stores.*

---

## How to Read the DFD

### For Developers:
- **Understand Integration Points**: See how components connect
- **Plan APIs**: Data flows indicate necessary endpoints
- **Design Database**: Data stores show required tables
- **Trace Data**: Follow data from input to output

### For Database Designers:
- **Identify Tables**: Each data store becomes one or more tables
- **Define Relationships**: Data flows show foreign key relationships
- **Plan Indexes**: Frequent queries indicate indexing needs
- **Optimize Storage**: Multiple processes accessing same store

### For Business Analysts:
- **Verify Requirements**: Ensure all business processes covered
- **Identify Gaps**: Missing data flows indicate missing features
- **Understand Impact**: Changes affect connected components
- **Document Workflows**: Processes match business workflows

---

## Repository Structure

```
alx-airbnb-project-documentation/
├── features-and-functionalities/
│   ├── README.md
│   └── airbnb-features-overview.png
├── use-case-diagram/
│   ├── README.md
│   └── use-case-diagram.png
├── user-stories/
│   ├── README.md
│   └── user-stories.md
├── data-flow-diagram/
│   ├── README.md (this file)
│   └── data-flow.png
├── requirements/
├── design/
└── project-plan/
```

---

## Conclusion

This Data Flow Diagram provides a comprehensive view of how data moves through the Airbnb Clone system. It serves as a blueprint for:
- API design and implementation
- Database schema design
- Integration planning
- System testing
- Performance optimization
- Security analysis

By understanding these data flows, the development team can build a system where data moves efficiently and securely from input to storage to output, ensuring a robust and scalable platform.

---

*This documentation is part of the Airbnb Clone project documentation suite. Last updated: November 2024*

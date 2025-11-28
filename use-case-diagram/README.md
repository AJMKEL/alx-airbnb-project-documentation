# Airbnb Clone - Use Case Diagram

## Overview
This document presents the use case diagram for the Airbnb Clone backend system. A use case diagram is a visual representation that shows how different types of users (actors) interact with the system to accomplish specific goals. It helps identify the key functionalities from a user's perspective and illustrates the relationships between actors and their use cases.

## Purpose
The use case diagram serves multiple purposes:
- **Visualize System Functionality**: Provides a clear picture of what the system does from the user's perspective
- **Identify Key Actors**: Shows all types of users who interact with the system
- **Document Interactions**: Captures how users accomplish their goals through system features
- **Communication Tool**: Helps stakeholders, developers, and designers understand system requirements
- **Scope Definition**: Clearly defines the boundaries of the system and its capabilities

---

## Actors

Actors represent the different types of users or external systems that interact with the Airbnb Clone platform. Each actor has specific goals and use cases they can perform.

### 1. Guest (Unauthenticated User)
**Description**: A visitor to the platform who has not yet registered or logged in.

**Primary Goals**:
- Browse available properties without commitment
- Learn about the platform and its offerings
- Create an account to access full functionality
- Log in to an existing account

**Characteristics**:
- Limited access to platform features
- Cannot make bookings or contact hosts
- Can view public property listings and details
- Must register or log in for transactional features

---

### 2. Registered Guest
**Description**: An authenticated user who books properties and uses the platform as a traveler.

**Primary Goals**:
- Find and book properties for travel
- Manage personal profile and preferences
- Communicate with hosts
- Leave reviews after stays
- Manage bookings and payments

**Characteristics**:
- Full access to booking features
- Can save favorite properties
- Maintains booking history
- Has payment methods on file
- Can rate and review properties

---

### 3. Host
**Description**: An authenticated user who lists properties for rent on the platform.

**Primary Goals**:
- List and manage rental properties
- Set pricing and availability
- Respond to booking requests
- Communicate with guests
- Review guests after stays
- Track earnings and performance

**Characteristics**:
- Can be both a host and a guest
- Manages multiple properties
- Sets house rules and cancellation policies
- Receives payouts for bookings
- Monitors property performance metrics

---

### 4. Administrator
**Description**: A platform staff member with elevated privileges for managing the system.

**Primary Goals**:
- Oversee platform operations
- Manage users and content
- Resolve disputes
- Monitor platform health
- Configure system settings
- Generate reports and analytics

**Characteristics**:
- Full access to all system features
- Can moderate content and users
- Views comprehensive analytics
- Handles customer support escalations
- Manages platform policies

---

### 5. Payment Gateway (External System)
**Description**: An external payment processing service (e.g., Stripe, PayPal) that handles financial transactions.

**Primary Goals**:
- Process payment transactions
- Handle refunds
- Verify payment methods
- Ensure PCI DSS compliance

**Characteristics**:
- Integrates via API
- Processes transactions securely
- Returns transaction status
- Handles multiple currencies

---

## Use Cases by Actor

### Guest (Unauthenticated) Use Cases

#### UC-1: Browse Properties
**Description**: View available properties without logging in  
**Actor**: Guest  
**Preconditions**: None  
**Main Flow**:
1. Guest accesses the platform homepage
2. System displays featured properties
3. Guest can view property details, photos, and basic information
4. System shows pricing and availability (limited)

**Postconditions**: Guest has viewed property listings  
**Business Value**: Attracts potential customers and showcases platform offerings

---

#### UC-2: Search Properties
**Description**: Search for properties using filters  
**Actor**: Guest  
**Preconditions**: None  
**Main Flow**:
1. Guest enters search criteria (location, dates, guests)
2. System validates input
3. System displays matching properties
4. Guest can apply additional filters

**Postconditions**: Guest views search results  
**Business Value**: Helps users discover relevant properties quickly

---

#### UC-3: Register Account
**Description**: Create a new user account  
**Actor**: Guest  
**Preconditions**: Valid email address  
**Main Flow**:
1. Guest navigates to registration page
2. Guest provides email, password, and personal information
3. System validates information
4. System creates account
5. System sends verification email
6. Guest verifies email address

**Postconditions**: New registered user account created  
**Business Value**: Converts visitors to registered users

---

#### UC-4: Login
**Description**: Access existing account  
**Actor**: Guest  
**Preconditions**: Valid account exists  
**Main Flow**:
1. Guest enters email and password
2. System authenticates credentials
3. System generates JWT token
4. User is redirected to dashboard

**Postconditions**: User is authenticated  
**Business Value**: Provides secure access to platform features

---

### Registered Guest Use Cases

#### UC-5: Manage Profile
**Description**: Update personal information and preferences  
**Actor**: Registered Guest  
**Preconditions**: User is authenticated  
**Main Flow**:
1. User accesses profile settings
2. User updates information (name, photo, phone, bio)
3. System validates changes
4. System saves updated profile

**Postconditions**: Profile information is updated  
**Business Value**: Enables personalization and trust-building

---

#### UC-6: Create Booking
**Description**: Reserve a property for specific dates  
**Actor**: Registered Guest  
**Preconditions**: User is authenticated, property is available  
**Main Flow**:
1. Guest selects property and dates
2. System checks availability
3. System calculates total price
4. Guest reviews booking details
5. Guest confirms booking
6. System processes payment
7. System creates booking record
8. System sends confirmation

**Postconditions**: Booking is created and confirmed  
**Business Value**: Core revenue-generating transaction  
**Related Use Cases**: UC-12 (Process Payment)

---

#### UC-7: View Bookings
**Description**: See list of current and past bookings  
**Actor**: Registered Guest  
**Preconditions**: User is authenticated  
**Main Flow**:
1. Guest navigates to bookings page
2. System retrieves user's bookings
3. System displays booking list with details

**Postconditions**: Guest views booking history  
**Business Value**: Provides transparency and booking management

---

#### UC-8: Cancel Booking
**Description**: Cancel an existing reservation  
**Actor**: Registered Guest  
**Preconditions**: User has active booking  
**Main Flow**:
1. Guest selects booking to cancel
2. System displays cancellation policy
3. Guest confirms cancellation
4. System calculates refund amount
5. System cancels booking
6. System processes refund
7. System notifies host

**Postconditions**: Booking is cancelled, refund processed  
**Business Value**: Provides flexibility while protecting hosts  
**Related Use Cases**: UC-13 (Process Refund)

---

#### UC-9: Leave Review
**Description**: Rate and review a property after stay  
**Actor**: Registered Guest  
**Preconditions**: Booking is completed  
**Main Flow**:
1. Guest accesses completed booking
2. Guest provides ratings (1-5 stars) for multiple categories
3. Guest writes review comment
4. System validates review
5. System saves review (hidden until host reviews or deadline)
6. System notifies host

**Postconditions**: Review is submitted  
**Business Value**: Builds trust and improves property quality

---

#### UC-10: Send Message to Host
**Description**: Communicate with property host  
**Actor**: Registered Guest  
**Preconditions**: User is authenticated  
**Main Flow**:
1. Guest navigates to property or booking
2. Guest clicks message host
3. Guest composes message
4. System sends message
5. System notifies host

**Postconditions**: Message is sent to host  
**Business Value**: Facilitates communication and customer service

---

#### UC-11: Add to Wishlist
**Description**: Save favorite properties for later  
**Actor**: Registered Guest  
**Preconditions**: User is authenticated  
**Main Flow**:
1. Guest views property
2. Guest clicks "Add to Wishlist"
3. System saves property to user's wishlist
4. System confirms action

**Postconditions**: Property is saved to wishlist  
**Business Value**: Increases engagement and future bookings

---

### Host Use Cases

#### UC-12: Create Property Listing
**Description**: Add a new rental property to the platform  
**Actor**: Host  
**Preconditions**: User is authenticated, verified as host  
**Main Flow**:
1. Host navigates to "List Property"
2. Host enters property details (title, description, location)
3. Host sets pricing and availability
4. Host uploads photos
5. Host specifies amenities and house rules
6. System validates listing
7. System creates property listing

**Postconditions**: New property listing is created  
**Business Value**: Expands platform inventory

---

#### UC-13: Update Property Listing
**Description**: Modify existing property information  
**Actor**: Host  
**Preconditions**: Host owns the property  
**Main Flow**:
1. Host selects property to edit
2. Host modifies information
3. System validates changes
4. System updates listing

**Postconditions**: Property information is updated  
**Business Value**: Keeps listings accurate and current

---

#### UC-14: Manage Availability Calendar
**Description**: Set available/blocked dates for property  
**Actor**: Host  
**Preconditions**: Host owns the property  
**Main Flow**:
1. Host accesses property calendar
2. Host blocks or unblocks specific dates
3. Host sets seasonal pricing (optional)
4. System updates availability

**Postconditions**: Property availability is updated  
**Business Value**: Prevents booking conflicts

---

#### UC-15: Respond to Booking Request
**Description**: Accept or decline booking requests  
**Actor**: Host  
**Preconditions**: Booking request exists  
**Main Flow**:
1. Host receives booking request notification
2. Host reviews guest profile and request details
3. Host accepts or declines request
4. System updates booking status
5. System notifies guest

**Postconditions**: Booking is confirmed or declined  
**Business Value**: Gives hosts control over guests

---

#### UC-16: Review Guest
**Description**: Rate and review guest after their stay  
**Actor**: Host  
**Preconditions**: Booking is completed  
**Main Flow**:
1. Host accesses completed booking
2. Host rates guest on cleanliness, communication, rules
3. Host writes review comment
4. System validates review
5. System saves review

**Postconditions**: Guest review is submitted  
**Business Value**: Maintains platform quality and accountability

---

#### UC-17: View Earnings
**Description**: Track income and payout information  
**Actor**: Host  
**Preconditions**: Host is authenticated  
**Main Flow**:
1. Host navigates to earnings dashboard
2. System displays total earnings, pending payouts
3. System shows transaction history
4. Host can filter by date range

**Postconditions**: Host views earnings information  
**Business Value**: Provides financial transparency

---

#### UC-18: Receive Payout
**Description**: Get paid for completed bookings  
**Actor**: Host  
**Preconditions**: Booking completed, payout scheduled  
**Main Flow**:
1. System identifies completed bookings
2. System calculates host payout (minus fees)
3. System initiates payout to host's account
4. Payment gateway processes transfer
5. System confirms payout
6. System notifies host

**Postconditions**: Host receives payment  
**Business Value**: Core host revenue mechanism  
**Related Use Cases**: UC-12 (Process Payment via Payment Gateway)

---

### Administrator Use Cases

#### UC-19: Manage Users
**Description**: View, suspend, or delete user accounts  
**Actor**: Administrator  
**Preconditions**: Admin is authenticated  
**Main Flow**:
1. Admin accesses user management panel
2. Admin searches for specific user
3. Admin views user details
4. Admin can suspend, ban, or delete account
5. System updates user status

**Postconditions**: User account status is modified  
**Business Value**: Maintains platform safety and policy compliance

---

#### UC-20: Moderate Content
**Description**: Review and remove inappropriate content  
**Actor**: Administrator  
**Preconditions**: Admin is authenticated  
**Main Flow**:
1. Admin reviews flagged content (listings, reviews, messages)
2. Admin determines if content violates policies
3. Admin removes or approves content
4. System updates content status
5. System notifies user if content removed

**Postconditions**: Content is moderated  
**Business Value**: Maintains platform quality and safety

---

#### UC-21: Resolve Disputes
**Description**: Mediate conflicts between guests and hosts  
**Actor**: Administrator  
**Preconditions**: Admin is authenticated, dispute exists  
**Main Flow**:
1. Admin reviews dispute details
2. Admin examines evidence from both parties
3. Admin makes decision on resolution
4. System processes refund/compensation if needed
5. System notifies both parties

**Postconditions**: Dispute is resolved  
**Business Value**: Maintains user trust and satisfaction

---

#### UC-22: Generate Reports
**Description**: Create analytics and business reports  
**Actor**: Administrator  
**Preconditions**: Admin is authenticated  
**Main Flow**:
1. Admin selects report type
2. Admin sets parameters (date range, metrics)
3. System retrieves data
4. System generates report
5. Admin exports or views report

**Postconditions**: Report is generated  
**Business Value**: Provides business insights for decision-making

---

#### UC-23: Configure Platform Settings
**Description**: Modify system-wide configurations  
**Actor**: Administrator  
**Preconditions**: Admin is authenticated  
**Main Flow**:
1. Admin accesses settings panel
2. Admin modifies settings (fees, policies, features)
3. System validates changes
4. System applies new settings

**Postconditions**: Platform settings are updated  
**Business Value**: Allows platform customization and optimization

---

### Payment Gateway Use Cases

#### UC-24: Process Payment
**Description**: Handle payment transaction for booking  
**Actor**: Payment Gateway (External System)  
**Triggered By**: Registered Guest (UC-6: Create Booking)  
**Main Flow**:
1. System sends payment request to gateway
2. Gateway validates payment method
3. Gateway processes transaction
4. Gateway returns success/failure status
5. System records transaction

**Postconditions**: Payment is processed  
**Business Value**: Enables secure financial transactions

---

#### UC-25: Process Refund
**Description**: Return funds for cancelled bookings  
**Actor**: Payment Gateway (External System)  
**Triggered By**: Registered Guest (UC-8: Cancel Booking) or Administrator  
**Main Flow**:
1. System calculates refund amount
2. System sends refund request to gateway
3. Gateway processes refund
4. Gateway returns confirmation
5. System updates transaction record

**Postconditions**: Refund is processed  
**Business Value**: Handles cancellations fairly

---

## Use Case Relationships

### Include Relationships
These represent mandatory sub-use cases that are always executed as part of another use case:

- **UC-6 (Create Booking) includes UC-24 (Process Payment)**: Every booking requires payment processing
- **UC-8 (Cancel Booking) includes UC-25 (Process Refund)**: Cancellations trigger refund processing
- **UC-3 (Register Account) includes UC-4 (Login)**: Registration automatically logs user in
- **UC-18 (Receive Payout) includes UC-24 (Process Payment)**: Payouts use payment gateway

### Extend Relationships
These represent optional use cases that may occur under specific conditions:

- **UC-6 (Create Booking) extends to UC-10 (Send Message)**: Guest may message host during booking
- **UC-6 (Create Booking) extends to UC-9 (Leave Review)**: After completion, guest can review
- **UC-12 (Create Property) extends to UC-14 (Manage Calendar)**: Host may set availability immediately
- **UC-21 (Resolve Disputes) extends to UC-25 (Process Refund)**: Dispute may result in refund

### Generalization Relationships
These show inheritance between actors or use cases:

- **Registered Guest and Host** generalize from **User**: Both are authenticated users with shared capabilities
- **All User Types** can perform **Login** and **Manage Profile**

---

## System Boundary

The system boundary defines what is inside the Airbnb Clone platform and what is external:

### Inside the System:
- User authentication and management
- Property listing management
- Booking and reservation handling
- Review and rating system
- Messaging and notifications
- Administrative functions
- Database operations
- Business logic processing

### Outside the System (External):
- Payment Gateway (Stripe, PayPal)
- Email service providers
- SMS notification services
- Cloud storage for images
- Third-party authentication (OAuth providers)
- Map and location services

---

## Use Case Diagram Structure

The use case diagram includes the following elements:

### System Boundary Box
A rectangle labeled "Airbnb Clone System" contains all use cases that are part of the platform.

### Actors (Stick Figures)
- Guest (Unauthenticated User) - positioned on left
- Registered Guest - positioned on left
- Host - positioned on left
- Administrator - positioned on top or right
- Payment Gateway - positioned on right (external)

### Use Cases (Ovals)
Each use case is represented as an oval inside the system boundary:
- Authentication: Register, Login
- Property Management: Create Listing, Update Listing, Manage Calendar
- Booking: Create Booking, View Bookings, Cancel Booking
- Reviews: Leave Review, Review Guest
- Communication: Send Message
- Payments: Process Payment, Process Refund, Receive Payout
- Admin: Manage Users, Moderate Content, Resolve Disputes, Generate Reports

### Relationships (Lines)
- **Solid lines**: Connect actors to use cases they can perform
- **Dashed arrows with <<include>>**: Show mandatory sub-use cases
- **Dashed arrows with <<extend>>**: Show optional extensions
- **Solid arrows**: Show generalization relationships

---

## Visual Documentation

![Use Case Diagram](Air%20Bnb%20Use%20Case%20Diagram.png)

*The diagram above illustrates all actors and their interactions with the Airbnb Clone system through various use cases. It shows the relationships between use cases and clearly defines the system boundary.*

---

## Use Case Priorities

### High Priority (MVP)
These use cases are essential for the minimum viable product:
- UC-3: Register Account
- UC-4: Login
- UC-2: Search Properties
- UC-6: Create Booking
- UC-12: Create Property Listing
- UC-24: Process Payment

### Medium Priority
These use cases enhance core functionality:
- UC-7: View Bookings
- UC-8: Cancel Booking
- UC-9: Leave Review
- UC-10: Send Message
- UC-13: Update Property Listing
- UC-14: Manage Availability Calendar
- UC-15: Respond to Booking Request

### Low Priority (Future Enhancement)
These use cases can be added in later iterations:
- UC-11: Add to Wishlist
- UC-17: View Earnings
- UC-22: Generate Reports
- Advanced admin features

---

## How to Create This Diagram in Visily

### Step 1: Set Up Canvas
1. Go to [Visily.ai](https://visily.ai)
2. Create a new project
3. Select "Use Case Diagram" template or blank canvas
4. Set canvas to landscape orientation

### Step 2: Add System Boundary
1. Draw a large rectangle to represent the system
2. Label it "Airbnb Clone System"
3. This will contain all use cases

### Step 3: Add Actors
1. Place stick figures outside the system boundary
2. Position Guest and Registered Guest on the left
3. Position Host on the left (below guests)
4. Position Admin on top or right
5. Position Payment Gateway on the right
6. Label each actor clearly

### Step 4: Add Use Cases
1. Inside the system boundary, add ovals for each use case
2. Group related use cases together:
   - Authentication (top left)
   - Property Management (center top)
   - Bookings (center)
   - Reviews (right)
   - Payments (right side)
   - Admin functions (top right)

### Step 5: Connect Actors to Use Cases
1. Draw lines from actors to use cases they perform
2. Guest → Browse, Search, Register, Login
3. Registered Guest → All guest use cases + booking, messaging, reviews
4. Host → Property management, respond to bookings, receive payouts
5. Admin → All administrative use cases
6. Payment Gateway → Payment processing use cases

### Step 6: Add Relationships
1. Add <<include>> relationships:
   - Create Booking → Process Payment
   - Cancel Booking → Process Refund
2. Add <<extend>> relationships:
   - Create Booking --extend--> Send Message
   - Create Booking --extend--> Leave Review (after completion)

### Step 7: Format and Polish
1. Use consistent colors (blue for authentication, green for bookings, etc.)
2. Align use cases for visual clarity
3. Ensure all text is readable
4. Add a legend explaining symbols if needed

### Step 8: Export
1. Click Export or Download
2. Select PNG format
3. Choose high resolution (300 DPI)
4. Save as "use-case-diagram.png"

---

## Best Practices Applied

1. **Clear Actor Identification**: All user types are clearly defined with distinct roles
2. **Focused Use Cases**: Each use case represents a single, complete goal
3. **Proper Naming**: Use cases start with verbs (Create, Manage, Process)
4. **System Boundary**: Clearly shows what's inside vs. outside the system
5. **Relationship Clarity**: Include and extend relationships are properly used
6. **Completeness**: All major system functions are represented
7. **User Perspective**: Use cases are written from the user's point of view

---

## Repository Structure

```
alx-airbnb-project-documentation/
├── features-and-functionalities/
│   ├── README.md
│   └── airbnb-features-overview.png
├── use-case-diagram/
│   ├── README.md (this file)
│   └── use-case-diagram.png
├── requirements/
├── design/
└── project-plan/
```

---

## Conclusion

This use case diagram provides a comprehensive view of how different actors interact with the Airbnb Clone system. It serves as a bridge between business requirements and technical implementation, ensuring that all stakeholder needs are captured and addressed in the system design.

The diagram will be used throughout the development process to:
- Guide API endpoint design
- Inform database schema decisions
- Define authorization rules
- Structure frontend user flows
- Plan testing scenarios
- Communicate with stakeholders

By clearly visualizing these interactions, the team can ensure that all necessary functionality is implemented and that the system meets user expectations.

---

## References

- UML Use Case Diagram Standards
- Airbnb Clone Project Requirements
- Software Engineering Best Practices
- Visily.ai Documentation

---

*This document is part of the Airbnb Clone project documentation. For questions or clarifications, please refer to the main project repository.*

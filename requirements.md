# Backend Feature Requirements Specifications

## 📋 Document Information

**Project**: Airbnb Clone Backend  
**Document Version**: 1.0  
**Last Updated**: November 2025  
**Author**: ALX Software Engineering Student

---

## Table of Contents

1. [User Authentication System](#1-user-authentication-system)
2. [Property Management System](#2-property-management-system)
3. [Booking System](#3-booking-system)

---

## 1. User Authentication System

### 1.1 Feature Overview

The User Authentication System provides secure user registration, login, and session management functionality. It supports both traditional email/password authentication and OAuth integration for social logins.

### 1.2 Functional Requirements

#### 1.2.1 User Registration

**Description**: Allow new users to create accounts as either guests or hosts.

**API Endpoint**:
```
POST /api/v1/auth/register
```

**Input Specifications**:
```json
{
  "email": "string (required, valid email format)",
  "password": "string (required, min 8 characters)",
  "first_name": "string (required, 2-50 characters)",
  "last_name": "string (required, 2-50 characters)",
  "phone_number": "string (optional, valid phone format)",
  "role": "enum (required, values: 'guest', 'host')",
  "date_of_birth": "date (required, user must be 18+)",
  "profile_picture": "string (optional, URL or base64)"
}
```

**Output Specifications**:

Success Response (201 Created):
```json
{
  "status": "success",
  "message": "User registered successfully",
  "data": {
    "user_id": "uuid",
    "email": "string",
    "first_name": "string",
    "last_name": "string",
    "role": "string",
    "created_at": "timestamp",
    "access_token": "string (JWT)",
    "refresh_token": "string"
  }
}
```

Error Response (400 Bad Request):
```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email already exists"
    }
  ]
}
```

**Validation Rules**:
- Email must be unique and valid format (RFC 5322)
- Password must contain:
  - Minimum 8 characters
  - At least one uppercase letter
  - At least one lowercase letter
  - At least one number
  - At least one special character
- Phone number must match international format (E.164)
- Date of birth must indicate user is 18 years or older
- First name and last name cannot contain numbers or special characters
- Role must be either 'guest' or 'host'

**Performance Criteria**:
- Response time: < 500ms for 95th percentile
- Database query time: < 100ms
- Password hashing time: < 200ms (using bcrypt with 12 rounds)
- Support 100 concurrent registration requests

---

#### 1.2.2 User Login

**Description**: Authenticate users and provide JWT tokens for session management.

**API Endpoint**:
```
POST /api/v1/auth/login
```

**Input Specifications**:
```json
{
  "email": "string (required, valid email format)",
  "password": "string (required)",
  "remember_me": "boolean (optional, default: false)"
}
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "message": "Login successful",
  "data": {
    "user_id": "uuid",
    "email": "string",
    "first_name": "string",
    "last_name": "string",
    "role": "string",
    "profile_picture": "string",
    "access_token": "string (JWT, expires in 1 hour)",
    "refresh_token": "string (expires in 7 days or 30 days if remember_me)",
    "token_type": "Bearer",
    "expires_in": "integer (seconds)"
  }
}
```

Error Response (401 Unauthorized):
```json
{
  "status": "error",
  "message": "Invalid credentials",
  "error_code": "INVALID_CREDENTIALS"
}
```

Error Response (429 Too Many Requests):
```json
{
  "status": "error",
  "message": "Too many login attempts. Please try again in 15 minutes",
  "error_code": "RATE_LIMIT_EXCEEDED",
  "retry_after": "integer (seconds)"
}
```

**Validation Rules**:
- Email must be in valid format
- Password is required (no length validation at login)
- Rate limiting: Maximum 5 failed attempts per email per 15 minutes
- Account lockout after 10 failed attempts within 1 hour

**Performance Criteria**:
- Response time: < 300ms for 95th percentile
- Database query time: < 50ms
- Password verification time: < 200ms
- Support 1000 concurrent login requests
- Token generation time: < 50ms

**Security Requirements**:
- Use bcrypt for password comparison
- Implement exponential backoff for failed attempts
- Log all login attempts (successful and failed)
- IP-based rate limiting
- Support 2FA (optional, future enhancement)

---

#### 1.2.3 OAuth Authentication

**Description**: Allow users to authenticate using third-party providers (Google, Facebook).

**API Endpoints**:
```
GET /api/v1/auth/oauth/google
GET /api/v1/auth/oauth/facebook
POST /api/v1/auth/oauth/callback
```

**Input Specifications** (Callback):
```json
{
  "provider": "enum (required, values: 'google', 'facebook')",
  "code": "string (required, OAuth authorization code)",
  "state": "string (required, CSRF token)"
}
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "message": "OAuth authentication successful",
  "data": {
    "user_id": "uuid",
    "email": "string",
    "first_name": "string",
    "last_name": "string",
    "profile_picture": "string",
    "provider": "string",
    "is_new_user": "boolean",
    "access_token": "string (JWT)",
    "refresh_token": "string"
  }
}
```

**Validation Rules**:
- State parameter must match the one generated during OAuth initiation
- Authorization code must be valid and not expired
- Provider must be supported ('google' or 'facebook')
- Email from OAuth provider must be verified

**Performance Criteria**:
- Response time: < 800ms (including external OAuth API call)
- Token exchange time: < 500ms
- Database operations: < 100ms

---

#### 1.2.4 Token Refresh

**Description**: Refresh expired access tokens using refresh tokens.

**API Endpoint**:
```
POST /api/v1/auth/refresh
```

**Input Specifications**:
```json
{
  "refresh_token": "string (required)"
}
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "data": {
    "access_token": "string (JWT)",
    "refresh_token": "string (new refresh token)",
    "token_type": "Bearer",
    "expires_in": "integer (seconds)"
  }
}
```

**Validation Rules**:
- Refresh token must be valid and not expired
- Refresh token must not be revoked
- Refresh token can only be used once (rotation)

**Performance Criteria**:
- Response time: < 100ms
- Support 500 concurrent refresh requests

---

#### 1.2.5 User Logout

**Description**: Invalidate user session and revoke tokens.

**API Endpoint**:
```
POST /api/v1/auth/logout
```

**Headers**:
```
Authorization: Bearer <access_token>
```

**Input Specifications**:
```json
{
  "refresh_token": "string (required)"
}
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "message": "Logged out successfully"
}
```

**Performance Criteria**:
- Response time: < 100ms
- Token revocation time: < 50ms

---

### 1.3 Technical Requirements

#### 1.3.1 Database Schema

**Users Table**:
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255),
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    phone_number VARCHAR(20),
    role VARCHAR(20) NOT NULL CHECK (role IN ('guest', 'host', 'admin')),
    date_of_birth DATE NOT NULL,
    profile_picture TEXT,
    is_email_verified BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    oauth_provider VARCHAR(50),
    oauth_provider_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP,
    CONSTRAINT age_check CHECK (date_of_birth <= CURRENT_DATE - INTERVAL '18 years')
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_oauth ON users(oauth_provider, oauth_provider_id);
```

**Refresh Tokens Table**:
```sql
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    is_revoked BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent TEXT
);

CREATE INDEX idx_refresh_tokens_user ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_hash ON refresh_tokens(token_hash);
```

#### 1.3.2 JWT Token Structure

**Access Token Claims**:
```json
{
  "sub": "user_id (uuid)",
  "email": "string",
  "role": "string",
  "type": "access",
  "iat": "issued_at (timestamp)",
  "exp": "expires_at (timestamp, 1 hour from iat)",
  "jti": "token_id (uuid)"
}
```

**Refresh Token Claims**:
```json
{
  "sub": "user_id (uuid)",
  "type": "refresh",
  "iat": "issued_at (timestamp)",
  "exp": "expires_at (timestamp, 7 or 30 days from iat)",
  "jti": "token_id (uuid)"
}
```

#### 1.3.3 Security Measures

- Password hashing: bcrypt with cost factor 12
- JWT signing algorithm: RS256 (RSA with SHA-256)
- Token storage: Redis for blacklisting and session management
- Rate limiting: Redis-based with sliding window algorithm
- HTTPS only for all authentication endpoints
- CORS configuration for allowed origins
- Content Security Policy headers
- XSS protection headers

---

### 1.4 Non-Functional Requirements

- **Availability**: 99.9% uptime
- **Scalability**: Support 10,000 concurrent users
- **Data Retention**: Keep login logs for 90 days
- **Compliance**: GDPR compliant for user data
- **Monitoring**: Log all authentication events
- **Backup**: Daily automated backups of user data

---

## 2. Property Management System

### 2.1 Feature Overview

The Property Management System allows hosts to create, update, delete, and manage property listings with detailed information including amenities, pricing, availability, and images.

### 2.2 Functional Requirements

#### 2.2.1 Create Property Listing

**Description**: Allow hosts to create new property listings.

**API Endpoint**:
```
POST /api/v1/properties
```

**Headers**:
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Input Specifications**:
```json
{
  "title": "string (required, 10-100 characters)",
  "description": "string (required, 50-5000 characters)",
  "property_type": "enum (required, values: 'apartment', 'house', 'villa', 'cottage', 'cabin', 'studio')",
  "location": {
    "address": "string (required)",
    "city": "string (required)",
    "state": "string (required)",
    "country": "string (required)",
    "postal_code": "string (required)",
    "latitude": "float (required, -90 to 90)",
    "longitude": "float (required, -180 to 180)"
  },
  "price_per_night": "decimal (required, min: 1.00, max: 100000.00)",
  "currency": "string (required, ISO 4217 code, e.g., 'USD')",
  "max_guests": "integer (required, min: 1, max: 50)",
  "bedrooms": "integer (required, min: 0)",
  "beds": "integer (required, min: 1)",
  "bathrooms": "float (required, min: 0.5)",
  "amenities": [
    "string (array of amenity IDs or names)"
  ],
  "house_rules": "string (optional, max 2000 characters)",
  "cancellation_policy": "enum (required, values: 'flexible', 'moderate', 'strict')",
  "check_in_time": "time (required, format: HH:MM)",
  "check_out_time": "time (required, format: HH:MM)",
  "minimum_stay": "integer (required, min: 1, default: 1)",
  "maximum_stay": "integer (optional, min: minimum_stay)",
  "images": [
    {
      "url": "string (required, valid URL or base64)",
      "caption": "string (optional)",
      "is_primary": "boolean (default: false)"
    }
  ]
}
```

**Output Specifications**:

Success Response (201 Created):
```json
{
  "status": "success",
  "message": "Property created successfully",
  "data": {
    "property_id": "uuid",
    "host_id": "uuid",
    "title": "string",
    "slug": "string (URL-friendly)",
    "property_type": "string",
    "location": {
      "address": "string",
      "city": "string",
      "state": "string",
      "country": "string",
      "postal_code": "string",
      "coordinates": {
        "latitude": "float",
        "longitude": "float"
      }
    },
    "price_per_night": "decimal",
    "currency": "string",
    "max_guests": "integer",
    "bedrooms": "integer",
    "beds": "integer",
    "bathrooms": "float",
    "amenities": ["array of amenity objects"],
    "status": "enum (pending_approval, active, inactive)",
    "created_at": "timestamp",
    "updated_at": "timestamp"
  }
}
```

Error Response (400 Bad Request):
```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    {
      "field": "price_per_night",
      "message": "Price must be greater than 0"
    }
  ]
}
```

Error Response (403 Forbidden):
```json
{
  "status": "error",
  "message": "Only hosts can create property listings",
  "error_code": "INSUFFICIENT_PERMISSIONS"
}
```

**Validation Rules**:
- User role must be 'host'
- Title must be unique per host
- At least one image is required, maximum 20 images
- At least one image must be marked as primary
- Check-out time must be after check-in time
- Maximum stay must be greater than minimum stay
- Price must be positive
- Coordinates must be valid geographic coordinates
- Amenities must exist in the amenities reference table
- Property type must be from predefined list

**Performance Criteria**:
- Response time: < 800ms for 95th percentile
- Image upload time: < 2 seconds per image
- Database insertion time: < 200ms
- Support 50 concurrent property creation requests

---

#### 2.2.2 Get Property Details

**Description**: Retrieve detailed information about a specific property.

**API Endpoint**:
```
GET /api/v1/properties/{property_id}
```

**Input Specifications**:

Query Parameters:
```
?include=reviews,host,availability
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "data": {
    "property_id": "uuid",
    "host": {
      "host_id": "uuid",
      "first_name": "string",
      "profile_picture": "string",
      "member_since": "date",
      "response_rate": "percentage",
      "is_superhost": "boolean"
    },
    "title": "string",
    "description": "string",
    "property_type": "string",
    "location": {
      "address": "string",
      "city": "string",
      "state": "string",
      "country": "string",
      "coordinates": {
        "latitude": "float",
        "longitude": "float"
      }
    },
    "price_per_night": "decimal",
    "currency": "string",
    "max_guests": "integer",
    "bedrooms": "integer",
    "beds": "integer",
    "bathrooms": "float",
    "amenities": [
      {
        "amenity_id": "uuid",
        "name": "string",
        "icon": "string"
      }
    ],
    "images": [
      {
        "image_id": "uuid",
        "url": "string",
        "caption": "string",
        "is_primary": "boolean"
      }
    ],
    "house_rules": "string",
    "cancellation_policy": "string",
    "check_in_time": "time",
    "check_out_time": "time",
    "minimum_stay": "integer",
    "maximum_stay": "integer",
    "rating": {
      "average": "float (1.0-5.0)",
      "count": "integer",
      "breakdown": {
        "cleanliness": "float",
        "accuracy": "float",
        "communication": "float",
        "location": "float",
        "check_in": "float",
        "value": "float"
      }
    },
    "total_reviews": "integer",
    "availability": {
      "is_available": "boolean",
      "next_available_date": "date"
    },
    "created_at": "timestamp",
    "updated_at": "timestamp"
  }
}
```

Error Response (404 Not Found):
```json
{
  "status": "error",
  "message": "Property not found",
  "error_code": "PROPERTY_NOT_FOUND"
}
```

**Performance Criteria**:
- Response time: < 200ms for 95th percentile
- Cache hit rate: > 80% for frequently accessed properties
- Database query time: < 100ms
- Support 1000 concurrent read requests

---

#### 2.2.3 Update Property Listing

**Description**: Allow hosts to update their property listings.

**API Endpoint**:
```
PUT /api/v1/properties/{property_id}
PATCH /api/v1/properties/{property_id}
```

**Headers**:
```
Authorization: Bearer <access_token>
```

**Input Specifications**:

For PATCH (partial update), all fields are optional:
```json
{
  "title": "string (optional)",
  "description": "string (optional)",
  "price_per_night": "decimal (optional)",
  "max_guests": "integer (optional)",
  "amenities": ["array (optional)"],
  "status": "enum (optional, values: 'active', 'inactive')"
}
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "message": "Property updated successfully",
  "data": {
    "property_id": "uuid",
    "updated_fields": ["array of field names"],
    "updated_at": "timestamp"
  }
}
```

Error Response (403 Forbidden):
```json
{
  "status": "error",
  "message": "You can only update your own properties",
  "error_code": "UNAUTHORIZED_UPDATE"
}
```

**Validation Rules**:
- User must be the property owner
- Cannot update property with active bookings (except status and price)
- All validation rules from creation apply to updated fields
- Status change from 'active' to 'inactive' requires confirmation

**Performance Criteria**:
- Response time: < 400ms
- Database update time: < 150ms
- Cache invalidation time: < 50ms

---

#### 2.2.4 Delete Property Listing

**Description**: Allow hosts to delete their property listings.

**API Endpoint**:
```
DELETE /api/v1/properties/{property_id}
```

**Headers**:
```
Authorization: Bearer <access_token>
```

**Query Parameters**:
```
?force=false (boolean, default: false)
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "message": "Property deleted successfully",
  "data": {
    "property_id": "uuid",
    "deleted_at": "timestamp"
  }
}
```

Error Response (409 Conflict):
```json
{
  "status": "error",
  "message": "Cannot delete property with active bookings",
  "error_code": "ACTIVE_BOOKINGS_EXIST",
  "data": {
    "active_bookings_count": "integer",
    "next_checkout_date": "date"
  }
}
```

**Validation Rules**:
- User must be the property owner
- Cannot delete with active or upcoming bookings (unless force=true)
- Soft delete by default (can be recovered within 30 days)
- Hard delete after 30 days or if force=true

**Performance Criteria**:
- Response time: < 300ms
- Database soft delete time: < 100ms

---

#### 2.2.5 Search and Filter Properties

**Description**: Search properties with advanced filtering options.

**API Endpoint**:
```
GET /api/v1/properties/search
```

**Input Specifications**:

Query Parameters:
```
?location=string (city or coordinates)
&check_in=date (YYYY-MM-DD)
&check_out=date (YYYY-MM-DD)
&guests=integer
&min_price=decimal
&max_price=decimal
&property_type=string (comma-separated)
&amenities=string (comma-separated amenity IDs)
&bedrooms=integer
&bathrooms=integer
&instant_book=boolean
&min_rating=float (1.0-5.0)
&sort_by=string (price_asc, price_desc, rating, distance)
&page=integer (default: 1)
&limit=integer (default: 20, max: 100)
```

**Output Specifications**:

Success Response (200 OK):
```json
{
  "status": "success",
  "data": {
    "properties": [
      {
        "property_id": "uuid",
        "title": "string",
        "property_type": "string",
        "location": {
          "city": "string",
          "state": "string",
          "country": "string"
        },
        "price_per_night": "decimal",
        "currency": "string",
        "rating": "float",
        "total_reviews": "integer",
        "primary_image": "string (URL)",
        "max_guests": "integer",
        "bedrooms": "integer",
        "bathrooms": "float",
        "is_available": "boolean",
        "distance": "float (km, if location search)"
      }
    ],
    "pagination": {
      "current_page": "integer",
      "total_pages": "integer",
      "total_results": "integer",
      "per_page": "integer",
      "has_next": "boolean",
      "has_previous": "boolean"
    },
    "filters_applied": {
      "location": "string",
      "date_range": "string",
      "guests": "integer",
      "price_range": "string"
    }
  }
}
```

**Validation Rules**:
- Check-out date must be after check-in date
- Maximum date range: 365 days
- Guests must be at least 1
- Price range: min_price < max_price
- Page must be positive integer
- Limit must be between 1 and 100

**Performance Criteria**:
- Response time: < 500ms for 95th percentile
- Database query time: < 300ms
- Support 500 concurrent search requests
- Use Elasticsearch for full-text search (optional)
- Cache popular searches for 5 minutes

---

### 2.3 Technical Requirements

#### 2.3.1 Database Schema

**Properties Table**:
```sql
CREATE TABLE properties (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(100) NOT NULL,
    slug VARCHAR(120) UNIQUE NOT NULL,
    description TEXT NOT NULL,
    property_type VARCHAR(50) NOT NULL,
    address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    country VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    price_per_night DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    max_guests INTEGER NOT NULL,
    bedrooms INTEGER NOT NULL,
    beds INTEGER NOT NULL,
    bathrooms DECIMAL(3, 1) NOT NULL,
    house_rules TEXT,
    cancellation_policy VARCHAR(20) NOT NULL,
    check_in_time TIME NOT NULL,
    check_out_time TIME NOT NULL,
    minimum_stay INTEGER NOT NULL DEFAULT 1,
    maximum_stay INTEGER,
    status VARCHAR(20) NOT NULL DEFAULT 'pending_approval',
    is_deleted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    CONSTRAINT price_positive CHECK (price_per_night > 0),
    CONSTRAINT guests_positive CHECK (max_guests > 0),
    CONSTRAINT checkout_after_checkin CHECK (check_out_time > check_in_time)
);

CREATE INDEX idx_properties_host ON properties(host_id);
CREATE INDEX idx_properties_location ON properties(city, country);
CREATE INDEX idx_properties_price ON properties(price_per_night);
CREATE INDEX idx_properties_coordinates ON properties USING GIST (
    ll_to_earth(latitude, longitude)
);
CREATE INDEX idx_properties_status ON properties(status) WHERE is_deleted = FALSE;
```

**Property Images Table**:
```sql
CREATE TABLE property_images (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID NOT NULL REFERENCES properties(id) ON DELETE CASCADE,
    url TEXT NOT NULL,
    caption VARCHAR(200),
    is_primary BOOLEAN DEFAULT FALSE,
    display_order INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_property_images_property ON property_images(property_id);
```

**Property Amenities Table** (Many-to-Many):
```sql
CREATE TABLE property_amenities (
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    amenity_id UUID REFERENCES amenities(id) ON DELETE CASCADE,
    PRIMARY KEY (property_id, amenity_id)
);
```

#### 2.3.2 File Storage

- Store images in AWS S3 or Cloudinary
- Image formats: JPEG, PNG, WebP
- Maximum image size: 10MB per image
- Generate thumbnails: 200x200, 400x400, 800x800
- Use CDN for image delivery
- Implement lazy loading for images

---

### 2.4 Non-Functional Requirements

- **Availability**: 99.95% uptime
- **Scalability**: Support 100,000 active listings
- **Search Performance**: < 500ms response time
- **Image Loading**: < 2 seconds for all property images
- **Data Retention**: Keep deleted properties for 30 days
- **Backup**: Real-time replication and daily backups

---

## 3. Booking System

### 3.1 Feature Overview

The Booking System manages the entire booking lifecycle including creation, confirmation, cancellation, and status tracking. It integrates with the payment system and ensures no double bookings occur.

### 3.2 Functional Requirements

#### 3.2.1 Create Booking

**Description**: Allow guests to create a booking for a property.

**API Endpoint**:
```
POST /api/v1/bookings
```

**Headers**:
```
Authorization: Bearer <access_token>
```

**Input Specifications**:
```json
{
  "property_id": "uuid (required)",
  "check_in_date": "date (required, format: YYYY-MM-DD)",
  "check_out_date": "date (required, format: YYYY-MM-DD)",
  "number_of_guests": "integer (required, min: 1)",
  "guest_details": {
    "adults": "integer (required, min: 1)",
    "children": "integer (optional, min: 0)",
    "infants": "integer (optional, min: 0)"
  },
  "special_requests": "string (optional, max: 500 characters)",
  "payment_method_id": "string (required, Stripe payment method ID)",
  "promo_code": "string (optional)"
}
```

**Output Specifications**:

Success Response (201 Created):
```json
{
  "status": "success",
  "message": "Booking created successfully",
  "data": {
    "booking_id": "uuid",
    "property_id": "uuid",
    "guest_id": "uuid",
    "host_id": "uuid",
    "check_in_date": "date",
    "check_out_date": "date",
    "number_of_nights": "integer",
    "number_of_guests": "integer",
    "booking_status": "enum (pending, confirmed)",
    "pricing": {
      "price_per_night": "decimal",
      "subtotal": "decimal",
      "service_fee": "decimal",
      "tax": "decimal",
      "discount": "decimal",
      "total_amount": "decimal",
      "currency": "string"
    },
    "payment_status": "enum (pending, completed, failed)",
    "cancellation_policy": "string",
    "created_at": "timestamp",
    "confirmation_code": "string (8-character alphanumeric)"
  }
}
```

Error Response (409 Conflict):
```json
{
  "status": "error",
  "message": "Property is not available for selected dates",
  "error_code": "DATES_UNAVAILABLE",
  "data": {
    "unavailable_dates": ["array of dates"],
    "next_available_date": "date"
  }
}
```

Error Response (400 Bad Request):
```json
{
  "status": "error",
  "message

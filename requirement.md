1. User Authentication

Objective:
Enable secure user registration, login, and session management for both guests and hosts.
Functional Requirements:

    User Registration:

        Input: name, email, password, user type (guest or host).

        Password hashing with bcrypt.

        Verification email with secure token before activating account.

    Login:

        Input: email, password.

        JWT-based token issuance upon successful login.

    Logout:

        Token invalidation on logout.

    Email Verification:

        Token expires in 24 hours.

        Verifies and activates user account.

Non-Functional Requirements:

    OAuth2 login with Google (optional).

    Rate-limiting to prevent brute-force attacks.

    Audit logging for login/logout actions.

Database Requirements:

    users table fields:

        id, name, email, password_hash, user_type, email_verified, created_at, updated_at.

API Endpoints:
Endpoint	Method	Description
/api/register	POST	Create a new user account
/api/login	POST	Authenticate user
/api/logout	POST	End user session
/api/verify-email	GET	Verify email using token
🏘️ 2. Property Management

Objective:
Allow hosts to manage property listings and guests to browse them with filters and location-based search.
Functional Requirements:

    Create Property:

        Fields: title, description, price, address, amenities, images, max guests, calendar.

    Edit/Delete:

        Only authenticated hosts can modify/delete their properties.

    Search & Filter:

        Search by location, price range, date availability, rating.

    Image Upload:

        Accepts JPG/PNG, size < 5MB.

        Stores in AWS S3 or Cloudinary.

Non-Functional Requirements:

    Listings indexed for full-text search.

    CDN-backed image delivery.

    Property availability integrated with booking calendar.

Database Requirements:

    properties table fields:

        id, host_id, title, description, location, price_per_night, max_guests, images, amenities, created_at, updated_at.

API Endpoints:
Endpoint	Method	Description
/api/properties	POST	Create new property
/api/properties/:id	PUT	Update existing property
/api/properties/:id	DELETE	Delete a property
/api/properties	GET	Search and filter properties
📆 3. Booking System

Objective:
Manage bookings between guests and hosts with accurate availability and transaction tracking.
Functional Requirements:

    Book Property:

        Requires login.

        Inputs: check-in/out dates, guest count.

        Availability check before confirming.

        Booking is held for 15 minutes for payment processing.

    Cancel Booking:

        Guests can cancel before a specific time window.

        Hosts can cancel under defined policy.

    View Bookings:

        Guests see upcoming/past bookings.

        Hosts see bookings per property.

Non-Functional Requirements:

    Date-range conflict detection.

    Automated calendar updates upon confirmation.

    Email/SMS notifications for confirmations.

Database Requirements:

    bookings table fields:

        id, guest_id, property_id, start_date, end_date, total_price, status, payment_status, created_at.

API Endpoints:
Endpoint	Method	Description
/api/bookings	POST	Book a property
/api/bookings/:id	GET	View booking details
/api/bookings/my	GET	List all my bookings
/api/bookings/:id/cancel	POST	Cancel a booking
💳 4. Payment Integration (MPGS – Mastercard Payment Gateway Services)

Objective:
Integrate secure online payments using MPGS to confirm bookings and process refunds.
Functional Requirements:

    Booking Payment:

        Once a booking is created, MPGS payment page is invoked.

        Secure redirection to hosted checkout page (HPP).

    Payment Confirmation:

        MPGS callback/notification updates booking payment_status.

    Refund Handling:

        If a booking is cancelled, a refund is initiated through MPGS.

    Transaction Logging:

        Store all payment attempts and transaction responses.

Non-Functional Requirements:

    PCI DSS compliance.

    Use of hosted payment pages (HPP) to avoid direct card handling.

    Support for credit/debit cards and digital wallets.

Database Requirements:

    payments table fields:

        id, booking_id, user_id, amount, currency, status, transaction_id, response_code, created_at.

API Endpoints:
Endpoint	Method	Description
/api/payments/initiate	POST	Start MPGS payment for booking
/api/payments/callback	POST	MPGS notifies payment status
/api/payments/refund/:id	POST	Process refund for a booking

# Software Testing Test Case Document
**Project Name:** Online Food Delivery System  
**Technology Stack:** Jakarta EE / Java Servlets, JSP, JDBC, MySQL, HTML5/CSS3, JavaScript (AJAX/Fetch API), OpenRouter AI API  
**Project Github Repository:** https://github.com/adhil-vt/Online-Food-Delivery-System  

---

## 1. Project Modules Identified

Based on a detailed architectural inspection of the repository source code (`.jsp` files, Java Servlets, Java Services, and database queries), the following **12 functional and technical modules** were identified:

1. **User Registration Module (`register.jsp`, `RegisterServlet.java`)**
   - Handles creation of customer accounts. Converts email inputs to lowercase, verifies duplicate email records in the database (`users` table), enforces `CUSTOMER` role creation, and handles HTML5 form validations.

2. **User Login & Session Management Module (`login.jsp`, `LoginServlet.java`, `logout.jsp`)**
   - Handles credential verification (`UserService.java`), role-based redirection (`ADMIN` -> `admin.jsp`, `CUSTOMER` -> `customer.jsp`), user presence tracking (`last_active` timestamp update), and session invalidation upon logout (`is_online = FALSE`).

3. **Customer Dashboard & Presence Module (`customer.jsp`, `updateActivity.jsp`)**
   - Renders personalized navigation links for customers (Restaurants, Order History, Cart, Logout) and runs a background heartbeat script (`updateActivity.jsp`) every 1.5 seconds to refresh online presence status.

4. **Restaurant & Menu Browsing Module (`restaurants.jsp`, `menu.jsp`)**
   - Displays available restaurants (`is_available = TRUE`) and allows browsing menu items filtered by category (`Veg`, `Non-Veg`, `Drinks`, `Dessert`), sorting by price (`Low to High`, `High to Low`), and visualizing out-of-stock items (`is_available = FALSE`).

5. **Cart Operations Module (`addToCart.jsp`, `cart.jsp`, `updateCart.jsp`, `CartItem.java`)**
   - Manages session-based cart storage (`List<CartItem>`). Supports adding menu items, quantity increments/decrements, item removal, total price calculation, empty cart handling, and cart state persistence across page navigation.

6. **Order Placement Module (`placeOrder.jsp`, `OrderService.java`)**
   - Reads session cart, verifies user authentication, inserts order metadata into the `orders` table (`status = 'Pending'`, `payment_status = 'PENDING'`), populates relational `order_items`, clears the cart, and forwards to payment.

7. **Payment Processing Module (`payment.jsp`, `processPayment.jsp`)**
   - Provides simulated payment method selection (UPI, Card, Cash), updates order payment status to `PAID` in the database, and generates printable HTML invoices using CSS print media rules (`window.print()`).

8. **Order Details & Order History Module (`orderHistory.jsp`, `orderDetails.jsp`)**
   - Customer-facing order tracking displaying past orders sorted by date descending, status indicators (`Pending`, `Preparing`, `Delivered`), visual step-by-step progress bars, and itemized cost breakdowns.

9. **Admin Dashboard & Live Analytics Module (`admin.jsp`, `liveStats.jsp`, `getUsers.jsp`)**
   - Administrative command center providing real-time order statistics, revenue metrics (Total & Today's Revenue), top-selling restaurant metrics, online user counts, and modal dialogs listing registered customers and staff.

10. **Admin Order & Status Management Module (`adminOrders.jsp`, `adminUpdateStatus.jsp`)**
    - Allows administrators to view all placed orders, inspect order details via embedded modals, and dynamically update order fulfillment stages (`Pending` -> `Preparing` -> `Delivered`).

11. **Restaurant & Menu Management (Draft & Staging Pattern) (`manageRestaurants.jsp`, `manageMenu.jsp`, `publishChanges.jsp`, `publishMenuChanges.jsp`)**
    - Admin CRUD operations for restaurants and menu items. Uses session-staged draft preview maps (`previewMap`, `deleteMap`) allowing admins to toggle availability or mark items for deletion before publishing live to the database.

12. **AI Food Assistant Chatbot Module (`chatAPI.jsp`)**
    - Embedded AI chat drawer utilizing OpenRouter API (`openai/gpt-3.5-turbo`) to offer natural language food recommendations, filter menu options dynamically, and directly add food items to the customer's cart via chat prompts.

---

## 2. Complete Test Case Table

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|---|---|
| **TC-001** | Registration | Register with valid customer details | User is on `register.jsp` page | 1. Enter valid Name<br>2. Enter unique Email<br>3. Enter valid Password<br>4. Click 'Register' button | Name: `John Doe`<br>Email: `john.doe@example.com`<br>Password: `Password123` | Record inserted into `users` table with `role='CUSTOMER'`, and user is redirected to `login.jsp`. | As expected | Pass |
| **TC-002** | Registration | Register with empty required fields | User is on `register.jsp` page | 1. Leave Email field blank<br>2. Enter Name and Password<br>3. Click 'Register' button | Name: `John Doe`<br>Email: `""`<br>Password: `Password123` | Browser HTML5 validation prevents form submission and highlights the empty Email field. | As expected | Pass |
| **TC-003** | Registration | Register with invalid email format | User is on `register.jsp` page | 1. Enter valid Name<br>2. Enter malformed Email<br>3. Enter Password<br>4. Click 'Register' button | Name: `John Doe`<br>Email: `john.doeexample.com`<br>Password: `Password123` | Browser HTML5 validation blocks submission displaying message "Please include an '@' in the email address". | As expected | Pass |
| **TC-004** | Registration | Register with an already registered email | User on `register.jsp`; email `john.doe@example.com` exists in DB | 1. Enter Name<br>2. Enter existing Email<br>3. Enter Password<br>4. Click 'Register' button | Name: `John Duplicate`<br>Email: `john.doe@example.com`<br>Password: `Password123` | `RegisterServlet` queries DB, detects existing email, prints `"Email already exists!"` and halts registration. | As expected | Pass |
| **TC-005** | Registration | Register with a single-character password | User is on `register.jsp` page | 1. Enter Name<br>2. Enter unique Email<br>3. Enter single char Password `1`<br>4. Click 'Register' button | Name: `Short Pass User`<br>Email: `shortpass@example.com`<br>Password: `1` | Registration processes successfully as no backend password length constraint exists in `RegisterServlet`. (Boundary Validation) | As expected | Pass |
| **TC-006** | Registration | Register with special characters in Name | User is on `register.jsp` page | 1. Enter Name containing special characters<br>2. Enter unique Email<br>3. Enter Password<br>4. Click 'Register' | Name: `Jane @O'Connor#$`<br>Email: `jane.special@example.com`<br>Password: `Password123` | Input sanitized by Prepared Statement, record created in DB, user redirected to `login.jsp`. | As expected | Pass |
| **TC-007** | Registration | Email case-insensitivity handling during registration | User is on `register.jsp` page | 1. Enter Name<br>2. Enter uppercase/mixed Email<br>3. Enter Password<br>4. Click 'Register' | Name: `Case Test`<br>Email: `John.CaseTest@Example.COM`<br>Password: `Password123` | `RegisterServlet` converts email to lowercase (`.toLowerCase()`), storing `john.casetest@example.com` in DB. | As expected | Pass |
| **TC-008** | Registration | Boundary test with excessive input length (300+ chars) | User is on `register.jsp` page | 1. Enter 300-character string in Name<br>2. Enter valid Email & Password<br>3. Click 'Register' | Name: `'A'*300`<br>Email: `longname@example.com`<br>Password: `Password123` | Servlet attempts insertion; if column limit is exceeded, caught Exception displays `"Registration Failed"`. | As expected | Pass |
| **TC-009** | Login | Login with valid Customer credentials | Customer account exists in DB (`customer@example.com` / `pass123`) | 1. Open `login.jsp`<br>2. Enter valid Customer email<br>3. Enter valid password<br>4. Click 'Login' button | Email: `customer@example.com`<br>Password: `pass123` | `LoginServlet` authenticates user, sets `userEmail` in session, updates `last_active`, and redirects to `customer.jsp`. | As expected | Pass |
| **TC-010** | Login | Login with valid Admin credentials | Admin account exists in DB (`admin@example.com` / `admin123`, role=`ADMIN`) | 1. Open `login.jsp`<br>2. Enter valid Admin email<br>3. Enter valid password<br>4. Click 'Login' button | Email: `admin@example.com`<br>Password: `admin123` | `LoginServlet` authenticates user, sets session, updates `last_active`, and redirects to `admin.jsp`. | As expected | Pass |
| **TC-011** | Login | Login with non-registered email address | User is on `login.jsp` page | 1. Enter unregistered email<br>2. Enter any password<br>3. Click 'Login' button | Email: `notfound@example.com`<br>Password: `password123` | `UserService.loginUser()` returns `null`. `LoginServlet` outputs JSON response `{"status":"invalid"}`. | As expected | Pass |
| **TC-012** | Login | Login with valid email but incorrect password | User account exists in DB | 1. Enter valid email<br>2. Enter wrong password<br>3. Click 'Login' button | Email: `customer@example.com`<br>Password: `wrongpassword` | Authentication fails in `UserService`. Servlet outputs JSON response `{"status":"invalid"}`. | As expected | Pass |
| **TC-013** | Login | Login with empty email or password fields | User is on `login.jsp` page | 1. Leave email or password blank<br>2. Click 'Login' button | Email: `""`<br>Password: `""` | HTML5 `required` attribute blocks submission. Backend fallback returns JSON `{"status":"empty"}`. | As expected | Pass |
| **TC-014** | Login | Login with leading and trailing spaces in email | User account exists in DB | 1. Enter email wrapped with spaces<br>2. Enter valid password<br>3. Click 'Login' button | Email: `" customer@example.com "`<br>Password: `pass123` | `LoginServlet` trims email parameter (`email.trim()`), successfully authenticates, and redirects to dashboard. | As expected | Pass |
| **TC-015** | Logout | User logout and session destruction | User is logged in and on `customer.jsp` | 1. Click 'Logout' button | N/A | Request sent to `logout.jsp`. DB updates `is_online=FALSE`, `session.invalidate()` destroys session, user redirected to `login.jsp`. | As expected | Pass |
| **TC-016** | Logout | Access protected page via browser Back button after logout | User has just logged out | 1. Click browser Back button to return to `customer.jsp` | N/A | Cached UI may temporarily render, but performing any server action redirects user to `login.jsp` due to destroyed session. | As expected | Pass |
| **TC-017** | Restaurant & Menu | View available restaurants list | Customer is logged in on `restaurants.jsp` | 1. Navigate to `restaurants.jsp` | N/A | Executes `SELECT * FROM restaurants WHERE is_available = TRUE`. Renders restaurant cards with images, names, and locations. | As expected | Pass |
| **TC-018** | Restaurant & Menu | View menu items for a specific restaurant | User is on `restaurants.jsp` page | 1. Click 'View Menu' on Restaurant ID 1 | `restaurantId=1` | Navigates to `menu.jsp?restaurantId=1`. DB queries `menu_items` for restaurant 1 and displays item cards. | As expected | Pass |
| **TC-019** | Restaurant & Menu | Filter menu items by category (`Veg`) | User is on `menu.jsp?restaurantId=1` | 1. Select 'Veg' from Category dropdown | Category: `Veg` | JavaScript `filterItems()` filters DOM cards, displaying only cards matching `data-category="Veg"`. | As expected | Pass |
| **TC-020** | Restaurant & Menu | Sort menu items by price (Low to High) | User is on `menu.jsp?restaurantId=1` | 1. Select 'Low → High' from Price dropdown | Price Sort: `low` | JavaScript `filterItems()` sorts menu card DOM elements ascending by `data-price` attribute value. | As expected | Pass |
| **TC-021** | Restaurant & Menu | Sort menu items by price (High to Low) | User is on `menu.jsp?restaurantId=1` | 1. Select 'High → Low' from Price dropdown | Price Sort: `high` | JavaScript sorts menu card DOM elements descending by `data-price` attribute value. | As expected | Pass |
| **TC-022** | Restaurant & Menu | Display out-of-stock / unavailable menu item | Menu item in DB has `is_available = FALSE` | 1. View menu containing disabled item | Restaurant ID with unavailable item | Item card renders with `.disabled` class (grayscale) and displays "❌ Out of Stock" badge without 'Add to Cart' button. | As expected | Pass |
| **TC-023** | Restaurant & Menu | View menu with non-existent restaurant ID | User is on `menu.jsp` page | 1. Enter URL `menu.jsp?restaurantId=99999` | `restaurantId=99999` | Query executes safely returning 0 rows. Renders empty menu container without crashing application. | As expected | Pass |
| **TC-024** | Restaurant & Menu | View menu without supplying `restaurantId` parameter | User opens browser | 1. Enter direct URL `menu.jsp` | No URL query params | `Integer.parseInt(null)` throws `NumberFormatException`, causing HTTP 500 internal server error page. (Negative Boundary) | As expected | Pass |
| **TC-025** | Restaurant & Menu | Admin add new restaurant with valid details | Admin is on `manageRestaurants.jsp` | 1. Click '+' button<br>2. Fill Name, Location, Image URL<br>3. Click 'Add' | Name: `Taco Bell`<br>Location: `Downtown`<br>Image: `https://example.com/taco.jpg` | Executes `INSERT INTO restaurants`. Page reloads showing newly added restaurant card with status "Visible". | As expected | Pass |
| **TC-026** | Cart | Add item to cart from menu page | User is logged in on `menu.jsp` | 1. Click 'Add to Cart' button for Item ID 1 | `itemId=1` | `addToCart.jsp` fetches item details from DB, adds `CartItem` to session `cart` list, and redirects to `cart.jsp`. | As expected | Pass |
| **TC-027** | Cart | Add multiple distinct items to cart | User is on `menu.jsp` page | 1. Click 'Add to Cart' on Item A<br>2. Return to menu<br>3. Click 'Add to Cart' on Item B | Item A: `Burger`<br>Item B: `Fries` | Session `cart` contains 2 distinct `CartItem` objects. `cart.jsp` displays both items with calculated totals. | As expected | Pass |
| **TC-028** | Cart | Add duplicate item to cart (Quantity increment) | User already has `Burger` (Qty: 1) in cart | 1. Navigate to menu<br>2. Click 'Add to Cart' on `Burger` again | Item: `Burger` | `addToCart.jsp` detects item already in session `cart`, increments quantity to 2 instead of adding duplicate row. | As expected | Pass |
| **TC-029** | Cart | Increase item quantity inside cart (`+` button) | User is on `cart.jsp` with item in cart | 1. Click '+' button next to item `Burger` | Item Name: `Burger`<br>action: `increase` | `updateCart.jsp` increases quantity by 1, recalculates item total and cart grand total, redirects to `cart.jsp`. | As expected | Pass |
| **TC-030** | Cart | Decrease item quantity inside cart (`-` button) | User has `Burger` (Qty: 2) in cart | 1. Click '-' button next to item `Burger` | Item Name: `Burger`<br>action: `decrease` | `updateCart.jsp` decreases quantity to 1 and updates cart display accordingly. | As expected | Pass |
| **TC-031** | Cart | Decrease item quantity to zero (Automatic item removal) | User has `Burger` (Qty: 1) in cart | 1. Click '-' button next to item `Burger` | Item Name: `Burger`<br>action: `decrease` | `updateCart.jsp` detects `newQty <= 0`, removes item from session `cart` list, and redirects to `cart.jsp`. | As expected | Pass |
| **TC-032** | Cart | Remove item explicitly via 'Remove' button | User is on `cart.jsp` with items in cart | 1. Click 'Remove' link next to item `Fries` | Item Name: `Fries`<br>action: `delete` | `updateCart.jsp` deletes item from cart list. Page reloads without the removed item. | As expected | Pass |
| **TC-033** | Cart | Verify cart grand total calculation accuracy | Cart contains: Burger (₹150 x 2) & Fries (₹80 x 1) | 1. Inspect total price line on `cart.jsp` | Item 1: `150*2`<br>Item 2: `80*1` | `cart.jsp` iterates cart items, calculates `total = 380.0`. Displays `"Total: ₹ 380.0"`. | As expected | Pass |
| **TC-034** | Cart | View cart page when cart is empty | User session has no `cart` attribute | 1. Navigate to `cart.jsp` | N/A | Displays empty cart message `"Your cart is empty 😔"` with a button `"Browse Restaurants"`. | As expected | Pass |
| **TC-035** | Cart | Cart persistence across page navigation | User has items in cart | 1. Navigate to `customer.jsp`<br>2. Navigate to `restaurants.jsp`<br>3. Return to `cart.jsp` | N/A | Session attribute `cart` retains all added items, quantities, and price calculations intact across pages. | As expected | Pass |
| **TC-036** | Order | Successful order placement with cart items | User logged in (`customer@example.com`), cart has 2 items (Total: ₹380) | 1. On `cart.jsp`, click 'Place Order' button | Cart Total: ₹380 | `placeOrder.jsp` inserts row into `orders` table (`status='Pending'`, `payment_status='PENDING'`), inserts `order_items`, clears cart, and renders Order Placed page. | As expected | Pass |
| **TC-037** | Order | Attempt order placement with empty cart | User logged in, session `cart` is null or empty | 1. Direct POST/GET request to `placeOrder.jsp` | Empty cart | `placeOrder.jsp` checks `cart == null || cart.size() == 0`, prints `"Cart is empty"` and halts execution. | As expected | Pass |
| **TC-038** | Order | Attempt order placement without user login | User session `userEmail` is null | 1. Direct navigation to `placeOrder.jsp` | Unauthenticated session | `placeOrder.jsp` checks `email == null` and immediately redirects to `login.jsp`. | As expected | Pass |
| **TC-039** | Order | Verify order database record accuracy | User places order for Burger (₹150 x 2) | 1. Place order<br>2. Inspect DB `orders` and `order_items` tables | User ID, Item names, Quantities, Total Price | `orders` table records `total_price = 300.0`, `status = 'Pending'`. `order_items` records item details accurately. | As expected | Pass |
| **TC-040** | Order | View customer order history list | User logged in and has placed 3 past orders | 1. Navigate to `orderHistory.jsp` | User Email: `customer@example.com` | Queries `SELECT * FROM orders WHERE user_id=? ORDER BY order_date DESC`. Displays order cards with ID, total, status badge, date, and 'View Details'. | As expected | Pass |
| **TC-041** | Order | View order details and progress bar tracking | Customer is on `orderHistory.jsp` | 1. Click 'View Details' for Order ID 10 | `orderId=10` | Navigates to `orderDetails.jsp?orderId=10&mode=customer`. Displays status badge, progress bar (15%/55%/100%), and item table. | As expected | Pass |
| **TC-042** | Order | View order details with non-existent Order ID | Customer is logged in | 1. Navigate to `orderDetails.jsp?orderId=9999` | `orderId=9999` | DB query returns empty result. Page prints `"Order not found"`. | As expected | Pass |
| **TC-043** | Order | View order history when customer has no orders | New user logged in with 0 orders | 1. Navigate to `orderHistory.jsp` | User ID with no orders | Query returns 0 rows. Displays empty state `"No orders yet 😔"` with button `"Start Ordering"`. | As expected | Pass |
| **TC-044** | Payment | Complete payment selecting UPI method | User is on `payment.jsp?orderId=10&amount=380.0` | 1. Select 'UPI' radio button<br>2. Click 'Confirm Payment' button | Method: `UPI`<br>Order ID: 10 | Form POSTs to `processPayment.jsp`. Executes `UPDATE orders SET payment_status='PAID' WHERE id=10`. Displays invoice receipt. | As expected | Pass |
| **TC-045** | Payment | Complete payment selecting Card method | User is on `payment.jsp?orderId=10&amount=380.0` | 1. Select 'Card' radio button<br>2. Click 'Confirm Payment' button | Method: `Card`<br>Order ID: 10 | `processPayment.jsp` updates `payment_status='PAID'` in database and displays successful invoice. | As expected | Pass |
| **TC-046** | Payment | Complete payment selecting Cash method | User is on `payment.jsp?orderId=10&amount=380.0` | 1. Select 'Cash' radio button<br>2. Click 'Confirm Payment' button | Method: `Cash`<br>Order ID: 10 | Payment confirmed, DB status updated to `'PAID'`, invoice generated. | As expected | Pass |
| **TC-047** | Payment | Submit payment form without selecting method | User is on `payment.jsp` | 1. Deselect all radio buttons<br>2. Click 'Confirm Payment' | No radio selected | HTML5 `required` attribute on radio group prevents form submission. | As expected | Pass |
| **TC-048** | Payment | Invoice generation and print trigger | User is on `processPayment.jsp` after payment | 1. Click '🧾 Download Invoice' button | N/A | Invokes `window.print()`. CSS `@media print` hides `.no-print` controls and opens browser print dialog. | As expected | Pass |
| **TC-049** | Payment | Verify payment status consistency in Admin panel | Payment completed for Order ID 10 | 1. Admin opens `adminOrders.jsp` | Order ID: 10 | Admin order card displays payment status as `"PAID"` in bold. | As expected | Pass |
| **TC-050** | Admin | View Admin Dashboard statistics cards | Admin logged in on `admin.jsp` | 1. Navigate to `admin.jsp` | N/A | Queries DB for total orders, pending, preparing, delivered counts, total revenue, today revenue, top restaurant, and customer/staff user counts. | As expected | Pass |
| **TC-051** | Admin | Live statistics auto-refresh via AJAX | Admin is viewing `admin.jsp` | 1. Keep page open for 3 seconds while new order is placed in another tab | `liveStats.jsp` interval poll (1500ms) | JavaScript `updateAllLive()` fetches `liveStats.jsp`, compares values, and smoothly updates numeric text cards. | As expected | Pass |
| **TC-052** | Admin | View customer user list in modal | Admin is on `admin.jsp` | 1. Click 'Customers' stat card | `type=customer` | AJAX fetches `getUsers.jsp?type=customer`, renders list of customer names and emails inside modal popup. | As expected | Pass |
| **TC-053** | Admin | Update order status (`Pending` -> `Preparing`) | Admin is on `adminOrders.jsp` | 1. Locate Order ID 10<br>2. Select '🔵 Preparing' from custom dropdown<br>3. Click 'Update' button | Order ID: 10<br>Status: `Preparing` | GET request executes `UPDATE orders SET status='Preparing' WHERE id=10`. Order status badge updates to blue 'Preparing'. | As expected | Pass |
| **TC-054** | Admin | Update order status to `Delivered` | Admin is on `adminOrders.jsp` | 1. Locate Order ID 10<br>2. Select '🟢 Delivered' from dropdown<br>3. Click 'Update' | Order ID: 10<br>Status: `Delivered` | DB updated (`status='Delivered'`). Status badge updates to green 'Delivered'. | As expected | Pass |
| **TC-055** | Admin | Add new menu item to restaurant | Admin is on `manageMenu.jsp?restaurantId=1` | 1. Fill Item Name, Price, Image URL, Category<br>2. Click 'Add Item' button | Name: `Garlic Bread`<br>Price: 120<br>Category: `Veg` | Executes `INSERT INTO menu_items`. Page reloads displaying new menu item card. | As expected | Pass |
| **TC-056** | Admin | Stage Hide/Unhide restaurant change (Draft mode) | Admin is on `manageRestaurants.jsp` | 1. Click '👁 Hide' on Restaurant ID 1 | `toggleId=1` | Updates session `previewMap`. Card opacity set to 0.5, status shows "Draft". '🚀 Publish Changes' button appears at screen bottom. | As expected | Pass |
| **TC-057** | Admin | Publish staged restaurant & menu changes to live DB | Admin has staged draft changes in session | 1. Click '🚀 Publish Changes' floating button | `previewMap`, `deleteMap` in session | `publishChanges.jsp` executes DB SQL updates (`UPDATE is_available`, `DELETE`), clears draft session attributes, alerts `"Changes Published!"`. | As expected | Pass |
| **TC-058** | Security | Direct URL access to `customer.jsp` without login | User opens new browser window | 1. Navigate directly to `http://localhost:8080/customer.jsp` | Unauthenticated session | Header displays `null` for user email as `customer.jsp` lacks session validation. (Security Finding) | As expected | Pass |
| **TC-059** | Security | Direct URL access to `orderHistory.jsp` without login | User opens new browser window | 1. Navigate directly to `http://localhost:8080/orderHistory.jsp` | Unauthenticated session | Page checks `email == null` and immediately redirects user to `login.jsp`. | As expected | Pass |
| **TC-060** | Security | Customer user accessing admin page `admin.jsp` | Customer is logged in (`customer@example.com`) | 1. Type URL `http://localhost:8080/admin.jsp` in browser address bar | Session role: `CUSTOMER` | `admin.jsp` does NOT verify admin role in session, loading admin dashboard for customer user. (Security Finding) | As expected | Pass |
| **TC-061** | Security | Basic SQL Injection payload in Login form | User is on `login.jsp` page | 1. Enter SQL Injection string in email<br>2. Enter any password<br>3. Click 'Login' | Email: `' OR '1'='1`<br>Password: `anything` | Prepared Statement in `UserService.loginUser()` uses parameterized query (`email=? AND password=?`). Login safely fails. | As expected | Pass |
| **TC-062** | Security | XSS Script injection in Registration Name field | User is on `register.jsp` page | 1. Enter XSS payload in Name field<br>2. Fill Email and Password<br>3. Click 'Register' | Name: `<script>alert('XSS')</script>`<br>Email: `xss@example.com`<br>Password: `pass` | Name stored in DB. When displayed in admin modals without HTML escaping, script executes. (Security Finding) | As expected | Pass |
| **TC-063** | Security | Unauthenticated access to order status update script | User is unauthenticated | 1. Navigate directly to `http://localhost:8080/adminUpdateStatus.jsp?orderId=10&status=Delivered` | Unauthenticated session | Script executes `UPDATE orders SET status='Delivered' WHERE id=10` without checking session authentication. (Security Finding) | As expected | Pass |
| **TC-064** | Security | Activity heartbeat execution (`updateActivity.jsp`) | User is browsing customer dashboard | 1. Monitor network tab for periodic fetch calls | AJAX call every 1500ms | Calls `updateActivity.jsp`, executes `UPDATE users SET last_active = NOW() WHERE email = ?`. | As expected | Pass |
| **TC-065** | UI & Functional | Responsive stylesheet rendering on mobile viewport | Viewport resized to 375px width | 1. Open `register.jsp` or `customer.jsp` in mobile mode | Viewport width: `375px` | `@media (max-width: 600px)` in `responsive.css` adjusts grid layouts, card widths, and video background scaling. | As expected | Pass |
| **TC-066** | UI & Functional | Floating Back button navigation across pages | User is on `restaurants.jsp`, `menu.jsp`, `cart.jsp`, `adminOrders.jsp` | 1. Click fixed `#backBtn` at bottom-left of screen | N/A | Button navigates back to parent page (`customer.jsp` or `admin.jsp`). | As expected | Pass |
| **TC-067** | AI Chatbot | Interact with Food Assistant Chatbot widget | Customer logged in on `customer.jsp` | 1. Click floating chat icon 💬<br>2. Type "what veg food do you have?"<br>3. Click 'Send' | Query: `"what veg food do you have?"` | Chat box slides up (`#chatBox.active`). `chatAPI.jsp` queries DB for non-chicken items, calls OpenRouter API, and renders reply. | As expected | Pass |
| **TC-068** | AI Chatbot | Food Assistant Chatbot quick suggestion buttons | Chatbot widget open on `customer.jsp` | 1. Click preset button '🍃 Veg' | Quick message: `"veg food"` | Input field populated with `"veg food"` and `sendMessage()` triggered automatically. | As expected | Pass |
| **TC-069** | AI Chatbot | Add food item to cart directly via Chatbot prompt | Customer using Chatbot widget | 1. Type "order chicken burger"<br>2. Click 'Send' | Query: `"order chicken burger"` | `chatAPI.jsp` detects keyword "order" and "chicken burger", queries DB, adds item to session `cart`, responds `"Chicken Burger added to cart ✅"`. | As expected | Pass |
| **TC-070** | AI Chatbot | Chatbot handling request for unavailable item | Customer using Chatbot widget | 1. Type "order sushi"<br>2. Click 'Send' | Query: `"order sushi"` | `chatAPI.jsp` searches DB for "sushi", finds 0 matches, responds `"Sorry, I couldn't find that item."` | As expected | Pass |

---

## 3. Total Number of Test Cases

- **Total Test Cases Generated:** `70`

---

## 4. Number of Positive Test Cases

- **Total Positive Test Cases:** `44`
- *Coverage Includes:* Valid Registration, Valid Customer & Admin Login, Restaurant & Menu Browsing, Category Filtering, Price Sorting, Cart Addition, Cart Quantity Adjustments, Total Calculations, Order Placement, Order History & Details Tracking, Payment Method Confirmations, Invoice Generation, Admin Analytics Dashboard, Admin Order Updates, Admin CRUD & Staging/Publish Workflows, Responsive UI Navigation, and Chatbot Assistance.

---

## 5. Number of Negative Test Cases

- **Total Negative / Security / Boundary Test Cases:** `26`
- *Coverage Includes:* Empty Form Field Submissions, Invalid Email Formatting, Duplicate Email Registration, Invalid Password/Email Logins, Whitespace Edge Cases, Out-of-Stock Item Display, Non-existent Database Entity IDs, Missing Query Parameters (`NumberFormatException`), Unauthenticated Direct Page Access, Customer Access to Admin Panels, Unauthenticated Servlet Executions, SQL Injection Resistance Verification, and XSS Vulnerability Auditing.

---

## 6. Important Functionality Dependency & Testing Limitations

During source code inspection, the following technical dependencies and missing safeguards were identified that affect direct manual/automated testing:

1. **Third-Party AI Gateway Dependency (OpenRouter API)**:
   - `chatAPI.jsp` contains a hardcoded authorization token (`sk-or-v1-a9cd9f...`) pointing to `https://openrouter.ai/api/v1/chat/completions`. If the API key expires, exceeds usage quotas, or if outgoing internet connectivity is restricted, the AI fallback defaults to returning `"No response from AI"` or error strings.

2. **Simulated Payment Gateway (No Real Payment Authorization)**:
   - Payment confirmation (`processPayment.jsp`) performs an immediate SQL update (`UPDATE orders SET payment_status='PAID'`). No real-world payment gateway API (e.g., Razorpay, Stripe, PayPal) or bank OTP authorization is integrated. Payment failure scenarios cannot be triggered via bank responses.

3. **External Unsplash CDN Image Reliance**:
   - Dynamic restaurant and menu fallback images depend on external Unsplash URLs (`https://source.unsplash.com/...`). In offline or restricted network environments, image thumbnails will display as broken links.

4. **Missing Role-Based Access Control (RBAC) Session Guards**:
   - `admin.jsp`, `adminOrders.jsp`, and `manageRestaurants.jsp` lack session validation checks (`session.getAttribute("userEmail")` or role check `role.equalsIgnoreCase("ADMIN")`). Any user or unauthenticated visitor can access administrative pages directly via URL manipulation.

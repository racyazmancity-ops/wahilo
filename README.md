# WASHILO for WordPress

Two parts, built from the WASHILO Business & Software Requirements v1.0:

| Package | What it is |
|---|---|
| `washilo-core.zip` | **WASHILO Core plugin** – the operational application: catalog, per-piece pricing, booking, orders, riders, QR packages, outlet workflow, payments, subscriptions, loyalty, coupons, corporate & hotel accounts, franchises, notifications, reports, audit log and a REST API for the mobile apps. |
| `washilo-theme.zip` | **WASHILO theme** – the public website (Home, Services, Pricing, How It Works, Corporate, Hotel, Offers & Rewards, About, Contact, FAQs, Journal) designed mobile-first, plus the page shells for the customer, rider and outlet apps. |

## Install (10 minutes)

1. WordPress 6.2+ and PHP 7.4+ (8.x recommended), MySQL/MariaDB. Use HTTPS.
2. **Plugins → Add New → Upload** `washilo-core.zip` → Activate.
   This creates the database tables, roles, sample catalog (Pakistan / Lahore, 3 outlets, 29 items, 5 services, example prices), Silver/Gold/Platinum plans, two coupons, and the app pages: Book a Pickup, Track Order, Find an Outlet, My Account, Subscriptions, Rider App, Outlet App.
3. **Appearance → Themes → Upload** `washilo-theme.zip` → Activate.
   This creates the marketing pages, sets the homepage, pretty permalinks, and the header/footer menus.
   (Install the plugin first, then the theme, so the menus include the app pages.)
4. **Settings → Permalinks → Save** once (refreshes URLs on some hosts).

## First-time setup checklist

Everything below is under the **WASHILO** menu in wp-admin.

1. **Locations** – edit the sample country/city/outlets/service areas or add your own. Set each outlet's slot capacity and each area's delivery fee and serving outlet.
2. **Services & Items** – adjust categories and items. Suits are bundles (e.g. Pent Coat = Coat + Trouser, counts as 2 pieces against a subscription).
3. **Pricing** – the sample prices are examples only. Edit the global price list, then override per country, city, outlet, or corporate/hotel contract where needed. Empty cells inherit.
4. **Staff & Riders** – create logins for riders, outlet staff/managers, admins and franchise owners, each assigned to an outlet. Riders and outlet staff are sent straight to their app after login.
5. **Settings** – time slots, fees, express %, payment methods, bank details, loyalty rules, status labels, support contacts.
6. **Before launch, change these settings:**
   - Turn **off** "Show login codes on screen (development)" once an SMS webhook is connected.
   - Keep the **test online gateway** off unless you are testing.
   - Set the support email (it receives admin alerts).
7. **Appearance → Customize → WASHILO homepage & contact** – hero text, stats, phone, WhatsApp, email, social links, logo.

## The apps

| Who | Where | What they do |
|---|---|---|
| Customer | `/book/` | Mixed-service order by piece/suit → pickup + delivery, pickup only, drop-off, or drop-off + delivery → area & slot (capacity-checked) → coupon / points / subscription → payment → confirmation. Guests get an account from their mobile number. |
| Customer | `/track/` | Timeline of every status, items (booked vs received), totals, packages, handover code when ready, bank-transfer reference, cancel, invoice, review. |
| Customer | `/account/` | Orders, subscription (pause/resume/cancel/change/auto-renew), rewards & referral link, saved addresses, support tickets, profile, notifications; company/hotel statement. Login by mobile OTP or email + password. |
| Rider | `/rider/` | Today's tasks → start → count items vs booking (customer confirmation) → photos → generate & print QR label → complete pickup. Deliveries: scan QR + customer's 4-digit code + proof photo + cash collected. Failed task with reason and reschedule. |
| Outlet staff | `/outlet-app/` | Kanban board (incoming → received → processing → QC → packed → ready), QR scan to receive/pack/dispatch, item verification, packages & label reprints (permission), assign pickup/delivery riders, customer collection by QR or code, record payments, photos, history. Walk-in orders at the counter. |
| Admin | wp-admin → WASHILO | Dashboard, Orders (filters, CSV, full order panel), Payments (verify bank transfers), Pricing, Services & Items, Staff & Riders, Corporate & Hotels (approve, contract prices, users, statements), Subscriptions, Coupons & Loyalty, Support Tickets, Reports, Locations, Franchises, Audit Log, Settings. |

QR labels encode only a random reference (`/?wsh_scan=CODE`). Staff scanning a label with their phone camera land straight on that order in their app; anyone else lands on the tracking page.

## Roles

Super Admin, Admin, Franchise Owner, Outlet Manager, Outlet Staff, Rider, Customer, Corporate User, Hotel User. Franchise owners and outlet staff only see orders, staff and reports for their own outlets. Franchise owners can override prices for their outlets only if the franchise allows it.

## Integrations

- **SMS / WhatsApp** – set a webhook URL in Settings. WASHILO POSTs JSON `{channel, to, event, message}` with a bearer token. Point it at your provider or an automation service. OTP codes use the SMS webhook.
- **Email** – uses WordPress `wp_mail`; install an SMTP plugin for reliable delivery.
- **Online payments** – add a gateway class with the `washilo_gateways` filter:

```php
add_filter( 'washilo_gateways', function ( $g ) {
	$g['mygateway'] = new class extends WSH_Gateway_Base {
		public function id() { return 'mygateway'; }
		public function label() { return 'Card / Wallet'; }
		public function is_online() { return true; }
		public function start( $order, $payment ) {
			return array( 'redirect' => 'https://provider.example/checkout?...' );
		}
	};
	return $g;
} );
// In your provider's callback: WSH_Payments::record( $order_id, $amount, 'mygateway', $provider_ref );
```

  Card details are never stored by WASHILO.
- **Push notifications** – hook `washilo_send_push` (receives user, order, event, message).
- **Message text** – filter `washilo_notification_message`.
- **Front-end translations** – filter `washilo_js_strings` (English ⇒ your language map). PHP strings use text domains `washilo` and `washilo-theme`.

## REST API (for Android / iOS)

Base: `/wp-json/washilo/v1`. Apps authenticate with WordPress Application Passwords (Users → Profile) or the OTP endpoints.

Public: `GET /config`, `GET /catalog?outlet_id=`, `GET /outlets`, `GET /outlets/nearest?lat=&lng=`, `GET /slots?outlet_id=`, `POST /quote`, `POST /orders`, `GET /track?order=&phone=`, `GET /plans`, `GET /offers`, `POST /otp/request`, `POST /otp/verify`, `POST /login`, `POST /account-request`, `POST /tickets`.
Customer: `GET|POST /me`, `GET /me/orders`, `GET /me/tickets`, `GET /me/statement?month=`, `GET /orders/{id}`, `POST /orders/{id}/cancel|rate|bank-reference`, `POST /subscriptions`, `POST /subscriptions/action`, `POST /tickets/{id}/rate`.
Rider: `GET /rider/tasks?scope=open|history`, `POST /rider/tasks/{id}/start|complete|deliver|fail`.
Staff: `POST /scan`, `GET /staff/board`, `GET /staff/riders`, `POST /staff/orders` (walk-in), `GET /staff/orders/{id}`, `POST /staff/orders/{id}/status|verify|package|photo|assign|payment|collect`.

## Notes

- The QR generator and camera scanner load from cdnjs.cloudflare.com; fonts load from Google Fonts. To self-host, use the `washilo_qr_lib_url` and `washilo_scanner_lib_url` filters.
- The site installs as a PWA (manifest + service worker that caches static files only, never personal pages).
- A daily job renews subscriptions, sends renewal and payment reminders, and clears expired OTPs. On low-traffic sites, set a real server cron for `wp-cron.php`.
- Order statuses can be renamed in Settings, but the order of steps is fixed so orders cannot skip a stage.
- Data is kept if the plugin is deactivated.

## Not included yet (spec Phase 4 / future)

Native Android/iOS apps (the API is ready for them), route optimisation, AI support, demand forecasting, machine/RFID integration, and a live payment gateway for your bank (add one with the filter above).

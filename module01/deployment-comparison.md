# Deployment Comparison: Bluebird Bakery Online Ordering

## Business scenario

Bluebird Bakery is an invented independent bakery with one storefront in Fort Wayne, Indiana. Its 14 employees include eight bakers and decorators, four counter employees, one delivery driver, and the owner-manager. The bakery sells walk-in products and custom cakes, and it expects about $850,000 in annual revenue. Because ingredients, payroll, rent, and equipment consume most of that revenue, the business limits all technology spending—including internet service, point-of-sale support, devices, backups, and hosting—to about $600 per month.

## Workload and requirements

The workload is Bluebird's online ordering and pickup-scheduling web application. Customers use it to browse the current menu, submit and pay for orders, choose pickup times, and receive confirmations. It has the following requirements:

- **Outside access:** It must be reachable securely from customers' phones and computers outside the bakery.
- **Availability:** Customers order after closing, so it must stay available overnight and should not depend on an employee leaving a personal device running.
- **Growth:** Normal use is modest, but demand can rise to roughly five times the normal level before holidays.
- **Physical access:** No employee needs direct access to its hardware; staff use the application through a browser.
- **Budget:** Hosting, backups, and related cloud charges must remain at or below **$180 per month**.

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

## Five deployment verdicts

| Deployment level | What works | What breaks | Verdict |
|---|---|---|---|
| **VirtualBox on a laptop** | It has almost no new hardware cost, isolates the server in a VM, and is adequate for a short demonstration. | A personal laptop may be shut down, removed from the bakery, disconnected, or asleep. Home-grade power and internet, port forwarding, and one-device failure make dependable overnight public access unrealistic. | **Reject for production.** Cheap testing does not satisfy continuous public availability. |
| **Hyper-V on a workstation** | A dedicated Windows workstation can run the application in an isolated VM and allows checkpoints before risky changes. | The workstation, bakery power, and bakery internet are still single points of failure. Secure internet exposure, patching, monitoring, and off-site backups become the owner's responsibility. | **Reject for production.** More stable than a laptop, but not dependable enough for ordering after closing. |
| **Proxmox host** | A dedicated host supports centralized VM management, scheduled backups, and room for other local services. | One host is not high availability, and adding redundant hosts, storage, power protection, and IT support would exceed the workload's scale and likely its $180 monthly limit. A bakery outage would still take ordering offline. | **Technically capable, but reject.** Its management and redundancy burden are disproportionate for this business. |
| **Physical PC** | It is straightforward, avoids a virtualization layer, and could run the small application on inexpensive hardware. | Hardware failure affects the entire service, recovery is slower without a portable VM, and the owner must manage remote access, security, backups, cooling, power, and internet uptime. | **Reject.** Simplicity does not overcome the single-machine and single-location risks. |
| **Azure** | Azure can provide public HTTPS access, off-site hosting, monitoring, backups, and the ability to increase capacity during holiday demand without bakery staff touching hardware. A modest configuration can be kept within the $180 monthly ceiling when cost alerts and limits are used. | It requires careful account security and cost monitoring, depends on internet connectivity for administration, and creates some platform dependence. A poor configuration could increase cost or expose data. | **Accept.** It is the only option here that directly meets the outside-access and overnight-availability requirements without local hardware. |

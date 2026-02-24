# Training Video Scripts -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Video Series Overview

| # | Title | Duration | Audience |
|---|-------|----------|----------|
| V01 | Platform Overview & Architecture | 15 min | All |
| V02 | Admin Console Walkthrough | 12 min | Admins |
| V03 | Product Catalog Management | 10 min | Admins |
| V04 | Customer Onboarding Flow | 12 min | Admins, CSR |
| V05 | Billing Cycle Deep Dive | 15 min | Admins, Finance |
| V06 | Prepaid Balance & Top-Up | 10 min | End Users |
| V07 | USSD Self-Service Guide | 8 min | End Users |
| V08 | Partner Portal & Settlement | 12 min | Partners, Admins |
| V09 | NOC Dashboard & Fault Management | 10 min | NOC Engineers |
| V10 | Developer Quickstart | 15 min | Developers |
| V11 | Revenue Assurance & Fraud Detection | 12 min | RA Analysts |
| V12 | Smart Metering & STS Tokens | 10 min | Utility Admins |

---

## 2. V01: Platform Overview & Architecture (15 min)

### Script

**[0:00 - 0:30] Opening**
"Welcome to ERP-BSS-OSS, the open-source, carrier-grade BSS/OSS platform. In this video, we will walk through the platform architecture, its 30 microservices, and how they work together to deliver a complete telecom and utilities solution."

**[0:30 - 3:00] What is BSS/OSS?**
"BSS -- Business Support Systems -- handles everything the subscriber interacts with: billing, orders, products, and customer management. OSS -- Operations Support Systems -- handles what happens behind the scenes: network provisioning, resource inventory, and fault management."

*[Show Mermaid diagram: C4 Context]*

**[3:00 - 7:00] Architecture Deep Dive**
"Our platform runs 30 microservices organized into four domains: BSS Core, OSS Core, Specialized Services, and Foundation Services."

*[Show architecture diagram with service inventory]*

"The core Rust crates -- bss-core, bss-api, bss-billing, bss-crm, bss-ordering -- provide the domain logic. The Go microservice stubs handle service-specific HTTP endpoints."

**[7:00 - 10:00] Data Architecture**
"We use a polyglot persistence strategy. PostgreSQL for transactional data -- customers, orders, invoices. Redis for real-time balance caching with sub-millisecond lookups. ClickHouse for analytics -- CDR aggregation, revenue reports. MongoDB for audit logs and dynamic configuration."

*[Show data flow diagram]*

**[10:00 - 13:00] Event-Driven Architecture**
"Every state change in the system publishes a CloudEvents event to Kafka. When an order is created, the billing service and provisioning service both consume that event independently. This decoupling enables us to scale each service independently."

**[13:00 - 14:30] TM Forum Compliance**
"The platform implements TM Forum Open APIs: TMF620 for Product Catalog, TMF622 for Order Management, TMF629 for Customer Management, TMF678 for Billing, and more. This means you can integrate with any TM Forum-compliant system."

**[14:30 - 15:00] Closing**
"In our next video, we will walk through the admin console. See you there."

---

## 3. V05: Billing Cycle Deep Dive (15 min)

### Script

**[0:00 - 0:30] Opening**
"In this video, we will walk through the complete billing cycle -- from CDR collection through invoice delivery."

**[0:30 - 4:00] The Billing Pipeline**
"The billing pipeline has six stages."

*[Show billing cycle workflow diagram]*

1. "First, CDRs flow from the network through our mediation engine, which normalizes and deduplicates them."
2. "The rating engine prices each CDR against the subscriber's tariff plan."
3. "At billing cycle time, the billing engine aggregates all charges."
4. "Discounts are applied in order: plan, volume, loyalty, promotional, manual."
5. "Tax is calculated per jurisdiction."
6. "The invoice is generated and delivered."

**[4:00 - 8:00] Admin Console Demo**
*[Screen recording: Navigate to Billing > Run Billing Cycle]*

"Here I am selecting the January monthly cycle. You can see the progress bar showing 50,000 customers being processed. The dashboard shows invoices generated, average invoice amount, and any errors."

**[8:00 - 11:00] Handling Errors**
*[Show error queue]*

"Sometimes an invoice fails -- maybe the customer has inconsistent data, or a tariff plan is missing. These go to the error queue. Let me click on one... this customer had a missing tariff for international data. I can fix it by assigning the correct tariff, then re-process this single invoice."

**[11:00 - 13:00] Dunning Configuration**
*[Show dunning settings screen]*

"After invoices are sent, the dunning engine monitors for payment. Here are our five dunning levels configured. Day 7: SMS reminder. Day 14: email warning. Day 30: outgoing calls barred. Day 45: full suspension. Day 90: termination."

**[13:00 - 15:00] Revenue Assurance Check**
"Finally, the revenue assurance module automatically reconciles CDRs against invoice line items. If there is a variance above our threshold, an alert is generated for investigation."

---

## 4. V07: USSD Self-Service Guide (8 min)

### Script

**[0:00 - 0:20] Opening**
"This video shows how to use USSD on your phone for balance checks, top-ups, and data purchases."

**[0:20 - 2:00] Balance Check**
*[Show phone screen]*
"Dial star-one-two-three-hash. The main menu appears. Press 1 for balance. You see your airtime balance, data remaining, and SMS remaining."

**[2:00 - 4:00] Top-Up via USSD**
"From the main menu, press 2 for top-up. Select an amount -- 5 dollars, 10 dollars, or enter a custom amount. Confirm with your mobile money PIN. Done -- your balance is updated immediately."

**[4:00 - 6:00] Buy a Data Bundle**
"Press 3 for data bundles. You see daily, weekly, and monthly options. Select the 5 GB monthly bundle for $5. Confirm. Your data balance is now updated."

**[6:00 - 7:30] Buy Electricity Token**
"Press 5 for electricity. Enter your meter number. Enter the amount -- $20. The system tells you 80 kWh. Confirm payment. You receive an SMS with a 20-digit token. Enter it on your meter."

**[7:30 - 8:00] Closing**
"That is USSD self-service. It works on any phone, no internet needed."

---

## 5. V10: Developer Quickstart (15 min)

### Script

**[0:00 - 0:30] Opening**
"In this video, we will set up a local development environment and make our first API call to ERP-BSS-OSS."

**[0:30 - 4:00] Environment Setup**
*[Terminal screen recording]*

```bash
# Clone the repository
git clone https://github.com/abiolaogu/BSS-OSS.git
cd BSS-OSS

# Start dependencies
docker-compose up -d

# Build the project
cargo build

# Run the API server
cargo run --bin bss-api-server
```

"Docker Compose brings up PostgreSQL, Redis, MongoDB, RabbitMQ, Kafka, and the observability stack."

**[4:00 - 8:00] Making API Calls**
*[Show curl commands]*

```bash
# Health check
curl http://localhost:8080/health

# Create a customer
curl -X POST http://localhost:8080/api/v1/customers \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: test-tenant" \
  -d '{"name": "John Doe", "customer_type": "individual"}'
```

**[8:00 - 12:00] Understanding the Codebase**
"Let me show you the project structure. The `crates/` directory contains Rust domain logic. The `services/` directory contains the 30 microservices as Go stubs. Let me walk through a handler..."

**[12:00 - 14:00] Running Tests**
```bash
cargo test --all
```

**[14:00 - 15:00] Next Steps**
"Explore the API documentation, try creating products and orders, and check out the Jaeger traces at localhost:16686 to see how requests flow through the system."

---

## 6. Production Notes

### Recording Setup
- Screen resolution: 1920x1080
- Codec: H.264 (MP4)
- Audio: Clear narration with noise cancellation
- Annotations: Highlight cursor clicks and key inputs
- Branding: BSS-OSS logo in bottom-right corner

### Distribution
- YouTube (unlisted) for embedding in LMS
- Download link in internal wiki
- Chapters/timestamps for easy navigation

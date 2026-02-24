# ERP-Platform API Documentation

> **Document ID:** ERP-PLAT-API-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Specification:** OpenAPI 3.1.0
> **Related Documents:** [14-Technical-Specifications.md](./14-Technical-Specifications.md), [22-Postman-Collection.md](./22-Postman-Collection.md)

---

## OpenAPI 3.1 Specification

```yaml
openapi: "3.1.0"
info:
  title: ERP-Platform API
  version: "1.0.0"
  description: |
    Unified control plane API for the 20-module ERP suite.
    Provides product catalog, subscription management, entitlement queries,
    tenant provisioning, module registry, marketplace, audit, notifications,
    web hosting, and activation wizard services.
  contact:
    name: ERP Platform Engineering
    email: platform-engineering@erp.example.com

servers:
  - url: http://localhost:8091
    description: Subscription Hub (local development)
  - url: http://localhost:8080
    description: Generic services (local development)
  - url: https://api.erp-platform.example.com
    description: Production

security:
  - BearerAuth: []

tags:
  - name: Health
    description: Health check endpoints (no auth required)
  - name: Products
    description: Product catalog operations
  - name: Subscriptions
    description: Subscription management
  - name: Entitlements
    description: Entitlement queries
  - name: Tenants
    description: Tenant provisioning and management
  - name: Audit
    description: Audit log operations
  - name: Marketplace
    description: Module marketplace
  - name: Modules
    description: Module registry
  - name: Notifications
    description: Notification hub
  - name: WebHosting
    description: Web hosting management
  - name: Activation
    description: Activation wizard

paths:
  /healthz:
    get:
      tags: [Health]
      summary: Health check
      description: Returns service health status. No authentication required.
      security: []
      responses:
        "200":
          description: Service is healthy
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: "#/components/schemas/SubscriptionHubHealth"
                  - $ref: "#/components/schemas/ServiceHealth"
              examples:
                subscription-hub:
                  value:
                    status: ok
                    catalog_version: "2026-02-23"
                generic-service:
                  value:
                    status: healthy
                    module: ERP-Platform
                    service: tenant-provisioner

  /v1/products:
    get:
      tags: [Products]
      summary: List all products
      description: Returns the complete product catalog including modules and bundles.
      security: []
      responses:
        "200":
          description: Product catalog
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Catalog"
              example:
                version: "2026-02-23"
                products:
                  - sku: erp-crm
                    id: erp-crm
                    name: ERP CRM
                    repo: ERP-CRM
                    type: module
                    standalone: true
                    category: business-ops
                    capabilities: 12
        "405":
          $ref: "#/components/responses/MethodNotAllowed"

  /v1/subscriptions:
    post:
      tags: [Subscriptions]
      summary: Create subscription
      description: |
        Creates a new subscription for a tenant. Bundle SKUs are automatically
        resolved to their constituent module SKUs. Duplicates are removed.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/SubscriptionRequest"
            examples:
              single:
                summary: Single module subscription
                value:
                  tenant_id: acme-corp
                  plan_type: single
                  skus: [erp-crm]
              bundle:
                summary: Bundle subscription
                value:
                  tenant_id: acme-corp
                  plan_type: bundle
                  skus: [professional]
              suite:
                summary: Suite (custom) subscription
                value:
                  tenant_id: acme-corp
                  plan_type: suite
                  skus: [erp-crm, erp-finance, erp-bi]
      responses:
        "201":
          description: Subscription created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/SubscriptionRecord"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthorized"
        "405":
          $ref: "#/components/responses/MethodNotAllowed"

  /v1/subscriptions/{tenant_id}:
    get:
      tags: [Subscriptions]
      summary: Get tenant subscription
      description: Retrieves the subscription record for a specific tenant.
      parameters:
        - name: tenant_id
          in: path
          required: true
          schema:
            type: string
          example: acme-corp
      responses:
        "200":
          description: Subscription found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/SubscriptionRecord"
        "404":
          $ref: "#/components/responses/NotFound"
        "405":
          $ref: "#/components/responses/MethodNotAllowed"

  /v1/entitlements/{tenant_id}:
    get:
      tags: [Entitlements]
      summary: Get tenant entitlements
      description: Returns the list of module SKUs the tenant is entitled to access.
      parameters:
        - name: tenant_id
          in: path
          required: true
          schema:
            type: string
          example: acme-corp
      responses:
        "200":
          description: Entitlements found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/EntitlementResponse"
              example:
                tenant_id: acme-corp
                entitlements: [erp-crm, erp-workspace, erp-bi]
        "404":
          $ref: "#/components/responses/NotFound"

  /v1/tenant-provisioner:
    get:
      tags: [Tenants]
      summary: List tenants
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Tenant list
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ListResponse"
        "400":
          $ref: "#/components/responses/MissingTenantID"
    post:
      tags: [Tenants]
      summary: Provision tenant
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: true
      responses:
        "201":
          description: Tenant provisioned
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/CreateResponse"
        "400":
          $ref: "#/components/responses/MissingTenantID"

  /v1/tenant-provisioner/{id}:
    get:
      tags: [Tenants]
      summary: Get tenant by ID
      parameters:
        - $ref: "#/components/parameters/XTenantID"
        - $ref: "#/components/parameters/ResourceID"
      responses:
        "200":
          description: Tenant found
        "400":
          $ref: "#/components/responses/MissingTenantID"
    put:
      tags: [Tenants]
      summary: Update tenant
      parameters:
        - $ref: "#/components/parameters/XTenantID"
        - $ref: "#/components/parameters/ResourceID"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: true
      responses:
        "200":
          description: Tenant updated
        "400":
          $ref: "#/components/responses/MissingTenantID"
    delete:
      tags: [Tenants]
      summary: Decommission tenant
      parameters:
        - $ref: "#/components/parameters/XTenantID"
        - $ref: "#/components/parameters/ResourceID"
      responses:
        "200":
          description: Tenant decommissioned
        "400":
          $ref: "#/components/responses/MissingTenantID"

  /v1/audit:
    get:
      tags: [Audit]
      summary: List audit entries
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Audit log entries
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ListResponse"
    post:
      tags: [Audit]
      summary: Create audit entry
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
      responses:
        "201":
          description: Audit entry created

  /v1/marketplace:
    get:
      tags: [Marketplace]
      summary: List marketplace modules
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Marketplace listings
    post:
      tags: [Marketplace]
      summary: Install marketplace module
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "201":
          description: Module installed

  /v1/module-registry:
    get:
      tags: [Modules]
      summary: List registered modules
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Module list
    post:
      tags: [Modules]
      summary: Register module
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "201":
          description: Module registered

  /v1/notification-hub:
    get:
      tags: [Notifications]
      summary: List notifications
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Notifications list
    post:
      tags: [Notifications]
      summary: Send notification
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "201":
          description: Notification sent

  /v1/web-hosting:
    get:
      tags: [WebHosting]
      summary: List hosting configurations
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Hosting configurations
    post:
      tags: [WebHosting]
      summary: Configure hosting
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "201":
          description: Hosting configured

  /v1/activation-wizard:
    get:
      tags: [Activation]
      summary: List wizard states
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "200":
          description: Wizard states
    post:
      tags: [Activation]
      summary: Create wizard step
      parameters:
        - $ref: "#/components/parameters/XTenantID"
      responses:
        "201":
          description: Wizard step created

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT token from ERP-IAM (OIDC)

  parameters:
    XTenantID:
      name: X-Tenant-ID
      in: header
      required: true
      schema:
        type: string
      description: Tenant identifier for multi-tenant isolation
      example: acme-corp
    ResourceID:
      name: id
      in: path
      required: true
      schema:
        type: string
      description: Resource identifier

  schemas:
    SubscriptionHubHealth:
      type: object
      properties:
        status:
          type: string
          example: ok
        catalog_version:
          type: string
          example: "2026-02-23"

    ServiceHealth:
      type: object
      properties:
        status:
          type: string
          example: healthy
        module:
          type: string
          example: ERP-Platform
        service:
          type: string
          example: tenant-provisioner

    Product:
      type: object
      properties:
        sku:
          type: string
          example: erp-crm
        id:
          type: string
        name:
          type: string
          example: ERP CRM
        repo:
          type: string
          example: ERP-CRM
        type:
          type: string
          enum: [module, bundle]
        standalone:
          type: boolean
        category:
          type: string
        capabilities:
          type: integer
        includes:
          type: array
          items:
            type: string

    Catalog:
      type: object
      properties:
        version:
          type: string
        products:
          type: array
          items:
            $ref: "#/components/schemas/Product"

    SubscriptionRequest:
      type: object
      required: [tenant_id, plan_type, skus]
      properties:
        tenant_id:
          type: string
          minLength: 1
        plan_type:
          type: string
          enum: [single, bundle, suite]
        skus:
          type: array
          items:
            type: string
          minItems: 1

    SubscriptionRecord:
      type: object
      properties:
        tenant_id:
          type: string
        plan_type:
          type: string
        skus:
          type: array
          items:
            type: string

    EntitlementResponse:
      type: object
      properties:
        tenant_id:
          type: string
        entitlements:
          type: array
          items:
            type: string

    ListResponse:
      type: object
      properties:
        items:
          type: array
          items:
            type: object
        event_topic:
          type: string

    CreateResponse:
      type: object
      properties:
        item:
          type: object
        event_topic:
          type: string

    Error:
      type: object
      properties:
        error:
          type: string

  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
          example:
            error: tenant subscription not found
    MethodNotAllowed:
      description: Method not allowed
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
          example:
            error: method not allowed
    MissingTenantID:
      description: Missing X-Tenant-ID header
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"
          example:
            error: missing X-Tenant-ID
```

---

*For Postman collection, see [22-Postman-Collection.md](./22-Postman-Collection.md). For technical specifications, see [14-Technical-Specifications.md](./14-Technical-Specifications.md).*

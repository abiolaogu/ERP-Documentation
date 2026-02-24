# ERP-Platform Postman Collection

> **Document ID:** ERP-PLAT-PC-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Related Documents:** [21-API-Documentation.md](./21-API-Documentation.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## Setup Instructions

1. Import the collection JSON below into Postman.
2. Import the environment JSON below.
3. Set the environment to "ERP-Platform Local".
4. Run the "Health Check" request first to verify connectivity.

---

## Environment Variables

```json
{
    "id": "erp-platform-env",
    "name": "ERP-Platform Local",
    "values": [
        {
            "key": "base_url",
            "value": "http://localhost:8091",
            "type": "default",
            "enabled": true
        },
        {
            "key": "service_url",
            "value": "http://localhost:8080",
            "type": "default",
            "enabled": true
        },
        {
            "key": "tenant_id",
            "value": "demo-tenant",
            "type": "default",
            "enabled": true
        },
        {
            "key": "jwt_token",
            "value": "",
            "type": "secret",
            "enabled": true
        },
        {
            "key": "resource_id",
            "value": "",
            "type": "default",
            "enabled": true
        }
    ]
}
```

---

## Collection JSON

```json
{
    "info": {
        "name": "ERP-Platform API",
        "description": "Complete API collection for ERP-Platform unified control plane",
        "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
        "version": "1.0.0"
    },
    "auth": {
        "type": "bearer",
        "bearer": [
            {
                "key": "token",
                "value": "{{jwt_token}}",
                "type": "string"
            }
        ]
    },
    "item": [
        {
            "name": "Health",
            "item": [
                {
                    "name": "Subscription Hub Health",
                    "request": {
                        "method": "GET",
                        "url": "{{base_url}}/healthz",
                        "auth": { "type": "noauth" }
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 200', function () {",
                                    "    pm.response.to.have.status(200);",
                                    "});",
                                    "pm.test('Status is ok', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.status).to.eql('ok');",
                                    "});",
                                    "pm.test('Catalog version present', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.catalog_version).to.be.a('string');",
                                    "});"
                                ]
                            }
                        }
                    ]
                },
                {
                    "name": "Tenant Provisioner Health",
                    "request": {
                        "method": "GET",
                        "url": "{{service_url}}/healthz",
                        "auth": { "type": "noauth" }
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 200', function () {",
                                    "    pm.response.to.have.status(200);",
                                    "});",
                                    "pm.test('Status is healthy', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.status).to.eql('healthy');",
                                    "});"
                                ]
                            }
                        }
                    ]
                }
            ]
        },
        {
            "name": "Products",
            "item": [
                {
                    "name": "List All Products",
                    "request": {
                        "method": "GET",
                        "url": "{{base_url}}/v1/products"
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 200', function () {",
                                    "    pm.response.to.have.status(200);",
                                    "});",
                                    "pm.test('Contains products array', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.products).to.be.an('array');",
                                    "    pm.expect(json.products.length).to.be.greaterThan(0);",
                                    "});",
                                    "pm.test('Has version field', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.version).to.be.a('string');",
                                    "});"
                                ]
                            }
                        }
                    ]
                }
            ]
        },
        {
            "name": "Subscriptions",
            "item": [
                {
                    "name": "Create Single Subscription",
                    "request": {
                        "method": "POST",
                        "url": "{{base_url}}/v1/subscriptions",
                        "header": [
                            { "key": "Content-Type", "value": "application/json" }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\"tenant_id\": \"{{tenant_id}}\", \"plan_type\": \"single\", \"skus\": [\"erp-crm\"]}"
                        }
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 201', function () {",
                                    "    pm.response.to.have.status(201);",
                                    "});",
                                    "pm.test('Has tenant_id', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.tenant_id).to.eql(pm.environment.get('tenant_id'));",
                                    "});",
                                    "pm.test('SKUs contain erp-crm', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.skus).to.include('erp-crm');",
                                    "});"
                                ]
                            }
                        }
                    ]
                },
                {
                    "name": "Create Bundle Subscription",
                    "request": {
                        "method": "POST",
                        "url": "{{base_url}}/v1/subscriptions",
                        "header": [
                            { "key": "Content-Type", "value": "application/json" }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\"tenant_id\": \"bundle-tenant\", \"plan_type\": \"bundle\", \"skus\": [\"enterprise\"]}"
                        }
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 201', function () {",
                                    "    pm.response.to.have.status(201);",
                                    "});",
                                    "pm.test('Enterprise resolves to 20 modules', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.skus.length).to.eql(20);",
                                    "});"
                                ]
                            }
                        }
                    ]
                },
                {
                    "name": "Get Subscription",
                    "request": {
                        "method": "GET",
                        "url": "{{base_url}}/v1/subscriptions/{{tenant_id}}"
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 200', function () {",
                                    "    pm.response.to.have.status(200);",
                                    "});",
                                    "pm.test('Returns correct tenant', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.tenant_id).to.eql(pm.environment.get('tenant_id'));",
                                    "});"
                                ]
                            }
                        }
                    ]
                },
                {
                    "name": "Get Subscription (404)",
                    "request": {
                        "method": "GET",
                        "url": "{{base_url}}/v1/subscriptions/nonexistent-tenant"
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 404', function () {",
                                    "    pm.response.to.have.status(404);",
                                    "});",
                                    "pm.test('Returns not found error', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.error).to.include('not found');",
                                    "});"
                                ]
                            }
                        }
                    ]
                }
            ]
        },
        {
            "name": "Entitlements",
            "item": [
                {
                    "name": "Get Entitlements",
                    "request": {
                        "method": "GET",
                        "url": "{{base_url}}/v1/entitlements/{{tenant_id}}"
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Status code is 200', function () {",
                                    "    pm.response.to.have.status(200);",
                                    "});",
                                    "pm.test('Has entitlements array', function () {",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.entitlements).to.be.an('array');",
                                    "});"
                                ]
                            }
                        }
                    ]
                }
            ]
        },
        {
            "name": "Tenant Provisioner",
            "item": [
                {
                    "name": "List Tenants",
                    "request": {
                        "method": "GET",
                        "url": "{{service_url}}/v1/tenant-provisioner",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
                        ]
                    }
                },
                {
                    "name": "Create Tenant",
                    "request": {
                        "method": "POST",
                        "url": "{{service_url}}/v1/tenant-provisioner",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
                            { "key": "Content-Type", "value": "application/json" }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\"name\": \"Acme Corporation\", \"domain\": \"acme.example.com\"}"
                        }
                    }
                },
                {
                    "name": "Get Tenant",
                    "request": {
                        "method": "GET",
                        "url": "{{service_url}}/v1/tenant-provisioner/{{resource_id}}",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
                        ]
                    }
                },
                {
                    "name": "Update Tenant",
                    "request": {
                        "method": "PUT",
                        "url": "{{service_url}}/v1/tenant-provisioner/{{resource_id}}",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
                            { "key": "Content-Type", "value": "application/json" }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\"name\": \"Acme Corp Updated\"}"
                        }
                    }
                },
                {
                    "name": "Delete Tenant",
                    "request": {
                        "method": "DELETE",
                        "url": "{{service_url}}/v1/tenant-provisioner/{{resource_id}}",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
                        ]
                    }
                },
                {
                    "name": "Missing Tenant ID (400)",
                    "request": {
                        "method": "GET",
                        "url": "{{service_url}}/v1/tenant-provisioner"
                    },
                    "event": [
                        {
                            "listen": "test",
                            "script": {
                                "exec": [
                                    "pm.test('Returns 400 without X-Tenant-ID', function () {",
                                    "    pm.response.to.have.status(400);",
                                    "    var json = pm.response.json();",
                                    "    pm.expect(json.error).to.eql('missing X-Tenant-ID');",
                                    "});"
                                ]
                            }
                        }
                    ]
                }
            ]
        },
        {
            "name": "Audit Service",
            "item": [
                {
                    "name": "List Audit Entries",
                    "request": {
                        "method": "GET",
                        "url": "{{service_url}}/v1/audit",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
                        ]
                    }
                },
                {
                    "name": "Create Audit Entry",
                    "request": {
                        "method": "POST",
                        "url": "{{service_url}}/v1/audit",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
                            { "key": "Content-Type", "value": "application/json" }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\"action\": \"test.audit.created\", \"details\": \"Manual test entry\"}"
                        }
                    }
                }
            ]
        },
        {
            "name": "Marketplace",
            "item": [
                {
                    "name": "List Marketplace",
                    "request": {
                        "method": "GET",
                        "url": "{{service_url}}/v1/marketplace",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
                        ]
                    }
                },
                {
                    "name": "Install Module",
                    "request": {
                        "method": "POST",
                        "url": "{{service_url}}/v1/marketplace",
                        "header": [
                            { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
                            { "key": "Content-Type", "value": "application/json" }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\"module_id\": \"example-module\", \"version\": \"1.0.0\"}"
                        }
                    }
                }
            ]
        }
    ]
}
```

---

## Running the Collection

1. **Import**: File > Import > paste the collection JSON above.
2. **Environment**: File > Import > paste the environment JSON above.
3. **Select Environment**: Set "ERP-Platform Local" as active.
4. **Run**: Use the Collection Runner to execute all requests in sequence.
5. **Review**: Check the test results tab for pass/fail status.

---

*For API documentation, see [21-API-Documentation.md](./21-API-Documentation.md). For webhook specifications, see [23-Webhook-Specifications.md](./23-Webhook-Specifications.md).*

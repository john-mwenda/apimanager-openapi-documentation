# API Catalog Server

## Overview

The API Catalog Server is a centralized repository for storing and serving OpenAPI specification files in JSON format.

It provides a consistent location for API documentation consumed by developer portals, API documentation viewers such as Scalar, Swagger UI, and Redoc, and other authorized applications.

The catalog is hosted using **Nginx**, which serves the JSON files directly without requiring a backend application.

## 1. Directory Structure

**Root directory:** `/var/www/api-catalog-server/`

API specifications should be organized by subsidiary or business domain.

```text
/var/www/api-catalog-server/
├── README.md
├── life/
│   ├── policy-api.json
│   └── claims-api.json
├── general/
│   ├── motor-api.json
│   └── claims-api.json
├── micro/
│   └── insurance-api.json
└── pensions/
    └── members-api.json
```

Each JSON file should contain a valid OpenAPI specification, preferably OpenAPI 3.x.

## 2. URL Structure

The catalog is exposed through the `/api-catalog/` URL path.

**Base URL:**

```text
http://<server-hostname>/api-catalog/
```

Example endpoints:

| Subsidiary | Endpoint |
|---|---|
| Life Insurance | `/api-catalog/life/policy-api.json` |
| General Insurance | `/api-catalog/general/motor-api.json` |
| Microinsurance | `/api-catalog/micro/insurance-api.json` |
| Pensions | `/api-catalog/pensions/members-api.json` |

The configured domain name and HTTPS settings depend on the deployment environment.

## 3. Nginx Configuration

The virtual host should be configured in `/etc/nginx/conf.d/` or another included Nginx configuration file.

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    # Default website
    location / {
        try_files $uri $uri/ =404;
    }

    # API catalog specifications
    location /api-catalog/ {
        alias /var/www/api-catalog-server/;

        try_files $uri =404;
        autoindex off;

        add_header Access-Control-Allow-Origin "*" always;
        add_header X-Content-Type-Options "nosniff" always;
    }
}
```

The primary Nginx configuration must include `/etc/nginx/mime.types` to return the correct `application/json` content type for JSON files.

**Note:** This configuration serves existing files within the catalog directory, not exclusively JSON files. Access to non-JSON files should be restricted if the directory contains operational documentation or other internal files.

## 4. Adding an API Specification

### Step 1: Identify the subsidiary

Determine which subsidiary or business domain owns the API.

### Step 2: Create the directory if necessary

```bash
sudo mkdir -p /var/www/api-catalog-server/life
```

### Step 3: Copy the OpenAPI specification

```bash
sudo cp policy-api.json \
  /var/www/api-catalog-server/life/
```

### Step 4: Configure file permissions

```bash
sudo chmod 644 \
  /var/www/api-catalog-server/life/policy-api.json
```

The Nginx worker user must have permission to read the file and traverse its parent directories.

### Step 5: Validate the JSON

```bash
python3 -m json.tool \
  /var/www/api-catalog-server/life/policy-api.json \
  > /dev/null
```

This validates JSON syntax only. OpenAPI schema validation should also be performed before publication.

### Step 6: Verify the published endpoint

```bash
curl -i \
  http://localhost/api-catalog/life/policy-api.json
```

Expected response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

Adding or updating JSON files normally does not require restarting Nginx.

## 5. Naming Conventions

Use consistent naming conventions for files and directories.

- Use lowercase characters.
- Separate words using hyphens.
- Avoid spaces and special characters.
- Use `.json` as the file extension.
- Organize specifications under the owning subsidiary or domain.

Examples:

```text
life/policy-api.json
general/motor-claims-api.json
micro/customer-api.json
pensions/member-registration-api.json
```

Where multiple API versions need to coexist, use versioned directories:

```text
life/
└── policy/
    ├── v1/
    │   └── openapi.json
    └── v2/
        └── openapi.json
```

Do not overwrite specifications for an existing API version with breaking contract changes without an approved versioning process.

## 6. Security and Access Control

The API Catalog Server hosts API documentation only. It does not execute API operations, handle authentication for downstream APIs, or route business transactions.

Security requirements:

- Publish only approved API specifications.
- Do not store passwords, API keys, tokens, or other secrets in specification files.
- Restrict sensitive or internal specifications through appropriate network controls or authentication.
- Use HTTPS in production environments.
- Configure CORS only for approved documentation consumers where necessary.
- Disable directory listing.
- Restrict write access to authorized administrators or deployment processes.

## 7. Operations and Troubleshooting

### Validate Nginx configuration

```bash
sudo nginx -t
```

### Reload configuration changes

```bash
sudo systemctl reload nginx
```

### Check Nginx status

```bash
sudo systemctl status nginx
```

### Monitor access logs

```bash
sudo tail -f /var/log/nginx/access.log
```

### Monitor error logs

```bash
sudo tail -f /var/log/nginx/error.log
```

### Common issues

| Issue | Possible cause |
|---|---|
| 404 Not Found | File missing, incorrect URL or alias configuration |
| 403 Forbidden | Insufficient filesystem permissions or SELinux restrictions |
| Incorrect content type | Missing JSON MIME mapping |
| Browser CORS error | Missing or incorrect `Access-Control-Allow-Origin` header |
| Invalid specification | Malformed JSON or noncompliant OpenAPI schema |

On Oracle Linux, ensure the directory has an appropriate SELinux context, such as `httpd_sys_content_t`.

## 8. Maintenance and Governance

All API specification changes should follow the organization's API governance process.

Before publishing or updating a specification:

1. Confirm the API owner and subsidiary.
2. Validate JSON syntax and OpenAPI schema compliance.
3. Review API versioning and backward compatibility.
4. Ensure no confidential information or credentials are exposed.
5. Publish the approved specification.
6. Verify that the developer portal can retrieve and display the updated documentation.

Obsolete specifications should be archived or removed according to the approved API lifecycle policy.

---

**Maintained by:** Integration and API Management Team  
**Platform:** Nginx  
**Storage directory:** `/var/www/api-catalog-server/`  
**URL prefix:** `/api-catalog/`  
**Specification format:** OpenAPI JSON


## Overview

The Certificate Issuance API lets you issue a verifiable certificate from your own systems — a course platform, an HR tool, an exam pipeline — without a human filling out a form. You send us your certificate PDF and the holder's details; we stamp it with a verifiable cover page, record it, and email the finished certificate to the issuer and holder — the same delivery behavior as the web form.

**Base URL:** Certivoo runs as two fully independent deployments, one per country, each with its own accounts, database, and API keys:

| Deployment | Base URL |
|---|---|
| UAE | `https://certivoo.ae/api/v1` |
| Pakistan | `https://certivoo.pk/api/v1` |

Use the base URL matching where your issuer account was created. A key from one deployment will not authenticate against the other — there's no cross-deployment lookup, so a request sent to the wrong base URL fails with `401`, not a routing error.

## Authentication

Every request needs an API key in the `Authorization` header:

```
Authorization: Bearer vega_live_8f3a9c2e1b...
```

Generate a key from your Certivoo account under **Settings → API Keys**. Keys are shown once at creation — store them the way you'd store any production secret. There's no session or CSRF token involved; the key is the whole story.

## Issue a Certificate

```
POST /certificates/issue
Content-Type: multipart/form-data
```

This is a multipart request, not JSON, because you're sending a file alongside form fields.

### Request body

| Field                   | Type   | Required | Description                                             |
|--------------------------|--------|----------|-----------------------------------------------------------|
| `title`                  | string | yes      | Certificate title (10–60 characters)                     |
| `holder_name`             | string | yes      | Full name of the person receiving the certificate         |
| `holder_email`            | string | yes      | Holder's email address                                     |
| `holder_identification`   | string | yes      | Student/employee ID, passport number, or similar reference |
| `pdf_upload`              | file   | yes      | Your certificate PDF, before Certivoo's cover page is added |

### Example request

```bash
curl -X POST https://certivoo.ae/api/v1/certificates/issue \
  -H "Authorization: Bearer vega_live_8f3a9c2e1b..." \
  -F "title=Certificate of Completion - Data Science" \
  -F "holder_name=Jane Doe" \
  -F "holder_email=jane@example.com" \
  -F "holder_identification=STUDENT-4471" \
  -F "pdf_upload=@./base_certificate.pdf"
```

### Example response — `201 Created`

```json
{
  "reference": "5-20260717-1",
  "issued_at": "2026-07-17T09:12:00Z",
  "status": "issued"
}
```

`reference` is the only pointer you need. The certificate PDF itself is emailed to the issuer and holder, not returned in this response.

**A note on what's deliberately missing:** this response does not include the certificate's file hash or the file itself. That's not an oversight — certificates can be revoked after issuance, and a hash only proves a file's *contents* haven't changed, not whether it's still *valid*. If we handed back a hash, integrators would be tempted to verify certificates locally by comparing hashes, which would silently accept a certificate we've since revoked. Verification is intentionally a single-source operation: it always goes through `POST /certificates/verify` on the *same deployment that issued the certificate* (or that deployment's verify page — `certivoo.ae/certificates/verify` or `certivoo.pk/certificates/verify`), which checks revocation and issuer status at request time, not just file integrity.

### Errors

| Status | Meaning                                                        |
|--------|-----------------------------------------------------------------|
| `400`  | Malformed request — missing a required field                    |
| `401`  | Missing, invalid, or revoked API key                            |
| `422`  | A field failed validation (e.g. PDF exceeds the size limit)     |
| `429`  | Daily issuance limit reached for this account                   |

Error bodies follow a consistent shape:

```json
{
  "error": "validation_error",
  "field": "pdf_upload",
  "message": "File size must be 2.0 MB or less."
}
```

## Rate limits

Each account has a daily certificate issuance limit tied to its plan. The current usage isn't exposed via a header yet — if you're issuing in bulk, build in retry-with-backoff on `429`.

## Versioning

The API is versioned in the URL path (`/v1/`). Breaking changes will ship under a new version rather than mutating `/v1/` in place; additive changes (new optional fields, new response fields) may appear without a version bump.

# Certivoo API Documentation

[Certivoo](https://certivoo.ae) is a certificate verification platform that lets issuers — universities, training providers, employers — issue tamper-evident, verifiable certificates. It runs as two independent deployments: [certivoo.ae](https://certivoo.ae) (UAE) and [certivoo.pk](https://certivoo.pk) (Pakistan).

This repository contains the documentation for the **Certificate Issuance API**, arriving in the next release of Certivoo. The API lets issuers issue certificates programmatically from their own systems — a course platform, an HR tool, an exam pipeline — instead of filling out a web form for each certificate.

## Contents

| File | Description |
|---|---|
| [certificate_issuance.md](certificate_issuance.md) | Full API reference — authentication, request/response, errors, rate limits, and the design reasoning behind the response shape |
| [openapi.yaml](openapi.yaml) | Machine-readable OpenAPI 3.0 specification |

## Exploring the spec

GitHub displays `openapi.yaml` as plain YAML. To browse it interactively, paste its contents into the [Swagger Editor](https://editor.swagger.io) — you'll get a rendered, clickable reference with example requests and responses.

## About

Written and maintained by Suhail — Bitcoin educator, author, and builder of cryptographic trust infrastructure. Feedback and questions are welcome via issues on this repository.

# API CRM Integration Documentation

This documentation describes the API CRM integration system that handles authentication, API calls, and logging for the Algar Telecom CRM API.

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Authentication Flow](#authentication-flow)
4. [Main Components](#main-components)
5. [Logging System](#logging-system)
6. [Usage Examples](#usage-examples)
7. [Error Handling](#error-handling)
8. [Snowflake Integration](#snowflake-integration)

## Overview

The API CRM integration system provides a robust interface to interact with the Algar Telecom CRM API. It handles:

- OAuth 2.0 authentication with automatic token refresh
- API request management
- Comprehensive logging
- Snowflake database integration for credential storage and logging

## System Architecture

The system follows a modular design with these key components:

1. **Authentication Module**: Handles OAuth 2.0 token management
2. **API Client**: Makes HTTP requests to the CRM API
3. **Logging System**: Records detailed information about each operation
4. **Snowflake Integration**: Stores credentials and logs in Snowflake tables

## Authentication Flow

The system uses OAuth 2.0 with the following flow:

1. Retrieve stored credentials from Snowflake
2. Make API calls using the stored access token
3. If authentication fails (401 error):
   - Request a new grant code
   - Exchange the grant code for a new access token
   - Update the stored token in Snowflake
   - Retry the API call

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Get Token  │────▶│  API Call   │────▶│   Success   │
└─────────────┘     └─────────────┘     └─────────────┘
                          │
                          │ 401 Error
                          ▼
                    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
                    │ Get Grant   │────▶│ Get Access  │────▶│  Update     │
                    │ Code        │     │ Token       │     │  Token      │
                    └─────────────┘     └─────────────┘     └─────────────┘
                                                                  │
                                                                  ▼
                                                            ┌─────────────┐
                                                            │ Retry API   │
                                                            │ Call        │
                                                            └─────────────┘
```

## Main Components

### Credential Management

Credentials are stored in the `DWDEV.DATASCIENCE.API_CRM_CREDENTIALS` Snowflake table:

- `CLIENT_ID`: OAuth client ID
- `CLIENT_SECRET`: OAuth client secret
- `REFRESH_TOKEN`: Current access token

### Token Refresh Function

The `refresh_token()` function handles the OAuth 2.0 token refresh process:

1. Requests a grant code from `/oauth/grant-code`
2. Exchanges the grant code for an access token via `/oauth/access-token`
3. Returns the new access token

### Main API Function

The `main()` function is the primary interface for making API calls:

```python
def main(client_id, url, payload):
    # Makes API calls with automatic token refresh
    # Returns the API response text
```

## Logging System

Two logging implementations are available:

### Simple Payload Logging

Records basic information about each API call:
- Timestamp
- Payload details
- HTTP status code

### Comprehensive Logging

Records detailed information about the entire process:
- Timestamps for each step
- Log levels (INFO, WARNING, ERROR, SUCCESS, DEBUG)
- Process stages
- Messages
- HTTP status codes
- Response times (in milliseconds)
- Error details

The comprehensive logging system stores logs in the `DWDEV.DATASCIENCE.API_CRM_LOGS` Snowflake table.

## Usage Examples

### Basic API Call

```python
from api_crm import main, CLIENT_ID

payload = {
    "circuitQuantity": 1,
    "maximumReLoyaltyDate": "2024-09-30 00:00:00.000",
    "workedAllCustomerRoot": "SIM",
    "workLogNote": "Criação via api",
    "consultorDocumentNumber": "02893814603",
    "customerSuccessDocumentNumber": "02893814603",
    "primaryDocumentNumber": "13612744000109",
    "circuits": ["A119099847"],
    "createdBy": "CLEUTORM",
    "positionId": "1016",
    "customerId": 3449299
}

result = main(
    CLIENT_ID,
    "https://api-sandbox.algartelecom.com.br/algar-telecom/corporate/algar-crm/v1/activityCustomerLoyalty",
    payload
)
print(result)
```

### Processing Multiple Payloads

```python
payloads = [
    {
        "circuitQuantity": 1,
        "maximumReLoyaltyDate": "2024-09-30 00:00:00.000",
        "workedAllCustomerRoot": "SIM",
        "workLogNote": "Criação via api",
        "consultorDocumentNumber": "02893814603",
        "customerSuccessDocumentNumber": "02893814603",
        "primaryDocumentNumber": "13612744000109",
        "circuits": ["A119099847"],
        "createdBy": "CLEUTORM",
        "positionId": "1016",
        "customerId": 3449299
    },
    # Additional payloads...
]

url = "https://api-sandbox.algartelecom.com.br/algar-telecom/corporate/algar-crm/v1/activityCustomerLoyalty"

for payload in payloads:
    main(CLIENT_ID, url, payload)
```

## Error Handling

The system includes comprehensive error handling:

- HTTP errors (4xx, 5xx)
- Connection timeouts
- Authentication failures
- Unexpected exceptions

All errors are logged with detailed information to help with troubleshooting.

## Snowflake Integration

### Tables

1. **API_CRM_CREDENTIALS**
   - Stores OAuth credentials
   - Schema: `CLIENT_ID`, `CLIENT_SECRET`, `REFRESH_TOKEN`

2. **API_CRM_LOGS** (Comprehensive Logging)
   - Schema:
     ```
     TIMESTAMP   VARCHAR,
     LEVEL       VARCHAR(10),
     ETAPA       VARCHAR(50),
     MENSAGEM    VARCHAR(2000),
     STATUS_CODE NUMBER,
     ELAPSED_MS  FLOAT,
     DETALHE     VARCHAR(4000)
     ```

3. **API_CRM_LOGS** (Simple Logging)
   - Schema:
     ```
     TIMESTAMP VARCHAR,
     CIRCUITQUANTITY NUMBER,
     MAXIMUMRELOYALTYDATE VARCHAR,
     WORKEDALLCUSTOMERROOT VARCHAR,
     WORKLOGNOTE VARCHAR,
     CONSULTORDOCUMENTNUMBER VARCHAR,
     CUSTOMERSUCCESSDOCUMENTNUMBER VARCHAR,
     PRIMARYDOCUMENTNUMBER VARCHAR,
     CIRCUITS VARCHAR,
     CREATEDBY VARCHAR,
     POSITIONID VARCHAR,
     CUSTOMERID NUMBER,
     STATUS_CODE NUMBER
     ```

### Snowflake Session Management

The system uses Snowpark's `get_active_session()` to interact with Snowflake, allowing it to run directly within Snowflake's compute environment.
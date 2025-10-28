# Azure Managed Redis Token Validator

This PowerShell script helps validate Microsoft Entra ID (formerly Azure Active Directory) tokens for use with Azure Managed Redis Cache resources. It performs comprehensive token validation and access policy verification to help diagnose authentication issues.

Use this script if your application is having trouble using Microsoft Entra ID to authenticate connections to Redis. It will analyze the Entra token you provide and output any issues it finds. 

Note that if your application is failing to acquire a token at all, this script won't be helpful. To diagnose issues preventing token acquisition, we recommend consulting [Entra access token documentation](https://learn.microsoft.com/entra/identity-platform/access-tokens).

## Features

- Decodes and validates JWT token structure
- Verifies token claims (audience, issuer, expiration)
- Checks access policy assignments on the Redis cache
- Provides detailed validation results with color-coded output
- Includes troubleshooting tips for common issues

## Prerequisites

- PowerShell 5.1 or later
- One of the following:
  - Azure CLI (az) installed and authenticated

## Usage

```powershell
.\Validate-RedisToken.ps1 -Token $token -ResourceId "/subscriptions/{sub-id}/resourceGroups/{rg-name}/providers/Microsoft.Cache/redisEnterprise/{cache-name}"
```

### Parameters

- `Token`: The Microsoft Entra ID JWT token to validate
- `ResourceId`: The Azure resource ID of the Azure Managed Redis Cache

### Getting a Token for Testing
Token acquisition options for testing or production diagnostics.

- Azure CLI

```bash
az account get-access-token --resource https://redis.azure.com --query accessToken -o tsv
```

- PowerShell (Az module)

```powershell
$token = (Get-AzAccessToken -ResourceUrl 'https://redis.azure.com').Token
```

- From your application (capture)

If you need to capture a token from a running application (debugger snapshot, secure trace collector, etc.), follow strict security practices (see below). Capture only what you need and avoid storing tokens in plaintext.

Security best practices when handling captured tokens:

- Store captured tokens only in encrypted/sealed storage and restrict access to a minimal set of principals.
- Purge captured tokens immediately after testing or diagnostics are complete.
- Never log full token values. If logging is unavoidable for debugging, redact the value and rotate the credential immediately afterward.

These steps help ensure captured tokens are used safely during diagnostics and are not a source of credential leakage.

## Example Output

The script provides detailed validation results including:
- Token structure verification
- Claims analysis (aud, iss, exp, etc.)
- Access policy assignment checks
- Color-coded success/warning/error messages
- Troubleshooting tips for failed validations

## Security Notes

- Never store or log tokens in plain text
- Rotate credentials after capturing tokens for diagnostics
- Use secure channels when sharing tokens for troubleshooting
- Ensure minimal required permissions when checking access policies

## Common Issues and Troubleshooting

1. Token Audience Issues
   - Ensure the token is requested with the correct resource: `https://redis.azure.com`
   - Use the `.default` scope for access token requests

2. Access Policy Problems
   - Ensure you have Reader or higher role to view assignments
   - Verify the user/service principal/managed identity has an access policy assignment

3. Expiration Issues
   - Tokens typically expire after 1 hour
   - Request a new token if expired

4. DefaultAzureCredential with Multiple Managed Identities
   - When multiple managed identities are assigned to a resource, DefaultAzureCredential follows this behavior:
     * First tries the system-assigned managed identity (if exists)
     * Then attempts user-assigned managed identities in an undefined order
     * This can lead to using the wrong identity if not explicitly specified

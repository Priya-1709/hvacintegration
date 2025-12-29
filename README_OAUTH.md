# OAuth 2.0 Authentication Manager

This document describes the OAuth 2.0 Authentication Manager for ServiceTitan integration, which provides automatic token management with caching and refresh capabilities.

## Overview

The OAuth 2.0 Authentication Manager handles ServiceTitan API authentication using the client credentials flow. It provides:

- **Automatic token acquisition** using OAuth 2.0 client credentials
- **Token caching** to minimize API calls
- **Automatic token refresh** before expiration
- **Thread-safe operations** for concurrent requests
- **Error handling** with automatic retry on authentication failures
- **Legacy token support** for backward compatibility

## Configuration

### OAuth 2.0 Configuration (Recommended)

Set these environment variables for OAuth authentication:

```bash
# OAuth client credentials
ST_CLIENT_ID=your_oauth_client_id_here
ST_CLIENT_SECRET=your_oauth_client_secret_here

# ServiceTitan configuration
ST_TENANT_ID=your_tenant_id_here
ST_BOOKING_PROVIDER_ID=your_booking_provider_id_here
ST_APP_KEY=your_app_key_here

# Optional OAuth settings
ST_AUTH_URL=https://auth-integration.servicetitan.io/connect/token
ST_TOKEN_REFRESH_BUFFER=300  # Refresh 5 minutes before expiry
```

### Legacy Token Configuration (Fallback)

For backward compatibility, you can still use static access tokens:

```bash
# Legacy configuration
SERVICETITAN_ACCESS_TOKEN=your_static_access_token_here
SERVICETITAN_TENANT_ID=your_tenant_id_here
SERVICETITAN_PROVIDER_ID=your_booking_provider_id_here
SERVICETITAN_APP_KEY=your_app_key_here
```

## Usage

### Basic Usage

```python
from api.oauth_manager import get_oauth_manager, get_servicetitan_headers

# Get OAuth manager (singleton)
oauth_manager = get_oauth_manager()

# Get access token
token = oauth_manager.get_access_token()

# Get complete headers for API requests
headers = get_servicetitan_headers(app_key="your_app_key")

# Make API request
import requests
response = requests.get("https://api.servicetitan.io/...", headers=headers)
```

### Advanced Usage

```python
from api.oauth_manager import ServiceTitanOAuthManager, OAuthError

# Create custom OAuth manager
oauth_manager = ServiceTitanOAuthManager(
    client_id="your_client_id",
    client_secret="your_client_secret", 
    tenant_id="your_tenant_id",
    auth_url="https://auth-integration.servicetitan.io/connect/token",
    refresh_buffer_seconds=300
)

try:
    # Get authorization header
    auth_header = oauth_manager.get_authorization_header()
    
    # Get token information
    token_info = oauth_manager.get_token_info()
    print(f"Token expires at: {token_info['expires_at']}")
    
    # Manually invalidate token (forces refresh on next request)
    oauth_manager.invalidate_token()
    
except OAuthError as e:
    print(f"OAuth error: {e.message}")
```

### Error Handling

```python
from api.oauth_manager import handle_auth_error, OAuthError
import requests

# Make API request with automatic auth error handling
try:
    response = requests.get(api_url, headers=headers)
    handle_auth_error(response)  # Automatically handles 401/403 errors
    
    # Process successful response
    data = response.json()
    
except OAuthError as e:
    print(f"Authentication failed: {e.message}")
    # Token will be automatically refreshed on next request
```

## Features

### Token Caching

The OAuth manager caches access tokens to minimize API calls:

- Tokens are cached in memory until expiration
- Multiple requests use the same cached token
- Automatic refresh before expiration (configurable buffer)

### Thread Safety

The OAuth manager is thread-safe for concurrent applications:

- Thread-safe token acquisition and caching
- Atomic token refresh operations
- Safe for use in multi-threaded environments

### Automatic Refresh

Tokens are automatically refreshed before expiration:

- Configurable refresh buffer (default: 5 minutes before expiry)
- Automatic retry on authentication failures
- Transparent to application code

### Error Handling

Comprehensive error handling for various scenarios:

- Invalid client credentials
- Network connectivity issues
- ServiceTitan API errors
- Token expiration and refresh failures

## Integration with Existing Code

The OAuth manager integrates seamlessly with existing ServiceTitan API modules:

### Job Type Mapper

```python
from api.job_type_mapper import map_job_type

# OAuth authentication is handled automatically
job_type_id = map_job_type("Plumbing Repair")
```

### Booking Creator

```python
from api.booking_creator import create_booking

# OAuth authentication is handled automatically
booking_response = create_booking(booking_data, job_type_id, external_id)
```

### Webhook Handler

The main webhook handler automatically uses OAuth when configured:

```python
# Environment variables are automatically detected
# OAuth is used if ST_CLIENT_ID and ST_CLIENT_SECRET are set
# Falls back to legacy tokens if SERVICETITAN_ACCESS_TOKEN is set
```

## Testing

### Unit Tests

Run OAuth manager tests:

```bash
python -m pytest tests/test_oauth_manager.py -v
```

### Manual Testing

Test OAuth credentials:

```bash
python test_oauth_credentials.py
```

### Integration Testing

Test with existing integration tests:

```bash
python -m pytest tests/test_integration.py -v
```

## Troubleshooting

### Common Issues

1. **Invalid Client Error**
   ```
   Error: {"error":"invalid_client"}
   ```
   - Verify client ID and secret are correct
   - Ensure credentials are for the correct environment (integration vs production)
   - Check that OAuth application is properly configured in ServiceTitan

2. **Token Expiration**
   ```
   Error: Authentication failed - token may be expired
   ```
   - OAuth manager handles this automatically
   - Check network connectivity to auth endpoint
   - Verify system clock is accurate

3. **Permission Errors**
   ```
   Error: Authorization failed - insufficient permissions
   ```
   - Verify OAuth application has required scopes
   - Check ServiceTitan API permissions for your application

### Debug Mode

Enable debug logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)

# OAuth manager will log token acquisition and refresh operations
```

### Environment Validation

Validate your environment configuration:

```python
from api.oauth_manager import create_oauth_manager_from_env

try:
    oauth_manager = create_oauth_manager_from_env()
    token_info = oauth_manager.get_token_info()
    print("OAuth configuration is valid")
except Exception as e:
    print(f"Configuration error: {e}")
```

## Migration from Legacy Tokens

To migrate from static access tokens to OAuth:

1. **Obtain OAuth credentials** from ServiceTitan Developer Portal
2. **Update environment variables** to use OAuth configuration
3. **Test the integration** with new credentials
4. **Remove legacy token variables** once OAuth is working

The system supports both authentication methods simultaneously for smooth migration.

## Security Considerations

- **Never commit credentials** to version control
- **Use environment variables** for all sensitive configuration
- **Rotate credentials regularly** as per security policies
- **Monitor token usage** for unusual patterns
- **Use HTTPS only** for all API communications

## Production Deployment

For production deployment:

1. **Use production OAuth credentials** (not integration/sandbox)
2. **Set production auth URL**: `https://auth.servicetitan.io/connect/token`
3. **Configure monitoring** for authentication failures
4. **Set up alerts** for high error rates
5. **Test failover scenarios** and token refresh

## Support

For OAuth-related issues:

- **ServiceTitan Developer Portal**: OAuth application configuration
- **ServiceTitan Support**: API access and permissions
- **Integration Tests**: Verify functionality with test credentials
- **Debug Logging**: Enable detailed logging for troubleshooting
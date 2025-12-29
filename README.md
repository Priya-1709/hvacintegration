# Retell ServiceTitan Integration

A Python serverless function deployed on Vercel that processes Retell AI webhook data and creates ServiceTitan bookings using the ServiceTitan Booking Provider API.

## Features

- **Dynamic Job Type Mapping**: Automatically maps spoken job types to ServiceTitan job types without hardcoding
- **Idempotent Booking Creation**: Prevents duplicate bookings using unique external IDs
- **Comprehensive Error Handling**: Handles various ServiceTitan API error scenarios
- **Production Ready**: Deployed as a serverless function with proper environment variable management

## Project Structure

```
├── api/                           # Vercel serverless functions
│   ├── webhook.py                # Main webhook handler
│   ├── validator.py              # Input validation and normalization
│   ├── job_type_mapper.py        # Dynamic job type mapping
│   ├── external_id_generator.py  # Unique ID generation
│   ├── booking_creator.py        # ServiceTitan booking creation
│   └── error_handler.py          # Error handling utilities
├── tests/                        # Test suite
│   ├── test_integration.py       # Integration tests
│   └── test_validator_properties.py # Property-based tests
├── requirements.txt              # Python dependencies
├── vercel.json                   # Vercel deployment configuration
├── .env.example                  # Environment variables template
└── README.md                     # This file
```

## Environment Variables

The integration supports two authentication methods:

### OAuth 2.0 Authentication (Recommended)

For automatic token management with OAuth 2.0 client credentials:

- **`ST_CLIENT_ID`**: OAuth client ID from ServiceTitan Developer Portal
- **`ST_CLIENT_SECRET`**: OAuth client secret from ServiceTitan Developer Portal
- **`ST_TENANT_ID`**: Your ServiceTitan tenant ID
- **`ST_BOOKING_PROVIDER_ID`**: Your ServiceTitan booking provider ID
- **`ST_APP_KEY`**: ServiceTitan application key (optional)
- **`ST_AUTH_URL`**: OAuth token endpoint (default: integration environment)

### Legacy Token Authentication (Fallback)

For backward compatibility with static access tokens:

- **`SERVICETITAN_ACCESS_TOKEN`**: Your ServiceTitan API access token
- **`SERVICETITAN_TENANT_ID`**: Your ServiceTitan tenant ID
- **`SERVICETITAN_PROVIDER_ID`**: Your ServiceTitan booking provider ID
- **`SERVICETITAN_APP_KEY`**: ServiceTitan application key (optional)

See [OAuth Documentation](README_OAUTH.md) for detailed configuration and usage.

### Required Variables (Legacy)

- **`SERVICETITAN_ACCESS_TOKEN`**: Your ServiceTitan API access token
  - Obtain from ServiceTitan Developer Portal
  - Must have permissions for Job Types API and Booking Provider API
  
- **`SERVICETITAN_TENANT_ID`**: Your ServiceTitan tenant ID
  - Found in ServiceTitan Developer Portal under your application settings
  
- **`SERVICETITAN_BOOKING_PROVIDER_ID`**: Your ServiceTitan booking provider ID
  - Configure in ServiceTitan under Settings > Booking Providers

### Optional Variables

- **`SERVICETITAN_API_BASE_URL`**: ServiceTitan API base URL (default: `https://api.servicetitan.io`)
  - Only change if using a different ServiceTitan environment

## Local Development

### Prerequisites

- Python 3.9 or higher
- pip package manager
- ServiceTitan API credentials

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd retell-servicetitan-integration
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your ServiceTitan API credentials
   ```

4. **Run tests**
   ```bash
   pytest tests/
   ```

## Deployment

### Vercel Deployment

This project is designed for deployment on Vercel's serverless platform.

#### Prerequisites

- Vercel account
- Vercel CLI installed (`npm install -g vercel`)
- ServiceTitan API credentials

#### Deployment Steps

1. **Install Vercel CLI** (if not already installed)
   ```bash
   npm install -g vercel
   ```

2. **Login to Vercel**
   ```bash
   vercel login
   ```

3. **Set up environment variables in Vercel**
   
   You can set environment variables either through the Vercel dashboard or CLI:
   
   **Option A: Using Vercel Dashboard**
   - Go to your project settings in Vercel dashboard
   - Navigate to "Environment Variables"
   - Add the required variables listed above
   
   **Option B: Using Vercel CLI**
   ```bash
   vercel env add SERVICETITAN_ACCESS_TOKEN
   vercel env add SERVICETITAN_TENANT_ID
   vercel env add SERVICETITAN_BOOKING_PROVIDER_ID
   vercel env add SERVICETITAN_API_BASE_URL
   ```

4. **Deploy to Vercel**
   ```bash
   vercel --prod
   ```

5. **Verify deployment**
   - Your webhook endpoint will be available at: `https://your-project.vercel.app/api/webhook`
   - Test the endpoint with a sample Retell AI payload

#### Environment Variable Configuration

For production deployment, configure these environment variables in your Vercel project:

| Variable | Description | Required |
|----------|-------------|----------|
| `SERVICETITAN_ACCESS_TOKEN` | ServiceTitan API access token | Yes |
| `SERVICETITAN_TENANT_ID` | ServiceTitan tenant ID | Yes |
| `SERVICETITAN_BOOKING_PROVIDER_ID` | ServiceTitan booking provider ID | Yes |
| `SERVICETITAN_API_BASE_URL` | ServiceTitan API base URL | No (defaults to https://api.servicetitan.io) |

### Alternative Deployment Platforms

While optimized for Vercel, this serverless function can be adapted for other platforms:

- **AWS Lambda**: Modify the handler function signature
- **Google Cloud Functions**: Adjust the request/response handling
- **Azure Functions**: Update the function binding configuration

## API Documentation

### Webhook Endpoint

**URL**: `POST /api/webhook`

**Request Body**: JSON payload from Retell AI containing customer booking information

**Required Fields**:
- `firstName` (string): Customer's first name
- `lastName` (string): Customer's last name  
- `phone` (string): Customer's phone number
- `email` (string): Customer's email address
- `address` (object): Customer's address with street, city, state, zip, country
- `jobType` (string): Spoken job type from customer
- `summary` (string): Job description/summary
- `startTime` (string): Appointment start time (UTC ISO string)
- `endTime` (string): Appointment end time (UTC ISO string)

**Optional Fields**:
- `source` (string): Lead source
- `isFirstTimeClient` (string): Whether customer is first-time client

**Response Format**:
```json
{
  "status": "BOOKED|ALREADY_EXISTS|ERROR",
  "externalId": "RETELL-{timestamp}-{phone}",
  "jobTypeId": 123,
  "message": "Booking created successfully"
}
```

## Monitoring and Troubleshooting

### Common Issues

1. **Authentication Errors (401/403)**
   - Verify ServiceTitan access token is valid and not expired
   - Check that token has required API permissions

2. **Job Type Not Found**
   - Ensure spoken job type matches available ServiceTitan job types
   - Check that job types are marked as active in ServiceTitan

3. **Duplicate Booking (409)**
   - This is expected behavior for idempotency
   - System returns "ALREADY_EXISTS" status

### Logs

Monitor function logs in Vercel dashboard under "Functions" tab for debugging.

## Testing

### Running Tests

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest tests/ --cov=api

# Run property-based tests only
pytest tests/test_validator_properties.py
```

### Test Types

- **Unit Tests**: Test individual components with specific examples
- **Property-Based Tests**: Test universal properties across randomized inputs
- **Integration Tests**: Test end-to-end webhook processing with mocked APIs

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

## License

[Add your license information here]
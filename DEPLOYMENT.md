# Deployment Guide: Retell ServiceTitan Integration

This guide provides step-by-step instructions for deploying the Retell ServiceTitan Integration webhook to production.

## Prerequisites

Before deploying, ensure you have:

1. **ServiceTitan API Access**
   - Valid ServiceTitan developer account
   - API access token with required permissions
   - Tenant ID and Booking Provider ID

2. **Vercel Account**
   - Free or paid Vercel account
   - Vercel CLI installed locally

3. **Required Permissions**
   - ServiceTitan Job Types API access
   - ServiceTitan Booking Provider API access

## Pre-Deployment Checklist

- [ ] ServiceTitan API credentials obtained and tested
- [ ] Vercel account created and CLI installed
- [ ] All tests passing locally (`pytest tests/`)
- [ ] Environment variables documented and ready
- [ ] Webhook endpoint URL planned for Retell AI configuration

## Step-by-Step Deployment

### 1. Prepare ServiceTitan Configuration

#### Obtain API Credentials

1. **Access Token**
   - Log into ServiceTitan Developer Portal
   - Navigate to "Applications" > Your App > "API Keys"
   - Generate or copy your access token
   - Ensure token has these scopes:
     - `job-types:read`
     - `booking-provider:write`

2. **Tenant ID**
   - Found in ServiceTitan Developer Portal
   - Located under "Applications" > Your App > "Settings"
   - Format: Usually a numeric ID

3. **Booking Provider ID**
   - Configure in ServiceTitan main application
   - Go to "Settings" > "Booking Providers"
   - Create or identify your booking provider
   - Note the provider ID

#### Test API Access

Before deployment, verify your credentials work:

```bash
# Test Job Types API access
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     "https://api.servicetitan.io/jpm/v2/tenant/YOUR_TENANT_ID/job-types"

# Should return list of job types
```

### 2. Set Up Vercel Project

#### Install Vercel CLI

```bash
npm install -g vercel
```

#### Login to Vercel

```bash
vercel login
```

#### Initialize Project

From your project directory:

```bash
vercel
```

Follow the prompts:
- Link to existing project or create new one
- Choose project name (e.g., `retell-servicetitan-integration`)
- Select framework: "Other"
- Confirm settings

### 3. Configure Environment Variables

#### Option A: Vercel Dashboard

1. Go to [vercel.com/dashboard](https://vercel.com/dashboard)
2. Select your project
3. Go to "Settings" > "Environment Variables"
4. Add each variable:

| Name | Value | Environment |
|------|-------|-------------|
| `SERVICETITAN_ACCESS_TOKEN` | Your API access token | Production, Preview, Development |
| `SERVICETITAN_TENANT_ID` | Your tenant ID | Production, Preview, Development |
| `SERVICETITAN_BOOKING_PROVIDER_ID` | Your booking provider ID | Production, Preview, Development |
| `SERVICETITAN_API_BASE_URL` | `https://api.servicetitan.io` | Production, Preview, Development |

#### Option B: Vercel CLI

```bash
# Set production environment variables
vercel env add SERVICETITAN_ACCESS_TOKEN production
vercel env add SERVICETITAN_TENANT_ID production
vercel env add SERVICETITAN_BOOKING_PROVIDER_ID production
vercel env add SERVICETITAN_API_BASE_URL production

# Set preview environment variables (optional)
vercel env add SERVICETITAN_ACCESS_TOKEN preview
vercel env add SERVICETITAN_TENANT_ID preview
vercel env add SERVICETITAN_BOOKING_PROVIDER_ID preview
vercel env add SERVICETITAN_API_BASE_URL preview
```

### 4. Deploy to Production

#### Deploy Command

```bash
vercel --prod
```

#### Verify Deployment

1. **Check Deployment Status**
   - Vercel will provide a deployment URL
   - Example: `https://retell-servicetitan-integration.vercel.app`

2. **Test Webhook Endpoint**
   ```bash
   curl -X POST https://your-project.vercel.app/api/webhook \
        -H "Content-Type: application/json" \
        -d '{
          "firstName": "Test",
          "lastName": "Customer",
          "phone": "555-123-4567",
          "email": "test@example.com",
          "address": {
            "street": "123 Main St",
            "city": "Anytown",
            "state": "CA",
            "zip": "12345",
            "country": "USA"
          },
          "jobType": "Plumbing",
          "summary": "Test booking",
          "startTime": "2024-01-15T10:00:00Z",
          "endTime": "2024-01-15T11:00:00Z"
        }'
   ```

3. **Expected Response**
   ```json
   {
     "status": "BOOKED",
     "externalId": "RETELL-1705320000-4567",
     "jobTypeId": 123,
     "message": "Booking created successfully"
   }
   ```

### 5. Configure Retell AI Webhook

#### Update Retell AI Configuration

1. Log into your Retell AI dashboard
2. Navigate to webhook settings
3. Set webhook URL to: `https://your-project.vercel.app/api/webhook`
4. Configure webhook to send POST requests
5. Test webhook delivery from Retell AI

#### Webhook Security (Optional)

For additional security, consider:

1. **IP Whitelisting**
   - Configure Vercel to only accept requests from Retell AI IPs
   - Add IP restrictions in Vercel project settings

2. **Webhook Signatures**
   - Implement webhook signature verification
   - Add signature validation in webhook handler

## Post-Deployment Verification

### 1. Monitor Function Logs

1. Go to Vercel dashboard
2. Select your project
3. Navigate to "Functions" tab
4. Monitor logs for any errors or issues

### 2. Test End-to-End Flow

1. **Trigger Test Call**
   - Make a test call through Retell AI
   - Provide booking information
   - Verify webhook is triggered

2. **Verify ServiceTitan Booking**
   - Check ServiceTitan dashboard
   - Confirm booking was created
   - Verify all customer details are correct

3. **Test Error Scenarios**
   - Try invalid job types
   - Test duplicate bookings
   - Verify error handling works correctly

### 3. Performance Monitoring

Monitor these metrics:
- **Response Times**: Should be under 5 seconds
- **Error Rates**: Should be minimal
- **Success Rates**: Should be high for valid requests

## Troubleshooting

### Common Deployment Issues

#### 1. Environment Variables Not Loading

**Symptoms**: 500 errors, authentication failures

**Solutions**:
- Verify environment variables are set in Vercel dashboard
- Check variable names match exactly (case-sensitive)
- Redeploy after adding variables: `vercel --prod`

#### 2. ServiceTitan API Errors

**Symptoms**: 401/403 errors in logs

**Solutions**:
- Verify access token is valid and not expired
- Check API permissions in ServiceTitan Developer Portal
- Test API access manually with curl

#### 3. Function Timeout

**Symptoms**: 504 Gateway Timeout errors

**Solutions**:
- Check ServiceTitan API response times
- Optimize API calls (reduce unnecessary requests)
- Consider implementing retry logic with exponential backoff

#### 4. Job Type Mapping Failures

**Symptoms**: "Job type not found" errors

**Solutions**:
- Verify job types exist and are active in ServiceTitan
- Check job type names match expected format
- Review job type mapping logic

### Debugging Steps

1. **Check Function Logs**
   ```bash
   vercel logs --follow
   ```

2. **Test Locally**
   ```bash
   # Set up local environment
   cp .env.example .env
   # Edit .env with your credentials
   
   # Run tests
   pytest tests/
   
   # Test specific functionality
   python -c "from api.job_type_mapper import map_job_type; print(map_job_type('Plumbing', 'token', 'tenant'))"
   ```

3. **Verify API Connectivity**
   ```bash
   # Test ServiceTitan API directly
   curl -H "Authorization: Bearer YOUR_TOKEN" \
        "https://api.servicetitan.io/jpm/v2/tenant/YOUR_TENANT/job-types"
   ```

## Maintenance

### Regular Tasks

1. **Monitor API Token Expiration**
   - ServiceTitan tokens may expire
   - Set up monitoring/alerts for authentication failures
   - Rotate tokens before expiration

2. **Update Dependencies**
   - Regularly update Python packages
   - Test thoroughly before deploying updates
   - Monitor security advisories

3. **Review Logs**
   - Weekly review of function logs
   - Monitor for new error patterns
   - Track success/failure rates

### Scaling Considerations

- **Rate Limits**: Monitor ServiceTitan API rate limits
- **Concurrent Requests**: Vercel handles scaling automatically
- **Error Handling**: Implement circuit breakers for high error rates

## Security Best Practices

1. **Environment Variables**
   - Never commit credentials to version control
   - Use Vercel's encrypted environment variable storage
   - Rotate credentials regularly

2. **API Security**
   - Monitor for unusual API usage patterns
   - Implement request logging for audit trails
   - Consider API rate limiting

3. **Error Handling**
   - Don't expose sensitive information in error messages
   - Log errors securely for debugging
   - Implement proper error categorization

## Support and Monitoring

### Monitoring Setup

Consider implementing:
- **Health Check Endpoint**: Simple endpoint to verify service status
- **Metrics Collection**: Track booking success rates, response times
- **Alerting**: Set up alerts for high error rates or service downtime

### Getting Help

- **Vercel Support**: For deployment and platform issues
- **ServiceTitan Support**: For API-related questions
- **Project Issues**: Use GitHub issues for bug reports and feature requests

## Rollback Procedure

If issues arise after deployment:

1. **Quick Rollback**
   ```bash
   # Rollback to previous deployment
   vercel rollback
   ```

2. **Identify Issues**
   - Check function logs
   - Review recent changes
   - Test problematic scenarios

3. **Fix and Redeploy**
   - Fix issues locally
   - Test thoroughly
   - Deploy with `vercel --prod`

This deployment guide ensures a smooth, secure, and maintainable production deployment of your Retell ServiceTitan Integration webhook.
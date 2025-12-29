# Production Deployment Checklist

Use this checklist to ensure all requirements are met before deploying the Retell ServiceTitan Integration to production.

## Pre-Deployment Requirements

### ✅ ServiceTitan API Setup
- [ ] ServiceTitan developer account created
- [ ] API access token obtained with required permissions:
  - [ ] `job-types:read` permission
  - [ ] `booking-provider:write` permission
- [ ] Tenant ID identified and documented
- [ ] Booking Provider ID configured in ServiceTitan
- [ ] API credentials tested manually (curl/Postman)

### ✅ Code Quality and Testing
- [ ] All unit tests passing (`pytest tests/`)
- [ ] All property-based tests passing
- [ ] Integration tests completed successfully
- [ ] Code reviewed and approved
- [ ] No hardcoded credentials or test data in code
- [ ] Error handling tested for all scenarios

### ✅ Environment Configuration
- [ ] `.env.example` file updated with all required variables
- [ ] Environment variables documented in README
- [ ] Production environment variables prepared:
  - [ ] `SERVICETITAN_ACCESS_TOKEN`
  - [ ] `SERVICETITAN_TENANT_ID`
  - [ ] `SERVICETITAN_BOOKING_PROVIDER_ID`
  - [ ] `SERVICETITAN_API_BASE_URL` (optional)

### ✅ Vercel Configuration
- [ ] `vercel.json` configured with proper runtime settings
- [ ] Vercel account set up and CLI installed
- [ ] Project linked to Vercel
- [ ] Environment variables configured in Vercel dashboard

## Deployment Process

### ✅ Pre-Deployment Testing
- [ ] Local testing completed with real ServiceTitan credentials
- [ ] Job type mapping tested with actual ServiceTitan job types
- [ ] Booking creation tested end-to-end
- [ ] Error scenarios tested (invalid job types, duplicate bookings)

### ✅ Production Deployment
- [ ] Environment variables set in Vercel production environment
- [ ] Deployed to production using `vercel --prod`
- [ ] Deployment successful (no build errors)
- [ ] Function accessible at production URL
- [ ] Health check performed on webhook endpoint

### ✅ Post-Deployment Verification
- [ ] Webhook endpoint responds to test requests
- [ ] ServiceTitan API integration working
- [ ] Job type mapping functioning correctly
- [ ] Booking creation successful in ServiceTitan
- [ ] Error handling working as expected
- [ ] Function logs accessible and clean

## Integration Testing

### ✅ Retell AI Integration
- [ ] Webhook URL configured in Retell AI
- [ ] Test webhook delivery from Retell AI successful
- [ ] End-to-end flow tested (Retell AI → Webhook → ServiceTitan)
- [ ] Response format compatible with Retell AI expectations

### ✅ ServiceTitan Integration
- [ ] Bookings appearing correctly in ServiceTitan dashboard
- [ ] Customer information mapping correctly
- [ ] Job types mapping to correct ServiceTitan job types
- [ ] Appointment times set correctly (timezone handling)
- [ ] External IDs preventing duplicate bookings

## Security and Compliance

### ✅ Security Measures
- [ ] No credentials committed to version control
- [ ] Environment variables encrypted in Vercel
- [ ] API tokens have minimal required permissions
- [ ] Error messages don't expose sensitive information
- [ ] Request/response logging configured appropriately

### ✅ Data Handling
- [ ] Customer data handled according to privacy requirements
- [ ] No unnecessary data stored or logged
- [ ] Data transmission encrypted (HTTPS)
- [ ] Compliance with relevant data protection regulations

## Monitoring and Maintenance

### ✅ Monitoring Setup
- [ ] Function logs accessible in Vercel dashboard
- [ ] Error tracking configured
- [ ] Performance monitoring in place
- [ ] Alert thresholds defined for:
  - [ ] High error rates
  - [ ] Function timeouts
  - [ ] API authentication failures

### ✅ Documentation
- [ ] README.md updated with deployment instructions
- [ ] API documentation complete
- [ ] Environment variables documented
- [ ] Troubleshooting guide available
- [ ] Deployment guide created

### ✅ Backup and Recovery
- [ ] Rollback procedure documented
- [ ] Previous deployment versions accessible
- [ ] Recovery plan for API credential issues
- [ ] Contact information for support documented

## Performance Requirements

### ✅ Response Time Targets
- [ ] Webhook response time under 5 seconds
- [ ] ServiceTitan API calls optimized
- [ ] Error responses returned quickly
- [ ] Timeout handling implemented

### ✅ Reliability Targets
- [ ] 99%+ uptime expected
- [ ] Graceful handling of ServiceTitan API downtime
- [ ] Retry logic for transient failures
- [ ] Circuit breaker pattern for repeated failures

## Final Sign-Off

### ✅ Stakeholder Approval
- [ ] Technical review completed
- [ ] Security review completed
- [ ] Business requirements validated
- [ ] Deployment approved by project owner

### ✅ Go-Live Readiness
- [ ] All checklist items completed
- [ ] Support team notified of deployment
- [ ] Monitoring alerts configured
- [ ] Documentation accessible to support team
- [ ] Rollback plan confirmed and tested

## Post-Deployment Tasks

### ✅ Immediate (Within 24 hours)
- [ ] Monitor function logs for errors
- [ ] Verify first production bookings successful
- [ ] Confirm Retell AI integration working
- [ ] Check ServiceTitan booking creation

### ✅ Short-term (Within 1 week)
- [ ] Review performance metrics
- [ ] Analyze error patterns
- [ ] Optimize based on real usage patterns
- [ ] Gather feedback from users

### ✅ Ongoing Maintenance
- [ ] Weekly log review scheduled
- [ ] Monthly performance review scheduled
- [ ] Quarterly security review scheduled
- [ ] API token rotation schedule established

---

**Deployment Date**: _______________

**Deployed By**: _______________

**Approved By**: _______________

**Production URL**: _______________

**Notes**: 
_Use this space for any deployment-specific notes or issues encountered_

---

## Emergency Contacts

- **Technical Lead**: _______________
- **ServiceTitan Support**: _______________
- **Vercel Support**: _______________
- **Retell AI Support**: _______________

## Quick Reference

- **Production URL**: `https://your-project.vercel.app/api/webhook`
- **Vercel Dashboard**: `https://vercel.com/dashboard`
- **ServiceTitan Developer Portal**: `https://developer.servicetitan.io`
- **Function Logs**: `vercel logs --follow`
- **Rollback Command**: `vercel rollback`
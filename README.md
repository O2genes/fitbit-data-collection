# Fitbit Research Data Collection System

## Overview
This web application enables researchers to collect Fitbit data from study participants through OAuth 2.0 authorization. The system allows participants to securely grant access to their Fitbit data for research purposes.

## Current Implementation Status

### ✅ Working Components
- OAuth 2.0 authorization flow with PKCE
- User consent and authorization interface
- Token exchange functionality
- Privacy policy and terms of service pages

### ⚠️ Security Issues to Address
- **CRITICAL**: Client secret exposed in frontend JavaScript
- No backend server for secure token management
- Manual token copying process

### 🔧 Missing Components
- Backend server infrastructure
- Database for secure token and participant data storage
- Automated data collection from Fitbit API
- Token refresh automation
- Data analysis tools

## Fitbit Web API Key Information

### Token Management
- **Access Token Lifespan**: 8 hours (28,800 seconds)
- **Refresh Token**: Single-use, doesn't expire
- **Continuous Access**: Possible through automated token refresh
- **User Control**: Users can revoke access anytime

### Available Data Types
- **Activity**: Steps, distance, calories, active minutes
- **Heart Rate**: Continuous and resting heart rate data
- **Sleep**: Duration, stages, efficiency
- **Profile**: Age, gender, height, weight (if provided)
- **Intraday Data**: High-frequency data (requires special approval)

### Rate Limits
- Personal apps: 150 requests per hour per user
- Research apps: Higher limits available with approval

## Recommended Architecture

### Phase 1: Secure Backend Implementation
```
Frontend (Current) → Backend Server → Database
     ↓                     ↓            ↓
  User Interface     Token Management  Data Storage
  OAuth Redirect     API Calls        Participant Data
  Participant UI     Data Collection   Analytics
```

### Phase 2: Automated Data Collection
```
Scheduler → Token Refresh → Fitbit API → Data Processing → Database
    ↓           ↓              ↓            ↓              ↓
  Cron Jobs   Auto-refresh   Bulk Data   Clean/Validate   Long-term
  6-hour      Before         Collection  Transform Data   Storage
  intervals   Expiration     All Users   Research Format  Analysis
```

## Security Improvements Needed

### 1. Move Client Secret to Backend
- ❌ Current: Client secret in JavaScript (public)
- ✅ Required: Client secret in secure backend environment

### 2. Implement Secure Token Storage
- Database with encryption for tokens
- Separate user IDs from tokens
- Secure API endpoints for data access

### 3. Add Authentication for Researchers
- Researcher login system
- Role-based access control
- Audit logs for data access

## Implementation Recommendations

### Backend Technology Options
1. **Node.js/Express** - Easy to implement, good for API handling
2. **Python/Django** - Excellent for data science and analysis
3. **PHP/Laravel** - Simple deployment, good documentation

### Database Requirements
```sql
-- Participants table
participants (
  id, email, consent_date, status, created_at
)

-- Tokens table (encrypted)
fitbit_tokens (
  participant_id, access_token, refresh_token, 
  expires_at, scopes, user_id, created_at
)

-- Data collection table
fitbit_data (
  participant_id, data_type, date, raw_data, 
  processed_data, collected_at
)
```

### Automated Data Collection Flow
1. **Token Refresh**: Every 6 hours (before 8-hour expiration)
2. **Data Collection**: Daily batch collection for all participants
3. **Data Storage**: Structured storage with participant anonymization
4. **Error Handling**: Retry failed requests, notify of revoked access

## For Research Use

### Intraday Data Access
- Requires separate application to Fitbit
- Provides minute-by-minute data
- Essential for detailed health research

### Multi-User Data Collection
- Single authorization URL for all participants
- Unique tokens per participant
- Bulk data collection capabilities

### Data Analysis Considerations
- Export capabilities (CSV, JSON)
- Statistical analysis integration
- Participant anonymization
- HIPAA/GDPR compliance features

## Next Steps

### Immediate (Security)
1. Create backend server environment
2. Move client secret to server-side
3. Implement secure token storage

### Short-term (Functionality)
1. Build automated token refresh system
2. Create data collection scheduler
3. Add researcher dashboard

### Long-term (Research Features)
1. Apply for intraday data access
2. Build data analysis tools
3. Add participant management features
4. Implement data export functionality

## Getting Started for Researchers

### Prerequisites
- Fitbit Developer Account
- Research application approval
- Backend hosting environment
- Database setup

### Configuration Steps
1. Update Fitbit app settings with secure redirect URL
2. Set up environment variables for sensitive data
3. Configure database and encryption
4. Deploy secure backend server
5. Test authorization flow with sample participants

## Support Resources
- [Fitbit Web API Documentation](https://dev.fitbit.com/build/reference/web-api/)
- [OAuth 2.0 Guide](https://dev.fitbit.com/build/reference/web-api/developer-guide/authorization/)
- [Research FAQ](https://enterprise.fitbit.com/researchers/faqs/)
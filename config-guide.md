# Configuration Guide for Fitbit Research Backend

## Environment Variables Setup

Create a `.env` file in your backend directory with the following variables:

```bash
# Fitbit API Configuration
FITBIT_CLIENT_ID=your_fitbit_client_id_here
FITBIT_CLIENT_SECRET=your_fitbit_client_secret_here
FITBIT_REDIRECT_URI=https://your-domain.com/api/auth/callback

# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/fitbit_research
REDIS_URL=redis://localhost:6379

# Security
ENCRYPTION_KEY=your_32_character_encryption_key_here
JWT_SECRET=your_jwt_secret_for_researcher_auth

# Server Configuration
PORT=3000
NODE_ENV=production

# Email Configuration (for notifications)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password

# Research Configuration
RESEARCH_ADMIN_EMAIL=researcher@university.edu
DATA_RETENTION_DAYS=365
```

## Database Setup

### PostgreSQL Schema
```sql
-- Create database
CREATE DATABASE fitbit_research;

-- Participants table
CREATE TABLE participants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    fitbit_user_id VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255),
    consent_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Encrypted tokens table
CREATE TABLE fitbit_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    participant_id UUID REFERENCES participants(id) ON DELETE CASCADE,
    access_token_encrypted TEXT NOT NULL,
    refresh_token_encrypted TEXT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    scopes TEXT NOT NULL,
    iv TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Data collection logs
CREATE TABLE data_collection_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    participant_id UUID REFERENCES participants(id) ON DELETE CASCADE,
    data_type VARCHAR(50) NOT NULL,
    collection_date DATE NOT NULL,
    status VARCHAR(20) NOT NULL,
    data_size INTEGER,
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Raw fitbit data storage
CREATE TABLE fitbit_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    participant_id UUID REFERENCES participants(id) ON DELETE CASCADE,
    data_type VARCHAR(50) NOT NULL,
    date DATE NOT NULL,
    raw_data JSONB NOT NULL,
    processed_data JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(participant_id, data_type, date)
);

-- Indexes for performance
CREATE INDEX idx_participants_fitbit_user_id ON participants(fitbit_user_id);
CREATE INDEX idx_tokens_participant_id ON fitbit_tokens(participant_id);
CREATE INDEX idx_tokens_expires_at ON fitbit_tokens(expires_at);
CREATE INDEX idx_data_participant_date ON fitbit_data(participant_id, date);
CREATE INDEX idx_data_type_date ON fitbit_data(data_type, date);
```

## Fitbit Developer App Configuration

### 1. Register Your Application
1. Go to [Fitbit Developer Portal](https://dev.fitbit.com/apps)
2. Click "Register a New App"
3. Fill out the application details:
   - **Application Name**: Your research project name
   - **Description**: Brief description of your research
   - **Application Website**: Your institution's website
   - **Organization**: Your research institution
   - **Organization Website**: Institution website
   - **OAuth 2.0 Application Type**: Server
   - **Callback URL**: `https://your-domain.com/api/auth/callback`
   - **Default Access Type**: Read & Write (or Read Only if sufficient)

### 2. Required Scopes for Research
Select appropriate scopes based on your research needs:
- `activity`: Steps, distance, calories, active minutes
- `heartrate`: Heart rate data
- `sleep`: Sleep duration and stages
- `profile`: Basic profile information
- `weight`: Weight and BMI data
- `nutrition`: Food log data (if applicable)
- `location`: GPS data (if applicable)

### 3. Apply for Intraday Data Access
If you need minute-by-minute data:
1. Submit a research application through Fitbit
2. Provide study protocol and IRB approval
3. Wait for approval (can take 2-4 weeks)

## Server Deployment

### Option 1: Docker Deployment
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
EXPOSE 3000

CMD ["node", "backend-example.js"]
```

### Option 2: Traditional Server Setup
1. Install Node.js 16+
2. Install PostgreSQL 12+
3. Install Redis (optional, for caching)
4. Set up SSL certificate for HTTPS
5. Configure reverse proxy (Nginx)

## Security Checklist

- [ ] Client secret stored in environment variables only
- [ ] Database connection encrypted (SSL)
- [ ] Tokens encrypted at rest
- [ ] HTTPS enabled on all endpoints
- [ ] Rate limiting implemented
- [ ] Input validation on all endpoints
- [ ] Audit logging enabled
- [ ] Regular security updates
- [ ] Backup strategy implemented
- [ ] Access control for researchers

## Monitoring and Maintenance

### Daily Tasks
- Check token refresh success rates
- Monitor data collection completion
- Review error logs

### Weekly Tasks
- Verify participant consent status
- Check data integrity
- Review security logs

### Monthly Tasks
- Update dependencies
- Review and archive old data
- Participant outreach if needed

## Compliance Considerations

### HIPAA Compliance (if applicable)
- Business Associate Agreements
- Encrypted data transmission
- Access audit trails
- Data retention policies

### GDPR Compliance (if applicable)
- Explicit consent collection
- Right to erasure implementation
- Data portability features
- Privacy impact assessments

### IRB Requirements
- Participant consent forms
- Data anonymization procedures
- Study protocol adherence
- Adverse event reporting

## Troubleshooting

### Common Issues
1. **Token Refresh Failures**
   - Check token expiration timing
   - Verify refresh token hasn't been used
   - Confirm participant hasn't revoked access

2. **Data Collection Gaps**
   - Verify participant device sync
   - Check API rate limits
   - Review network connectivity

3. **Database Connection Issues**
   - Verify connection string
   - Check database server status
   - Review SSL certificate validity

### Support Resources
- [Fitbit Web API Documentation](https://dev.fitbit.com/build/reference/web-api/)
- [Developer Community Forum](https://community.fitbit.com/t5/Web-API-Development/bd-p/fitbit-web-api)
- [API Support Contact](https://dev.fitbit.com/build/reference/web-api/help/) 
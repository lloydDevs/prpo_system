## PR to PO System - Production Setup Guide

### Security Checklist

Before deploying to production, ensure you have completed the following:

#### 1. Authentication & Credentials
- [ ] Remove hardcoded username/password from Login component
- [ ] Implement proper database-backed authentication
- [ ] Use bcrypt or similar for password hashing
- [ ] Remove console.log statements with sensitive data

#### 2. Environment Variables
- [ ] Create `.env.local` with production values
- [ ] Update `DB_PASSWORD` with a strong password
- [ ] Update `SESSION_SECRET` with a random 32+ character string
- [ ] Update `JWT_SECRET` if using JWT tokens
- [ ] Never commit `.env` to version control

#### 3. Database
- [ ] Create production database
- [ ] Run migration scripts
- [ ] Set up database backups
- [ ] Configure database user with limited permissions
- [ ] Enable SSL connections to database

#### 4. API Configuration
- [ ] Update `VITE_API_URL` to production domain
- [ ] Update `CORS_ORIGIN` to production domain
- [ ] Remove localhost references

#### 5. Build & Deployment
- [ ] Run `npm run build` and verify output
- [ ] Test production build locally with `npm run preview`
- [ ] Set up automated deployment pipeline
- [ ] Configure SSL/TLS certificates
- [ ] Enable GZIP compression

#### 6. Monitoring & Logging
- [ ] Set up error tracking (Sentry, etc.)
- [ ] Enable application logs
- [ ] Set up monitoring/alerting
- [ ] Configure backup strategies

#### 7. Performance
- [ ] Enable caching headers
- [ ] Optimize bundle size
- [ ] Enable code splitting
- [ ] Lazy load components where possible

### Database Setup Script

```sql
-- Create database
CREATE DATABASE prpo_system_prod;
USE prpo_system_prod;

-- Create entries table
CREATE TABLE entries (
  id INT AUTO_INCREMENT PRIMARY KEY,
  pr_number VARCHAR(50) UNIQUE NOT NULL,
  office_section VARCHAR(100),
  date DATE,
  fund_cluster VARCHAR(100),
  responsibility_code VARCHAR(50),
  purpose LONGTEXT,
  requested_by VARCHAR(100),
  approved_by VARCHAR(100),
  requested_by_position VARCHAR(100),
  approved_by_position VARCHAR(100),
  note LONGTEXT,
  items JSON,
  filename VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  UNIQUE KEY unique_pr_number (pr_number),
  INDEX idx_date (date),
  INDEX idx_office (office_section)
);

-- Create users table
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  role ENUM('admin', 'user', 'approver') DEFAULT 'user',
  is_active BOOLEAN DEFAULT TRUE,
  last_login TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_username (username),
  INDEX idx_email (email)
);

-- Create audit log table
CREATE TABLE audit_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  action VARCHAR(50),
  entity_type VARCHAR(50),
  entity_id INT,
  changes JSON,
  ip_address VARCHAR(45),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  INDEX idx_user_id (user_id),
  INDEX idx_created_at (created_at)
);

-- Create production user (remember to hash password!)
-- Use bcrypt to hash the password before inserting
INSERT INTO users (username, password_hash, email, role) VALUES 
('admin', 'bcrypt_hashed_password_here', 'admin@ched.gov.ph', 'admin');
```

### Environment Variables (Production)

```env
# Frontend
VITE_API_URL=https://api.prpo-system.gov.ph
VITE_APP_NAME="PR to PO System - Production"

# Backend
BACKEND_PORT=443
NODE_ENV=production

# Database Configuration
DB_HOST=prod-db.example.com
DB_USER=prpo_prod_user
DB_PASSWORD=your_very_strong_password_here
DB_NAME=prpo_system_prod
DB_PORT=3306
DB_POOL_SIZE=20

# Security
SESSION_SECRET=your_random_session_secret_at_least_32_chars
JWT_SECRET=your_random_jwt_secret_at_least_32_chars
CORS_ORIGIN=https://prpo-system.gov.ph

# Session Configuration
SESSION_TIMEOUT=3600000
SESSION_TIMEOUT_MINUTES=60

# Logging
LOG_LEVEL=warn
```

### Deployment Steps

#### Step 1: Build the Application
```bash
npm install
npm run lint
npm run build
```

#### Step 2: Prepare Server
```bash
# SSH into production server
ssh user@production-server

# Create application directory
mkdir -p /var/www/prpo-system
cd /var/www/prpo-system

# Copy built files
# scp -r dist/* user@production-server:/var/www/prpo-system/
```

#### Step 3: Configure Web Server (Nginx Example)
```nginx
server {
    listen 443 ssl http2;
    server_name prpo-system.gov.ph;

    ssl_certificate /etc/ssl/certs/prpo-system.gov.ph.crt;
    ssl_certificate_key /etc/ssl/private/prpo-system.gov.ph.key;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    root /var/www/prpo-system;
    index index.html;

    # Cache static files
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Handle SPA routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Disable access to sensitive files
    location ~ /\. {
        deny all;
    }
}
```

#### Step 4: Set Up Backend (Node.js with PM2)
```bash
# Install PM2
npm install -g pm2

# Start backend
pm2 start server.js --name "prpo-backend"

# Set PM2 to start on boot
pm2 startup
pm2 save
```

#### Step 5: SSL/TLS Certificate
```bash
# Using Certbot with Let's Encrypt
sudo certbot certonly --nginx -d prpo-system.gov.ph
```

### Backup & Recovery

```bash
# Backup database
mysqldump -u root -p prpo_system_prod > /backup/prpo_system_$(date +%Y%m%d_%H%M%S).sql

# Restore database
mysql -u root -p prpo_system_prod < /backup/prpo_system_20240101_120000.sql
```

### Monitoring & Maintenance

- Set up automated daily backups
- Monitor disk space and database size
- Review access logs regularly
- Keep dependencies updated
- Monitor error rates and performance metrics

### Support
For production deployment issues, contact: admin@ched.gov.ph

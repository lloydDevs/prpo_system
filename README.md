# PR to PO System

A React + Vite web application that converts Purchase Requests (PR) to Purchase Orders (PO) with PDF/Excel export capabilities. This system is designed for the Commission on Higher Education (CHED) MIMAROPA Region.

## Features

- **User Authentication**: Secure login with cookie-based sessions
- **Purchase Request Form**: Comprehensive form for entering purchase requisition details
- **Multi-format Export**: Generate PDF and Excel (.xlsx) documents
- **Item Management**: Add, edit, and remove line items dynamically
- **Database Integration**: Store entries in MySQL database
- **Responsive Design**: Bootstrap-based UI for desktop and mobile
- **Error Handling**: Global error boundary for graceful error handling

## Tech Stack

### Frontend
- **React 19** - UI framework
- **Vite 6** - Build tool and dev server with HMR
- **Bootstrap 5** - Responsive CSS framework
- **React Router** - Client-side routing
- **jsPDF + jsPDF-AutoTable** - PDF generation
- **XLSX & xlsx-populate** - Excel file handling
- **js-cookie** - Cookie management

### Backend
- **Node.js + Express** - REST API server
- **MySQL 2** - Database driver
- **CORS** - Cross-origin request handling
- **Body Parser** - Request body parsing

## Prerequisites

- **Node.js** (v16 or higher)
- **npm** (v8 or higher)
- **MySQL** (v8 or higher)

## Installation

### 1. Clone the Repository
```bash
git clone https://github.com/lloydDevs/prpo_system.git
cd prpo_system
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Setup

Create a `.env` file in the root directory:

```env
# Frontend
VITE_API_URL=http://localhost:3001

# Backend (if running backend locally)
BACKEND_PORT=3001
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=prpo_system
DB_PORT=3306

# Security
SESSION_SECRET=your_secret_key_here
NODE_ENV=development
```

### 4. Database Setup

Create the MySQL database:

```sql
CREATE DATABASE prpo_system;
USE prpo_system;

-- Create entries table
CREATE TABLE entries (
  id INT AUTO_INCREMENT PRIMARY KEY,
  pr_number VARCHAR(50),
  office_section VARCHAR(100),
  date VARCHAR(20),
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
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Create users table for authentication
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  email VARCHAR(100),
  role VARCHAR(20) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert default user (remember to hash password in production)
INSERT INTO users (username, password, email, role) VALUES 
('mimaropa@01', 'mimaropa@01', 'admin@ched.gov.ph', 'admin');
```

## Running the Application

### Development Mode

#### Frontend Only (with mock API):
```bash
npm run dev
```
The app will be available at `http://localhost:5173`

#### With Backend (Full Stack):

**Terminal 1 - Frontend:**
```bash
npm run dev
```

**Terminal 2 - Backend:**
```bash
npm run backend
# or if you have a separate backend project
cd backend
npm install
npm start
```

### Production Build

#### Build Frontend:
```bash
npm run build
```

The optimized files will be in the `dist/` directory.

#### Preview Build Locally:
```bash
npm run preview
```

## Project Structure

```
prpo_system/
├── src/
│   ├── main.jsx              # React entry point
│   ├── App.jsx               # Main app component with routing
│   ├── assets/               # Images and static assets
│   ├── components/
│   │   ├── DataForm.jsx      # Main purchase request form
│   │   ├── ItemDetails/      # Line items management
│   │   ├── Login/            # Authentication component
│   │   ├── Home.jsx          # Home/dashboard page
│   │   ├── SavedEntries.jsx  # View saved entries
│   │   ├── ErrorBoundary.jsx # Global error handling
│   │   └── NavBar/           # Navigation component
│   └── components/*.css      # Component stylesheets
├── public/                   # Static files
├── index.html                # HTML template
├── vite.config.js            # Vite configuration
├── eslint.config.js          # ESLint rules
├── package.json              # Dependencies and scripts
└── .env                       # Environment variables (create this)
```

## Available Scripts

```bash
# Start development server with HMR
npm run dev

# Build for production
npm run build

# Preview production build locally
npm run preview

# Run ESLint checks
npm run lint

# Format code (recommended to add: prettier)
npm run format
```

## API Endpoints

The application expects the following backend endpoints:

### Authentication
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout

### Entries
- `GET /api/entries` - Get all entries
- `POST /api/entries` - Create new entry
- `GET /api/entries/:id` - Get specific entry
- `PUT /api/entries/:id` - Update entry
- `DELETE /api/entries/:id` - Delete entry

## Usage

### 1. Login
- Username: `mimaropa@01`
- Password: `mimaropa@01`
- Note: Change these credentials in production

### 2. Create Purchase Request
1. Navigate to "Generate PR"
2. Fill in primary information (PR Number, Office, Date, etc.)
3. Add item details (Stock No., Unit, Description, Quantity, Unit Cost)
4. Click "Add Item" to add more items
5. Select export format (PDF or Excel)
6. Click "Generate PR" to export

### 3. View Saved Entries
- Navigate to "Entries" to view all saved purchase requests
- Entries are saved to the database and can be retrieved later

## Security Considerations

⚠️ **Important for Production:**

1. **Change Default Credentials**: Update the login username/password in the database
2. **Use Environment Variables**: Store sensitive data in `.env` file (never commit it)
3. **Add Password Hashing**: Use bcrypt to hash passwords
4. **Enable HTTPS**: Use SSL certificates in production
5. **Add CSRF Protection**: Implement CSRF tokens
6. **Validate Input**: Add server-side validation for all inputs
7. **Database Security**: 
   - Use strong database passwords
   - Limit database user permissions
   - Use connection pooling
8. **Session Management**: Implement proper session timeouts
9. **Rate Limiting**: Add rate limiting to prevent brute force attacks
10. **CORS Configuration**: Restrict CORS to trusted domains

## Known Issues & TODOs

- [ ] Implement proper user authentication system
- [ ] Add password hashing and security
- [ ] Create backend API routes
- [ ] Add database connection pool management
- [ ] Implement unit and integration tests
- [ ] Add loading states and better error messages
- [ ] Implement Excel export functionality
- [ ] Add data validation on form submission
- [ ] Create admin dashboard for managing entries
- [ ] Add email notifications

## Deployment

### Deploy to Vercel (Frontend)
```bash
npm install -g vercel
vercel
```

### Deploy to Heroku (Backend)
```bash
heroku create your-app-name
git push heroku main
```

### Deploy to AWS, Google Cloud, or Azure
See respective cloud provider documentation.

## Contributing

1. Create a feature branch (`git checkout -b feature/AmazingFeature`)
2. Commit your changes (`git commit -m 'Add AmazingFeature'`)
3. Push to the branch (`git push origin feature/AmazingFeature`)
4. Open a Pull Request

## Troubleshooting

### Port Already in Use
```bash
# Windows
netstat -ano | findstr :3001
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :3001
kill -9 <PID>
```

### Database Connection Issues
- Verify MySQL is running
- Check database credentials in `.env`
- Ensure database tables exist

### CORS Errors
- Check backend CORS configuration
- Verify frontend API URL in `.env`

## License

ISC

## Support

For issues and questions, please open a GitHub issue at: https://github.com/lloydDevs/prpo_system/issues

## Contact

- CHED MIMAROPA Region
- Email: admin@ched.gov.ph

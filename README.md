# 🏛️ SIELANGMERAH - Sistem Evaluasi Kinerja Perwakilan Perdagangan Metode Jarak Jauh

Platform digital untuk **evaluasi kinerja perwakilan perdagangan Indonesia di luar negeri** menggunakan metode jarak jauh. Sistem ini dikembangkan oleh Kementerian Perdagangan Republik Indonesia dengan arsitektur full-stack modern yang terdiri dari backend API berbasis FastAPI dan frontend React dengan arsitektur monorepo yang scalable.

## 🏗️ Arsitektur Sistem

### Backend (FastAPI)
- **API Layer**: REST API dengan dokumentasi otomatis
- **Service Layer**: Business logic dan orchestration
- **Repository Layer**: Data access dan database operations
- **Auth Layer**: JWT authentication dengan role-based authorization
- **Database**: PostgreSQL dengan Redis untuk caching

### Frontend (React)
- **Main App**: React 19 dengan TypeScript dan Vite
- **State Management**: Redux Toolkit dengan persistence
- **UI Framework**: Tailwind CSS dengan Shadcn UI components
- **Monorepo**: Turborepo dengan shared packages

## 🚀 Quick Start

### Prerequisites

Pastikan sistem Anda memiliki:

- **Docker** & **Docker Compose** (Recommended)
- **Node.js** >= 20.0.0
- **pnpm** >= 9.15.4
- **Python** >= 3.11 (untuk development tanpa Docker)
- **PostgreSQL** >= 14 (jika tidak menggunakan Docker)

### Installation

1. **Clone repositories:**
```bash
# Clone main project
git clone <main-repo-url> sielangmerah
cd sielangmerah

# Clone backend
git clone https://github.com/DaffaJatmiko/backend-perwadag.git backend

# Clone frontend  
git clone https://github.com/Ahmdfdhilah/frontend_perwadag.git frontend
```

2. **Setup dengan Docker (Recommended):**
```bash
# Start backend services
cd backend
docker compose up -d postgres redis
docker compose up -d app
docker compose exec app alembic upgrade head

# Start frontend
cd ../frontend
pnpm install
pnpm dev
```

3. **Akses aplikasi:**
- **Frontend**: `http://localhost:3000` atau `http://localhost:5173`
- **Backend API**: `http://localhost:8000`
- **API Docs**: `http://localhost:8000/docs`

## 📜 Environment Configuration

### Backend Environment (.env)
```env
# Database
DATABASE_URL=postgresql://postgres:password@postgres:5432/perwadag
POSTGRES_SERVER=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=perwadag

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# JWT Security
SECRET_KEY=your-super-secret-jwt-key-min-32-characters
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7

# CORS
CORS_ORIGINS=http://localhost:3000,http://localhost:5173

# Email (untuk password reset)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# File Upload
UPLOADS_PATH=static/uploads
MAX_FILE_SIZE=10485760  # 10MB

# Application
PROJECT_NAME=SIELANGMERAH - Sistem Evaluasi Kinerja Perwakilan Perdagangan Metode Jarak Jauh
VERSION=1.0.0
DEBUG=true
```

### Frontend Environment (.env.local)
```env
VITE_API_BASE_URL=http://localhost:8000/api/v1
VITE_APP_NAME=SIELANGMERAH
VITE_APP_VERSION=1.0.0
```

## 💻 Development

### Backend Development

#### Dengan Docker:
```bash
cd backend

# Start dependencies
docker compose up -d postgres redis

# Start aplikasi dengan hot reload
docker compose up -d app

# Database migration
docker compose exec app alembic upgrade head

# View logs
docker compose logs -f app

# Execute commands
docker compose exec app python scripts/seed_data.py
```

#### Tanpa Docker:
```bash
cd backend

# Setup virtual environment
python -m venv venv
source venv/bin/activate  # Linux/macOS
# atau venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Setup database lokal
createdb sielangmerah

# Update .env untuk local
# POSTGRES_SERVER=localhost
# REDIS_HOST=localhost

# Run migration
alembic upgrade head

# Start application
python main.py
```

### Frontend Development

```bash
cd frontend

# Install dependencies (WAJIB menggunakan pnpm)
pnpm install

# Start development server
pnpm dev

# Start aplikasi tertentu
pnpm --filter vite-react-app dev

# Build untuk production
pnpm build

# Lint dan format code
pnpm lint
pnpm format

# Type checking
pnpm check-types
```

## 🚀 Production Deployment

### Server Prerequisites

Deployment dilakukan di server dengan struktur:
```
/var/www/
├── sielangmerah/
│   ├── backend/
│   ├── frontend/
│   └── nginx/
```

### 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker & Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install Node.js & pnpm
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g pnpm

# Install Nginx
sudo apt install nginx -y
```

### 2. Deploy Backend

```bash
# Clone dan setup backend
cd /var/www/sielangmerah
git clone https://github.com/DaffaJatmiko/backend-perwadag.git backend
cd backend

# Setup production environment
cp .env.example .env
# Edit .env dengan konfigurasi production

# Start services
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Run migration
docker compose exec app alembic upgrade head
```

#### Backend Production Docker Compose (docker-compose.prod.yml)
```yaml
version: '3.8'
services:
  app:
    command: ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
    environment:
      - DEBUG=false
    volumes:
      - ./static/uploads:/app/static/uploads
      - ./logs:/app/logs
    restart: unless-stopped

  postgres:
    restart: unless-stopped
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    restart: unless-stopped
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 3. Deploy Frontend

```bash
# Clone dan setup frontend
cd /var/www/sielangmerah
git clone https://github.com/Ahmdfdhilah/frontend_perwadag.git frontend
cd frontend

# Setup production environment
cp apps/vite-react-app/.env.example apps/vite-react-app/.env.local
# Edit dengan konfigurasi production:
# VITE_API_BASE_URL=https://yourdomain.com/api/v1

# Install dependencies dan build
pnpm install
pnpm build

# Build output akan ada di apps/vite-react-app/dist/
```

### 4. Nginx Configuration

#### /etc/nginx/sites-available/sielangmerah
```nginx
server {
    listen 80;
    server_name sielangmerah.com www.sielangmerah.com;

    # Frontend static files
    location / {
        root /var/www/sielangmerah/frontend/apps/vite-react-app/dist;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Backend API
    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Upload file size limit
        client_max_body_size 10M;
    }

    # Backend docs
    location /docs {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend uploaded files
    location /static/ {
        alias /var/www/sielangmerah/backend/static/;
        expires 30d;
        add_header Cache-Control "public";
    }
}
```

#### Enable site dan restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/sielangmerah /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 5. SSL Configuration (Recommended)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get SSL certificate
sudo certbot --nginx -d sielangmerah.com -d www.sielangmerah.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### 6. Process Management

#### Backend dengan Systemd (Alternative)
```bash
# Create service file
sudo nano /etc/systemd/system/perwadag-backend.service
```

```ini
[Unit]
Description=SIELANGMERAH Backend API
After=network.target

[Service]
Type=exec
User=www-data
WorkingDirectory=/var/www/sielangmerah/backend
Environment=PATH=/var/www/sielangmerah/backend/venv/bin
ExecStart=/var/www/sielangmerah/backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable sielangmerah-backend
sudo systemctl start sielangmerah-backend
```

## 📊 Monitoring & Maintenance

### Health Checks
```bash
# Backend health
curl http://localhost:8000/health

# Frontend check
curl http://localhost/

# Database check
docker compose exec postgres pg_isready -U postgres
```

### Logs
```bash
# Backend logs
docker compose logs -f app

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# System logs
journalctl -u sielangmerah-backend -f
```

### Backup

#### Database Backup:
```bash
# Create backup
docker compose exec postgres pg_dump -U postgres perwadag > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore backup
docker compose exec -T postgres psql -U postgres perwadag < backup_file.sql
```

#### Files Backup:
```bash
# Backup uploaded files
tar -czf uploads_backup_$(date +%Y%m%d_%H%M%S).tar.gz backend/static/uploads/
```

### Updates

#### Backend Update:
```bash
cd /var/www/sielangmerah/backend
git pull origin main
docker compose build --no-cache app
docker compose up -d app
docker compose exec app alembic upgrade head
```

#### Frontend Update:
```bash
cd /var/www/sielangmerah/frontend
git pull origin main
pnpm install
pnpm build
sudo systemctl reload nginx
```

## 🛠️ Tech Stack

### Backend
- **FastAPI** - Modern Python web framework
- **PostgreSQL** - Primary database
- **Redis** - Caching dan session storage
- **SQLAlchemy** - ORM dengan Alembic migrations
- **JWT** - Authentication dengan bcrypt hashing
- **Docker** - Containerization

### Frontend
- **React 19** - Modern React dengan concurrent features
- **TypeScript** - Type-safe JavaScript
- **Vite** - Fast build tool dan dev server
- **Redux Toolkit** - State management
- **Tailwind CSS** - Utility-first CSS framework
- **Radix UI** - Accessible UI primitives
- **PWA** - Progressive Web App capabilities

### Infrastructure
- **Nginx** - Reverse proxy dan static file serving
- **Docker Compose** - Multi-container orchestration
- **SSL/TLS** - HTTPS encryption
- **Systemd** - Process management

## 🔒 Security

- **JWT Authentication** - Secure token-based auth
- **Password Hashing** - bcrypt dengan salt
- **CORS Protection** - Configured allowed origins
- **Input Validation** - Pydantic schemas dan Zod
- **SQL Injection Protection** - SQLAlchemy ORM
- **Rate Limiting** - API request throttling
- **File Upload Security** - Type dan size validation
- **HTTPS** - SSL/TLS encryption

## 📄 Default Users

Setelah database migration, sistem akan memiliki user default:

**Password untuk semua user**: `@Kemendag123`

1. **Administrator**
   - Username: `administrator`
   - Email: `projectkemendag@gmail.com`
   - Role: `ADMIN`

2. **Inspektorat Users** (12 users)
   - Format: `{nama}_ir{nomor}`
   - Role: `INSPEKTORAT`

3. **Perwadag Users** (54 users)
   - Format: `{prefix}_{location}`
   - Role: `PERWADAG`

## 🆘 Troubleshooting

### Common Issues

#### Backend tidak start:
```bash
# Check logs
docker compose logs app

# Check database connection
docker compose exec app python -c "from src.core.database import engine; print(engine.execute('SELECT 1'))"
```

#### Frontend build error:
```bash
# Clear cache
pnpm clean
rm -rf node_modules
pnpm install

# Check TypeScript
pnpm check-types
```

#### Database connection error:
```bash
# Check PostgreSQL
docker compose ps postgres
docker compose exec postgres pg_isready -U postgres

# Reset database
docker compose down -v
docker compose up -d postgres redis
docker compose exec app alembic upgrade head
```

#### Nginx permission error:
```bash
# Fix permissions
sudo chown -R www-data:www-data /var/www/sielangmerah/
sudo chmod -R 755 /var/www/sielangmerah/
```

## 📚 Documentation

- **Backend API**: `http://localhost:8000/docs`
- **Backend README**: [backend/README.md](./backend/README.md)
- **Frontend README**: [frontend/README.md](./frontend/README.md)
- **API Documentation**: [backend/doc-api.md](./backend/doc-api.md)

## 🤝 Contributing

1. Fork repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add some amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## 📄 License

Proyek ini dilisensikan di bawah MIT License.

## 🆘 Support

Untuk bantuan dan pertanyaan:

1. Periksa dokumentasi di atas
2. Cek [backend issues](https://github.com/DaffaJatmiko/backend-perwadag/issues)
3. Cek [frontend issues](https://github.com/Ahmdfdhilah/frontend_perwadag/issues)
4. Buat issue baru jika diperlukan

---

**SIELANGMERAH - Sistem Evaluasi Kinerja Perwakilan Perdagangan Metode Jarak Jauh**  
Dibangun dengan ❤️ oleh Kementerian Perdagangan Republik Indonesia
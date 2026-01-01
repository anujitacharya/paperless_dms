# 📄 Paperless-ngx Docker Setup

> **A modern, self-hosted document management system** - Scan, organize, and digitize your documents with ease.

<div align="center">

![Paperless-ngx](https://img.shields.io/badge/Paperless--ngx-Latest-blue?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)
![License](https://img.shields.io/badge/License-GPL--3.0-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success?style=flat-square)

[Features](#features) • [Quick Start](#quick-start) • [Configuration](#configuration) • [Architecture](#architecture) • [Troubleshooting](#troubleshooting)

</div>

---

## 🎯 Overview

This repository contains a **production-ready Docker Compose setup** for Paperless-ngx, a powerful document management system that helps you go paperless. With optical character recognition (OCR), full-text search, and intelligent document classification, Paperless-ngx transforms how you manage documents.

**Key Benefits:**
- 🔒 **Self-Hosted**: Keep your documents on your own infrastructure
- 🚀 **Fast**: Lightning-quick search with PostgreSQL + Redis
- 📱 **Responsive**: Works seamlessly across desktop and mobile devices
- 🔍 **Smart OCR**: Automatic text recognition in multiple languages
- 🏷️ **Auto-Tagging**: Intelligent document classification and organization
- 🔐 **Secure**: Complete control over your sensitive documents

---

## ✨ Features

### Core Functionality
- **Document Management**: Upload, organize, and retrieve documents effortlessly
- **Full-Text Search**: Instantly find any document using powerful search capabilities
- **OCR Technology**: Convert scanned documents to searchable text
- **Multi-Language Support**: OCR in English and other languages (configurable)
- **Document Versioning**: Track changes and maintain document history
- **Bulk Operations**: Process multiple documents simultaneously

### Advanced Capabilities
- **Automatic Matching**: Smart algorithms for document classification
- **Custom Workflows**: Automate document processing pipelines
- **REST API**: Full API access for third-party integrations
- **Bulk Importer**: Import documents from external sources
- **Email Integration**: Process emails and attachments
- **Web Interface**: Beautiful, intuitive user interface

### Technical Features
- **PostgreSQL Database**: Reliable, scalable data persistence
- **Redis Cache**: High-performance caching layer
- **Docker Containerization**: Isolated, reproducible environments
- **Multi-Architecture Support**: amd64, ARM, and ARM64 hardware
- **Automatic Restart**: Resilient deployment with restart policies
- **Volume Management**: Persistent data storage with Docker volumes

---

## 🚀 Quick Start

### Prerequisites

- **Docker**: Version 20.10 or later
- **Docker Compose**: Version 1.29 or later
- **Disk Space**: Minimum 10GB for initial setup (scales with documents)
- **Memory**: Minimum 2GB RAM recommended

### Installation

1. **Clone or navigate to the project directory:**
   ```bash
   cd /path/to/paperless-ngx
   ```

2. **Review and customize environment variables:**
   ```bash
   # Edit the docker-compose.env file with your preferences
   nano docker-compose.env
   ```

3. **Pull the latest Docker images:**
   ```bash
   docker compose pull
   ```

4. **Start the services:**
   ```bash
   docker compose up -d
   ```

5. **Verify the setup:**
   ```bash
   docker compose ps
   ```

6. **Access Paperless-ngx:**
   - Open your browser to: **`http://localhost:8000`**
   - Default credentials are created on first run

### First Steps

After starting Paperless-ngx:

1. **Create an admin account** (if not auto-created)
2. **Configure OCR settings** in the admin panel
3. **Set up document matching rules** for auto-tagging
4. **Start importing documents** via:
   - Web upload interface
   - `./consume` folder (auto-import)
   - API integration

---

## ⚙️ Configuration

### Environment Variables

The `docker-compose.env` file controls Paperless-ngx configuration:

```env
# User mapping (match your system user)
USERMAP_UID=503
USERMAP_GID=20

# Timezone configuration
PAPERLESS_TIME_ZONE=Asia/Kolkata

# OCR Language (eng, deu, fra, etc.)
PAPERLESS_OCR_LANGUAGE=eng

# Security key (change this to a random value)
PAPERLESS_SECRET_KEY='YourSecureKeyHere'
```

### Common Configuration Options

#### OCR Settings
```env
# Multiple languages
PAPERLESS_OCR_LANGUAGE=eng+deu+fra

# OCR mode
PAPERLESS_OCR_MODE=racy  # fast, racy, or skip
```

#### Database
```env
# Credentials (pre-configured in compose file)
POSTGRES_DB=paperless
POSTGRES_USER=paperless
POSTGRES_PASSWORD=paperless  # Change this!
```

#### Document Storage
```env
# Import folder location
# Maps to ./consume directory

# Export folder location
# Maps to ./export directory

# Media storage (Docker volume)
# For thumbnails and processed documents
```

#### Security
```env
# Change the secret key to a strong random value
PAPERLESS_SECRET_KEY='GenerateASecureKeyHere'

# Optional: Set admin username
PAPERLESS_ADMIN_USER=admin

# Optional: Set admin password
PAPERLESS_ADMIN_PASSWORD=changeme
```

### Advanced Configuration

For complete configuration options, refer to the [Paperless-ngx Documentation](https://docs.paperless-ngx.com/).

---

## 🏗️ Architecture

### Service Components

```
┌─────────────────────────────────────────────────────────┐
│                    PAPERLESS-NGX                        │
│                   (Docker Compose)                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Webserver   │  │      DB      │  │    Broker    │ │
│  │   (Port      │  │  PostgreSQL  │  │     Redis    │ │
│  │    8000)     │  │   (Port      │  │   (Port      │ │
│  │              │  │    5432)     │  │    6379)     │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│        │                  │                   │        │
│        └──────────────────┼───────────────────┘        │
│                           │                            │
└─────────────────────────────────────────────────────────┘
         │                  │                   │
         ▼                  ▼                   ▼
    ┌─────────┐        ┌─────────┐        ┌─────────┐
    │ Volumes │        │ Database│        │  Cache  │
    │ (data,  │        │ Storage │        │ Storage │
    │ media)  │        │ (pgdata)│        │(redisdb)│
    └─────────┘        └─────────┘        └─────────┘

    ┌───────────┐              ┌──────────────┐
    │  Consume  │              │   Export     │
    │  Folder   │              │   Folder     │
    │(Auto-     │              │ (Download &  │
    │ Import)   │              │  Archive)    │
    └───────────┘              └──────────────┘
```

### Service Details

| Service | Image | Port | Purpose |
|---------|-------|------|---------|
| **webserver** | `ghcr.io/paperless-ngx/paperless-ngx:latest` | 8000 | Main application with web UI |
| **db** | `docker.io/library/postgres:18` | 5432 | Document metadata database |
| **broker** | `docker.io/library/redis:8` | 6379 | Task queue and caching |

### Data Persistence

- **`data/`**: Paperless application data and configuration
- **`media/`**: Processed documents, thumbnails, and previews
- **`pgdata/`**: PostgreSQL database files
- **`redisdata/`**: Redis cache persistence
- **`./consume/`**: Auto-import documents from this folder
- **`./export/`**: Export documents to this folder

---

## 📁 Directory Structure

```
paperless-ngx/
├── README.md                    # This file
├── docker-compose.yml           # Docker Compose configuration
├── docker-compose.env           # Environment variables
├── .env                         # Compose project settings
├── consume/                     # Auto-import folder
│   └── (place documents here)
└── export/                      # Export/download folder
    └── (exported documents)
```

---

## 🔧 Common Tasks

### Starting/Stopping Services

```bash
# Start all services
docker compose up -d

# Stop services (preserves data)
docker compose down

# Stop services and remove volumes (warning: deletes data!)
docker compose down -v

# View logs
docker compose logs -f webserver

# View specific service logs
docker compose logs -f db
```

### Managing Documents

```bash
# Place documents in consume folder for automatic import
cp /path/to/document.pdf ./consume/

# Export documents (via web UI or API)
# Exported files appear in ./export/
```

### Database Operations

```bash
# Access PostgreSQL shell
docker compose exec db psql -U paperless -d paperless

# Backup database
docker compose exec db pg_dump -U paperless paperless > backup.sql

# Restore database
docker compose exec -T db psql -U paperless paperless < backup.sql
```

### Maintenance

```bash
# Update to latest version
docker compose pull
docker compose up -d

# Clean up unused images
docker image prune

# View resource usage
docker stats

# Restart specific service
docker compose restart webserver
```

---

## 📊 Performance Optimization

### Redis Cache
- Automatically handles caching and task queue
- Improves search performance significantly
- Helps with OCR processing speed

### PostgreSQL Database
- Superior performance compared to SQLite
- Scales well with large document collections
- Supports concurrent access

### Recommended System Requirements

| Metric | Minimum | Recommended | Large Scale |
|--------|---------|-------------|------------|
| CPU | 2 cores | 4 cores | 8+ cores |
| RAM | 2GB | 4GB | 8GB+ |
| Storage | 10GB | 50GB+ | 500GB+ |
| Network | 100Mbps | 1Gbps | 10Gbps |

---

## 🔒 Security Best Practices

### Essential Setup

1. **Change Default Credentials:**
   ```env
   PAPERLESS_SECRET_KEY='GenerateAStrongRandomKeyHere'
   POSTGRES_PASSWORD='ChangeToStrongPassword'
   ```

2. **Restrict Network Access:**
   ```yaml
   # In docker-compose.yml, use:
   ports:
     - "127.0.0.1:8000:8000"  # Local access only
   ```

3. **Use HTTPS:**
   - Deploy behind nginx/Traefik with SSL certificates
   - Enable TLS for external access

4. **Regular Backups:**
   ```bash
   # Backup automation script
   docker compose exec -T db pg_dump -U paperless paperless > \
     backup-$(date +%Y%m%d-%H%M%S).sql
   ```

5. **Access Control:**
   - Disable public registration
   - Use strong admin passwords
   - Enable two-factor authentication if available

### Firewall Rules

```bash
# Allow only local access (recommended for home use)
sudo ufw allow from 127.0.0.1 to 127.0.0.1 port 8000

# Allow from specific IP (for remote access)
sudo ufw allow from 192.168.1.0/24 to any port 8000
```

---

## 🐛 Troubleshooting

### Common Issues

#### "Port 8000 already in use"
```bash
# Find process using port 8000
lsof -i :8000

# Or change the port in docker-compose.yml
ports:
  - "8001:8000"  # Use 8001 instead
```

#### "Permission denied" on consume folder
```bash
# Fix permissions
sudo chown -R 503:20 ./consume ./export

# Or adjust USERMAP_UID and USERMAP_GID in docker-compose.env
```

#### "Database connection failed"
```bash
# Check if db service is running
docker compose ps

# View database logs
docker compose logs db

# Restart services
docker compose restart
```

#### "Out of disk space"
```bash
# Check Docker disk usage
docker system df

# Clean up unused images and containers
docker system prune

# View volume sizes
docker volume ls -q | xargs docker volume inspect | grep -A 5 Mountpoint
```

#### "OCR not working"
```bash
# Verify OCR language is installed
docker compose logs webserver | grep -i ocr

# Check OCR configuration
docker compose exec webserver cat /etc/paperless.conf | grep OCR
```

### Debug Mode

```bash
# Enable debug logging
PAPERLESS_DEBUG=true docker compose up -d

# View comprehensive logs
docker compose logs -f --tail=100
```

### Getting Help

- **Official Documentation**: https://docs.paperless-ngx.com/
- **GitHub Issues**: https://github.com/paperless-ngx/paperless-ngx/issues
- **Community Forums**: https://github.com/paperless-ngx/paperless-ngx/discussions

---

## 📚 Additional Resources

### Documentation
- [Paperless-ngx Official Docs](https://docs.paperless-ngx.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Redis Documentation](https://redis.io/documentation)

### Related Projects
- [Paperless](https://github.com/the-paperless-project/paperless) - Original project
- [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) - Active fork

### Helpful Tools
- **DockerHub**: https://hub.docker.com/
- **Container Registry**: https://ghcr.io/

---

## 🤝 Contributing

Contributions to improve this Docker setup are welcome! Please:

1. Test changes thoroughly
2. Document any new configuration options
3. Update this README with improvements
4. Follow Docker best practices

---

## 📝 License

This Docker Compose setup is provided as-is. Paperless-ngx is licensed under the [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.html).

---

## 🙋 Support

- **Found a bug?** Open an issue on GitHub
- **Have a suggestion?** Start a discussion
- **Need help?** Check the troubleshooting section or official docs

---

## 📈 Roadmap

### Planned Improvements
- [ ] Add Traefik reverse proxy configuration
- [ ] Implement automated backup scripts
- [ ] Add monitoring and alerting setup
- [ ] Include docker-compose production variant
- [ ] Add health check configurations
- [ ] Create management CLI utility

---

<div align="center">

### Made with ❤️ for Document Management

**[⬆ back to top](#-paperless-ngx-docker-setup)**

</div>

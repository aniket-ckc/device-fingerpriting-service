# Device Fingerprinting Service

A high-performance microservice for device identification and fingerprinting using browser characteristics and signal analysis.

## Overview

This service collects and analyzes device signals (browser capabilities, hardware info, screen characteristics, etc.) to create unique device fingerprints and identify recurring devices with similarity scoring.

### Key Features

- **Fingerprint Generation**: Collect and normalize browser signals into device fingerprints
- **Device Identification**: Create and track device identities with observation history
- **Similarity Matching**: Compare fingerprints using weighted similarity algorithm
- **Device Tracking**: Maintain observation count, first/last seen timestamps, and risk scores
- **Privacy First**: Minimal signal collection, data normalization, and hashing support
- **Async/Await**: Full async support with SQLAlchemy for high throughput
- **Clean Architecture**: Layered design with clear separation of concerns

## Technology Stack

- **Framework**: FastAPI + Uvicorn
- **Database**: SQLite (development) / PostgreSQL (production)
- **ORM**: SQLAlchemy 2.0 with async support
- **Validation**: Pydantic v2
- **Migrations**: Alembic
- **Container**: Docker & Docker Compose
- **Testing**: pytest with async support

## Project Structure

```
device-fingerprinting/
├── app/
│   ├── api/v1/              # API layer - HTTP handlers
│   ├── application/         # Application layer - use cases
│   ├── domain/              # Domain layer - business logic
│   │   ├── entities/        # Fingerprint, Device, Observation
│   │   ├── services/        # FingerprintGenerator, Matcher
│   │   ├── repositories/    # Abstract interfaces
│   │   └── value_objects/   # Domain types
│   ├── infrastructure/      # Infrastructure layer
│   │   ├── database/        # SQLAlchemy models & repositories
│   │   ├── hashing/         # SHA256 utilities
│   │   └── schemas/         # Pydantic request/response schemas
│   ├── config/              # Settings & logging
│   └── main.py              # FastAPI app factory
├── tests/                   # Unit, integration, API tests
├── alembic/                 # Database migrations
├── pyproject.toml           # Project metadata & dependencies
├── requirements.txt         # Pinned dependencies
├── docker-compose.yml       # Local PostgreSQL setup
└── Dockerfile               # Production image
```

## Getting Started

### Prerequisites

- Python 3.10+
- PostgreSQL (for production)
- Docker & Docker Compose (optional)

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd device-fingerprinting
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Create `.env` file:
```bash
cp .env.example .env
```

### Running Locally

```bash
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Access API documentation at `http://localhost:8000/docs`

### Running with Docker

```bash
docker-compose up
```

Database will be available at `localhost:5432`

## API Endpoints

### Create/Identify Fingerprint
```
POST /api/v1/fingerprints

{
  "signals": {
    "userAgent": "...",
    "platform": "Win32",
    "language": "en-US",
    "timezone": "Asia/Kolkata",
    "screen": {
      "width": 1920,
      "height": 1080,
      "pixelRatio": 1
    },
    "hardware": {
      "cpuCores": 8,
      "memory": 16
    },
    "webgl": "...",
    "canvas": "..."
  }
}

Response:
{
  "fingerprint_id": "fp_8f3a91c2",
  "device_id": "device_123",
  "confidence": 0.94,
  "is_new": false,
  "similarity_score": 0.92
}
```

### Compare Fingerprints
```
POST /api/v1/fingerprints/compare?fingerprint_a=...&fingerprint_b=...

Response:
{
  "similarity_score": 0.91,
  "matched": true
}
```

### Get Device Information
```
GET /api/v1/devices/{device_id}

Response:
{
  "id": "device_123",
  "first_seen": "2024-01-15T10:30:00",
  "last_seen": "2024-01-20T14:45:00",
  "observation_count": 42,
  "risk_score": 0.15,
  "is_flagged": false
}
```

### Get Device History
```
GET /api/v1/devices/{device_id}/history

Response:
{
  "device_id": "device_123",
  "first_seen": "2024-01-15T10:30:00",
  "last_seen": "2024-01-20T14:45:00",
  "observations": 42
}
```

### Health Check
```
GET /api/v1/health

Response:
{
  "status": "healthy"
}
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `sqlite:///./fingerprint.db` | Development database |
| `DATABASE_URL_PROD` | `postgresql://user:password@localhost/fingerprint` | Production database |
| `ENVIRONMENT` | `development` | Environment mode |
| `LOG_LEVEL` | `INFO` | Logging level |

## Architecture

### Clean Architecture Layers

1. **API Layer**: HTTP request/response handling via FastAPI
2. **Application Layer**: Use cases and business workflows
3. **Domain Layer**: Core business logic, entities, and rules
4. **Infrastructure Layer**: Database, persistence, and utilities

### Design Patterns

- **Repository Pattern**: Abstract data access behind interfaces
- **Dependency Injection**: FastAPI dependencies for service composition
- **Domain-Driven Design**: Entities and value objects reflect business concepts
- **Weighted Similarity**: Multi-factor fingerprint comparison algorithm

## Testing

```bash
# Run all tests
pytest

# With coverage
pytest --cov=app

# Specific test file
pytest tests/unit/test_fingerprint_service.py
```

## Database Migrations

```bash
# Create migration
alembic revision --autogenerate -m "Add column"

# Apply migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```

## Performance Considerations

- Async/await for high concurrency
- Database indexing on frequently queried fields
- SHA256 hashing for fingerprint deduplication
- Optional Redis caching layer for similarity scores
- Connection pooling with SQLAlchemy

## Security

- Input validation via Pydantic
- SQL injection prevention via SQLAlchemy ORM
- CORS middleware configuration
- Environment-based secrets management
- Rate limiting (recommended for production)

## Privacy & Compliance

- Minimal signal collection
- Data normalization before storage
- Optional hashing/pseudonymization
- Observation data retention policies
- GDPR-compliant deletion mechanisms

## Roadmap

- [ ] Redis caching layer
- [ ] WebSocket support for real-time updates
- [ ] Machine learning-based risk scoring
- [ ] Advanced analytics dashboard
- [ ] GraphQL API
- [ ] Multi-tenancy support

## Contributing

1. Create a feature branch
2. Commit changes
3. Push and create a pull request
4. Ensure tests pass and code is formatted

## License

MIT License - see LICENSE file for details

## Support

For issues, questions, or contributions, please open an issue on the repository.

# RSS Aggregator (rssagg)

A RESTful API service built in Go that allows users to manage and follow RSS feeds and automatically collects new posts from those feeds.

## Features

- User authentication using API keys
- Create and manage RSS feeds
- Follow/unfollow RSS feeds
- Automatic background scraping of RSS feeds at regular intervals
- Retrieve posts from followed feeds

## Tech Stack

- **Go** - Main programming language
- **Chi Router** - HTTP routing and middleware
- **PostgreSQL** - Database
- **sqlc** - SQL query generator for Go
- **Goose** - Database migration tool

## Project Structure

- `/internal/auth` - Authentication utilities
- `/internal/database` - Generated database code (via sqlc)
- `/sql/queries` - SQL queries for code generation
- `/sql/schema` - Database schema migrations

## API Endpoints

### User Management
- `POST /v1/users` - Create a new user
- `GET /v1/users` - Get authenticated user

### Feed Management
- `POST /v1/feeds` - Create a new feed
- `GET /v1/feeds` - List all feeds

### Feed Following
- `POST /v1/feed_follows` - Follow a feed
- `GET /v1/feed_follows` - List followed feeds
- `DELETE /v1/feed_follows/{feedFollowID}` - Unfollow a feed

### Posts
- `GET /v1/posts` - Get posts from followed feeds

### Health Check
- `GET /v1/healthz` - Health check endpoint
- `GET /v1/err` - Error check endpoint

## Setup and Installation

### Prerequisites
- Go 1.24+
- PostgreSQL
- [sqlc](https://github.com/sqlc-dev/sqlc)
- [Goose](https://github.com/pressly/goose)

### Environment Variables
Create a `.env` file with:
```
PORT=8080
DB_URL=postgresql://username:password@localhost:5432/rssagg?sslmode=disable
```

### Database Setup
1. Create a database named `rssagg`
2. Run migrations with goose:
```bash
goose -dir sql/schema postgres "postgresql://username:password@localhost:5432/rssagg?sslmode=disable" up
```

### Running the Service
```bash
go build && ./rssagg
```

## Authentication

The API uses a simple API key authentication mechanism. When creating a user, an API key is automatically generated.

To authenticate requests, include an `Authorization` header with:
```
Authorization: ApiKey YOUR_API_KEY
```

## Usage Example

1. **Create a user**
```bash
curl -X POST http://localhost:8080/v1/users -d '{"name":"John Doe"}'
```

2. **Create a feed (using the API key)**
```bash
curl -X POST http://localhost:8080/v1/feeds \
     -H "Authorization: ApiKey YOUR_API_KEY" \
     -d '{"name":"Example Blog","url":"https://example.com/rss"}'
```

3. **Follow a feed**
```bash
curl -X POST http://localhost:8080/v1/feed_follows \
     -H "Authorization: ApiKey YOUR_API_KEY" \
     -d '{"feed_id":"FEED_UUID"}'
```

4. **Get posts from followed feeds**
```bash
curl http://localhost:8080/v1/posts \
     -H "Authorization: ApiKey YOUR_API_KEY"
```

## How It Works

The application has a background scraper process that periodically:
1. Fetches feeds that need updating
2. Parses the RSS XML
3. Extracts post information 
4. Stores new posts in the database

When users request posts, they only see posts from feeds they've chosen to follow.
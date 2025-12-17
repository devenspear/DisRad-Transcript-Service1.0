# Disruption Radar Transcript Service

A residential YouTube transcript fetching service built with Node.js/TypeScript. Designed to run behind a Cloudflare Tunnel for secure, residential IP-based transcript retrieval.

## Features

- Fetch YouTube video transcripts on-demand
- Batch processing (up to 10 videos per request)
- In-memory caching with configurable TTL
- API key authentication
- Rate limiting (100 requests/minute per IP)
- Health monitoring endpoint
- Winston logging with file rotation

## Tech Stack

- **Runtime**: Node.js
- **Language**: TypeScript
- **Framework**: Express.js
- **Transcript Library**: youtube-transcript
- **Logging**: Winston

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd transcript-service

# Install dependencies
npm install

# Create environment file
cp .env.example .env
# Edit .env with your API key
```

## Configuration

Create a `.env` file with the following variables:

```env
PORT=3100
API_KEY=your-secure-api-key-here
RATE_LIMIT_WINDOW=60000
RATE_LIMIT_MAX=100
CACHE_TTL=3600000
```

## Running the Service

```bash
# Development mode (with hot reload)
npm run dev

# Production build
npm run build
npm start
```

## API Endpoints

### Health Check

```
GET /health
```

No authentication required. Returns service status and system metrics.

**Response:**
```json
{
  "status": "healthy",
  "uptime": 3600,
  "uptimeFormatted": "1h 0m 0s",
  "version": "1.0.0",
  "timestamp": "2025-01-15T12:00:00.000Z",
  "system": {
    "platform": "win32",
    "hostname": "DevOfficeMiniPC",
    "memory": {
      "total": "16384MB",
      "free": "8192MB",
      "usage": "50%"
    },
    "cpu": "Intel Core i7"
  }
}
```

### Fetch Single Transcript

```
GET /transcript?videoId=<video-id>&language=<optional-language>
```

**Headers:**
- `X-API-Key`: Your API key (required)

**Query Parameters:**
- `videoId` (required): YouTube video ID or full URL
- `language` (optional): Language code (e.g., `en`, `es`)

**Response:**
```json
{
  "success": true,
  "videoId": "dQw4w9WgXcQ",
  "transcript": "Full transcript text...",
  "segments": [
    {
      "text": "Segment text",
      "offset": 0,
      "duration": 5000
    }
  ],
  "cached": false
}
```

### Fetch Batch Transcripts

```
POST /transcript/batch
```

**Headers:**
- `X-API-Key`: Your API key (required)
- `Content-Type`: application/json

**Body:**
```json
{
  "videoIds": ["video1", "video2", "video3"],
  "language": "en"
}
```

**Response:**
```json
{
  "success": true,
  "results": [...],
  "summary": {
    "total": 3,
    "successful": 3,
    "failed": 0
  }
}
```

### Cache Management

```
GET /transcript/cache     # Get cache statistics
DELETE /transcript/cache  # Clear cache
```

**Headers:**
- `X-API-Key`: Your API key (required)

## Deployment

The service runs on **Windows Server** at `192.168.0.50:3100`.

### Running as Windows Service

The service is configured to run via Windows Task Scheduler:

```cmd
# Start the service
schtasks /run /tn "TranscriptService"

# Check status
schtasks /query /tn "TranscriptService"
```

---

## Monitoring with Uptime Kuma

The service is monitored using [Uptime Kuma](https://github.com/louislam/uptime-kuma), a self-hosted monitoring dashboard.

### Uptime Kuma Installation

Uptime Kuma is installed at `C:\Projects\uptime-kuma` on the Windows Server.

**Access Dashboard:**
```
http://192.168.0.50:3001
```

### Service Management

**Start Uptime Kuma:**
```cmd
schtasks /run /tn "UptimeKuma"
```

**Check Status:**
```cmd
schtasks /query /tn "UptimeKuma"
```

**Stop Uptime Kuma:**
```cmd
taskkill /f /fi "WINDOWTITLE eq UptimeKuma"
```

### Monitor Configuration

The transcript service is monitored with these settings:

| Setting | Value |
|---------|-------|
| Monitor Type | HTTP(s) |
| Friendly Name | Disruption Radar Transcript Service |
| URL | `http://localhost:3100/health` |
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Method | GET |

### Features

- **Real-time Status**: Shows UP/DOWN status with response times
- **Uptime History**: Tracks uptime percentage over time
- **Notifications**: Can alert via Discord, Slack, Email, Telegram, etc.
- **Status Pages**: Create public status pages to share

### Adding Notifications

1. Open Uptime Kuma dashboard
2. Go to **Settings > Notifications**
3. Click **Setup Notification**
4. Choose your notification service (Discord, Slack, Email, etc.)
5. Configure webhook URL or SMTP settings
6. Test and save

### Auto-Start on Boot

Both services are configured to auto-start via Windows Task Scheduler:
- `UptimeKuma` - Monitoring dashboard
- `TranscriptService` - YouTube transcript API

---

## Project Structure

```
transcript-service/
├── src/
│   ├── index.ts              # Express app entry point
│   ├── middleware/
│   │   ├── auth.ts           # API key authentication
│   │   ├── logging.ts        # Request/response logging
│   │   └── rateLimit.ts      # Rate limiting
│   ├── routes/
│   │   ├── health.ts         # Health check endpoint
│   │   └── transcript.ts     # Transcript API endpoints
│   ├── services/
│   │   └── youtube.ts        # YouTube transcript service
│   └── utils/
│       └── logger.ts         # Winston logger config
├── logs/                     # Log files (auto-created)
├── package.json
├── tsconfig.json
└── README.md
```

## License

MIT License - Deven Spear

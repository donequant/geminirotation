# Gemini API Key Rotation Proxy

A proxy server that rotates through multiple Gemini API keys automatically.

## Directory Structure

```
.
├── app/
│   ├── main.py          # Main application
│   └── requirements.txt  # Python dependencies
├── config_d/
│   ├── keys.txt         # Your API keys (one per line)
│   └── keys_failed.txt  # Auto-generated failed keys log
├── log_d/
│   └── proxy.log        # Application logs
├── Dockerfile
├── docker-compose.yml
└── docker-compose-nobuild.yml
```

## Setup

1. Add your Gemini API keys to `config_d/keys.txt` (one key per line):
   ```
   AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   AIzaSyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
   ```

2. Build and run:
   ```bash
   # First time - build the image
   docker compose build
   
   # Start the service
   docker compose up -d
   ```

3. After building once, you can use the no-build compose file:
   ```bash
   docker compose -f docker-compose-nobuild.yml up -d
   ```

## Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check (no API key rotation) |
| `/status` | GET | Service status |
| `/reload-keys` | POST | Reload keys from file |
| `/*` | ALL | Proxy to Gemini API with key rotation |

## Configuration (Environment Variables)

All configuration is in `docker-compose.yml`:

| Variable | Default | Description |
|----------|---------|-------------|
| `MAX_ROTATION_TIMES` | 7 | Max retry attempts with different keys |
| `MODEL` | gemini-2.5-flash | Default model name |
| `GEMINI_URL` | https://generativelanguage.googleapis.com | Gemini API base URL |
| `GEMINI_STREAMING` | false | Enable streaming by default |
| `GEMINI_RAW_FORMAT` | false | Return raw API response format |

## Usage

Point your application to `http://localhost:1237` instead of the Gemini API URL.

Example:
```bash
# Health check
curl http://localhost:1237/health

# Make API request (will rotate keys automatically)

curl -X POST "http://sj.eu.org:1237/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1237" \
  -d '{"contents":[{"parts":[{"text":"Hello"}]}]}'



```

## Logs

- Application logs: `./log_d/proxy.log`
- Failed keys log: `./config_d/keys_failed.txt`

## Commands

```bash
# View logs
docker logs -f gemini-rotation-proxy

# Restart
docker compose restart

# Stop
docker compose down

# Reload keys without restart
curl -X POST http://localhost:1237/reload-keys
```

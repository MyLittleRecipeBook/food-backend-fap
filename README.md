# YouTube Recipe Video Search API - Fast and Reliable Video Discovery with Redis Caching

This FastAPI application provides a specialized search interface for recipe videos on YouTube, intelligently categorizing content into long-form and short-form videos while implementing Redis Sentinel for robust caching and high availability.

The application serves as a middleware between clients and the YouTube Data API, offering optimized search functionality specifically for recipe-related content. It features intelligent video duration classification, Redis-based caching for improved performance, and high availability through Redis Sentinel implementation. The service is containerized for consistent deployment and integrated with AWS infrastructure for scalable production use.

## Repository Structure
```
.
├── app/                          # Application source code directory
│   ├── __init__.py              # Python package initializer
│   └── main.py                  # Core application logic and API endpoints
├── buildspec.yaml               # AWS CodeBuild configuration for CI/CD
├── Dockerfile                   # Container definition for application deployment
└── requirements.txt            # Python dependency specifications
```

## Usage Instructions
### Prerequisites
- Python 3.12 or higher
- Docker
- YouTube API key
- Redis Sentinel cluster
- AWS account (for deployment)

Required environment variables:
```bash
YOUTUBE_API_KEY=your_youtube_api_key
SENTINEL_HOST=your_sentinel_host
SENTINEL_PORT=your_sentinel_port
SENTINEL_MASTER_NAME=your_master_name
```

### Installation

#### Local Development
```bash
# Clone the repository
git clone <repository-url>
cd <repository-name>

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the application
uvicorn app.main:app --host 0.0.0.0 --port 5000
```

#### Docker Installation
```bash
# Build the Docker image
docker build -t youtube-recipe-search .

# Run the container
docker run -p 5000:5000 \
  -e YOUTUBE_API_KEY=your_key \
  -e SENTINEL_HOST=your_host \
  -e SENTINEL_PORT=your_port \
  -e SENTINEL_MASTER_NAME=your_master_name \
  youtube-recipe-search
```

### Quick Start
1. Start the application using either local development or Docker method
2. Access the API endpoints:

```python
import requests

# Get long-form recipe videos
response = requests.get("http://localhost:5000/api/video/long")
print(response.json())

# Get short-form recipe videos
response = requests.get("http://localhost:5000/api/video/short")
print(response.json())
```

### More Detailed Examples
```python
# Example response structure
{
    [
        {
            "video_id": "abc123xyz",
            "title": "Perfect Pasta Recipe"
        },
        {
            "video_id": "def456uvw",
            "title": "Quick Cooking Tips #shorts"
        }
    ]
}
```

### Troubleshooting

Common Issues:
1. YouTube API Rate Limiting
   - Error: `HTTPException: 429 Too Many Requests`
   - Solution: Implement exponential backoff or check API quota

2. Redis Connection Issues
   - Error: `Failed to get master/replica`
   - Check Redis Sentinel configuration
   - Verify network connectivity
   - Ensure correct environment variables

3. Cache Performance
   - Enable debug logging:
   ```python
   logging.basicConfig(level=logging.DEBUG)
   ```
   - Monitor Redis cache hits/misses in logs
   - Cache TTL is set to 3600 seconds (1 hour)

## Data Flow
The application processes video searches through a caching layer before accessing the YouTube API, optimizing response times and reducing API quota usage.

```ascii
Client Request → FastAPI → Redis Cache Check
                              ↙           ↘
                    [Cache Hit]         [Cache Miss]
                      ↙                      ↘
                Return Data             YouTube API
                                           ↓
                                    Process Results
                                           ↓
                                    Store in Cache
                                           ↓
                                    Return Results
```

Key Component Interactions:
1. Incoming requests are first checked against Redis cache
2. Cache hits return immediately with stored data
3. Cache misses trigger YouTube API requests
4. Video duration is calculated and categorized
5. Results are cached for future requests
6. Redis Sentinel ensures high availability
7. Docker containerization enables consistent deployment
8. AWS CodeBuild handles automated builds and deployments
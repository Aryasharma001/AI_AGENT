# AI Agent - Web Search and Data Extraction Tool 🔍

An intelligent agent that performs automated web searches and extracts structured information from datasets using LLM processing. This tool allows users to upload CSV files or connect Google Sheets, perform customized searches, and extract specific information using natural language prompts.

## 🌟 Features

- CSV and Google Sheets data import
- Custom search query templates with dynamic entity replacement
- Automated web searching with rate limiting and caching
- LLM-powered information extraction
- Results export to CSV or Google Sheets
- Comprehensive error handling and logging
- Search result caching for improved performance
- Progress tracking for long-running operations

## 🚀 Getting Started

### Prerequisites

- Python 3.8+
- pip (Python package manager)
- Google Cloud Console account (for Google Sheets integration)
- API keys for:
  - Groq/OpenAI (LLM processing)
  - SerpAPI/ScraperAPI (web searching)
  - Google Sheets API (optional)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/ai-agent.git
cd ai-agent
```

2. Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Set up environment variables:
```bash
cp .env.example .env
```

Edit `.env` file with your API keys:
```
GROQ_API_KEY=your_groq_api_key
SERPAPI_KEY=your_serpapi_key
GOOGLE_SHEETS_CREDENTIALS=path_to_credentials.json
CACHE_DIR=./cache
LOG_DIR=./logs
```

### Project Structure

```
ai-agent/
├── app.py                 
├── src/
│   ├── search/
│   │   ├── __init__.py
│   │   ├── search.py     
│   │   └── cache.py      
│   ├── llm/
│   │   ├── __init__.py
│   │   └── processor.py  
│   ├── data/
│   │   ├── __init__.py
│   │   ├── loader.py     
│   │   └── exporter.py  
│   └── utils/
│       ├── __init__.py
│       ├── logger.py     
│       └── rate_limit.py 
├── cache/                
├── logs/                 
├── tests/               
└── requirements.txt
```
## Demo Video

[Watch the demo on Loom](https://www.loom.com/share/3892d630cff44e1284a9181e8aa1a2c9?sid=39118729-7d47-4d05-a723-aa90885c3d32)


## ⚙️ Configuration

### Rate Limiting

The application implements rate limiting to prevent API abuse and ensure stable operation:

```python
# src/utils/rate_limit.py
class RateLimit:
    def __init__(self, max_requests: int, time_window: int):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = []
    
    def can_proceed(self) -> bool:
        current_time = time.time()
        # Remove old requests
        self.requests = [req for req in self.requests 
                        if current_time - req < self.time_window]
        
        if len(self.requests) < self.max_requests:
            self.requests.append(current_time)
            return True
        return False
```

Default rate limits:
- SerpAPI: 100 requests per minute
- Groq API: 60 requests per minute
- Google Sheets API: 60 requests per minute

### Search Cache

The search cache system prevents redundant API calls and improves performance:

```python
# src/search/cache.py
class SearchCache:
    def __init__(self, cache_dir: str, expiry_days: int = 7):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(exist_ok=True)
        self.expiry_days = expiry_days
    
    def get_cache_key(self, query: str) -> str:
        return hashlib.md5(query.encode()).hexdigest()
    
    def get_cached_result(self, query: str) -> Optional[dict]:
        cache_key = self.get_cache_key(query)
        cache_file = self.cache_dir / f"{cache_key}.json"
        
        if cache_file.exists():
            # Check expiry
            if (time.time() - cache_file.stat().st_mtime) > (self.expiry_days * 86400):
                cache_file.unlink()
                return None
                
            with cache_file.open('r') as f:
                return json.load(f)
        return None
```

### Logging

Comprehensive logging is implemented to track application behavior and debug issues:

```python
# src/utils/logger.py
def setup_logger(name: str, log_dir: str) -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)
    
    # Create log directory if it doesn't exist
    Path(log_dir).mkdir(exist_ok=True)
    
    # File handler for general logs
    fh = logging.FileHandler(f"{log_dir}/{name}.log")
    fh.setLevel(logging.INFO)
    
    # File handler for errors
    error_fh = logging.FileHandler(f"{log_dir}/{name}_error.log")
    error_fh.setLevel(logging.ERROR)
    
    # Console handler
    ch = logging.StreamHandler()
    ch.setLevel(logging.INFO)
    
    # Formatting
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    fh.setFormatter(formatter)
    error_fh.setFormatter(formatter)
    ch.setFormatter(formatter)
    
    logger.addHandler(fh)
    logger.addHandler(error_fh)
    logger.addHandler(ch)
    
    return logger
```

## 🎮 Usage

1. Start the application:
```bash
streamlit run app.py
```

2. Upload data:
   - Click "Upload CSV" or enter Google Sheet URL
   - Select the column containing entities to search

3. Configure search:
   - Enter your search prompt template (e.g., "Find the email address for {company}")
   - Adjust any advanced settings (batch size, retry attempts, etc.)

4. Start processing:
   - Click "Start Processing" to begin the search and extraction
   - Monitor progress in the dashboard
   - Download results or export to Google Sheets when complete

## 🔧 Advanced Features

### Multiple Field Extraction

Extract multiple pieces of information in a single query:

```python
prompt = """
Extract the following information for {company}:
- Email address
- Physical address
- Phone number
- Founded year

Format the output as JSON.
"""
```

### Google Sheets Integration

Enable two-way Google Sheets sync:

1. Set up credentials in Google Cloud Console
2. Enable Google Sheets API
3. Download credentials JSON
4. Set path in `.env` file
5. Use "Export to Google Sheets" feature

### Error Handling

The application implements comprehensive error handling:

- API failures: Automatic retry with exponential backoff
- Rate limiting: Smart queuing and batch processing
- Data validation: Input sanitization and format verification
- Cache management: Automatic cleanup of expired entries
- Connection issues: Graceful degradation and status updates

## 📝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 🤝 Acknowledgments

- [Streamlit](https://streamlit.io/) for the web interface
- [Groq](https://groq.com/) for LLM processing
- [SerpAPI](https://serpapi.com/) for web search capabilities

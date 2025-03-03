# Sheppard
Sheppard Agency is an advanced AI assistant application that combines memory persistence, web research capabilities, and conversational intelligence. It's designed to provide users with a more contextualized and knowledgeable AI experience by remembering conversation history and actively researching information when needed. ALL LOCAL HOSTED. 
# Sheppard Agency

## Overview

Sheppard Agency is an advanced AI assistant application that combines memory persistence, web research capabilities, and conversational intelligence. It's designed to provide users with a more contextualized and knowledgeable AI experience by remembering conversation history and actively researching information when needed.

## Key Features

### Memory System
- **Short-term Recall**: Remembers recent interactions within the current conversation
- **Long-term Memory**: Stores important details, preferences, and facts across multiple sessions
- **Context Awareness**: Connects related information between different conversations

### Research System
- **Web Search Integration**: Connects to search engines for real-time information
- **Source Verification**: Evaluates and cites sources for credibility
- **Information Synthesis**: Combines multiple sources to provide comprehensive answers
- **Autonomous Browser**: Self-healing browsing with intelligent navigation capabilities

### LLM System
- **Conversational Interface**: Natural dialogue with understanding of context and nuance
- **Task Execution**: Can perform various tasks based on user requests
- **Reasoning Capabilities**: Processes complex questions with step-by-step reasoning

## Installation

### Requirements
- Python 3.10 or higher
- Redis server (for memory storage)
- ChromaDB (for vector embeddings)
- Ollama installed locally (for language models)

### Setup

1. Clone the repository:
```bash
git clone https://github.com/yourusername/sheppard_agency.git
cd sheppard_agency
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows, use: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

The key dependencies include:
- aiohttp
- rich
- selenium
- beautifulsoup4
- backoff
- tldextract
- chromadb
- redis

If Firecrawl integration is desired:
```bash
pip install firecrawl-py
```

4. Set up environment variables by creating a `.env` file in the project root:

```
# Ollama Configuration
OLLAMA_MODEL=mannix/dolphin-2.9-llama3-8b:latest
OLLAMA_EMBED_MODEL=mxbai-embed-large
OLLAMA_API_BASE=http://localhost:11434/api

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379

# ChromaDB Configuration
CHROMADB_PERSIST_DIRECTORY=./data/chromadb
CHROMADB_DISTANCE_FUNC=cosine

# Research Configuration
FIRECRAWL_API_KEY=your_firecrawl_api_key  # Optional

# Application Directories
DATA_DIR=./data
LOG_DIR=./logs
SCREENSHOT_DIR=./screenshots
TEMP_DIR=./temp

# System Configuration
MAX_RETRIES=3
REQUEST_TIMEOUT=60
EMBEDDING_DIMENSION=384
GPU_ENABLED=false  # Set to true if GPU available
```

5. Prepare Local Ollama Models:

Ensure Ollama is running locally and pull the required models:

```bash
ollama pull mannix/dolphin-2.9-llama3-8b:latest
ollama pull mxbai-embed-large
ollama pull Azazel-AI/llama-3.2-1b-instruct-abliterated.q8_0
```

6. Set up directory structure:

```bash
mkdir -p data/chromadb logs screenshots temp research_results
```

7. Start Redis Server:

```bash
# Linux/macOS
redis-server

# Windows (after installing Redis)
redis-server.exe
```

## Running the Application

### Method 1: Direct Execution

Run the main application:

```bash
python main.py
```

### Method 2: Using the ChatApp Interface

```python
# Example script to run the chat application
import asyncio
from src.core.chat import ChatApp
from src.core.system import system_manager

async def main():
    # Initialize system components
    success, error = await system_manager.initialize()
    if not success:
        print(f"System initialization failed: {error}")
        return
        
    # Create chat app
    chat_app = ChatApp()
    
    # Initialize chat app with system components
    await chat_app.initialize(
        memory_system=system_manager.memory_manager,
        research_system=system_manager.research_system,
        llm_system=system_manager.ollama_client
    )
    
    # Example interaction
    async for response in chat_app.process_input("Tell me about quantum computing"):
        print(response.content)
        
    # Clean up resources
    await chat_app.close()
    await system_manager.cleanup()

# Run the main function
if __name__ == "__main__":
    asyncio.run(main())
```

### Method 3: Using Command Line Interface

```python
import asyncio
from rich.console import Console
from src.core.commands import CommandHandler
from src.core.system import system_manager

async def main():
    # Initialize console
    console = Console()
    
    # Initialize system
    success, error = await system_manager.initialize()
    if not success:
        console.print(f"System initialization failed: {error}", style="bold red")
        return
    
    # Create command handler
    command_handler = CommandHandler(console, system_manager)
    
    # Show welcome message
    command_handler.show_welcome()
    
    # Main interaction loop
    while True:
        # Display prompt
        console.print("[green]You:[/green] ", end="")
        user_input = input()
        
        # Check for exit command
        if user_input.lower() in ['/exit', '/quit']:
            break
            
        # Handle commands
        if user_input.startswith('/'):
            await command_handler.handle_command(user_input)
        else:
            # Process normal input through chat system
            responses = system_manager.ollama_client.chat(
                messages=[{"role": "user", "content": user_input}],
                stream=True
            )
            
            console.print("[blue]Assistant:[/blue] ", end="")
            async for response in responses:
                if hasattr(response, 'content') and response.content:
                    console.print(response.content, end="")
            console.print()
    
    # Clean up
    await system_manager.cleanup()
    console.print("Goodbye!", style="bold blue")

if __name__ == "__main__":
    asyncio.run(main())
```

## Usage

### Basic Interaction
Interact with Sheppard through the command line or web interface by simply typing your questions or instructions.

```
> Tell me about quantum computing
> What was the last topic we discussed?
> I prefer explanations with examples
> Research the latest advancements in renewable energy
```

### Commands

- `/research [topic] [--depth=<1-5>]` - Research a topic with specified depth
- `/memory [query]` - Search through memory for specific information
- `/status` - Show current system status and components
- `/clear [--confirm]` - Clear chat history
- `/settings [setting] [value]` - View or modify chat settings
- `/browse <url> [--headless]` - Open browser to research specific URL
- `/save [type] [filename]` - Save research or conversation
- `/preferences [action] [key] [value]` - View or update user preferences
- `/help [command]` - Display available commands
- `/exit` - Exit the application

## Using the Research System

The research system is a core feature of Sheppard Agency, providing autonomous web browsing and information extraction capabilities.

### Basic Research Command

```
/research quantum computing --depth=3
```

This will:
1. Search for information about quantum computing
2. Extract content from relevant websites
3. Analyze and summarize the findings
4. Display formatted results

### Using the ResearchSystem Programmatically

```python
import asyncio
from src.research.system import ResearchSystem
from src.research.models import ResearchType
from rich.console import Console

async def research_example():
    console = Console()
    
    # Initialize research system
    research_system = ResearchSystem()
    await research_system.initialize()
    
    try:
        # Perform research
        console.print("Researching quantum computing...", style="bold blue")
        results = await research_system.research_topic(
            topic="quantum computing",
            research_type=ResearchType.WEB_SEARCH,
            depth=3
        )
        
        # Display results
        console.print(f"Found {len(results.get('findings', []))} findings")
        for finding in results.get('findings', []):
            console.print(f"[bold green]Source:[/bold green] {finding.get('url')}")
            console.print(f"[bold green]Title:[/bold green] {finding.get('title')}")
            console.print(f"[bold green]Summary:[/bold green] {finding.get('summary')}")
            console.print("---")
    
    finally:
        # Clean up
        await research_system.cleanup()

if __name__ == "__main__":
    asyncio.run(research_example())
```

### Using Autonomous Browser Mode

The system includes an `AutonomousBrowser` class that provides enhanced browsing capabilities with self-healing and intelligent navigation:

```python
from src.research.browser_control import AutonomousBrowser

# Initialize the autonomous browser
browser = AutonomousBrowser(
    headless=True,
    timeout=30,
    max_pages=5,
    auto_recovery=True,
    intelligent_retries=True
)
await browser.initialize()

# Use the intelligent navigation method
results = await browser.navigate_intelligently(
    query="quantum computing",
    depth=3,
    source_types=["scholarly", "scientific", "technical"],
    progress_callback=lambda p: print(f"Progress: {p*100:.0f}%")
)

# Analyze extracted information
print(f"Pages visited: {results['metadata']['pages_visited']}")
print(f"Domains visited: {results['metadata']['domains_visited']}")
for content in results['content']:
    print(f"URL: {content['url']}")
    print(f"Word count: {len(content.get('markdown', '').split())}")
    print("---")

# Cleanup when done
await browser.cleanup()
```

The autonomous browser includes several advanced features:
- **Self-healing**: Automatically recovers from navigation errors
- **Intelligent retries**: Uses different strategies for different error types
- **Content filtering**: Identifies and filters low-quality content
- **Link prioritization**: Prioritizes links based on relevance to the topic
- **Rate limiting**: Applies dynamic rate limiting to avoid blocking

### Deep Analysis Mode

For more comprehensive research:

```python
# Create a task for deep analysis
task_id = await research_system.task_manager.create_task(
    description="quantum computing advancements",
    research_type=ResearchType.DEEP_ANALYSIS,
    depth=5,
    metadata={"require_citations": True}
)

# Process the task
await research_system.task_manager.process_task(task_id)

# Get the detailed results
deep_results = await research_system.perform_deep_research(
    task_id,
    validation_level=ValidationLevel.HIGH
)

# Display the analysis
print(deep_results['analysis']['text'])
```

The deep analysis mode provides:
- More thorough content extraction
- Higher validation standards for sources
- Comprehensive cross-source analysis
- Citation extraction and verification

### Firecrawl Integration

To enable Firecrawl integration for enhanced content extraction:

1. Set the `FIRECRAWL_API_KEY` environment variable with your API key
2. Create a FirecrawlConfig instance with your desired settings
3. Pass it to the ResearchSystem constructor

```python
from src.research.models import FirecrawlConfig
from src.research.system import ResearchSystem

# Create Firecrawl configuration
firecrawl_config = FirecrawlConfig(
    api_key="your_api_key_here",
    max_pages=10,
    formats=["markdown", "html"]
)

# Initialize research system with Firecrawl
research_system = ResearchSystem(
    memory_manager=memory_manager,
    ollama_client=ollama_client,
    config=ResearchConfig(firecrawl=firecrawl_config)
)
await research_system.initialize()

# Now research will use Firecrawl when available
results = await research_system.research_topic("quantum computing")
```

## Memory Management

### Memory Persistence
- **Embedding Generation**: Uses `mxbai-embed-large` via Ollama for fixed-size embeddings
- **Database Storage**: Stores embeddings in ChromaDB with optional PostgreSQL and Redis integrations

### Context Summarization
- **Short Context Summarization**: Uses `Azazel-AI/llama-3.2-1b-instruct-abliterated.q8_0` for quick summarization
- **Long Context Summarization**: Uses `krith/mistral-nemo-instruct-2407-abliterated:IQ3_M` for extended summaries

## Configuration Options

### Memory System Settings

| Setting | Description | Default | Type |
|---------|-------------|---------|------|
| `MEMORY_PROVIDER` | Backend storage provider | `redis` | string |
| `MEMORY_RETENTION_PERIOD` | Days to keep memories | `90` | integer |
| `MEMORY_IMPORTANCE_THRESHOLD` | Minimum importance score to store memory | `0.5` | float |
| `EMBEDDING_MODEL` | Model used for vectorizing text | `mxbai-embed-large` | string |
| `MEMORY_CHUNK_SIZE` | Maximum token size for memory chunks | `500` | integer |

### Research System Settings

| Setting | Description | Default | Type |
|---------|-------------|---------|------|
| `RESEARCH_PROVIDER` | Search API provider | `serpapi` | string |
| `RESEARCH_MAX_SOURCES` | Maximum sources per query | `5` | integer |
| `RESEARCH_MAX_DEPTH` | How deep to search (1-3) | `2` | integer |
| `RESEARCH_CACHE_HOURS` | Hours to cache research results | `24` | integer |
| `SOURCE_CITATION_STYLE` | Citation format for sources | `mla` | string |

### LLM System Settings

| Setting | Description | Default | Type |
|---------|-------------|---------|------|
| `LLM_PROVIDER` | LLM API provider | `ollama` | string |
| `LLM_MODEL` | Default language model | `mannix/dolphin-2.9-llama3-8b:latest` | string |
| `MAX_CONTEXT_TOKENS` | Maximum token context window | `8192` | integer |
| `TEMPERATURE` | Creativity of responses (0-2) | `0.7` | float |
| `SYSTEM_PROMPT` | Base system instructions | see `prompts.py` | string |

## Architecture

Sheppard follows a modular architecture with three main systems:

1. **Memory System**: 
   - `memory/manager.py` - Central memory coordination
   - `memory/models.py` - Data structures for memory
   - `memory/storage/` - Storage backends (Redis, ChromaDB)

2. **Research System**:
   - `research/system.py` - Research orchestration
   - `research/models.py` - Data structures for research
   - `research/browser_manager.py` - Browser automation
   - `research/browser_control.py` - Enhanced autonomous browser
   - `research/content_processor.py` - Content extraction and analysis

3. **LLM System**:
   - `llm/system.py` - LLM coordination
   - `llm/client.py` - Ollama client

4. **Core**:
   - `core/chat.py` - Main application logic
   - `core/commands.py` - Command handling
   - `core/system.py` - System management

## Troubleshooting

### Common Issues:

1. **Browser Automation Errors**:
   - Ensure you have ChromeDriver installed or use the built-in undetected_chromedriver.
   - Set `headless=True` for server environments.

2. **Ollama API Connection Issues**:
   - Verify Ollama is running: `curl http://localhost:11434/api/version`
   - Ensure required models are downloaded.

3. **Memory Storage Errors**:
   - Check Redis connection with `redis-cli ping`
   - Ensure ChromaDB persistence directory exists and is writable.

4. **Firecrawl Integration Issues**:
   - Verify your API key is valid.
   - Check for rate limiting issues.

### Logs

Logs are stored in the `logs/` directory by default. Set `LOG_LEVEL=DEBUG` for verbose logging.

## Adding New Features

To extend Sheppard with new capabilities:

1. **New Memory Storage**:
   - Create a new provider in `memory/storage/`
   - Implement the `BaseStorage` interface

2. **New Research Provider**:
   - Add a new provider in `research/providers/`
   - Implement the `BaseResearchProvider` interface

3. **New LLM Integration**:
   - Create a new provider in `llm/providers/`
   - Implement the `BaseLLMProvider` interface

## License

[MIT License](LICENSE) - See the LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Licensing

This project uses a dual-licensing approach:

- Core functionality is licensed under the Mozilla Public License 2.0
- Enterprise features require a commercial license. Contact us at licensing@example.com 

For more details, see [LICENSE-CORE.txt](LICENSE-CORE.txt) and [COMMERCIAL-LICENSE.md](COMMERCIAL-LICENSE.md)

# LangChain Documentation Helper Bot 🦜🔗

An intelligent chatbot application that helps users navigate and query LangChain documentation using RAG (Retrieval-Augmented Generation) technology. The system crawls documentation, creates a vector database, and provides accurate answers with source citations through an interactive Streamlit interface.

## Overview

This project consists of two main components:
1. **Ingestion Pipeline**: Crawls LangChain documentation, chunks content, and stores it in a vector database
2. **Chat Interface**: Interactive Streamlit web app that answers user questions using the indexed documentation

## Features

- 🔍 **Smart Documentation Search**: Uses vector similarity search to find relevant documentation sections
- 💬 **Conversational Interface**: Maintains chat history for contextual responses
- 📚 **Source Citations**: Provides source URLs for all answers
- 🎨 **Modern Dark UI**: Clean, professional Streamlit interface with dark theme
- ⚡ **Async Processing**: High-performance document ingestion with concurrent batch processing
- 🔄 **Chat History Awareness**: Rephrases queries based on conversation context

## Tech Stack

### Core Technologies
- **Python 3.x**
- **LangChain**: Framework for building LLM applications
- **OpenAI GPT**: Language model for generating responses
- **Streamlit**: Web interface framework

### Vector Storage & Embeddings
- **ChromaDB**: Local vector database for document storage
- **OpenAI Embeddings** (text-embedding-3-small): Document embedding model

### Web Crawling & Extraction
- **Tavily API**: Advanced web crawling and content extraction
  - TavilyCrawl: Website crawling
  - TavilyExtract: Content extraction
  - TavilyMap: Site mapping

### Additional Libraries
- **python-dotenv**: Environment variable management
- **Pillow (PIL)**: Image processing for user avatars
- **certifi**: SSL certificate handling
- **requests**: HTTP requests for profile pictures

## Project Structure

```
.
├── backend/
│   ├── __init__.py
│   └── core.py              # RAG chain logic and query processing
├── chroma_db/               # Local vector database (generated)
├── consts.py               # Configuration constants
├── ingestion.py            # Documentation crawling and indexing
├── logger.py               # Colored logging utilities
├── main.py                 # Streamlit application entry point
├── .env                    # Environment variables (not tracked)
└── README.md              # This file
```

## Prerequisites

- Python 3.8 or higher
- OpenAI API key
- Tavily API key (for documentation crawling)

## Installation

1. Clone the repository:
```bash
git clone https://github.com/soroushr123/documentation-helper.git
cd documentation-helper
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install langchain langchain-openai langchain-community langchain-chroma \
            langchain-pinecone langchain-tavily streamlit python-dotenv \
            pillow certifi requests
```

4. Create a `.env` file in the project root:
```env
OPENAI_API_KEY=your_openai_api_key_here
TAVILY_API_KEY=your_tavily_api_key_here
```

## Usage

### Step 1: Ingest Documentation

First, run the ingestion pipeline to crawl and index the LangChain documentation:

```bash
python ingestion.py
```

This will:
- Crawl the LangChain documentation site (https://python.langchain.com/)
- Extract and chunk the content
- Create embeddings and store them in ChromaDB
- Display colored progress logs throughout the process

**Note**: The ingestion process may take several minutes depending on the documentation size.

### Step 2: Run the Chat Application

Launch the Streamlit web interface:

```bash
streamlit run main.py
```

The application will open in your browser at `http://localhost:8501`

### Using the Chat Interface

1. Enter your question in the text input field
2. Click "Submit" or press Enter
3. View the AI-generated answer with source citations
4. Continue the conversation - the bot maintains context from previous messages

## How It Works

### Ingestion Pipeline (`ingestion.py`)

```
1. TavilyCrawl → Crawls documentation website
2. Text Splitting → Breaks documents into chunks (4000 chars, 200 overlap)
3. Embeddings → Converts chunks to vector embeddings
4. Vector Storage → Stores in ChromaDB for similarity search
```

### Query Processing (`backend/core.py`)

```
1. User Query → Received from Streamlit interface
2. History-Aware Retrieval → Rephrases query based on chat history
3. Vector Search → Finds relevant documentation chunks
4. LLM Generation → Generates answer using retrieved context
5. Source Attribution → Returns answer with source URLs
```

### Two RAG Implementations

The project includes two RAG chain implementations:

**`run_llm()` - Production Version**
- Uses `create_retrieval_chain` for streamlined RAG
- Integrates history-aware retriever
- Returns structured response with context and answer

**`run_llm2()` - Alternative Implementation**
- Uses LCEL (LangChain Expression Language) for more control
- Custom chain composition with RunnablePassthrough
- Returns both context and answer separately

## Configuration

### Crawling Settings
Modify in `ingestion.py`:
```python
tavily_crawl.invoke({
    "url": "https://python.langchain.com/",
    "max_depth": 2,              # How deep to crawl
    "extract_depth": "advanced"   # Extraction detail level
})
```

### Chunking Settings
```python
RecursiveCharacterTextSplitter(
    chunk_size=4000,    # Size of each chunk
    chunk_overlap=200   # Overlap between chunks
)
```

### Model Settings
```python
ChatOpenAI(
    model="gpt-4o-mini",  # or "gpt-4-turbo"
    temperature=0,         # Deterministic responses
    verbose=True          # Debug logging
)
```

### Batch Processing
```python
await index_documents_async(
    documents=splitted_docs,
    batch_size=500  # Documents per batch
)
```

## Customization

### Changing Documentation Source

Edit the URL in `ingestion.py`:
```python
res = tavily_crawl.invoke({
    "url": "YOUR_DOCUMENTATION_URL",
    "max_depth": 2,
    "extract_depth": "advanced"
})
```

### Styling the Interface

Modify the CSS in `main.py`:
```python
st.markdown("""
<style>
    .stApp {
        background-color: #1E1E1E;
        color: #FFFFFF;
    }
    /* Add your custom styles */
</style>
""", unsafe_allow_html=True)
```

### User Profile

Replace placeholder data in `main.py`:
```python
user_name = "Your Name"
user_email = "your.email@example.com"
```

## Logging

The project includes a custom logging system with color-coded output:

- 🚀 **Purple**: Section headers
- ℹ️ **Cyan**: Information messages
- ✅ **Green**: Success messages
- ⚠️ **Yellow**: Warnings
- ❌ **Red**: Errors

## Performance Optimization

- **Async Processing**: Concurrent document indexing for faster ingestion
- **Batch Processing**: Documents processed in batches of 500
- **Embedding Optimization**: Chunk size of 50 for API efficiency
- **Retry Logic**: 10-second minimum retry interval for rate limiting

## Troubleshooting

### SSL Certificate Errors
The project includes SSL configuration using certifi:
```python
ssl_context = ssl.create_default_context(cafile=certifi.where())
```

### Vector Database Issues
Delete and recreate the `chroma_db` directory:
```bash
rm -rf chroma_db
python ingestion.py
```

### API Rate Limits
- Reduce batch size in `index_documents_async()`
- Increase retry intervals in OpenAIEmbeddings configuration

## Alternative Vector Stores

The code includes support for Pinecone (commented out):
```python
# Uncomment to use Pinecone instead of ChromaDB
# vectorstore = PineconeVectorStore(
#     index_name="langchain-docs-2025", 
#     embedding=embeddings
# )
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

[Specify your license here]

## Acknowledgments

- Built with [LangChain](https://github.com/langchain-ai/langchain)
- Powered by [OpenAI](https://openai.com/)
- Web crawling by [Tavily](https://tavily.com/)
- UI framework by [Streamlit](https://streamlit.io/)

## Support

For issues or questions, please open an issue on GitHub or contact the maintainer.

---

**Note**: Make sure to never commit your `.env` file with API keys to version control. Add it to `.gitignore` before pushing to GitHub.

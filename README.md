# India Civic Assistant

An AI-powered system to help Indian citizens access government services, benefits, and civic information in their local languages.

## Features

- Multi-language support (13 major Indian languages)
- State-specific information
- Government scheme discovery
- Application guidance
- Offline capability

## Setup

1. Create a virtual environment:
```bash
python -m venv venv
```

2. Activate the virtual environment:
- Windows: `venv\Scripts\activate`
- Linux/Mac: `source venv/bin/activate`

3. Install dependencies:
```bash
pip install -r requirements.txt
```

## Project Structure

```
india-civic-assistant/
├── src/                    # Source code
├── tests/                  # Test files
├── data/                   # Data files (schemes, mappings)
├── config/                 # Configuration files
├── requirements.txt        # Python dependencies
└── README.md              # This file
```

## Running Tests

```bash
pytest tests/
```

## Development

This project follows the specification in `.kiro/specs/india-civic-assistant/`.

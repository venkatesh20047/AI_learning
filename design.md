# Design Document: AI Learning & Productivity Assistant

## System Architecture

### Overview
The AI Learning & Productivity Assistant follows a three-tier architecture:
1. **Presentation Layer**: HTML/CSS/JavaScript frontend
2. **Application Layer**: FastAPI backend with service modules
3. **Integration Layer**: Google Gemini AI API

```
┌─────────────────────────────────────────┐
│         Frontend (Browser)              │
│  HTML + CSS + JavaScript                │
│  - Tab Navigation                       │
│  - Knowledge Level Selector             │
│  - Input Forms                          │
│  - Output Display                       │
└──────────────┬──────────────────────────┘
               │ HTTP/JSON
               ▼
┌─────────────────────────────────────────┐
│      FastAPI Backend (Python)           │
│  ┌───────────────────────────────────┐  │
│  │   API Endpoints (main.py)         │  │
│  └───────────┬───────────────────────┘  │
│              │                           │
│  ┌───────────▼───────────────────────┐  │
│  │   Service Layer                   │  │
│  │  - TopicExplanationService        │  │
│  │  - QuestionAnsweringService       │  │
│  │  - SummarizationService           │  │
│  │  - StudyNotesService              │  │
│  └───────────┬───────────────────────┘  │
│              │                           │
│  ┌───────────▼───────────────────────┐  │
│  │   GeminiService                   │  │
│  │  - Prompt Engineering             │  │
│  │  - API Integration                │  │
│  │  - Response Parsing               │  │
│  └───────────┬───────────────────────┘  │
└──────────────┼───────────────────────────┘
               │ Google GenAI SDK
               ▼
┌─────────────────────────────────────────┐
│      Google Gemini AI API               │
│      Model: gemini-2.5-flash            │
└─────────────────────────────────────────┘
```

## Component Design

### 1. Frontend Components

#### 1.1 User Interface (index.html)
**Purpose**: Provides the visual interface for user interactions

**Structure**:
- Header: App title and subtitle
- Tab Navigation: 4 feature tabs
- Knowledge Level Selector: 3 level buttons
- Input Section: Dynamic based on selected tab
- Output Section: Formatted AI responses
- Footer: Tips and guidance

**Key Elements**:
```html
- Tab buttons (data-tab attribute)
- Level buttons (data-level attribute)
- Input fields (text/textarea)
- Action buttons with loading states
- Output container with copy button
```

#### 1.2 Styling (style.css)
**Purpose**: Provides visual design and responsive layout

**Design System**:
- **Colors**: Purple gradient theme (#667eea to #764ba2)
- **Typography**: Segoe UI font family
- **Spacing**: Consistent padding and margins
- **Borders**: Rounded corners (10-20px radius)
- **Shadows**: Subtle box shadows for depth

**Responsive Breakpoints**:
- Desktop: > 768px
- Mobile: ≤ 768px

#### 1.3 Client Logic (script.js)
**Purpose**: Handles user interactions and API communication

**State Management**:
```javascript
- currentTab: Active feature tab
- currentLevel: Selected knowledge level
```

**Key Functions**:
- `setupTabSwitching()`: Tab navigation
- `setupLevelSwitching()`: Knowledge level selection
- `handleExplain()`: Topic explanation requests
- `handleAnswer()`: Question answering requests
- `handleSummarize()`: Summarization requests
- `handleNotes()`: Study notes requests
- `displayOutput()`: Render formatted responses
- `formatMarkdown()`: Convert markdown to HTML

### 2. Backend Components

#### 2.1 Main Application (app/main.py)
**Purpose**: FastAPI application setup and endpoint definitions

**Configuration**:
- CORS middleware for cross-origin requests
- Static file serving for frontend
- Service initialization

**Endpoints**:
```python
GET  /              → Serve frontend
GET  /health        → Health check
POST /api/explain   → Topic explanation
POST /api/answer    → Question answering
POST /api/summarize → Content summarization
POST /api/study-notes → Study notes generation
```

#### 2.2 Configuration (app/config.py)
**Purpose**: Application configuration and environment management

**Settings**:
```python
- GEMINI_API_KEY: API key from environment
- APP_NAME: Application name
- VERSION: Application version
- DEBUG: Debug mode flag
```

#### 2.3 Data Models (app/models.py)
**Purpose**: Pydantic models for request/response validation

**Enums**:
```python
- KnowledgeLevel: beginner, intermediate, advanced
- RequestType: explanation, question, summarization, study_notes
```

**Request Models**:
```python
- ExplanationRequest: topic, knowledge_level
- QuestionRequest: question, knowledge_level
- SummarizationRequest: content, knowledge_level
- StudyNotesRequest: topic, knowledge_level
```

**Response Models**:
```python
- Explanation: topic, content, knowledge_level, examples, analogies
- StepByStepAnswer: question, steps, summary, examples
- Summary: original_length, summary_length, key_points, technical_terms, content
- StudyNotes: topic, content, key_definitions, practice_questions
```

### 3. Service Layer

#### 3.1 GeminiService (app/services/gemini_service.py)
**Purpose**: Core AI integration and prompt engineering

**Responsibilities**:
- Initialize Gemini AI client
- Generate prompts for different request types
- Call Gemini API
- Parse and validate responses
- Provide fallback mock responses

**Prompt Engineering Strategy**:
Each prompt includes:
1. **Context**: Knowledge level and user background
2. **Instructions**: Specific formatting requirements
3. **Structure**: Required sections and organization
4. **Examples**: Expected output format

**Prompt Templates**:
```python
- get_explanation_prompt(): Topic explanation format
- get_question_prompt(): Step-by-step answer format
- get_summary_prompt(): Summary format
- get_study_notes_prompt(): Study notes format
```

**API Integration**:
```python
client.models.generate_content(
    model='gemini-2.5-flash',
    contents=prompt
)
```

#### 3.2 TopicExplanationService (app/services/explanation_service.py)
**Purpose**: Handle topic explanation requests

**Process**:
1. Receive topic and knowledge level
2. Generate explanation prompt
3. Call GeminiService
4. Extract examples and analogies
5. Return structured Explanation object

#### 3.3 QuestionAnsweringService (app/services/question_service.py)
**Purpose**: Handle step-by-step question answering

**Process**:
1. Receive question and knowledge level
2. Generate question prompt
3. Call GeminiService
4. Parse steps from response
5. Extract summary and examples
6. Return StepByStepAnswer object

#### 3.4 SummarizationService (app/services/summarization_service.py)
**Purpose**: Handle content summarization

**Process**:
1. Receive content and knowledge level
2. Generate summary prompt
3. Call GeminiService
4. Extract key points and technical terms
5. Return Summary object

#### 3.5 StudyNotesService (app/services/study_notes_service.py)
**Purpose**: Handle study notes generation

**Process**:
1. Receive topic and knowledge level
2. Generate study notes prompt
3. Call GeminiService
4. Extract definitions and practice questions
5. Return StudyNotes object

## Data Flow

### Example: Topic Explanation Request

```
1. User Input
   ↓
   User enters "Machine Learning" + selects "Beginner"
   ↓
2. Frontend (script.js)
   ↓
   handleExplain() → POST /api/explain
   ↓
3. Backend (main.py)
   ↓
   explain_topic() endpoint receives request
   ↓
4. Service Layer (explanation_service.py)
   ↓
   TopicExplanationService.explain_topic()
   ↓
5. AI Integration (gemini_service.py)
   ↓
   GeminiService.get_explanation_prompt()
   GeminiService.generate_response()
   ↓
6. Gemini AI API
   ↓
   Returns formatted explanation
   ↓
7. Response Processing
   ↓
   Extract examples and analogies
   Create Explanation object
   ↓
8. Frontend Display
   ↓
   formatMarkdown() → displayOutput()
   ↓
9. User sees formatted explanation
```

## Prompt Engineering Design

### Knowledge Level Adaptation

**Beginner Level**:
- Very simple language (explain like to a 10-year-old)
- Lots of concrete examples
- Analogies to familiar things
- Short sentences
- No jargon

**Intermediate Level**:
- Clear language with some technical terms
- Practical examples
- Balance between simplicity and accuracy
- Moderate detail

**Advanced Level**:
- Technical terminology
- Detailed mechanisms
- Advanced concepts
- In-depth examples

### Response Format Requirements

All prompts enforce:
1. **Headings**: Clear section headers (# and ##)
2. **Bullet Points**: Use • symbol for lists
3. **Structure**: Logical organization
4. **Examples**: Concrete, relatable examples
5. **Analogies**: Familiar comparisons

## Error Handling Strategy

### Error Categories

1. **Client Errors (4xx)**:
   - Invalid input
   - Missing required fields
   - Malformed requests

2. **Server Errors (5xx)**:
   - AI API failures
   - Service unavailable
   - Internal processing errors

### Error Response Format

```json
{
  "error": "Error type",
  "message": "Technical error message",
  "suggestions": ["User-friendly suggestion"]
}
```

### Fallback Mechanism

When Gemini API fails:
1. Log error details
2. Return mock response
3. Maintain user experience
4. Provide retry option

## Security Considerations

### API Key Management
- Stored in `.env` file
- Never committed to version control
- Loaded via environment variables
- Not exposed in client code

### Input Validation
- Pydantic models validate all inputs
- Type checking on all fields
- Length limits on text inputs
- Sanitization of user content

### CORS Configuration
- Allow all origins (development)
- Restrict in production
- Proper headers configuration

## Performance Optimization

### Frontend
- Minimal JavaScript dependencies
- CSS-only animations
- Efficient DOM manipulation
- Debounced API calls

### Backend
- Async/await for non-blocking I/O
- Connection pooling
- Response caching (future)
- Efficient prompt generation

### AI Integration
- Optimized prompt length
- Model selection (gemini-2.5-flash for speed)
- Timeout handling
- Retry logic

## Deployment Architecture

### Development
```
Local Machine
├── Python virtual environment
├── FastAPI dev server (auto-reload)
├── Static file serving
└── Environment variables from .env
```

### Production (Future)
```
Cloud Platform
├── Container (Docker)
├── Load Balancer
├── Multiple API instances
├── Environment secrets management
└── CDN for static files
```

## Testing Strategy

### Unit Tests
- Service layer functions
- Prompt generation
- Response parsing
- Error handling

### Integration Tests
- API endpoint testing
- End-to-end workflows
- Error scenarios
- Mock AI responses

### Manual Testing
- UI/UX validation
- Cross-browser compatibility
- Responsive design
- Accessibility

## Monitoring and Logging

### Logging Levels
- INFO: Normal operations
- WARNING: Fallback to mock responses
- ERROR: API failures, exceptions

### Metrics to Track
- API response times
- Error rates
- User requests per feature
- Knowledge level distribution

## Future Enhancements

### Technical Improvements
1. Database integration for history
2. Caching layer (Redis)
3. Rate limiting
4. WebSocket for real-time updates
5. Background task queue

### Feature Additions
1. User authentication
2. Saved conversations
3. Export functionality (PDF, Markdown)
4. Multi-language support
5. Voice input/output
6. Collaborative features
7. Custom prompt templates
8. Analytics dashboard

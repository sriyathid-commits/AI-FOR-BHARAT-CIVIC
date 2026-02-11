# Design Document: India Civic Assistant

## Overview

The India Civic Assistant is a conversational AI system that helps Indian citizens access government services, benefits, and civic information in their preferred language. The system addresses India's linguistic diversity by supporting 13 major Indian languages and provides state-specific information tailored to each user's geographic location.

The architecture follows a modular design with clear separation between language processing, knowledge retrieval, and response generation. This enables independent updates to language models, government scheme databases, and state-specific information without affecting other components.

Key design principles:
- **Language-first approach**: All components handle multilingual content natively
- **State-aware processing**: Geographic context influences information retrieval and presentation
- **Offline-capable**: Core functionality works with cached data when connectivity is limited
- **Privacy-preserving**: Personal information processing happens locally without external transmission
- **Extensible knowledge base**: Easy addition of new schemes, languages, and states

## Architecture

The system uses a pipeline architecture with the following major components:

```
User Input → Language Detection → Query Understanding → Knowledge Retrieval → Response Generation → Translation → User Output
                                         ↓
                                  State Resolution
                                         ↓
                                  Eligibility Matching
```

### Component Interaction Flow

1. **Input Processing Layer**
   - Language Detector identifies the input language
   - Query Parser extracts intent, entities, and context
   - State Resolver determines geographic scope

2. **Knowledge Layer**
   - Scheme Database stores government program information
   - Eligibility Engine matches user situations to scheme criteria
   - Document Repository maintains application requirements

3. **Response Layer**
   - Response Generator formulates answers based on retrieved knowledge
   - Translation Engine converts responses to user's language
   - Context Manager maintains conversation state

4. **Offline Layer**
   - Cache Manager stores frequently accessed information
   - Sync Engine updates cached data when online
   - Offline Query Handler processes requests using cached data

## Components and Interfaces

### 1. Language Detector

**Purpose**: Identifies the language of user input to enable appropriate processing and response.

**Interface**:
```
function detectLanguage(text: string): LanguageCode
  Input: text - User's input string
  Output: LanguageCode - ISO 639-1 code (e.g., "hi" for Hindi, "ta" for Tamil)
  
function getConfidence(): number
  Output: Confidence score between 0.0 and 1.0
```

**Implementation approach**:
- Use character n-gram analysis for script-based detection (Devanagari, Tamil, Bengali scripts)
- Maintain language-specific word frequency lists for disambiguation
- Support code-mixing detection (e.g., Hindi-English mixed text)
- Cache language preference per session to reduce repeated detection

### 2. Query Parser

**Purpose**: Extracts structured information from natural language queries including intent, entities, and context.

**Interface**:
```
function parseQuery(text: string, language: LanguageCode, context: ConversationContext): ParsedQuery
  Input: 
    - text: User's query
    - language: Detected language code
    - context: Previous conversation state
  Output: ParsedQuery containing:
    - intent: QueryIntent (SCHEME_SEARCH, ELIGIBILITY_CHECK, APPLICATION_HELP, GENERAL_INFO)
    - entities: Map of entity types to values (SCHEME_NAME, CATEGORY, STATE, etc.)
    - contextReferences: References to previous conversation turns
```

**Implementation approach**:
- Use intent classification model trained on government service queries
- Named entity recognition for scheme names, states, categories, and personal attributes
- Coreference resolution to link pronouns and references to previous context
- Handle informal language and regional vocabulary variations

### 3. State Resolver

**Purpose**: Determines the relevant state or union territory for providing location-specific information.

**Interface**:
```
function resolveState(query: ParsedQuery, userProfile: UserProfile): StateInfo
  Input:
    - query: Parsed query with potential location entities
    - userProfile: User's saved preferences including location
  Output: StateInfo containing:
    - stateCode: Two-letter state code
    - stateName: Full state name in user's language
    - confidence: Confidence in state resolution
    
function getApplicableJurisdictions(scheme: Scheme): List<StateCode>
  Input: scheme - Government scheme object
  Output: List of state codes where scheme is available
```

**Implementation approach**:
- Extract location entities from query (city names, district names, state names)
- Use IP-based geolocation as fallback when no explicit location provided
- Maintain mapping of cities/districts to states
- Handle union territories and special administrative regions

### 4. Scheme Database

**Purpose**: Stores comprehensive information about government schemes at central and state levels.

**Data Model**:
```
Scheme:
  - id: string
  - name: Map<LanguageCode, string>
  - description: Map<LanguageCode, string>
  - category: SchemeCategory (HEALTHCARE, EDUCATION, AGRICULTURE, etc.)
  - level: GovernmentLevel (CENTRAL, STATE)
  - applicableStates: List<StateCode>
  - eligibilityCriteria: List<Criterion>
  - benefits: Map<LanguageCode, List<string>>
  - applicationProcess: ApplicationProcess
  - documents: List<Document>
  - officialLinks: Map<StateCode, URL>
  - lastUpdated: timestamp
  - source: OfficialSource

Criterion:
  - type: CriterionType (AGE, INCOME, OCCUPATION, GENDER, etc.)
  - operator: ComparisonOperator (LESS_THAN, GREATER_THAN, EQUALS, IN_RANGE)
  - value: any
  - description: Map<LanguageCode, string>

ApplicationProcess:
  - steps: List<Step>
  - onlinePortal: URL (optional)
  - officeLocations: Map<StateCode, List<OfficeInfo>>
  - helpline: Map<StateCode, string>
  - timeline: string

Document:
  - name: Map<LanguageCode, string>
  - description: Map<LanguageCode, string>
  - required: boolean
  - alternatives: List<Document>
```

**Interface**:
```
function searchSchemes(query: ParsedQuery, state: StateInfo): List<Scheme>
  Input: Parsed query and state context
  Output: Ranked list of relevant schemes
  
function getSchemeById(id: string, language: LanguageCode): Scheme
  Input: Scheme identifier and desired language
  Output: Scheme with all text fields in specified language
  
function getSchemesByCategory(category: SchemeCategory, state: StateInfo): List<Scheme>
  Input: Category and state filter
  Output: All schemes in category applicable to state
```

**Implementation approach**:
- Store multilingual content with language-specific fields
- Index schemes by category, keywords, and eligibility criteria
- Maintain separate collections for central and state schemes
- Use vector embeddings for semantic search across scheme descriptions
- Regular update mechanism from official government sources

### 5. Eligibility Engine

**Purpose**: Matches user situations and attributes against scheme eligibility criteria.

**Interface**:
```
function checkEligibility(scheme: Scheme, userAttributes: Map<string, any>): EligibilityResult
  Input:
    - scheme: Government scheme to check
    - userAttributes: User-provided attributes (age, income, occupation, etc.)
  Output: EligibilityResult containing:
    - eligible: boolean
    - matchedCriteria: List<Criterion>
    - unmatchedCriteria: List<Criterion>
    - confidence: number
    
function findEligibleSchemes(userAttributes: Map<string, any>, state: StateInfo): List<Scheme>
  Input: User attributes and state
  Output: All schemes user is likely eligible for
```

**Implementation approach**:
- Evaluate each criterion against provided user attributes
- Handle missing attributes gracefully (partial matching)
- Support range-based criteria (age ranges, income brackets)
- Provide explanations for eligibility decisions
- Privacy-preserving: process attributes locally without storage

### 6. Response Generator

**Purpose**: Formulates natural language responses based on retrieved knowledge and user context.

**Interface**:
```
function generateResponse(
  query: ParsedQuery,
  schemes: List<Scheme>,
  eligibility: EligibilityResult,
  language: LanguageCode,
  context: ConversationContext
): Response
  Input: All relevant information for response formulation
  Output: Response containing:
    - text: Main response text
    - schemes: Structured scheme information
    - followUpSuggestions: List of suggested next questions
    - sources: Official source citations
```

**Implementation approach**:
- Template-based generation for structured information (scheme details, steps)
- Natural language generation for explanations and summaries
- Adapt tone and complexity based on query type
- Include citations to official sources
- Generate follow-up suggestions based on query intent

### 7. Translation Engine

**Purpose**: Translates content between supported Indian languages while preserving meaning and government terminology.

**Interface**:
```
function translate(text: string, fromLang: LanguageCode, toLang: LanguageCode): string
  Input: Text and source/target languages
  Output: Translated text
  
function translateScheme(scheme: Scheme, toLang: LanguageCode): Scheme
  Input: Scheme object and target language
  Output: Scheme with all text fields translated
```

**Implementation approach**:
- Use neural machine translation models trained on government domain text
- Maintain terminology glossary for consistent translation of scheme names and technical terms
- Preserve official scheme names in original language with translation in parentheses
- Handle code-mixing (Hinglish, Tanglish) appropriately
- Cache translations of frequently accessed content

### 8. Context Manager

**Purpose**: Maintains conversation state to enable contextual understanding across multiple turns.

**Interface**:
```
function updateContext(query: ParsedQuery, response: Response): ConversationContext
  Input: Current query and generated response
  Output: Updated conversation context
  
function getContext(): ConversationContext
  Output: Current conversation state
  
function clearContext(): void
  Effect: Resets conversation state (called at session end)

ConversationContext:
  - sessionId: string
  - language: LanguageCode
  - state: StateInfo
  - discussedSchemes: List<string>
  - userAttributes: Map<string, any>
  - conversationHistory: List<Turn>
  - timestamp: timestamp
```

**Implementation approach**:
- Store conversation history with timestamps
- Track mentioned schemes and entities
- Maintain user attributes shared during conversation
- Implement session timeout (30 minutes of inactivity)
- Clear personal data at session end for privacy

### 9. Cache Manager (Offline Support)

**Purpose**: Enables offline access to essential civic information by caching frequently accessed data.

**Interface**:
```
function cacheSchemes(schemes: List<Scheme>): void
  Input: Schemes to cache locally
  Effect: Stores schemes in local storage
  
function getCachedSchemes(state: StateInfo): List<Scheme>
  Input: State filter
  Output: Cached schemes for state
  
function syncCache(): SyncResult
  Effect: Updates cached data from server
  Output: Sync status and updated item count
  
function isOfflineMode(): boolean
  Output: Whether system is operating offline
```

**Implementation approach**:
- Cache top 100 most accessed schemes per state
- Store state-specific information packages
- Implement differential sync to minimize data transfer
- Mark cached data with last update timestamp
- Provide clear offline mode indicators to users

## Data Models

### User Profile (Session-scoped)
```
UserProfile:
  - sessionId: string
  - preferredLanguage: LanguageCode
  - state: StateInfo (optional)
  - attributes: Map<string, any> (age, occupation, etc.)
  - consentGiven: boolean
  - createdAt: timestamp
  - expiresAt: timestamp
```

### Query Intent Types
```
enum QueryIntent:
  - SCHEME_SEARCH: User looking for relevant schemes
  - ELIGIBILITY_CHECK: User wants to know if they qualify
  - APPLICATION_HELP: User needs application guidance
  - DOCUMENT_INFO: User asking about required documents
  - GENERAL_INFO: General civic information query
  - CLARIFICATION: User providing additional context
```

### Scheme Categories
```
enum SchemeCategory:
  - HEALTHCARE: Health insurance, medical assistance
  - EDUCATION: Scholarships, educational loans
  - AGRICULTURE: Farmer support, crop insurance
  - EMPLOYMENT: Job programs, skill development
  - HOUSING: Housing schemes, urban development
  - FINANCIAL_INCLUSION: Banking, insurance, pensions
  - SOCIAL_WELFARE: Women, children, elderly support
  - INFRASTRUCTURE: Rural development, connectivity
```

## Correctness Properties


A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Property 1: Language Detection Accuracy

*For any* text input in a supported Indian language, the Language Detector should identify the correct language with at least 95% accuracy.

**Validates: Requirements 1.1**

### Property 2: Language Response Consistency

*For any* user query, when the system detects the input language, the response should be generated in that same detected language.

**Validates: Requirements 1.2**

### Property 3: Language Preference Persistence

*For any* explicitly selected language preference, all subsequent queries in the same session should receive responses in that preferred language regardless of the input language.

**Validates: Requirements 1.4**

### Property 4: Translation Terminology Preservation

*For any* government scheme name or official terminology, translating to another supported language and back should preserve the key terms and official names.

**Validates: Requirements 1.5**

### Property 5: State Resolution Correctness

*For any* valid location input (city name, district name, or state name), the State Resolver should correctly identify the corresponding Indian state or union territory.

**Validates: Requirements 2.1**

### Property 6: State-Specific Scheme Inclusion

*For any* state query, the returned scheme list should include both applicable state-specific schemes and central government schemes.

**Validates: Requirements 2.2**

### Property 7: Geographic Eligibility Indication

*For any* scheme with limited geographic availability, the scheme information should clearly indicate which states or regions are eligible.

**Validates: Requirements 2.3**

### Property 8: State-Level Information Priority

*For any* scheme that has state-specific variations, when queried for a specific state, the system should present the state-specific version rather than only the central guidelines.

**Validates: Requirements 2.5**

### Property 9: Relevant Scheme Retrieval

*For any* user query describing a situation or need, the system should return at least one relevant scheme from the appropriate category.

**Validates: Requirements 3.1**

### Property 10: Complete Scheme Information

*For any* scheme presented in a response, the output should contain the scheme name, description, benefits, eligibility criteria, required documents, and application process (when applicable).

**Validates: Requirements 3.2, 4.1, 4.2**

### Property 11: Scheme Relevance Ranking

*For any* query that returns multiple schemes, the schemes should be ordered such that more relevant schemes (matching more query criteria) appear before less relevant ones.

**Validates: Requirements 3.5**

### Property 12: Application Portal Links

*For any* scheme with an online application portal, the response should include a valid URL to the official government website.

**Validates: Requirements 4.3**

### Property 13: Offline Process Information Completeness

*For any* scheme requiring offline application, the response should include physical office locations, office hours, and contact information.

**Validates: Requirements 4.4**

### Property 14: Deadline Communication

*For any* scheme with application deadlines, the response should clearly indicate the deadline date and any time-sensitive requirements.

**Validates: Requirements 4.5**

### Property 15: Colloquial Language Understanding

*For any* set of queries expressing the same intent using different colloquial terms or regional vocabulary, the system should identify the same query intent.

**Validates: Requirements 5.1**

### Property 16: Ambiguity Handling

*For any* ambiguous query that could match multiple intents or schemes, the system should respond with clarifying questions rather than making assumptions.

**Validates: Requirements 5.2**

### Property 17: Conversational Context Maintenance

*For any* multi-turn conversation where later queries reference earlier context (using pronouns or implicit references), the system should correctly resolve references to previously mentioned schemes or entities.

**Validates: Requirements 5.3**

### Property 18: Unintelligible Query Handling

*For any* query that cannot be parsed or understood, the system should request clarification rather than returning scheme recommendations.

**Validates: Requirements 5.4**

### Property 19: Scheme Name Variation Recognition

*For any* government scheme, all common name variations and synonyms across supported languages should resolve to the same scheme entity.

**Validates: Requirements 5.5**

### Property 20: Source Citation Presence

*For any* scheme in the knowledge base, the scheme data should include citations to official government sources.

**Validates: Requirements 6.1, 6.4**

### Property 21: Last Updated Timestamp

*For any* scheme information presented, the response should include the last verification or update date.

**Validates: Requirements 6.3**

### Property 22: Fallback Contact Information

*For any* query where the system lacks current or complete information, the response should include official government helpline numbers or website URLs as fallback resources.

**Validates: Requirements 6.5**

### Property 23: Spelling Error Tolerance

*For any* query with common spelling or grammatical errors, the system should still correctly identify the intended query and respond appropriately.

**Validates: Requirements 7.2**

### Property 24: Technical Term Explanation

*For any* response containing technical or legal terminology, the system should provide simple explanations alongside the technical terms.

**Validates: Requirements 7.5**

### Property 25: Session Data Deletion

*For any* user session, when the session ends or times out, all personally identifiable information collected during that session should be deleted and not retrievable.

**Validates: Requirements 8.1, 8.4**

### Property 26: Local Eligibility Processing

*For any* eligibility check involving personal details, the processing should occur locally without making external API calls or transmitting personal data.

**Validates: Requirements 8.2**

### Property 27: Privacy Notice Before Collection

*For any* interaction that requires collecting personal information, the system should present privacy information before requesting the personal details.

**Validates: Requirements 8.3**

### Property 28: Offline Scheme Access

*For any* cached scheme, when the system is in offline mode, the scheme information should be accessible and the response should indicate offline status.

**Validates: Requirements 9.1, 9.2**

### Property 29: Online Synchronization

*For any* transition from offline to online mode, the system should automatically synchronize cached data with the latest knowledge base updates.

**Validates: Requirements 9.3**

### Property 30: Offline Query Queueing

*For any* query that requires online processing while in offline mode, the query should be queued and processed once connectivity is restored, with user notification.

**Validates: Requirements 9.5**

### Property 31: Feedback Mechanism Availability

*For any* response provided to a user, the interface should include mechanisms for both rating the helpfulness and reporting inaccuracies.

**Validates: Requirements 10.1, 10.2**

### Property 32: Feedback Acknowledgment

*For any* issue report or feedback submission, the system should return an acknowledgment message with a unique reference number.

**Validates: Requirements 10.4**

## Error Handling

The system must gracefully handle various error conditions while maintaining user trust and providing helpful guidance:

### Language Detection Errors
- **Low confidence detection**: When language detection confidence is below 95%, prompt user to select their preferred language
- **Unsupported language**: Inform user of supported languages and offer to continue in Hindi or English
- **Mixed language input**: Detect primary language and respond accordingly, or ask for clarification

### Knowledge Base Errors
- **Scheme not found**: Suggest similar schemes or broader category searches
- **Outdated information**: Display last update date and provide official contact information
- **Missing state data**: Fall back to central scheme information and note state-specific details may vary

### State Resolution Errors
- **Ambiguous location**: Ask user to specify state when city/district names are ambiguous
- **Invalid location**: Suggest correct spelling or ask user to select from state list
- **Location not provided**: Use IP-based geolocation with user confirmation

### Eligibility Checking Errors
- **Insufficient information**: Request specific missing attributes needed for eligibility determination
- **Conflicting criteria**: Explain which criteria are met and which are not
- **Edge cases**: Provide conservative eligibility assessment and suggest contacting officials

### Translation Errors
- **Translation failure**: Fall back to English or Hindi with apology message
- **Terminology ambiguity**: Preserve original term with explanation
- **Character encoding issues**: Handle gracefully with Unicode normalization

### Offline Mode Errors
- **Cache miss**: Inform user that information requires online access and offer to queue query
- **Sync failure**: Retry with exponential backoff, inform user of sync status
- **Storage quota exceeded**: Prioritize most accessed schemes, allow user to manage cache

### Session and Privacy Errors
- **Session timeout**: Inform user and offer to restart conversation
- **Data deletion failure**: Log error and retry, ensure no PII persists
- **Consent not provided**: Do not collect personal information, offer general guidance only

### General Error Principles
- Always provide actionable next steps
- Never expose technical error details to users
- Log errors for system improvement
- Maintain conversation context when possible
- Offer alternative ways to get information (helplines, websites)

## Testing Strategy

The India Civic Assistant requires comprehensive testing across multiple dimensions: functional correctness, language accuracy, data quality, and user experience. We employ a dual testing approach combining unit tests for specific scenarios and property-based tests for universal correctness guarantees.

### Testing Approach

**Unit Tests**: Focus on specific examples, edge cases, and integration points
- Specific scheme queries with known expected results
- Edge cases like empty inputs, special characters, very long queries
- Error conditions and fallback behaviors
- Integration between components (e.g., query parser → knowledge retrieval)

**Property-Based Tests**: Verify universal properties across randomized inputs
- Language detection accuracy across all supported languages
- Translation consistency and terminology preservation
- State resolution correctness for all Indian states
- Scheme information completeness
- Privacy and data handling guarantees

**Configuration**: Each property-based test should run a minimum of 100 iterations to ensure comprehensive coverage through randomization.

**Test Tagging**: Each property test must include a comment tag referencing the design document property:
```
# Feature: india-civic-assistant, Property 1: Language Detection Accuracy
```

### Test Coverage by Component

**Language Detector**
- Unit tests: Specific text samples in each supported language
- Property tests: Random text generation in each language, verify 95%+ accuracy (Property 1)
- Edge cases: Mixed language text, very short inputs, special characters

**Query Parser**
- Unit tests: Known query patterns for each intent type
- Property tests: Colloquial language variations resolve to same intent (Property 15)
- Edge cases: Ambiguous queries, unintelligible input, context references

**State Resolver**
- Unit tests: Major cities and all state capitals
- Property tests: All valid location names resolve to correct state (Property 5)
- Edge cases: Ambiguous city names (e.g., cities with same name in different states)

**Scheme Database**
- Unit tests: Verify presence of major central schemes
- Property tests: All schemes have complete required fields (Property 10, 20, 21)
- Data validation: Regular checks against official government sources

**Eligibility Engine**
- Unit tests: Known eligibility scenarios for major schemes
- Property tests: Eligibility logic consistency across all schemes
- Edge cases: Missing attributes, boundary conditions (age limits, income thresholds)

**Response Generator**
- Unit tests: Response format for each query intent type
- Property tests: Complete information in responses (Property 10), technical term explanations (Property 24)
- Edge cases: Very long scheme lists, missing optional fields

**Translation Engine**
- Unit tests: Key government terms in all language pairs
- Property tests: Round-trip translation preserves terminology (Property 4)
- Edge cases: Scheme names, legal terms, numbers and dates

**Context Manager**
- Unit tests: Multi-turn conversations with known context
- Property tests: Context references resolve correctly (Property 17)
- Edge cases: Session timeout, context overflow, ambiguous references

**Cache Manager**
- Unit tests: Cache operations (store, retrieve, sync)
- Property tests: Offline access to cached schemes (Property 28), sync after reconnection (Property 29)
- Edge cases: Storage limits, corrupted cache, partial sync

**Privacy and Data Handling**
- Unit tests: Session lifecycle with PII
- Property tests: Data deletion after session end (Property 25), local processing (Property 26)
- Security tests: Verify no PII in logs, no external transmission during eligibility checks

### Language-Specific Testing

Each supported language requires dedicated test coverage:
- Minimum 100 test queries per language
- Common scheme names and variations in each language
- Colloquial terms and regional vocabulary
- Error messages and system prompts in each language

### Data Quality Testing

Regular validation of knowledge base:
- Verify all schemes have required fields
- Check for broken URLs to government portals
- Validate eligibility criteria are machine-readable
- Ensure multilingual content is synchronized
- Compare against official government scheme lists

### Integration Testing

End-to-end flows across components:
- Complete user journeys from query to application guidance
- Multi-turn conversations with context
- Offline-to-online transitions
- Language switching mid-conversation
- State-specific information retrieval

### Performance Testing

- Response time under 2 seconds for 95% of queries
- Language detection under 100ms
- Cache retrieval under 50ms
- Support for 1000 concurrent users
- Offline mode with 500MB cache size

### Accessibility Testing

- Screen reader compatibility for voice output
- Voice input accuracy across languages and accents
- Simple language readability (Grade 8 level or below)
- Clear error messages and guidance

### Continuous Testing

- Automated test suite runs on every code change
- Daily data quality checks against government sources
- Weekly language accuracy validation
- Monthly user feedback analysis
- Quarterly comprehensive security audit

This testing strategy ensures the India Civic Assistant maintains high accuracy, reliability, and trustworthiness while serving India's diverse population across languages and states.

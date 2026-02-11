# Implementation Plan: India Civic Assistant

## Overview

This implementation plan breaks down the India Civic Assistant into discrete coding tasks. The system will be built in Python, following a modular architecture that separates language processing, knowledge retrieval, and response generation. Each task builds incrementally, with early validation through testing to catch issues quickly.

The implementation prioritizes core functionality first (language detection, query parsing, knowledge retrieval) before adding advanced features (offline mode, voice support). Property-based tests are integrated throughout to validate universal correctness properties.

## Tasks

- [-] 1. Set up project structure and core dependencies
  - Create Python project with virtual environment
  - Install core dependencies: FastAPI (API framework), Pydantic (data validation), pytest (testing), hypothesis (property-based testing)
  - Set up project directory structure: `src/`, `tests/`, `data/`, `config/`
  - Create configuration management for supported languages and states
  - _Requirements: 1.3, 2.4_

- [ ] 2. Implement Language Detector component
  - [~] 2.1 Create LanguageDetector class with character n-gram analysis
    - Implement script detection for Devanagari, Tamil, Bengali, and other Indian scripts
    - Build language-specific word frequency lists for disambiguation
    - Return language code and confidence score
    - _Requirements: 1.1_
  
  - [ ]* 2.2 Write property test for language detection accuracy
    - **Property 1: Language Detection Accuracy**
    - Generate random text samples in each of the 13 supported languages
    - Verify detection accuracy meets 95% threshold across all languages
    - **Validates: Requirements 1.1**
  
  - [ ]* 2.3 Write unit tests for language detection edge cases
    - Test very short inputs (1-3 words)
    - Test mixed-language text (code-mixing)
    - Test inputs with special characters and numbers
    - _Requirements: 1.1_

- [ ] 3. Implement data models and State Resolver
  - [~] 3.1 Create Pydantic models for core data structures
    - Define Scheme, Criterion, ApplicationProcess, Document models
    - Define StateInfo, UserProfile, ConversationContext models
    - Implement validation rules for each model
    - _Requirements: 2.1, 2.4, 8.1_
  
  - [~] 3.2 Implement StateResolver class
    - Create mapping of cities/districts to states
    - Implement location entity extraction from text
    - Add IP-based geolocation fallback
    - _Requirements: 2.1_
  
  - [ ]* 3.3 Write property test for state resolution
    - **Property 5: State Resolution Correctness**
    - Generate random valid location names (cities, districts, states)
    - Verify correct state identification for all inputs
    - **Validates: Requirements 2.1**

- [ ] 4. Build Scheme Database and Knowledge Base
  - [~] 4.1 Design database schema for government schemes
    - Create SQLite database with schemes, eligibility criteria, and documents tables
    - Implement multilingual content storage (JSON fields for language maps)
    - Add indexes for efficient querying by category, state, and keywords
    - _Requirements: 3.4, 6.1_
  
  - [~] 4.2 Create data loading utilities
    - Build CSV/JSON import scripts for scheme data
    - Implement validation to ensure all required fields are present
    - Add sample data for major central government schemes (PM-KISAN, Ayushman Bharat, etc.)
    - _Requirements: 3.4, 6.1_
  
  - [ ]* 4.3 Write property test for scheme data completeness
    - **Property 10: Complete Scheme Information**
    - **Property 20: Source Citation Presence**
    - **Property 21: Last Updated Timestamp**
    - For all schemes in database, verify presence of required fields
    - Verify each scheme has name, description, benefits, eligibility, documents, source, and timestamp
    - **Validates: Requirements 3.2, 4.1, 4.2, 6.1, 6.3, 6.4**

- [~] 5. Checkpoint - Verify data models and database
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 6. Implement Query Parser
  - [~] 6.1 Create QueryParser class with intent classification
    - Implement intent detection for SCHEME_SEARCH, ELIGIBILITY_CHECK, APPLICATION_HELP, GENERAL_INFO
    - Build named entity recognition for scheme names, categories, and states
    - Add context reference resolution for multi-turn conversations
    - _Requirements: 5.1, 5.2, 5.3_
  
  - [~] 6.2 Add support for colloquial language and synonyms
    - Create synonym dictionaries for common scheme names in each language
    - Implement fuzzy matching for spelling errors
    - Handle regional vocabulary variations
    - _Requirements: 5.1, 5.5, 7.2_
  
  - [ ]* 6.3 Write property tests for query understanding
    - **Property 15: Colloquial Language Understanding**
    - **Property 18: Unintelligible Query Handling**
    - **Property 19: Scheme Name Variation Recognition**
    - **Property 23: Spelling Error Tolerance**
    - Generate queries with variations, colloquialisms, and errors
    - Verify consistent intent detection and appropriate handling
    - **Validates: Requirements 5.1, 5.4, 5.5, 7.2**

- [ ] 7. Implement Eligibility Engine
  - [~] 7.1 Create EligibilityEngine class
    - Implement criterion evaluation logic (age ranges, income thresholds, etc.)
    - Build matching algorithm to find eligible schemes based on user attributes
    - Handle partial matching when some attributes are missing
    - _Requirements: 3.1, 3.3_
  
  - [ ]* 7.2 Write property test for local eligibility processing
    - **Property 26: Local Eligibility Processing**
    - Verify no external API calls during eligibility checks
    - Monitor network activity during eligibility processing
    - **Validates: Requirements 8.2**
  
  - [ ]* 7.3 Write unit tests for eligibility edge cases
    - Test boundary conditions (exactly at age limit, income threshold)
    - Test missing attributes (partial information)
    - Test conflicting criteria
    - _Requirements: 3.3_

- [ ] 8. Implement knowledge retrieval and search
  - [~] 8.1 Create SchemeDatabase class with search functionality
    - Implement keyword-based search across scheme descriptions
    - Add category filtering and state-specific filtering
    - Implement relevance ranking algorithm
    - _Requirements: 3.1, 3.5_
  
  - [~] 8.2 Add semantic search using embeddings
    - Generate embeddings for scheme descriptions
    - Implement vector similarity search for better query matching
    - Combine keyword and semantic search results
    - _Requirements: 3.1_
  
  - [ ]* 8.3 Write property tests for scheme retrieval
    - **Property 6: State-Specific Scheme Inclusion**
    - **Property 9: Relevant Scheme Retrieval**
    - **Property 11: Scheme Relevance Ranking**
    - Generate random queries and verify relevant schemes are returned
    - Verify state-specific and central schemes both appear
    - Verify ranking order (more relevant schemes first)
    - **Validates: Requirements 2.2, 3.1, 3.5**

- [~] 9. Checkpoint - Verify core retrieval functionality
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Implement Translation Engine
  - [~] 10.1 Create TranslationEngine class
    - Integrate translation library (e.g., IndicTrans, Google Translate API)
    - Implement caching for frequently translated content
    - Build terminology glossary for government scheme names
    - _Requirements: 1.2, 1.5_
  
  - [~] 10.2 Add terminology preservation logic
    - Identify official scheme names and technical terms
    - Preserve original terms with translations in parentheses
    - Handle code-mixing appropriately
    - _Requirements: 1.5_
  
  - [ ]* 10.3 Write property test for translation
    - **Property 2: Language Response Consistency**
    - **Property 4: Translation Terminology Preservation**
    - Generate queries in random languages, verify responses in same language
    - Test round-trip translation preserves key terms
    - **Validates: Requirements 1.2, 1.5**

- [ ] 11. Implement Response Generator
  - [~] 11.1 Create ResponseGenerator class
    - Build template-based generation for structured information
    - Implement natural language generation for explanations
    - Add follow-up suggestion generation
    - _Requirements: 3.2, 4.1, 4.5_
  
  - [~] 11.2 Add comprehensive information inclusion
    - Ensure responses include all required scheme details
    - Add application process steps, documents, and links
    - Include deadline information when applicable
    - _Requirements: 3.2, 4.1, 4.2, 4.3, 4.4, 4.5_
  
  - [~] 11.3 Implement technical term explanation
    - Detect technical/legal terminology in responses
    - Add simple explanations in parentheses
    - Maintain glossary of common technical terms
    - _Requirements: 7.5_
  
  - [ ]* 11.4 Write property tests for response completeness
    - **Property 7: Geographic Eligibility Indication**
    - **Property 10: Complete Scheme Information**
    - **Property 12: Application Portal Links**
    - **Property 13: Offline Process Information Completeness**
    - **Property 14: Deadline Communication**
    - **Property 24: Technical Term Explanation**
    - Generate random schemes and verify all required information in responses
    - **Validates: Requirements 2.3, 3.2, 4.1, 4.2, 4.3, 4.4, 4.5, 7.5**

- [ ] 12. Implement Context Manager for conversation state
  - [~] 12.1 Create ContextManager class
    - Implement conversation history tracking
    - Build entity and scheme reference tracking
    - Add session timeout logic (30 minutes)
    - _Requirements: 5.3, 8.1_
  
  - [~] 12.2 Add privacy-preserving data handling
    - Implement automatic PII deletion at session end
    - Add consent tracking before collecting personal information
    - Ensure no PII in logs or external transmissions
    - _Requirements: 8.1, 8.3, 8.4_
  
  - [ ]* 12.3 Write property tests for context and privacy
    - **Property 17: Conversational Context Maintenance**
    - **Property 25: Session Data Deletion**
    - **Property 27: Privacy Notice Before Collection**
    - Generate multi-turn conversations, verify context resolution
    - Verify PII deletion after session end
    - Verify privacy notice before PII collection
    - **Validates: Requirements 5.3, 8.1, 8.3, 8.4**

- [ ] 13. Implement ambiguity and error handling
  - [~] 13.1 Add ambiguity detection to QueryParser
    - Identify queries matching multiple intents or schemes
    - Generate clarifying questions for ambiguous queries
    - _Requirements: 5.2_
  
  - [~] 13.2 Implement fallback mechanisms
    - Add helpline and website information for incomplete data
    - Handle unsupported languages gracefully
    - Provide actionable error messages
    - _Requirements: 6.5_
  
  - [ ]* 13.3 Write property tests for error handling
    - **Property 16: Ambiguity Handling**
    - **Property 22: Fallback Contact Information**
    - Generate ambiguous and unintelligible queries
    - Verify clarifying questions instead of assumptions
    - Verify fallback information when data is incomplete
    - **Validates: Requirements 5.2, 6.5**

- [~] 14. Checkpoint - Verify end-to-end query flow
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 15. Implement Cache Manager for offline support
  - [~] 15.1 Create CacheManager class
    - Implement local storage for top schemes per state
    - Build differential sync mechanism
    - Add cache expiration and update logic
    - _Requirements: 9.1, 9.3_
  
  - [~] 15.2 Add offline mode detection and handling
    - Detect network connectivity status
    - Provide offline indicators in responses
    - Implement query queueing for online-required requests
    - _Requirements: 9.2, 9.5_
  
  - [ ]* 15.3 Write property tests for offline functionality
    - **Property 28: Offline Scheme Access**
    - **Property 29: Online Synchronization**
    - **Property 30: Offline Query Queueing**
    - Simulate offline mode, verify cached scheme access
    - Verify sync occurs on reconnection
    - Verify query queueing when offline
    - **Validates: Requirements 9.1, 9.2, 9.3, 9.5**

- [ ] 16. Implement feedback and improvement mechanisms
  - [~] 16.1 Create FeedbackManager class
    - Add rating mechanism for responses
    - Implement inaccuracy reporting with reference numbers
    - Store anonymous usage patterns
    - _Requirements: 10.1, 10.2, 10.4_
  
  - [ ]* 16.2 Write property tests for feedback mechanisms
    - **Property 31: Feedback Mechanism Availability**
    - **Property 32: Feedback Acknowledgment**
    - Verify feedback mechanisms present in all responses
    - Verify acknowledgment with reference number for reports
    - **Validates: Requirements 10.1, 10.2, 10.4**

- [ ] 17. Build API layer and main application
  - [~] 17.1 Create FastAPI application with endpoints
    - Implement `/query` endpoint for user queries
    - Add `/language` endpoint for language preference setting
    - Create `/feedback` endpoint for user feedback
    - Add `/health` endpoint for system monitoring
    - _Requirements: 1.4, 10.1, 10.2_
  
  - [~] 17.2 Wire all components together
    - Connect Language Detector → Query Parser → Knowledge Retrieval → Response Generator → Translation
    - Integrate Context Manager for session handling
    - Add error handling and logging throughout
    - _Requirements: All_
  
  - [ ]* 17.3 Write integration tests for API endpoints
    - Test complete query flow through API
    - Test language preference persistence
    - Test feedback submission
    - _Requirements: 1.4, 10.1, 10.2_

- [ ] 18. Add state-specific information handling
  - [~] 18.1 Implement state-level scheme prioritization
    - Modify search to prioritize state-specific schemes
    - Add state variation handling for schemes
    - _Requirements: 2.2, 2.5_
  
  - [ ]* 18.2 Write property test for state prioritization
    - **Property 8: State-Level Information Priority**
    - Generate queries for schemes with state variations
    - Verify state-specific version presented for user's state
    - **Validates: Requirements 2.5**

- [ ] 19. Implement language preference persistence
  - [~] 19.1 Add session-based language preference storage
    - Store language preference in session context
    - Apply preference to all subsequent queries
    - _Requirements: 1.4_
  
  - [ ]* 19.2 Write property test for language persistence
    - **Property 3: Language Preference Persistence**
    - Set language preference, submit queries in different languages
    - Verify all responses use preferred language
    - **Validates: Requirements 1.4**

- [~] 20. Final checkpoint and integration validation
  - Ensure all tests pass, ask the user if questions arise.
  - Run complete test suite including all property-based tests
  - Verify end-to-end flows for major use cases
  - Check test coverage across all components

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property-based tests validate universal correctness properties with minimum 100 iterations each
- Unit tests validate specific examples and edge cases
- Checkpoints ensure incremental validation throughout development
- Python was selected as the implementation language for its strong NLP library ecosystem and rapid development capabilities

# AI Scheme Eligibility & Explanation Assistant - Requirements Document

## Project Overview

**Title:** AI Scheme Eligibility & Explanation Assistant (Local-First MVP)

**Problem Statement:** Eligible citizens fail to access government welfare schemes due to lack of awareness, complex eligibility rules, bureaucratic language, and poor personalization.

**Solution:** A web-based assistant that uses deterministic rule-based logic for eligibility determination and AI exclusively for explanation simplification.

## Functional Requirements

### 1. User Input Collection
- **FR-001:** System SHALL collect exactly five user inputs:
  - `age` (number, required)
  - `occupation` (string, enum-based selection, required)
  - `annual_income` (number in INR, required)
  - `state` (string, single state for MVP, required)
  - `category` (string: General/OBC/SC/ST/EWS, required)

- **FR-002:** System SHALL validate all inputs before processing
- **FR-003:** System SHALL provide clear error messages for invalid inputs

### 2. Eligibility Determination Engine
- **FR-004:** System SHALL use ONLY deterministic, rule-based logic for eligibility decisions
- **FR-005:** System SHALL process eligibility for each scheme independently using structured rules
- **FR-006:** For each scheme, system SHALL check:
  - State match (exact)
  - Age range (min_age ≤ user_age ≤ max_age)
  - Occupation match (from allowed_occupations array)
  - Income threshold (annual_income ≤ max_income)
  - Category match (if scheme specifies allowed_categories)

- **FR-007:** System SHALL return for each scheme:
  - `eligible`: boolean
  - `matched_rules`: array of satisfied conditions
  - `failed_rules`: array of unsatisfied conditions

### 3. AI Explanation Layer
- **FR-008:** System SHALL use AI ONLY for explaining eligibility results in simple language
- **FR-009:** AI SHALL NOT participate in eligibility decision-making
- **FR-010:** System SHALL generate explanations covering:
  - Why user is eligible/ineligible for each scheme
  - Scheme benefits in plain language
  - Step-by-step application process
  - Required documents (if applicable)

### 4. Output Generation
- **FR-011:** System SHALL display results in three sections:
  - Eligible schemes with benefits and application steps
  - Ineligible schemes with reasons for rejection
  - Summary of user's profile matching

- **FR-012:** System SHALL present information in text-only format
- **FR-013:** System SHALL use simple, non-bureaucratic language in all outputs

### 5. Data Management
- **FR-014:** System SHALL store scheme data in structured JSON format
- **FR-015:** Each scheme record SHALL contain:
  ```json
  {
    "scheme_id": "string",
    "scheme_name": "string", 
    "state": "string",
    "eligibility_rules": {
      "min_age": "number",
      "max_age": "number", 
      "allowed_occupations": ["array"],
      "max_income": "number",
      "allowed_categories": ["array", "optional"]
    },
    "benefits": "string",
    "application_steps": ["array"]
  }
  ```

## Non-Functional Requirements

### Performance
- **NFR-001:** Eligibility processing SHALL complete within 2 seconds
- **NFR-002:** AI explanation generation SHALL complete within 5 seconds
- **NFR-003:** System SHALL handle concurrent users (minimum 10)

### Reliability
- **NFR-004:** System SHALL maintain 99% uptime during demo period
- **NFR-005:** System SHALL gracefully handle AI service failures with fallback messages

### Usability
- **NFR-006:** Interface SHALL be accessible via standard web browsers
- **NFR-007:** System SHALL provide clear progress indicators during processing
- **NFR-008:** All text SHALL be readable at 12pt font minimum

### Security
- **NFR-009:** System SHALL NOT store user input data persistently
- **NFR-010:** System SHALL validate and sanitize all user inputs

## Strict Constraints

### Architecture Constraints
- **C-001:** Eligibility logic and AI explanation layers MUST be completely separated
- **C-002:** NO AI involvement in eligibility decision-making
- **C-003:** ALL eligibility rules MUST be deterministic and auditable
- **C-004:** Scheme data MUST be stored as structured JSON (no databases)

### Scope Constraints  
- **C-005:** Maximum 5-7 government schemes only
- **C-006:** Single state OR single category coverage (not both)
- **C-007:** Text-only user interface (no graphics, charts, or media)
- **C-008:** No user authentication or login system
- **C-009:** No integration with real government APIs
- **C-010:** No web scraping of live data

### Technical Constraints
- **C-011:** Local-first architecture (minimal external dependencies)
- **C-012:** Web-based deployment only
- **C-013:** No mobile app development

## Explicit Out-of-Scope Items

- Multi-state scheme coverage
- Real-time government API integration
- User account management and data persistence
- Document upload and verification
- Payment processing or application submission
- Mobile application development
- Multi-language support
- Advanced UI/UX features (animations, responsive design)
- Scheme recommendation algorithms
- Historical application tracking
- Notification systems

## Success Criteria for MVP Demo

### Primary Success Criteria
1. **Functional Demo:** Complete user journey from input to explanation within 10 seconds
2. **Accuracy:** 100% correct eligibility determination for test cases
3. **Clarity:** AI explanations understandable to non-technical users
4. **Separation:** Clear demonstration that AI does not influence eligibility decisions

### Secondary Success Criteria
1. **Coverage:** All 5-7 schemes properly configured and testable
2. **Edge Cases:** Proper handling of boundary conditions (age limits, income thresholds)
3. **Error Handling:** Graceful degradation when AI service is unavailable
4. **Performance:** Sub-5-second total response time for complete workflow

## Assumptions and Dependencies

### Assumptions
- **A-001:** Users have basic web browser access
- **A-002:** Users can provide accurate personal information
- **A-003:** Scheme eligibility rules are publicly available and static
- **A-004:** AI service (OpenAI/similar) will be available during demo

### Dependencies
- **D-001:** Access to AI API service for explanation generation
- **D-002:** Web hosting platform for deployment
- **D-003:** Government scheme documentation for rule extraction
- **D-004:** Test user personas for validation

## Technical Architecture Requirements

### System Components
1. **Input Validation Module:** Form handling and data validation
2. **Eligibility Engine:** Pure rule-based logic processor  
3. **AI Interface Module:** Explanation generation wrapper
4. **Output Formatter:** Result presentation layer
5. **Scheme Data Store:** JSON-based configuration files

### Data Flow
```
User Input → Validation → Eligibility Engine → Results + AI Explanation → Output Display
```

### Integration Points
- AI service API (explanation only)
- Static JSON scheme database
- Web frontend interface

## Acceptance Criteria

The MVP will be considered complete when:
1. All functional requirements (FR-001 through FR-015) are implemented
2. All strict constraints (C-001 through C-013) are enforced
3. Primary success criteria are demonstrable
4. System processes at least 3 different user profiles correctly
5. AI explanations are generated without influencing eligibility decisions

---

**Document Version:** 1.0  
**Last Updated:** January 23, 2026  
**Approval Required:** Technical Lead, Product Owner
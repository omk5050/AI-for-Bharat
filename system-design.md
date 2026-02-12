# System Design Document - AI Scheme Eligibility Assistant

## 1. High-Level Architecture Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Web UI        │    │  Eligibility     │    │  Scheme Data    │
│   (HTML/JS)     │───▶│  Engine          │───▶│  Store (JSON)   │
│                 │    │  (Pure Logic)    │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │
         │                       ▼
         │              ┌──────────────────┐    ┌─────────────────┐
         │              │  Eligibility     │    │  AI Service     │
         └──────────────▶│  Results         │───▶│  (Explanation   │
                        │  Formatter       │    │   Only)         │
                        └──────────────────┘    └─────────────────┘
```

**Core Principle:** Strict separation between deterministic eligibility logic and AI explanation generation.

## 2. Component Breakdown

### 2.1 UI Layer
**Purpose:** Collect user input and display results
**Technology:** HTML forms + vanilla JavaScript
**Responsibilities:**
- Input validation (client-side)
- Form submission handling
- Results display formatting
- Error message presentation

### 2.2 Eligibility Engine
**Purpose:** Pure rule-based eligibility determination
**Technology:** JavaScript/Node.js
**Responsibilities:**
- Load scheme rules from JSON
- Apply deterministic logic to user inputs
- Generate structured eligibility results
- NO AI interaction whatsoever

### 2.3 Scheme Data Store
**Purpose:** Static storage of scheme rules and metadata
**Technology:** JSON files
**Structure:** One JSON file per scheme
**Responsibilities:**
- Store eligibility criteria
- Store scheme benefits and application steps
- Provide structured data to eligibility engine

### 2.4 Explanation Layer (AI)
**Purpose:** Convert technical results into plain language
**Technology:** OpenAI API or similar
**Responsibilities:**
- Receive eligibility results (not raw user data)
- Generate human-readable explanations
- Simplify bureaucratic language
- NO eligibility decision-making

## 3. Data Flow (Step-by-Step)

```
1. User Input Collection
   ├─ User fills form (age, occupation, income, state, category)
   ├─ Client-side validation
   └─ Submit to eligibility engine

2. Eligibility Processing
   ├─ Load all scheme JSON files
   ├─ For each scheme:
   │  ├─ Check state match
   │  ├─ Check age range
   │  ├─ Check occupation match
   │  ├─ Check income threshold
   │  └─ Check category (if applicable)
   └─ Generate structured results

3. AI Explanation Generation
   ├─ Send eligibility results to AI (NOT raw user data)
   ├─ AI generates plain language explanations
   └─ Return formatted explanations

4. Output Display
   ├─ Combine eligibility results + AI explanations
   ├─ Format for web display
   └─ Present to user
```

## 4. Eligibility Evaluation Algorithm

```pseudocode
FUNCTION evaluateEligibility(userProfile, schemes):
    results = []
    
    FOR each scheme in schemes:
        eligibilityResult = {
            scheme_id: scheme.scheme_id,
            scheme_name: scheme.scheme_name,
            eligible: false,
            matched_rules: [],
            failed_rules: [],
            benefits: scheme.benefits,
            application_steps: scheme.application_steps
        }
        
        // State check
        IF userProfile.state != scheme.state:
            eligibilityResult.failed_rules.push("state_mismatch")
            CONTINUE to next scheme
        ELSE:
            eligibilityResult.matched_rules.push("state_match")
        
        // Age check
        IF userProfile.age < scheme.eligibility_rules.min_age OR 
           userProfile.age > scheme.eligibility_rules.max_age:
            eligibilityResult.failed_rules.push("age_criteria")
        ELSE:
            eligibilityResult.matched_rules.push("age_criteria")
        
        // Occupation check
        IF userProfile.occupation NOT IN scheme.eligibility_rules.allowed_occupations:
            eligibilityResult.failed_rules.push("occupation_criteria")
        ELSE:
            eligibilityResult.matched_rules.push("occupation_criteria")
        
        // Income check
        IF userProfile.annual_income > scheme.eligibility_rules.max_income:
            eligibilityResult.failed_rules.push("income_criteria")
        ELSE:
            eligibilityResult.matched_rules.push("income_criteria")
        
        // Category check (if scheme specifies categories)
        IF scheme.eligibility_rules.allowed_categories EXISTS:
            IF userProfile.category NOT IN scheme.eligibility_rules.allowed_categories:
                eligibilityResult.failed_rules.push("category_criteria")
            ELSE:
                eligibilityResult.matched_rules.push("category_criteria")
        
        // Final eligibility determination
        IF eligibilityResult.failed_rules.length == 0:
            eligibilityResult.eligible = true
        
        results.push(eligibilityResult)
    
    RETURN results
```

## 5. AI Explanation Flow

### 5.1 Exact Inputs AI Receives
```json
{
  "user_profile_summary": {
    "age_range": "25-30",
    "occupation_category": "farmer", 
    "income_bracket": "below_2_lakh",
    "state": "maharashtra",
    "category": "general"
  },
  "eligibility_results": [
    {
      "scheme_name": "PM-KISAN",
      "eligible": true,
      "matched_criteria": ["age", "occupation", "income"],
      "benefits": "₹6000 per year direct transfer",
      "application_steps": ["Visit CSC", "Submit documents", "Wait for verification"]
    }
  ]
}
```

### 5.2 What AI is NOT Allowed to Do
- **NEVER** make eligibility decisions
- **NEVER** modify eligibility criteria
- **NEVER** access raw user personal data
- **NEVER** suggest scheme modifications
- **NEVER** provide legal advice
- **ONLY** explain pre-determined results in simple language

### 5.3 AI Prompt Template
```
You are explaining government scheme eligibility results. 
RULES:
- Only explain the provided results, never change eligibility decisions
- Use simple, non-bureaucratic language
- Focus on benefits and next steps for eligible schemes
- For ineligible schemes, clearly explain why without suggesting workarounds

Results to explain: {eligibility_results}
User profile: {user_profile_summary}
```

## 6. Example Scheme JSON Structure

```json
{
  "scheme_id": "pm_kisan_2024",
  "scheme_name": "PM-KISAN Samman Nidhi",
  "state": "maharashtra",
  "eligibility_rules": {
    "min_age": 18,
    "max_age": 70,
    "allowed_occupations": ["farmer", "agricultural_worker"],
    "max_income": 200000,
    "allowed_categories": ["general", "obc", "sc", "st"]
  },
  "benefits": "₹6,000 per year in three equal installments of ₹2,000 each, directly transferred to bank account",
  "application_steps": [
    "Visit nearest Common Service Center (CSC)",
    "Carry Aadhaar card, bank passbook, and land documents",
    "Fill PM-KISAN application form",
    "Submit documents for verification",
    "Receive SMS confirmation within 7 days"
  ]
}
```

## 7. Example Eligibility Evaluation Output

```json
{
  "user_profile": {
    "age": 35,
    "occupation": "farmer",
    "annual_income": 150000,
    "state": "maharashtra",
    "category": "general"
  },
  "evaluation_results": [
    {
      "scheme_id": "pm_kisan_2024",
      "scheme_name": "PM-KISAN Samman Nidhi",
      "eligible": true,
      "matched_rules": ["state_match", "age_criteria", "occupation_criteria", "income_criteria", "category_criteria"],
      "failed_rules": [],
      "benefits": "₹6,000 per year in three equal installments",
      "application_steps": ["Visit CSC", "Submit documents", "Wait for verification"],
      "ai_explanation": "Great news! You qualify for PM-KISAN because you're a farmer in Maharashtra under 70 years old with annual income below ₹2 lakh. You'll receive ₹6,000 yearly in your bank account."
    }
  ],
  "summary": {
    "total_schemes_checked": 5,
    "eligible_schemes": 1,
    "ineligible_schemes": 4
  }
}
```

## 8. Error Handling and Edge Cases

### 8.1 Input Validation Errors
- **Missing fields:** Clear field-specific error messages
- **Invalid age:** "Age must be between 0 and 120"
- **Invalid income:** "Income must be a positive number"
- **Invalid occupation:** Show dropdown with valid options

### 8.2 System Errors
- **JSON file corruption:** Graceful degradation, skip corrupted schemes
- **AI service unavailable:** Show eligibility results without explanations
- **Network timeout:** Retry mechanism with fallback messages

### 8.3 Edge Cases
- **Age boundary conditions:** Inclusive range checking (18 ≤ age ≤ 70)
- **Income exactly at threshold:** Use ≤ comparison (inclusive)
- **Multiple occupation matches:** User selects primary occupation
- **Scheme with no category restrictions:** Skip category check entirely

## 9. Explicit Non-Goals

### What This System Will NOT Do
- **Real-time data sync** with government databases
- **Document verification** or upload processing  
- **Application submission** to actual government portals
- **User account management** or data persistence
- **Multi-language support** beyond English/Hindi
- **Mobile app development** or responsive design
- **Payment processing** or financial transactions
- **Legal advice** or official eligibility guarantees
- **Scheme recommendation algorithms** based on user behavior
- **Integration with external APIs** beyond AI explanation service

## 10. Design Justification for Hackathon Constraints

### Why This Architecture Works for MVP Demo

**Simplicity Over Scalability**
- JSON files instead of databases: Faster setup, easier debugging
- Vanilla JavaScript: No framework learning curve, immediate productivity
- Monolithic structure: Easier to demo, fewer moving parts

**Clear Separation of Concerns**
- Eligibility engine isolation: Easy to audit and verify correctness
- AI explanation wrapper: Can be disabled without breaking core functionality
- Static data store: Predictable behavior, no database setup required

**Demo-Friendly Features**
- Fast local execution: No network dependencies for core logic
- Visible data flow: Easy to trace decisions during presentation
- Error resilience: System works even if AI service fails
- Minimal dependencies: Reduces setup complexity for judges

**Hackathon Time Constraints**
- No authentication: Saves 2-3 hours of development time
- Single state focus: Reduces data preparation effort
- Text-only UI: Faster development, focuses on logic over aesthetics
- Limited scheme count: Allows thorough testing of each scheme

This design prioritizes demonstrable functionality and clear logic flow over enterprise patterns, making it ideal for hackathon evaluation and rapid development.
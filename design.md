# Design Document: AI-Powered Document Assistant

## 1. System Architecture Overview

### 1.1 High-Level Architecture
The AI-Powered Document Assistant follows a mobile-first, cloud-hybrid architecture with offline-first capabilities:

```
┌─────────────────────────────────────────────────────────────┐
│                    Mobile Application Layer                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Glassmorphism│  │ Voice-First  │  │  Multimodal  │      │
│  │   UI Layer   │  │  Interface   │  │ Input Handler│      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                   Local Processing Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Offline OCR  │  │ Local Cache  │  │ Encryption   │      │
│  │   Engine     │  │   Manager    │  │   Module     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    Cloud Services Layer                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  AI Engine   │  │  RAG System  │  │    Scam      │      │
│  │   (LLM)      │  │ (Knowledge)  │  │  Detector    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Technology Stack

**Mobile Frontend:**
- Framework: React Native / Flutter
- UI Library: Custom Glassmorphism components
- State Management: Redux / MobX
- Local Storage: SQLite + Encrypted File System
- Offline Support: Service Workers / Background Sync

**Backend Services:**
- API Gateway: Node.js / FastAPI
- LLM Integration: OpenAI GPT-4 / Anthropic Claude / Google Gemini
- OCR: Google Cloud Vision API / Tesseract (offline)
- Speech Services: Google Cloud Speech-to-Text/Text-to-Speech
- Vector Database: Pinecone / Weaviate (for RAG)

**Data Storage:**
- Primary Database: PostgreSQL (user data, document metadata)
- Document Storage: AWS S3 / Google Cloud Storage (encrypted)
- Cache Layer: Redis
- Knowledge Base: Vector embeddings + structured data

**Security:**
- Encryption: AES-256 for data at rest, TLS 1.3 for transit
- Authentication: JWT + Biometric (fingerprint/face)
- Compliance: Indian data residency requirements

## 2. Core Component Design

### 2.1 Multimodal Input Handler

**Component Responsibilities:**
- Accept and normalize inputs from multiple sources
- Route inputs to appropriate processing pipelines
- Handle input validation and error recovery

**Input Processing Pipeline:**
```
User Input → Input Validator → Format Normalizer → Content Extractor → AI Engine
```

**Supported Input Types:**
1. **Camera Capture**: Edge detection → Image enhancement → OCR
2. **File Upload**: Format detection → Conversion → Text extraction
3. **Voice Input**: Speech-to-text → Language detection → Intent parsing
4. **Text Input**: Direct processing → Language detection

**Design Decisions:**
- Use on-device ML for edge detection to reduce latency
- Implement progressive image enhancement (quick preview → full quality)
- Support batch processing for multi-page documents
- Queue inputs when offline, process when online

### 2.2 OCR Engine

**Architecture:**
- **Online Mode**: Google Cloud Vision API (high accuracy)
- **Offline Mode**: Tesseract OCR (local processing)
- **Hybrid Mode**: Quick local scan → Cloud refinement

**Processing Steps:**
1. Image preprocessing (deskew, denoise, contrast enhancement)
2. Text region detection
3. Character recognition
4. Post-processing (spell check, format correction)
5. Confidence scoring

**Optimization:**
- Cache OCR results locally
- Use adaptive quality based on network speed
- Implement progressive OCR (quick scan → detailed scan)

### 2.3 AI Engine (Document Analysis)

**Core Architecture:**

```
Document Text → Classifier → Type-Specific Analyzer → Fact Verifier → Output Generator
```

**Stage 1: Document Classification**
- **Model**: Fine-tuned LLM classifier
- **Input**: Extracted text + metadata
- **Output**: Document type + confidence score
- **Fallback**: If confidence < 80%, ask user for clarification

**Document Type Taxonomy:**
```
Legal Documents
├── Court Summons
├── Legal Notices
└── Property Documents

Financial Documents
├── Bank Notices
├── Loan Agreements
├── Insurance Policies
└── Tax Notices

Commercial Documents
├── Terms & Conditions
├── Privacy Policies
└── Employment Contracts

Government Documents
├── Government Schemes
├── Tax Notices
└── Educational Certificates
```

**Stage 2: Type-Specific Analysis**

Each document type has a specialized analysis template:

**Example: Bank Notice Analyzer**
```python
class BankNoticeAnalyzer:
    def analyze(self, document_text):
        return {
            "deadline": extract_deadline(document_text),
            "penalty_amount": extract_monetary_values(document_text),
            "action_required": identify_required_actions(document_text),
            "defense_options": query_rag_system("bank notice defense"),
            "escalation_path": get_escalation_hierarchy(),
            "urgency_score": calculate_urgency(deadline, penalty)
        }
```

**Example: Legal Summons Analyzer**
```python
class LegalSummonsAnalyzer:
    def analyze(self, document_text):
        return {
            "case_type": classify_legal_case(document_text),
            "severity": assess_severity(case_type),
            "court_details": extract_court_info(document_text),
            "legal_rights": query_rag_system(f"IPC rights {case_type}"),
            "lawyer_required": assess_lawyer_necessity(severity),
            "legal_aid_info": get_legal_aid_options(user_location)
        }
```

**Stage 3: Emotion & Tone Detection**
- Analyze language patterns for threatening/urgent/confusing content
- Score: 🔴 Threatening (8-10) | 🟡 Confusing (4-7) | 🟢 Informational (1-3)
- Adjust response tone based on detected emotion

### 2.4 RAG System (Fact Verification)

**Architecture:**
```
Document Claim → Embedding Generator → Vector Search → Knowledge Retrieval → Verification Logic → Result
```

**Knowledge Base Structure:**

```
Knowledge_Base
├── Indian_Laws
│   ├── IPC (Indian Penal Code)
│   ├── Consumer_Protection_Act
│   ├── IT_Act
│   └── Civil_Procedure_Code
├── Regulatory_Guidelines
│   ├── RBI_Guidelines
│   ├── SEBI_Guidelines
│   └── IRDAI_Guidelines
├── Case_Law
│   ├── Supreme_Court_Judgments
│   └── High_Court_Precedents
├── Government_Schemes
│   └── Eligibility_Rules
└── Standard_Practices
    └── Industry_Benchmarks
```

**Verification Process:**
1. Extract verifiable claims from document
2. Generate embeddings for each claim
3. Search vector database for relevant legal/regulatory content
4. Compare claim against retrieved knowledge
5. Generate verification result with legal reference

**Example Verification:**
```
Claim: "Non-payment will result in immediate arrest"
→ Search: "civil debt arrest India"
→ Retrieved: IPC Section 87 - Civil debt cannot lead to arrest
→ Result: ❌ FALSE - Civil debt cannot lead to arrest (IPC Section 87)
```

### 2.5 Scam Detector

**Multi-Layer Detection System:**

**Layer 1: Sender Verification**
- Check email domain against official database
- Verify phone numbers against registered databases
- Flag mismatches (e.g., rbi_india@gmail.com vs @rbi.org.in)

**Layer 2: Content Analysis**
- Detect urgent/threatening language patterns
- Identify requests for sensitive information (OTP, password, PIN)
- Check for grammatical errors and poor language quality
- Analyze tone consistency

**Layer 3: Link Analysis**
- Extract and analyze URLs
- Check against phishing databases
- Verify domain authenticity
- Flag shortened URLs

**Layer 4: Pattern Matching**
- Compare against known scam templates
- Check for common fraud phrases
- Analyze monetary request patterns

**Scam Score Calculation:**
```
Scam_Score = (
    sender_verification_score * 0.3 +
    content_analysis_score * 0.3 +
    link_analysis_score * 0.2 +
    pattern_matching_score * 0.2
)

If Scam_Score > 0.7: Display SCAM ALERT
```

### 2.6 Voice Interface System

**Architecture:**
```
User Speech → STT Engine → Language Detection → AI Processing → TTS Engine → Audio Output
```

**Speech-to-Text (STT):**
- Primary: Google Cloud Speech-to-Text
- Fallback: Azure Speech Services
- Offline: On-device speech recognition (limited)
- Supported: All 22 Indian languages + dialects

**Language Detection:**
- Auto-detect from first 3-5 seconds of speech
- Support code-mixing (Hinglish, Tanglish, etc.)
- Allow manual language override

**Text-to-Speech (TTS):**
- Use native speaker voice models for each language
- Adjust pace based on content complexity
- Add pauses for emphasis
- Support emotional tone modulation

**Voice-Only Mode Features:**

- Complete workflow without visual interaction
- Audio feedback for all actions
- Voice commands for navigation
- Haptic feedback for confirmation

**Conversation Flow Example:**
```
User: "Mujhe bank se ek notice aaya hai"
AI: "Main samajh gaya. Aap notice ka photo upload karna chahte hain ya text paste karna chahte hain?"
User: "Photo upload karunga"
AI: "Theek hai. Camera khol raha hun. Jab ready ho, screen pe tap karein."
[Photo captured]
AI: "Photo mil gayi. Processing kar raha hun... Kripya 10 second rukein."
```

### 2.7 Action Generator

**Context-Aware Action Planning:**

**Scenario Detection:**
```python
def detect_scenario(document_analysis):
    if document_analysis.requires_reply:
        return "REPLY_REQUIRED"
    elif document_analysis.urgency_score >= 8:
        return "LEGAL_EMERGENCY"
    else:
        return "AWARENESS_ONLY"
```

**Action Plan Templates:**

**1. Reply Required Scenario:**
```
Immediate Actions (0-2 hours):
├── Generate reply letter with legal references
├── Provide email/physical sending instructions
└── Set 2-hour reminder

Follow-up Actions (3 days):
├── Check for response
├── Escalate to ombudsman if needed
└── Prepare complaint draft

Backup Plan:
├── Consumer forum complaint template
└── Legal aid contact information
```

**2. Legal Emergency Scenario:**
```
Urgent Checklist:
├── Court date countdown
├── Lawyer directory (verified)
├── Free legal aid eligibility check
├── Document collection checklist
├── Court appearance preparation
└── Multiple reminders (3 days, 1 day, morning)
```

**3. Awareness Only Scenario:**
```
Summary Report:
├── Key points to remember
├── Settings to update
├── Calendar reminders for future dates
└── PDF summary generation
```

**Letter Generation System:**
- Use legal templates with placeholders
- Insert user-specific information
- Add relevant legal references
- Format professionally
- Provide both English and native language versions

### 2.8 Glassmorphism UI Design

**Design Principles:**
- Frosted glass effect with backdrop blur
- Translucent layers with subtle shadows
- Vibrant gradient backgrounds
- Smooth animations and transitions
- Touch-friendly large buttons

**Color Coding System:**
- 🔴 Red: Critical/Urgent (deadlines, penalties, threats)
- 🟡 Yellow: Warning/Caution (hidden clauses, confusing terms)
- 🟢 Green: Positive/Rights (user benefits, legal protections)
- 🔵 Blue: Informational (general content)

**Component Library:**

```
UI Components:
├── GlassCard (document summary cards)
├── GlassButton (action buttons)
├── GlassModal (dialogs and popups)
├── GlassInput (text input fields)
├── GlassProgress (loading indicators)
└── GlassAvatar (AI assistant representation)
```

**Screen Layouts:**

**Home Screen:**
- Large "Upload Document" button (center)
- Voice mode toggle (top-right)
- Recent documents (scrollable list)
- Pending actions badge

**Analysis Screen:**
- Document type header
- Urgency indicator
- Tabbed sections: Risks | Hidden Clauses | Your Rights | Actions
- Voice explanation button (floating)
- Q&A chat interface (bottom)

**Voice-Only Screen:**
- Large AI avatar (animated)
- Waveform visualization
- Minimal text (large font)
- Tap-to-speak button

## 3. Data Models

### 3.1 Document Model
```typescript
interface Document {
  id: string;
  userId: string;
  uploadedAt: Date;
  processedAt: Date;
  
  // Input data
  inputType: 'camera' | 'file' | 'text' | 'voice';
  originalFile?: File;
  extractedText: string;
  
  // Classification
  documentType: DocumentType;
  classificationConfidence: number;
  
  // Analysis results
  analysis: DocumentAnalysis;
  urgencyScore: number;
  emotionTone: 'threatening' | 'confusing' | 'informational';
  
  // Verification
  verifiedClaims: VerifiedClaim[];
  scamScore: number;
  isScam: boolean;
  
  // Actions
  actionPlan: ActionPlan;
  reminders: Reminder[];
  
  // Status
  status: 'pending' | 'completed' | 'archived';
  userFeedback?: Feedback;
}
```

### 3.2 Document Analysis Model
```typescript
interface DocumentAnalysis {
  documentType: DocumentType;
  
  // Common fields
  deadline?: Date;
  keyDates: Date[];
  monetaryAmounts: MonetaryValue[];
  parties: Party[];
  
  // Type-specific fields
  bankNotice?: BankNoticeAnalysis;
  legalSummons?: LegalSummonsAnalysis;
  termsAndConditions?: TermsAnalysis;
  insurancePolicy?: InsuranceAnalysis;
  
  // Extracted insights
  criticalRisks: string[];
  hiddenClauses: string[];
  userRights: string[];
  expertTips: string[];
}
```

### 3.3 Action Plan Model
```typescript
interface ActionPlan {
  scenario: 'REPLY_REQUIRED' | 'LEGAL_EMERGENCY' | 'AWARENESS_ONLY';
  
  immediateActions: Action[];
  followUpActions: Action[];
  backupPlan?: Action[];
  
  generatedLetters: GeneratedLetter[];
  reminders: Reminder[];
  resources: Resource[];
}

interface Action {
  id: string;
  title: string;
  description: string;
  deadline?: Date;
  priority: 'high' | 'medium' | 'low';
  status: 'pending' | 'completed';
  instructions: string[];
}
```

### 3.4 User Model
```typescript
interface User {
  id: string;
  phoneNumber: string;
  email?: string;
  
  // Preferences
  preferredLanguage: string;
  voiceModeEnabled: boolean;
  notificationPreferences: NotificationPreferences;
  
  // Location (for legal aid)
  state: string;
  district?: string;
  
  // Usage stats
  documentsProcessed: number;
  deadlinesMet: number;
  moneySaved: number;
  
  // Security
  biometricEnabled: boolean;
  lastLogin: Date;
}
```

## 4. API Design

### 4.1 Document Processing API

**POST /api/v1/documents/upload**
```json
Request:
{
  "inputType": "camera",
  "file": "<base64_encoded_image>",
  "language": "hi"
}

Response:
{
  "documentId": "doc_123",
  "status": "processing",
  "estimatedTime": 15
}
```

**GET /api/v1/documents/{documentId}/analysis**
```json
Response:
{
  "documentId": "doc_123",
  "documentType": "bank_notice",
  "urgencyScore": 8,
  "analysis": {
    "deadline": "2026-02-21T23:59:59Z",
    "penaltyAmount": 25000,
    "criticalRisks": [...],
    "hiddenClauses": [...],
    "userRights": [...]
  },
  "actionPlan": {...},
  "scamScore": 0.1,
  "isScam": false
}
```

### 4.2 Voice Interface API

**POST /api/v1/voice/transcribe**
```json
Request:
{
  "audio": "<base64_encoded_audio>",
  "format": "wav"
}

Response:
{
  "text": "Mujhe bank se ek notice aaya hai",
  "detectedLanguage": "hi",
  "confidence": 0.95
}
```

**POST /api/v1/voice/synthesize**
```json
Request:
{
  "text": "आपका document analyze हो गया है",
  "language": "hi",
  "emotionalTone": "reassuring"
}

Response:
{
  "audio": "<base64_encoded_audio>",
  "duration": 3.5
}
```

### 4.3 RAG Verification API

**POST /api/v1/verify/claim**
```json
Request:
{
  "claim": "Non-payment will result in immediate arrest",
  "context": "bank loan notice"
}

Response:
{
  "isVerified": false,
  "verdict": "FALSE",
  "explanation": "Civil debt cannot lead to arrest under IPC Section 87",
  "legalReferences": [
    {
      "source": "IPC",
      "section": "87",
      "text": "..."
    }
  ]
}
```

## 5. Security Architecture

### 5.1 Data Encryption

**At Rest:**
- AES-256 encryption for all stored documents
- Encrypted SQLite database for local storage
- Secure keychain for encryption keys

**In Transit:**
- TLS 1.3 for all API communications
- Certificate pinning to prevent MITM attacks
- End-to-end encryption for sensitive documents

### 5.2 Authentication & Authorization

**Authentication Flow:**
```
1. User opens app
2. Biometric authentication (fingerprint/face)
3. Generate JWT token
4. Token refresh every 24 hours
5. Auto-lock after 5 minutes of inactivity
```

**Authorization Levels:**
- User can only access their own documents
- Admin can access aggregated analytics (anonymized)
- No third-party access to user documents

### 5.3 Privacy Measures

- Documents deleted from server after 30 days (user configurable)
- Option for local-only processing (no cloud upload)
- No document retention for AI training without explicit consent
- Anonymized analytics only
- Compliance with Indian data protection laws

## 6. Offline Architecture

### 6.1 Offline Capabilities

**Available Offline:**
- OCR processing (Tesseract)
- Basic document classification (on-device ML model)
- Key information extraction (dates, amounts)
- Document history viewing
- Previously analyzed documents

**Unavailable Offline:**
- Advanced AI analysis
- RAG fact verification
- Scam detection
- Voice synthesis (limited)
- Action plan generation

### 6.2 Sync Strategy

**When Online:**
```
1. Upload queued documents
2. Fetch enhanced analysis
3. Update knowledge base
4. Sync reminders and notifications
5. Download voice models (if needed)
```

**Conflict Resolution:**
- Server data takes precedence for analysis
- Local data preserved for user edits
- Merge strategy for document metadata

## 7. Performance Optimization

### 7.1 Response Time Targets

- Document upload acknowledgment: < 2 seconds
- OCR completion: < 5 seconds per page
- Initial AI analysis: < 15 seconds
- Voice transcription: < 2 seconds
- Voice synthesis: < 1 second
- App launch: < 3 seconds

### 7.2 Optimization Strategies

**Frontend:**
- Lazy loading for document history
- Image compression before upload
- Progressive rendering of analysis results
- Caching of frequently accessed data

**Backend:**
- LLM response streaming for faster perceived performance
- Parallel processing of document analysis stages
- Redis caching for common queries
- CDN for static assets

**Mobile:**
- On-device ML models for quick classification
- Background processing for non-urgent tasks
- Efficient battery usage (limit background sync)

## 8. Scalability Design

### 8.1 Horizontal Scaling

**API Servers:**
- Load balancer distributing requests
- Auto-scaling based on traffic
- Stateless design for easy scaling

**AI Processing:**
- Queue-based processing (RabbitMQ/Kafka)
- Worker pool for parallel document analysis
- GPU instances for ML inference

**Database:**
- Read replicas for query distribution
- Sharding by user ID
- Caching layer (Redis) for hot data

### 8.2 Capacity Planning

**Target Scale:**
- 1M+ concurrent users
- 10M+ documents processed daily
- 100K+ voice interactions per hour
- 99.9% uptime SLA

## 9. Monitoring & Analytics

### 9.1 System Metrics

- API response times
- Error rates by endpoint
- OCR accuracy rates
- AI classification accuracy
- Scam detection accuracy
- User engagement metrics

### 9.2 User Analytics

- Documents processed per user
- Most common document types
- Feature usage (voice mode, Q&A, etc.)
- User satisfaction scores
- Money saved estimates

## 10. Testing Strategy

### 10.1 Unit Testing

- Component-level tests for UI
- API endpoint tests
- ML model accuracy tests
- Encryption/decryption tests

### 10.2 Integration Testing

- End-to-end document processing flow
- Voice interface workflow
- Offline-to-online sync
- Multi-language support

### 10.3 User Acceptance Testing

- Usability testing with target users
- Voice-only mode testing with illiterate users
- Multi-language testing with native speakers
- Accessibility testing

## 11. Deployment Strategy

### 11.1 Mobile App Deployment

**iOS:**
- TestFlight for beta testing
- App Store submission
- Phased rollout (10% → 50% → 100%)

**Android:**
- Google Play Console
- Internal testing → Closed testing → Open testing
- Staged rollout

### 11.2 Backend Deployment

- Blue-green deployment for zero downtime
- Canary releases for new features
- Automated rollback on errors
- Database migration strategy

## 12. Compliance & Legal

### 12.1 Data Residency

- All user data stored in Indian data centers
- Compliance with Indian data protection laws
- GDPR-like data deletion rights

### 12.2 Disclaimers

- Clear disclaimer: "This is information, not legal advice"
- Encourage users to consult lawyers for serious matters
- Accuracy limitations disclosure

### 12.3 Content Moderation

- Filter inappropriate content
- Report suspicious documents to authorities (with user consent)
- Abuse prevention mechanisms

## 13. Future Enhancements

### 13.1 Phase 2 Features

- Document comparison tool
- Template library for common responses
- Community-sourced document reviews
- Integration with government portals (DigiLocker)
- Automated form filling

### 13.2 Phase 3 Features

- AI-powered negotiation assistance
- Direct lawyer consultation booking
- Family account sharing
- Business document support
- Blockchain-based document verification

## 14. Correctness Properties

### Property 1: Document Classification Accuracy
**Property:** For any uploaded document, the system SHALL classify it with at least 90% accuracy when compared to human expert classification.

**Validation:** Compare AI classification against expert-labeled test dataset of 1000+ documents across all types.

### Property 2: Deadline Extraction Correctness
**Property:** For any document containing a deadline, the system SHALL extract the correct date with 100% accuracy (no false negatives for critical deadlines).

**Validation:** Test with documents containing various date formats (DD/MM/YYYY, written dates, relative dates like "within 7 days").

### Property 3: Scam Detection Precision
**Property:** When the system flags a document as a scam (scam_score > 0.7), it SHALL be a genuine scam with at least 95% precision (minimize false positives).

**Validation:** Test with known scam dataset and legitimate document dataset, measure false positive rate.

### Property 4: Language Consistency
**Property:** For any user interaction, if the user inputs in language L, the system SHALL respond in the same language L throughout the session.

**Validation:** Test conversation flows in each supported language, verify no language switching occurs.

### Property 5: Offline-Online Sync Integrity
**Property:** When transitioning from offline to online mode, all queued documents SHALL be processed without data loss or corruption.

**Validation:** Simulate offline document uploads, verify all documents processed correctly after reconnection.

### Property 6: Encryption Integrity
**Property:** All documents stored locally or transmitted SHALL be encrypted, and decryption SHALL only be possible with valid user authentication.

**Validation:** Attempt to access encrypted files without authentication, verify access denied.

### Property 7: Voice Transcription Accuracy
**Property:** For clear audio input in any supported language, the system SHALL achieve at least 90% word-level transcription accuracy.

**Validation:** Test with native speaker audio samples across all supported languages.

### Property 8: Action Plan Completeness
**Property:** For any document requiring action, the system SHALL generate an action plan containing at minimum: immediate actions, timeline, and contact information.

**Validation:** Verify all action plans contain required fields for documents needing replies.

### Property 9: RAG Verification Accuracy
**Property:** When verifying legal claims, the system SHALL provide correct verification (true/false) with legal references in at least 95% of cases.

**Validation:** Test against known legal claims with verified ground truth.

### Property 10: Response Time Guarantee
**Property:** For any document under 10 pages, the system SHALL provide initial analysis results within 20 seconds of upload completion.

**Validation:** Measure end-to-end processing time for various document sizes and types.

---

## 15. Implementation Notes

### 15.1 Development Phases

**Phase 1 (MVP - 3 months):**
- Core document upload and OCR
- Basic AI classification and analysis
- Simple UI (non-glassmorphism)
- English and Hindi support
- Online-only mode

**Phase 2 (Full Features - 3 months):**
- Glassmorphism UI
- Voice interface
- All 22 languages
- RAG fact verification
- Scam detection
- Action generation

**Phase 3 (Optimization - 2 months):**
- Offline mode
- Performance optimization
- Advanced analytics
- User feedback integration

### 15.2 Team Structure

- 2 Mobile Developers (React Native/Flutter)
- 2 Backend Developers (Node.js/Python)
- 1 ML Engineer (LLM integration, RAG)
- 1 UI/UX Designer (Glassmorphism specialist)
- 1 QA Engineer
- 1 DevOps Engineer
- 1 Product Manager

### 15.3 Third-Party Dependencies

- OpenAI/Anthropic/Google (LLM)
- Google Cloud Vision (OCR)
- Google Cloud Speech (STT/TTS)
- Pinecone/Weaviate (Vector DB)
- AWS/GCP (Infrastructure)
- Twilio (SMS notifications)

---

**Document Version:** 1.0  
**Last Updated:** February 14, 2026  
**Status:** Ready for Implementation

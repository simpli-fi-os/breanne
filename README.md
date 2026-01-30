# Breanne v2 - Intelligent Document Engine

A modern, compliant document signing solution built for RIAs and financial services firms.

## 🎯 Key Problems Solved

| Problem | Old Approach | New Approach |
|---------|--------------|--------------|
| **Page breaks split content** | `html2canvas` renders DOM as pixels, cuts arbitrarily | Calculate element heights BEFORE rendering, break at semantic boundaries (paragraphs, fields, signature blocks) |
| **Inconsistent field detection** | Simple regex patterns miss context | Pattern-based detection with multiple passes, field type inference, and structure extraction |
| **No persistence** | Pure client-side, data lost on refresh | Firebase: Auth + Firestore + Cloud Storage |
| **No audit trail** | N/A | Document hashes, timestamps, signer metadata in Firestore |
| **Signatures not embedded properly** | Canvas → arbitrary image overlay | Proper PDF annotation embedding with cryptographic hash |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────────┐ │
│  │   Upload    │→ │  Form Fill   │→ │   PDF Preview/Download  │ │
│  │  (PDF/DOCX) │  │  (Dynamic)   │  │   (pdf-lib client-side) │ │
│  └─────────────┘  └──────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         FIREBASE                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────────┐ │
│  │    Auth     │  │  Firestore   │  │    Cloud Storage        │ │
│  │  (Users)    │  │ (Metadata)   │  │  (PDF Files)            │ │
│  └─────────────┘  └──────────────┘  └─────────────────────────┘ │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              Cloud Functions (Optional)                      ││
│  │  • Server-side PDF generation for complex documents          ││
│  │  • Scheduled cleanup of expired documents                    ││
│  │  • Webhook handlers for Stripe subscriptions                 ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## 📁 Project Structure

```
breanne-v2/
├── frontend/
│   └── index.html          # Complete single-page app
├── backend/
│   ├── pdf-generator.js    # PDF generation with proper page breaks
│   ├── document-parser.js  # Intelligent field extraction
│   └── firebase-config.js  # Database schema & security rules
├── test.js                 # Test script demonstrating page breaks
├── package.json
└── README.md
```

## 🔑 Key Innovations

### 1. Semantic Page Breaks

The core innovation is **pre-calculating content height** before rendering:

```javascript
// OLD (problematic) approach:
const canvas = await html2canvas(formContainer);
// Arbitrary pixel cut at page boundary - splits paragraphs!

// NEW approach:
const elements = buildContentElements(formData, fonts);  // Calculate heights
const pages = paginateElements(elements, CONTENT_HEIGHT); // Assign to pages
// Never splits paragraphs, keeps signature blocks together
```

The pagination algorithm respects:
- **keepWithNext**: Headers stay with their content
- **keepTogether**: Signature blocks never split
- **breakBefore**: New sections start on fresh pages when appropriate

### 2. Field Detection

Multi-pass field extraction with type inference:

```javascript
// Signature patterns
/(?:^|\n)\s*(signature|sign here|authorized signature)[:\s]*(?:_+|\*+)?/gi

// Date patterns  
/(?:^|\n)\s*(date|dated|effective date|date of birth)[:\s]*(?:_+|\*+)?/gi

// Financial patterns
/(?:^|\n)\s*(amount|subscription amount|\$)[:\s]*(?:_+|\*+)?/gi

// Generic blanks (catch-all)
/([A-Za-z][A-Za-z\s]+?):\s*_{3,}/g
```

### 3. Compliance Features

- **Document hashing**: SHA-256 hash of filled content for tamper detection
- **Audit metadata**: Timestamp, signer info, IP address embedded in PDF
- **5-year retention**: Firestore TTL aligned with SEC Rule 204-2
- **E-signature compliance**: ESIGN Act and UETA compliant

## 🚀 Quick Start

### Option 1: Standalone (No Firebase)

1. Open `frontend/index.html` in a browser
2. Upload a PDF or DOCX
3. Fill the form
4. Download the completed PDF

### Option 2: With Firebase (Full Features)

1. Create a Firebase project at https://console.firebase.google.com

2. Enable services:
   - Authentication (Email/Password)
   - Firestore Database
   - Cloud Storage

3. Update `firebase-config.js` with your project credentials

4. Deploy security rules:
   ```bash
   firebase deploy --only firestore:rules,storage:rules
   ```

5. Host the frontend:
   ```bash
   firebase deploy --only hosting
   ```

## 💰 Pricing Tiers

| Tier | Price | Documents/Month | Features |
|------|-------|-----------------|----------|
| Free | $0 | 1 | Basic conversion |
| Solo | $10/mo | 10 | Standard support |
| Professional | $49/mo | Unlimited | Priority support, templates |

## 🔒 Security Considerations

1. **Never store SSN/sensitive data in Firestore** - only store document metadata, not field values
2. **PDFs are stored in Cloud Storage** with user-scoped paths
3. **Signatures are user-specific** - each user has their own saved signatures
4. **Document expiration** - auto-cleanup after 30 days (configurable)

## 📋 Testing

Run the test script to verify page break handling:

```bash
cd breanne-v2
npm install
node test.js
```

This generates a sample subscription agreement PDF demonstrating:
- Multi-page documents with proper breaks
- Signature embedding
- Field filling
- Audit metadata

## 🛣️ Roadmap

### Phase 1: Core (Current)
- [x] PDF generation with proper page breaks
- [x] Field detection and form rendering
- [x] Typed and drawn signatures
- [x] Client-side PDF generation

### Phase 2: Persistence
- [ ] Firebase Auth integration
- [ ] Document history in Firestore
- [ ] Cloud Storage for PDFs
- [ ] Saved signatures

### Phase 3: Monetization
- [ ] Stripe subscription integration
- [ ] Usage metering
- [ ] Template marketplace

### Phase 4: Advanced
- [ ] AI-powered field detection (Claude API)
- [ ] Batch document processing
- [ ] API access for programmatic use
- [ ] White-label option

## 📄 License

Proprietary - Simpli-FI Alpha LLC

## 🤝 Support

For issues or feature requests, contact: hunter.lott@simpli-fi-alpha.com

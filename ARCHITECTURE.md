# nano4HR Technical Architecture - 2026 Modernization

## Overview

nano4HR is being modernized to a **React 19 + TypeScript** progressive web application with offline-first architecture, AI-powered features, and enterprise-grade reliability.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interface Layer                     │
│  (React 19 Components + TailwindCSS + shadcn/ui)            │
├─────────────────────────────────────────────────────────────┤
│                    State Management Layer                    │
│  (React Context + useReducer + Custom Hooks)               │
├─────────────────────────────────────────────────────────────┤
│                    Business Logic Layer                      │
│  (Feature Hooks + Utilities + AI Services)                 │
├─────────────────────────────────────────────────────────────┤
│                    Data Access Layer                         │
│  (IndexedDB + Sync Queue + Local Storage)                  │
├─────────────────────────────────────────────────────────────┤
│                    Service Worker Layer                      │
│  (Workbox + Caching + Background Sync)                     │
├─────────────────────────────────────────────────────────────┤
│                    Offline/Online Detection                  │
│  (Navigator.onLine + Service Worker Events)                │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Flow Architecture

### Online Mode
```
User Action
    ↓
React Component
    ↓
Custom Hook
    ↓
Business Logic
    ↓
IndexedDB (Local)
    ↓
Service Worker
    ↓
Network Request (if needed)
    ↓
Update IndexedDB
    ↓
Update UI
```

### Offline Mode
```
User Action
    ↓
React Component
    ↓
Custom Hook
    ↓
Business Logic
    ↓
IndexedDB (Local)
    ↓
Sync Queue
    ↓
Notify User (Offline)
    ↓
Update UI (from Local Data)
```

### Sync Process
```
Online Detection
    ↓
Check Sync Queue
    ↓
Batch Operations
    ↓
Send to Server (if backend exists)
    ↓
Handle Conflicts
    ↓
Update IndexedDB
    ↓
Clear Sync Queue
    ↓
Notify User (Synced)
```

---

## Component Architecture

### Page Components
```
App.tsx
├── Dashboard.tsx          # Main dashboard
├── TimeTracking.tsx       # Work time entry
├── Reports.tsx            # Analytics & reports
├── Profile.tsx            # User profile
├── Settings.tsx           # App settings
└── NotFound.tsx           # 404 page
```

### Layout Components
```
components/layout/
├── Header.tsx             # Top navigation
├── Sidebar.tsx            # Left navigation
├── MainLayout.tsx         # Main layout wrapper
└── PageContainer.tsx      # Page wrapper
```

### Feature Components
```
components/features/
├── TimeEntry/
│   ├── TimeEntryForm.tsx
│   ├── TimeEntryList.tsx
│   └── TimeEntryCard.tsx
├── Reports/
│   ├── DailyReport.tsx
│   ├── WeeklyReport.tsx
│   └── ReportChart.tsx
├── Profile/
│   ├── ProfileForm.tsx
│   ├── CertificationList.tsx
│   └── DocumentUpload.tsx
└── Settings/
    ├── GeneralSettings.tsx
    ├── NotificationSettings.tsx
    └── PrivacySettings.tsx
```

### UI Components (shadcn/ui)
```
components/ui/
├── button.tsx
├── card.tsx
├── dialog.tsx
├── form.tsx
├── input.tsx
├── label.tsx
├── select.tsx
├── tabs.tsx
├── toast.tsx
└── ... (30+ components)
```

---

## State Management Architecture

### Global State (Context)
```typescript
// AppContext - Global application state
- currentUser
- appSettings
- theme
- language
- notifications

// SyncContext - Offline sync state
- isSyncing
- syncQueue
- lastSyncTime
- syncErrors

// ThemeContext - Theme management
- theme (light/dark)
- toggleTheme
```

### Local State (useState)
```typescript
// Component-level state
- Form inputs
- UI state (modals, dropdowns)
- Loading states
- Error messages
```

### Custom Hooks
```typescript
// Feature hooks
- useWorkTime()         // Time tracking
- useTimeEntries()      // Entry management
- useReports()          // Report generation
- useProfile()          // User profile
- useSettings()         // App settings
- useOfflineSync()      // Offline sync
- useAIRecommendations()// AI features
- useNotifications()    // Notifications
```

---

## Data Persistence Architecture

### IndexedDB Schema

```typescript
// Database: nano4hr
// Version: 1

// Store: workSessions
{
  keyPath: 'id',
  indexes: [
    { name: 'date', keyPath: 'date' },
    { name: 'synced', keyPath: 'synced' },
    { name: 'userId', keyPath: 'userId' }
  ]
}

// Store: employees
{
  keyPath: 'id',
  indexes: [
    { name: 'email', keyPath: 'email', unique: true },
    { name: 'department', keyPath: 'department' }
  ]
}

// Store: leaves
{
  keyPath: 'id',
  indexes: [
    { name: 'userId', keyPath: 'userId' },
    { name: 'status', keyPath: 'status' },
    { name: 'startDate', keyPath: 'startDate' }
  ]
}

// Store: attendance
{
  keyPath: 'id',
  indexes: [
    { name: 'date', keyPath: 'date' },
    { name: 'userId', keyPath: 'userId' }
  ]
}

// Store: syncQueue
{
  keyPath: 'id',
  indexes: [
    { name: 'timestamp', keyPath: 'timestamp' },
    { name: 'status', keyPath: 'status' }
  ]
}

// Store: aiModels
{
  keyPath: 'id',
  indexes: [
    { name: 'type', keyPath: 'type' },
    { name: 'version', keyPath: 'version' }
  ]
}

// Store: settings
{
  keyPath: 'key'
}
```

### Data Models

```typescript
// Work Session
interface WorkSession {
  id: string;
  userId: string;
  date: string;
  startTime: number;
  endTime: number | null;
  breaks: Break[];
  duration: number;
  overtime: number;
  synced: boolean;
  syncedAt?: number;
  createdAt: number;
  updatedAt: number;
}

// Employee
interface Employee {
  id: string;
  name: string;
  email: string;
  position: string;
  department: string;
  phone?: string;
  certifications: Certification[];
  createdAt: number;
  updatedAt: number;
}

// Leave Request
interface LeaveRequest {
  id: string;
  userId: string;
  type: 'vacation' | 'sick' | 'personal' | 'other';
  startDate: string;
  endDate: string;
  reason: string;
  status: 'pending' | 'approved' | 'rejected';
  createdAt: number;
  updatedAt: number;
}

// Sync Queue Item
interface SyncQueueItem {
  id: string;
  action: 'create' | 'update' | 'delete';
  entity: 'workSession' | 'employee' | 'leave';
  data: any;
  timestamp: number;
  retries: number;
  status: 'pending' | 'syncing' | 'failed';
  error?: string;
}
```

---

## Service Worker Architecture

### Caching Strategy

```typescript
// Static Assets (Cache-First)
- HTML, CSS, JS
- Fonts
- Icons
- Images (with expiration)

// API Responses (Network-First)
- Work sessions
- Employee data
- Reports
- Settings

// Fallback (Offline Page)
- Show cached content
- Queue operations
- Notify user
```

### Background Sync

```typescript
// Sync Events
1. Online detection
2. Periodic sync (every 5 minutes)
3. Manual sync trigger
4. App focus detection

// Sync Process
1. Get items from sync queue
2. Batch operations
3. Send to server (if available)
4. Handle responses
5. Update local data
6. Clear queue
7. Notify user
```

### Push Notifications

```typescript
// Service Worker Push Handler
- Listen for push events
- Decode notification data
- Show notification
- Handle notification clicks
- Track engagement
```

---

## AI/ML Architecture

### TensorFlow.js Integration

```typescript
// Model Loading
1. Load model asynchronously after first paint
2. Cache model in IndexedDB
3. Update model periodically

// Inference Pipeline
1. Prepare input data
2. Run prediction
3. Post-process results
4. Cache predictions
5. Update UI

// Models
- Work Pattern Predictor
- Anomaly Detector
- Break Time Optimizer
- Fatigue Detector
```

### AI Features

```typescript
// Predictive Features
- Suggest work schedule
- Recommend break times
- Detect anomalies
- Predict fatigue
- Optimize productivity

// Personalization
- User preference learning
- Behavior analysis
- Recommendation ranking
- Notification timing
```

---

## Performance Architecture

### Code Splitting

```typescript
// Route-based splitting
- Dashboard (lazy loaded)
- TimeTracking (lazy loaded)
- Reports (lazy loaded)
- Profile (lazy loaded)
- Settings (lazy loaded)

// Feature-based splitting
- AI models (lazy loaded)
- Charts library (lazy loaded)
- Heavy utilities (lazy loaded)
```

### Asset Optimization

```typescript
// Images
- WebP format with fallback
- Responsive images
- Lazy loading
- Compression

// Fonts
- System fonts (fast)
- Google Fonts (preload critical)
- Font subsetting
- WOFF2 format

// CSS
- TailwindCSS purging
- CSS-in-JS optimization
- Critical CSS inline
```

### Performance Monitoring

```typescript
// Metrics
- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)
- Time to Interactive (TTI)
- First Contentful Paint (FCP)

// Tools
- Lighthouse CI
- Web Vitals library
- Error tracking
- Performance profiling
```

---

## Security Architecture

### Data Protection

```typescript
// Local Storage
- IndexedDB encryption (optional)
- Secure session storage
- No sensitive data in localStorage

// Service Worker
- HTTPS only
- Secure headers
- CSP (Content Security Policy)
- SRI (Subresource Integrity)
```

### Privacy

```typescript
// Data Handling
- All data stored locally
- No server transmission (offline-first)
- User consent for analytics
- GDPR compliance
- Data export/delete features
```

---

## Testing Architecture

### Unit Tests
```typescript
// Test files
- hooks/__tests__/useWorkTime.test.ts
- lib/__tests__/db.test.ts
- lib/__tests__/sync.test.ts
- lib/__tests__/ai.test.ts
```

### Component Tests
```typescript
// React Testing Library
- components/__tests__/TimeEntryForm.test.tsx
- components/__tests__/Reports.test.tsx
- pages/__tests__/Dashboard.test.tsx
```

### Integration Tests
```typescript
// Feature workflows
- Time entry creation and sync
- Offline operation and sync
- AI recommendation generation
- Notification delivery
```

### E2E Tests
```typescript
// User workflows
- Complete work day tracking
- Report generation
- Settings management
- Offline functionality
```

---

## Deployment Architecture

### Build Process
```
Source Code
    ↓
TypeScript Compilation
    ↓
React Build (Vite)
    ↓
Code Splitting
    ↓
Asset Optimization
    ↓
Service Worker Generation
    ↓
Manifest Generation
    ↓
Production Bundle
```

### Deployment Targets
```typescript
// Static Hosting
- Vercel
- Netlify
- GitHub Pages
- AWS S3 + CloudFront
- Manus Platform

// Requirements
- HTTPS
- Service Worker support
- HTTP/2
- Gzip compression
- Cache headers
```

### CI/CD Pipeline
```
Git Push
    ↓
Run Tests
    ↓
Lint & Format
    ↓
Build
    ↓
Lighthouse Audit
    ↓
Deploy to Staging
    ↓
E2E Tests
    ↓
Deploy to Production
    ↓
Monitor & Alert
```

---

## Scalability Architecture

### Horizontal Scaling
```typescript
// Multi-device sync
- Same user on multiple devices
- Conflict resolution
- Device-specific data
- Cross-device notifications
```

### Feature Scaling
```typescript
// Modular architecture
- Add new features without touching core
- Feature flags for gradual rollout
- Plugin system for extensions
- Custom hooks for reusability
```

### Performance Scaling
```typescript
// Optimization techniques
- Progressive loading
- Virtual scrolling for large lists
- Memoization of expensive computations
- Web Workers for heavy processing
```

---

## Technology Stack Details

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **UI Framework** | React | 19.2.1 | Component-based UI |
| **Language** | TypeScript | 5.6.3 | Type safety |
| **Build Tool** | Vite | 7.1.7 | Fast development & builds |
| **Styling** | TailwindCSS | 4.1.14 | Utility-first CSS |
| **Components** | shadcn/ui | Latest | Pre-built accessible components |
| **Routing** | Wouter | 3.3.5 | Lightweight client routing |
| **Forms** | React Hook Form | 7.64.0 | Efficient form handling |
| **Validation** | Zod | 4.1.12 | Schema validation |
| **State** | Context + useReducer | Built-in | Global state management |
| **Data Storage** | IndexedDB | Browser API | Offline data persistence |
| **ORM** | Dexie.js | 4.x | IndexedDB wrapper |
| **Service Worker** | Workbox | 7.x | Advanced caching |
| **AI/ML** | TensorFlow.js | 4.x | On-device inference |
| **Charts** | Recharts | 2.15.2 | Data visualization |
| **Icons** | Lucide React | 0.453.0 | Icon library |
| **Notifications** | Sonner | 2.0.7 | Toast notifications |
| **Testing** | Vitest | 2.1.4 | Unit testing |
| **Testing** | React Testing Library | 16.x | Component testing |
| **Linting** | ESLint | 9.x | Code quality |
| **Formatting** | Prettier | 3.6.2 | Code formatting |

---

## Conclusion

This architecture provides a modern, scalable, and maintainable foundation for nano4HR's evolution into a 2026-ready progressive web application. The layered approach ensures separation of concerns, the offline-first design preserves the core value proposition, and the modular structure enables future growth and feature additions.

---

**Document Version:** 1.0  
**Last Updated:** 2026-04-30  
**Status:** Ready for Development

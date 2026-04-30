# nano4HR Implementation Roadmap - 2026 Modernization

## Quick Start Guide

This roadmap provides a step-by-step implementation plan for modernizing nano4HR from vanilla JavaScript to React 19 + TypeScript.

---

## Phase 1: Foundation (Weeks 1-2)

### Week 1: Project Setup

#### Day 1-2: Repository & Environment
- [ ] Create new branch: `modernization/react-typescript`
- [ ] Initialize React 19 + TypeScript project
- [ ] Set up Vite configuration
- [ ] Configure TailwindCSS 4
- [ ] Set up shadcn/ui components
- [ ] Configure TypeScript strict mode
- [ ] Set up ESLint & Prettier

**Tasks:**
```bash
# Initialize project structure
pnpm create vite@latest nano4hr-modern --template react-ts
cd nano4hr-modern
pnpm install

# Add dependencies
pnpm add -D tailwindcss postcss autoprefixer
pnpm add @radix-ui/react-* lucide-react sonner
pnpm add wouter react-hook-form zod
```

#### Day 3-4: Design System
- [ ] Create color palette (HR professional blues)
- [ ] Define typography system
- [ ] Set up spacing system (8px base)
- [ ] Create CSS variables in `index.css`
- [ ] Configure Tailwind theme
- [ ] Set up dark mode support

**File: `client/src/index.css`**
```css
@import "tailwindcss";

@layer base {
  :root {
    --primary: #2563eb;
    --primary-foreground: #ffffff;
    --secondary: #64748b;
    --accent: #f59e0b;
    --background: #ffffff;
    --foreground: #1f2937;
    --border: #e5e7eb;
  }
  
  .dark {
    --background: #0f172a;
    --foreground: #f1f5f9;
    --border: #334155;
  }
}
```

#### Day 5: Component Library
- [ ] Set up shadcn/ui
- [ ] Create custom Button component
- [ ] Create custom Card component
- [ ] Create custom Form components
- [ ] Document component usage

**File: `client/src/components/ui/button.tsx`**
```typescript
import * as React from "react"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      ref={ref}
      {...props}
    />
  )
)
Button.displayName = "Button"

export { Button, buttonVariants }
```

### Week 2: Core Infrastructure

#### Day 1-2: Routing & Layout
- [ ] Set up Wouter routing
- [ ] Create MainLayout component
- [ ] Create Header component
- [ ] Create Sidebar component
- [ ] Create page structure

**File: `client/src/App.tsx`**
```typescript
import { Router, Route, Switch } from "wouter"
import { ThemeProvider } from "@/contexts/ThemeContext"
import MainLayout from "@/components/layout/MainLayout"
import Dashboard from "@/pages/Dashboard"
import TimeTracking from "@/pages/TimeTracking"
import Reports from "@/pages/Reports"
import Profile from "@/pages/Profile"
import Settings from "@/pages/Settings"
import NotFound from "@/pages/NotFound"

export default function App() {
  return (
    <ThemeProvider>
      <Router>
        <Switch>
          <Route path="/" component={Dashboard} />
          <Route path="/time-tracking" component={TimeTracking} />
          <Route path="/reports" component={Reports} />
          <Route path="/profile" component={Profile} />
          <Route path="/settings" component={Settings} />
          <Route component={NotFound} />
        </Switch>
      </Router>
    </ThemeProvider>
  )
}
```

#### Day 3-4: State Management
- [ ] Create AppContext
- [ ] Create SyncContext
- [ ] Create ThemeContext
- [ ] Set up useReducer for complex state
- [ ] Create custom hooks structure

**File: `client/src/contexts/AppContext.tsx`**
```typescript
import React, { createContext, useReducer, ReactNode } from "react"

interface AppState {
  currentUser: any
  appSettings: any
  notifications: any[]
  isLoading: boolean
}

interface AppContextType {
  state: AppState
  dispatch: React.Dispatch<any>
}

export const AppContext = createContext<AppContextType | undefined>(undefined)

const initialState: AppState = {
  currentUser: null,
  appSettings: {},
  notifications: [],
  isLoading: false,
}

function appReducer(state: AppState, action: any): AppState {
  switch (action.type) {
    case "SET_USER":
      return { ...state, currentUser: action.payload }
    case "SET_SETTINGS":
      return { ...state, appSettings: action.payload }
    case "ADD_NOTIFICATION":
      return { ...state, notifications: [...state.notifications, action.payload] }
    case "REMOVE_NOTIFICATION":
      return { ...state, notifications: state.notifications.filter(n => n.id !== action.payload) }
    case "SET_LOADING":
      return { ...state, isLoading: action.payload }
    default:
      return state
  }
}

export function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, initialState)

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  )
}

export function useApp() {
  const context = React.useContext(AppContext)
  if (!context) {
    throw new Error("useApp must be used within AppProvider")
  }
  return context
}
```

#### Day 5: Service Worker Setup
- [ ] Create service worker file
- [ ] Configure Workbox
- [ ] Set up caching strategies
- [ ] Implement offline detection

**File: `public/service-worker.ts`**
```typescript
/// <reference lib="webworker" />

declare const self: ServiceWorkerGlobalScope

const CACHE_NAME = "nano4hr-v1"
const STATIC_ASSETS = [
  "/",
  "/index.html",
  "/manifest.json",
]

// Install event
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(STATIC_ASSETS)
    })
  )
})

// Activate event
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_NAME) {
            return caches.delete(cacheName)
          }
        })
      )
    })
  )
})

// Fetch event
self.addEventListener("fetch", (event) => {
  const { request } = event

  // Skip non-GET requests
  if (request.method !== "GET") {
    return
  }

  // Cache-first for static assets
  if (request.destination === "style" || request.destination === "script") {
    event.respondWith(
      caches.match(request).then((response) => {
        return response || fetch(request)
      })
    )
    return
  }

  // Network-first for API calls
  event.respondWith(
    fetch(request)
      .then((response) => {
        if (!response || response.status !== 200) {
          return response
        }

        const responseToCache = response.clone()
        caches.open(CACHE_NAME).then((cache) => {
          cache.put(request, responseToCache)
        })

        return response
      })
      .catch(() => {
        return caches.match(request)
      })
  )
})
```

---

## Phase 2: Core Features (Weeks 3-4)

### Week 3: Time Tracking Module

#### Day 1-2: Data Models & Database
- [ ] Create TypeScript interfaces for work sessions
- [ ] Set up IndexedDB with Dexie.js
- [ ] Create database schema
- [ ] Implement CRUD operations

**File: `client/src/types/worktime.ts`**
```typescript
export interface WorkSession {
  id: string
  userId: string
  date: string
  startTime: number
  endTime: number | null
  breaks: Break[]
  duration: number
  overtime: number
  synced: boolean
  syncedAt?: number
  createdAt: number
  updatedAt: number
}

export interface Break {
  id: string
  startTime: number
  endTime: number
  duration: number
}

export interface DailySummary {
  date: string
  totalDuration: number
  overtime: number
  breaks: number
  sessions: WorkSession[]
}
```

**File: `client/src/lib/db.ts`**
```typescript
import Dexie, { Table } from "dexie"
import { WorkSession, Employee, LeaveRequest } from "@/types"

export class Nano4HRDatabase extends Dexie {
  workSessions!: Table<WorkSession>
  employees!: Table<Employee>
  leaves!: Table<LeaveRequest>
  syncQueue!: Table<any>

  constructor() {
    super("nano4hr")
    this.version(1).stores({
      workSessions: "++id, date, userId, synced",
      employees: "++id, email",
      leaves: "++id, userId, status, startDate",
      syncQueue: "++id, timestamp, status",
    })
  }
}

export const db = new Nano4HRDatabase()
```

#### Day 3-4: Time Tracking Components
- [ ] Create TimeEntryForm component
- [ ] Create TimeEntryList component
- [ ] Create TimeEntryCard component
- [ ] Implement start/stop functionality

**File: `client/src/components/features/TimeEntry/TimeEntryForm.tsx`**
```typescript
import { useState } from "react"
import { useForm } from "react-hook-form"
import { Button } from "@/components/ui/button"
import { useWorkTime } from "@/hooks/useWorkTime"
import { toast } from "sonner"

export default function TimeEntryForm() {
  const { startSession, endSession, currentSession } = useWorkTime()
  const [isRunning, setIsRunning] = useState(false)

  const handleStart = async () => {
    try {
      await startSession()
      setIsRunning(true)
      toast.success("Work session started")
    } catch (error) {
      toast.error("Failed to start session")
    }
  }

  const handleEnd = async () => {
    try {
      await endSession()
      setIsRunning(false)
      toast.success("Work session ended")
    } catch (error) {
      toast.error("Failed to end session")
    }
  }

  return (
    <div className="space-y-4">
      <div className="flex gap-4">
        <Button
          onClick={handleStart}
          disabled={isRunning}
          variant="default"
          size="lg"
        >
          Start Work
        </Button>
        <Button
          onClick={handleEnd}
          disabled={!isRunning}
          variant="outline"
          size="lg"
        >
          End Work
        </Button>
      </div>
      {currentSession && (
        <div className="p-4 bg-blue-50 rounded-lg">
          <p>Current session started at {new Date(currentSession.startTime).toLocaleTimeString()}</p>
        </div>
      )}
    </div>
  )
}
```

#### Day 5: Custom Hooks
- [ ] Create useWorkTime hook
- [ ] Create useTimeEntries hook
- [ ] Create useOfflineSync hook

**File: `client/src/hooks/useWorkTime.ts`**
```typescript
import { useState, useEffect } from "react"
import { db } from "@/lib/db"
import { WorkSession } from "@/types"

export function useWorkTime() {
  const [currentSession, setCurrentSession] = useState<WorkSession | null>(null)
  const [sessions, setSessions] = useState<WorkSession[]>([])
  const [isLoading, setIsLoading] = useState(false)

  useEffect(() => {
    loadCurrentSession()
  }, [])

  const loadCurrentSession = async () => {
    const today = new Date().toISOString().split("T")[0]
    const todaySessions = await db.workSessions
      .where("date")
      .equals(today)
      .toArray()

    const active = todaySessions.find(s => !s.endTime)
    setCurrentSession(active || null)
  }

  const startSession = async () => {
    setIsLoading(true)
    try {
      const session: WorkSession = {
        id: crypto.randomUUID(),
        userId: "current-user",
        date: new Date().toISOString().split("T")[0],
        startTime: Date.now(),
        endTime: null,
        breaks: [],
        duration: 0,
        overtime: 0,
        synced: false,
        createdAt: Date.now(),
        updatedAt: Date.now(),
      }

      await db.workSessions.add(session)
      setCurrentSession(session)
    } finally {
      setIsLoading(false)
    }
  }

  const endSession = async () => {
    if (!currentSession) return

    setIsLoading(true)
    try {
      const updated = {
        ...currentSession,
        endTime: Date.now(),
        duration: Date.now() - currentSession.startTime,
        updatedAt: Date.now(),
      }

      await db.workSessions.update(currentSession.id, updated)
      setCurrentSession(null)
    } finally {
      setIsLoading(false)
    }
  }

  return {
    currentSession,
    sessions,
    isLoading,
    startSession,
    endSession,
    loadCurrentSession,
  }
}
```

### Week 4: HR Management & Sync

#### Day 1-2: HR Features
- [ ] Create Employee management
- [ ] Create Leave request tracking
- [ ] Create Attendance records
- [ ] Implement data models

#### Day 3-4: Offline Sync
- [ ] Create sync queue system
- [ ] Implement conflict resolution
- [ ] Create sync status UI
- [ ] Add retry logic

**File: `client/src/lib/sync.ts`**
```typescript
import { db } from "./db"

export interface SyncQueueItem {
  id: string
  action: "create" | "update" | "delete"
  entity: string
  data: any
  timestamp: number
  retries: number
  status: "pending" | "syncing" | "failed"
  error?: string
}

export async function queueSync(
  action: "create" | "update" | "delete",
  entity: string,
  data: any
) {
  const item: SyncQueueItem = {
    id: crypto.randomUUID(),
    action,
    entity,
    data,
    timestamp: Date.now(),
    retries: 0,
    status: "pending",
  }

  await db.syncQueue.add(item)
}

export async function processSyncQueue() {
  const pendingItems = await db.syncQueue
    .where("status")
    .equals("pending")
    .toArray()

  for (const item of pendingItems) {
    try {
      await db.syncQueue.update(item.id, { status: "syncing" })

      // Simulate sync to server
      await new Promise(resolve => setTimeout(resolve, 1000))

      await db.syncQueue.update(item.id, { status: "pending" })
    } catch (error) {
      await db.syncQueue.update(item.id, {
        status: "failed",
        error: String(error),
        retries: item.retries + 1,
      })
    }
  }
}

export async function getSyncStatus() {
  const pending = await db.syncQueue.where("status").equals("pending").count()
  const failed = await db.syncQueue.where("status").equals("failed").count()

  return { pending, failed, total: pending + failed }
}
```

#### Day 5: Testing & Integration
- [ ] Write unit tests for hooks
- [ ] Write component tests
- [ ] Test offline functionality
- [ ] Integration testing

---

## Phase 3: AI & Advanced Features (Weeks 5-6)

### Week 5: AI Integration

#### Day 1-2: TensorFlow.js Setup
- [ ] Add TensorFlow.js dependency
- [ ] Create AI model loader
- [ ] Implement model caching
- [ ] Set up inference pipeline

**File: `client/src/lib/ai.ts`**
```typescript
import * as tf from "@tensorflow/tfjs"

export class AIEngine {
  private model: tf.LayersModel | null = null
  private isLoading = false

  async loadModel() {
    if (this.model || this.isLoading) return

    this.isLoading = true
    try {
      // Load pre-trained model
      this.model = await tf.loadLayersModel(
        "indexeddb://nano4hr-model"
      )
    } catch (error) {
      console.error("Failed to load AI model:", error)
    } finally {
      this.isLoading = false
    }
  }

  async predictWorkSchedule(historicalData: any[]) {
    if (!this.model) await this.loadModel()
    if (!this.model) return null

    try {
      const input = tf.tensor2d([historicalData])
      const prediction = this.model.predict(input) as tf.Tensor
      const result = await prediction.data()

      prediction.dispose()
      input.dispose()

      return Array.from(result)
    } catch (error) {
      console.error("Prediction failed:", error)
      return null
    }
  }

  async detectAnomalies(workSessions: any[]) {
    // Implement anomaly detection logic
    const anomalies = []
    // ... detection logic
    return anomalies
  }
}

export const aiEngine = new AIEngine()
```

#### Day 3-4: AI Features
- [ ] Work pattern prediction
- [ ] Anomaly detection
- [ ] Break time optimization
- [ ] Fatigue detection

#### Day 5: Push Notifications
- [ ] Set up Web Push API
- [ ] Create notification service
- [ ] Implement notification UI
- [ ] Add AI-powered timing

**File: `client/src/lib/notifications.ts`**
```typescript
export async function requestNotificationPermission() {
  if (!("Notification" in window)) {
    console.log("This browser does not support notifications")
    return false
  }

  if (Notification.permission === "granted") {
    return true
  }

  if (Notification.permission !== "denied") {
    const permission = await Notification.requestPermission()
    return permission === "granted"
  }

  return false
}

export async function showNotification(
  title: string,
  options?: NotificationOptions
) {
  if (Notification.permission === "granted") {
    const registration = await navigator.serviceWorker.ready
    registration.showNotification(title, options)
  }
}

export async function subscribeToNotifications() {
  const registration = await navigator.serviceWorker.ready
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: process.env.VITE_VAPID_PUBLIC_KEY,
  })
  return subscription
}
```

### Week 6: Polish & Optimization

#### Day 1-2: Performance
- [ ] Code splitting by route
- [ ] Lazy loading components
- [ ] Image optimization
- [ ] CSS optimization

#### Day 3-4: Testing
- [ ] Unit tests (80%+ coverage)
- [ ] Component tests
- [ ] Integration tests
- [ ] E2E tests

#### Day 5: Documentation
- [ ] API documentation
- [ ] Component library docs
- [ ] Setup guide
- [ ] Contributing guide

---

## Phase 4: Launch Preparation (Weeks 7-8)

### Week 7: Final Testing & Optimization

- [ ] Lighthouse audit (target > 90)
- [ ] Accessibility audit (WCAG 2.1 AA)
- [ ] Cross-browser testing
- [ ] Mobile device testing
- [ ] Performance profiling

### Week 8: Deployment & Monitoring

- [ ] Create GitHub release
- [ ] Deploy to staging
- [ ] Final verification
- [ ] Deploy to production
- [ ] Set up monitoring
- [ ] Gather feedback

---

## Deployment Checklist

### Pre-Deployment
- [ ] All tests passing
- [ ] Lighthouse score > 90
- [ ] No console errors
- [ ] Offline functionality verified
- [ ] All features tested on mobile
- [ ] Accessibility audit passed
- [ ] Performance acceptable
- [ ] Security review completed

### Deployment
- [ ] Build production bundle
- [ ] Deploy to hosting
- [ ] Verify deployment
- [ ] Set up monitoring
- [ ] Configure analytics
- [ ] Set up error tracking

### Post-Deployment
- [ ] Monitor error rates
- [ ] Track performance metrics
- [ ] Gather user feedback
- [ ] Plan improvements
- [ ] Schedule maintenance

---

## Key Metrics to Track

| Metric | Target | Tool |
|--------|--------|------|
| Lighthouse PWA | > 90 | Lighthouse CI |
| LCP (Load) | < 2.5s | Web Vitals |
| INP (Interactivity) | < 200ms | Web Vitals |
| CLS (Stability) | < 0.1 | Web Vitals |
| Bundle Size | < 500KB | Webpack Bundle Analyzer |
| Test Coverage | > 80% | Vitest |
| Accessibility | WCAG AA | axe DevTools |

---

## Success Criteria

- ✅ All core features migrated to React
- ✅ 100% TypeScript coverage
- ✅ Offline functionality preserved
- ✅ AI features implemented
- ✅ Performance improved
- ✅ Tests passing (> 80% coverage)
- ✅ Documentation complete
- ✅ Lighthouse score > 90
- ✅ Zero breaking changes for users
- ✅ Smooth deployment

---

**Document Version:** 1.0  
**Last Updated:** 2026-04-30  
**Status:** Ready for Implementation

# Development Guidelines - KNOBO Code Standards

**Version**: 1.1
**Last Updated**: 2025-01-27
**Scope**: Next.js Web Applications & Node.js Functions

This document defines our unified coding standards, architecture patterns, and best practices across all KNOBO projects to ensure consistency, maintainability, and scalability.

## Table of Contents

1. [Core Principles](#core-principles)
2. [AI Assistant Guidelines](#ai-assistant-guidelines)
3. [Code Reuse & Duplication Prevention](#code-reuse--duplication-prevention)
4. [Project Structure](#project-structure)
5. [SOLID Principles Implementation](#solid-principles-implementation)
6. [Clean Code Standards](#clean-code-standards)
7. [Comments & Documentation](#comments--documentation)
8. [Frontend Development](#frontend-development)
9. [Backend Development](#backend-development)
10. [Error Handling & Resilience](#error-handling--resilience)
11. [Performance Guidelines](#performance-guidelines)
12. [Implementation Checklist](#implementation-checklist)

---

## Core Principles

### 1. Feature-First Architecture
- Organize code by business domain/feature, not by technical layer
- Each feature owns its own types, actions, components, and utilities
- Shared utilities only for truly generic, reusable code

### 2. Explicit Dependencies
- Use dependency injection over hard-coded implementations
- Abstract external services behind interfaces
- Make dependencies swappable and modular

### 3. Fail Fast, Fail Clear
- Use comprehensive error handling with typed results
- Provide actionable error messages
- Log errors with sufficient context for debugging

---

## AI Assistant Guidelines

**CRITICAL**: Before writing any new code, AI assistants MUST follow this discovery process to prevent duplication and leverage existing code:

### 1. Codebase Discovery Process

**ALWAYS perform these checks before creating new code:**

1. **Search for existing types/interfaces**:
   ```bash
   # Search for existing types before creating new ones
   grep -r "interface.*User\|type.*User" src/
   grep -r "interface.*Company\|type.*Company" src/
   ```

2. **Check for existing utilities/functions**:
   ```bash
   # Search for similar functionality
   grep -r "function.*format\|const.*format" src/
   grep -r "export.*function.*validate" src/
   ```

3. **Look for existing services/hooks**:
   ```bash
   # Check for existing hooks and services
   find src/ -name "use-*.ts" -o -name "*-service.ts" -o -name "*-actions.ts"
   ```

4. **Review existing components**:
   ```bash
   # Find similar components
   find src/components -name "*form*" -o -name "*modal*" -o -name "*card*"
   ```

### 2. Code Reuse Priorities

**MANDATORY ORDER** - Check these locations before creating new code:

1. **`src/shared/`** - Generic utilities, hooks, constants
2. **`src/features/*/`** - Feature-specific types, components, services
3. **`src/lib/`** - Infrastructure services (auth, database, external APIs)
4. **`src/components/ui/`** - Base UI components

### 3. Before Creating New Code

**ASK THESE QUESTIONS:**

- ✅ "Does a similar type/interface already exist that I can extend?"
- ✅ "Is there an existing utility function that does something similar?"
- ✅ "Can I reuse an existing component and just modify props?"
- ✅ "Is there already a hook that handles similar state/logic?"
- ✅ "Does this service/action already exist in a different feature?"

### 4. Extending vs Creating New

**PREFER extending existing code:**

```typescript
// ✅ GOOD: Extend existing interface
interface CompanyProfile {
  id: string;
  name: string;
  // ... existing fields
}

interface EnrichedCompanyProfile extends CompanyProfile {
  enrichmentData: EnrichmentData;
  enrichmentStatus: 'pending' | 'completed' | 'failed';
}

// ❌ BAD: Duplicate similar interface
interface CompanyWithEnrichment {
  id: string;           // Duplicated
  name: string;         // Duplicated
  enrichmentData: EnrichmentData;
  enrichmentStatus: 'pending' | 'completed' | 'failed';
}
```

### 5. AI Assistant Commands

**Use these commands to check existing code:**

- `find src/ -name "*.ts" -o -name "*.tsx" | xargs grep -l "YourSearchTerm"`
- `grep -r "export.*interface\|export.*type" src/features/`
- `find src/shared -name "*.ts" | head -20` (check shared utilities)
- `ls src/features/*/types/` (check feature-specific types)

---

## Code Reuse & Duplication Prevention

### 1. Mandatory Checks Before Coding

**NEVER create new code without checking these locations:**

#### Types & Interfaces
```typescript
// Check these files FIRST:
// - src/features/*/types/
// - src/shared/types/
// - src/lib/types/

// Example: Before creating user-related types
// CHECK: src/features/auth/types/, src/features/company-profiles/types/
```

#### Utility Functions
```typescript
// Check these locations FIRST:
// - src/shared/utils/
// - src/features/*/utils/
// - src/lib/utils/

// Example: Before creating formatters
// CHECK: src/shared/utils/formatting.ts
```

#### Hooks
```typescript
// Check these patterns FIRST:
// - src/shared/hooks/use-*.ts
// - src/features/*/hooks/use-*.ts

// Example: Before creating useAsyncState
// CHECK: src/shared/hooks/use-async-operation.ts
```

### 2. Code Discovery Workflow

**STEP 1**: Search by functionality
```bash
# Example: Need form validation?
grep -r "validation\|validate" src/shared/ src/features/
```

**STEP 2**: Search by naming patterns
```bash
# Example: Need company-related code?
find src/ -name "*company*" -o -name "*profile*"
```

**STEP 3**: Check existing implementations
```bash
# Example: Need similar component?
find src/components -name "*form*" | head -5 | xargs cat
```

### 3. Reuse Strategies

#### Extend Existing Types
```typescript
// ✅ GOOD: Build on existing types
import { BaseUser } from '@/shared/types/base';

interface AdminUser extends BaseUser {
  permissions: Permission[];
  lastLoginAt: Date;
}
```

#### Compose Existing Utilities
```typescript
// ✅ GOOD: Combine existing functions
import { formatRevenue } from '@/shared/utils/formatting';
import { validateRequired } from '@/shared/hooks/use-form-validation';

function CompanyRevenueField({ revenue }: { revenue: number }) {
  return <span>{formatRevenue(revenue)}</span>;
}
```

#### Wrap Existing Components
```typescript
// ✅ GOOD: Enhance existing components
import { Card } from '@/components/ui/card';

function CompanyCard({ company }: { company: Company }) {
  return (
    <Card className="company-specific-styling">
      {/* Company-specific content */}
    </Card>
  );
}
```

### 4. Anti-Patterns to Avoid

```typescript
// ❌ BAD: Copy-paste similar functions
function formatUserRevenue(amount: number) {
  if (amount >= 1000000) return `${(amount / 1000000).toFixed(1)}M`;
  // ... duplicate of formatRevenue
}

// ❌ BAD: Duplicate similar interfaces
interface UserProfile {
  id: string;
  name: string;
  email: string;
}
interface CompanyProfile {
  id: string;        // Duplicated field
  name: string;      // Duplicated field
  industry: string;
}

// ✅ GOOD: Use generic base
interface BaseProfile {
  id: string;
  name: string;
}
interface UserProfile extends BaseProfile {
  email: string;
}
interface CompanyProfile extends BaseProfile {
  industry: string;
}
```

---

## Project Structure

### Next.js Web Applications

```
src/
├── app/                    # Next.js App Router pages
│   ├── (routes)/          # Route groups
│   └── api/               # API routes
├── features/              # Business domain modules
│   └── {feature-name}/
│       ├── actions.ts     # Server actions
│       ├── types/         # Feature-specific types
│       ├── hooks/         # Feature-specific hooks
│       ├── utils/         # Feature-specific utilities
│       └── database.ts    # Data layer (if needed)
├── components/            # UI components organized by feature
│   ├── ui/               # Generic UI components
│   └── {feature-name}/   # Feature-specific components
├── lib/                  # Infrastructure & shared services
│   ├── auth/             # Authentication
│   ├── database/         # Database connections
│   ├── external-apis/    # Third-party integrations
│   └── storage/          # File storage
└── shared/               # Truly generic utilities
    ├── hooks/            # Generic hooks
    ├── utils/            # Generic utilities
    └── constants/        # Application constants
```

### Node.js Functions

```
src/
├── functions/            # Cloud function handlers
├── features/            # Business domain modules (same structure)
├── lib/                 # Infrastructure services
├── shared/              # Generic utilities
└── types/               # Global type definitions
```

### Key Structure Rules

1. **Features are self-contained**: Each feature folder contains everything specific to that business domain
2. **Shared is truly generic**: Only put code in `shared/` if it's used across multiple features
3. **Lib contains infrastructure**: Database connections, external APIs, authentication - services that multiple features depend on
4. **Components mirror features**: UI components are organized by the same feature boundaries

---

## SOLID Principles Implementation

### Single Responsibility Principle (SRP)

**DO:**
```typescript
// ✅ Each function has one clear responsibility
export async function validateUserCredentials(credentials: UserCredentials): Promise<ValidationResult> {
  // Only validates credentials
}

export async function authenticateUser(credentials: UserCredentials): Promise<AuthResult> {
  // Only handles authentication logic
}
```

**DON'T:**
```typescript
// ❌ Function doing too many things
export async function loginUser(credentials: UserCredentials) {
  // Validates credentials
  // Authenticates user
  // Updates last login
  // Sends welcome email
  // Updates user preferences
}
```

### Open/Closed Principle (OCP)

**DO:**
```typescript
// ✅ Abstract interface allows extension without modification
interface EmailProvider {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

class EmailService {
  constructor(private provider: EmailProvider) {}

  async sendWelcomeEmail(user: User) {
    await this.provider.sendEmail(user.email, 'Welcome!', this.getWelcomeBody(user));
  }
}
```

**DON'T:**
```typescript
// ❌ Hard-coded dependency on specific provider
class EmailService {
  async sendWelcomeEmail(user: User) {
    const sendgrid = new SendGridAPI(); // Hard-coded dependency
    await sendgrid.send(/* ... */);
  }
}
```

### Dependency Inversion Principle (DIP)

**DO:**
```typescript
// ✅ Depend on abstractions, not concretions
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
}

export class UserService {
  constructor(private userRepo: IUserRepository) {}

  async updateUser(id: string, updates: Partial<User>): Promise<ActionResult<User>> {
    // Business logic using interface
  }
}
```

---

## Clean Code Standards

### File Size Limits
- **Maximum file size**: 400 lines
- **Maximum function length**: 20 lines
- **Maximum function parameters**: 5 parameters

### Naming Conventions

**Functions & Variables:**
```typescript
// ✅ Clear, descriptive names
const getUserById = (id: string) => { /* ... */ };
const isValidEmail = (email: string) => boolean;
const calculateTotalRevenue = (transactions: Transaction[]) => number;

// ❌ Unclear abbreviations
const getUsrById = (id: string) => { /* ... */ };
const chkEmail = (email: string) => boolean;
const calcTotRev = (trans: Transaction[]) => number;
```

**Constants:**
```typescript
// ✅ Use descriptive constants instead of magic numbers
const API_TIMEOUT_MS = 30000;
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_PAGE_SIZE = 50;

// ❌ Magic numbers
setTimeout(callback, 30000); // What is 30000?
if (attempts > 3) { /* ... */ } // Why 3?
```

**Types & Interfaces:**
```typescript
// ✅ Clear, descriptive type names
interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
}

type PaymentStatus = 'pending' | 'completed' | 'failed';
```

### Function Complexity

**DO:**
```typescript
// ✅ Single responsibility, clear flow
export async function createUserProfile(userData: CreateUserRequest): Promise<ActionResult<User>> {
  const validation = validateUserData(userData);
  if (!validation.success) {
    return validation;
  }

  const hashedPassword = await hashPassword(userData.password);
  const user = await userRepository.create({
    ...userData,
    password: hashedPassword
  });

  return { success: true, data: user };
}
```

**DON'T:**
```typescript
// ❌ Too complex, multiple responsibilities
export async function createUserProfile(userData: any) {
  // 50+ lines of validation logic
  // Email sending logic
  // Database operations
  // File upload handling
  // Analytics tracking
  // etc...
}
```

---

## Comments & Documentation

### When to Add Comments

**REQUIRED Comments:**

1. **Complex Business Logic:**
```typescript
// ✅ GOOD: Explain WHY, not what
/**
 * Calculate competitor relevance score based on industry overlap,
 * market segment similarity, and revenue range proximity.
 *
 * This algorithm weights industry match (40%), market segment (35%),
 * and revenue similarity (25%) to determine competitive threat level.
 */
function calculateRelevanceScore(company: Company, competitor: Competitor): number {
  const industryWeight = 0.4;
  const marketWeight = 0.35;
  const revenueWeight = 0.25;
  // Implementation...
}
```

2. **API Integration Points:**
```typescript
/**
 * Enriches company profile using external data sources.
 *
 * This integrates with multiple APIs in sequence:
 * 1. Brønnøysund Register (official Norwegian business data)
 * 2. Proff.no API (financial information)
 * 3. Internal ML model for market segment classification
 *
 * @param companyId - Internal company profile ID
 * @returns Enriched profile or error if any API fails
 */
export async function enrichCompanyProfile(companyId: string): Promise<ActionResult<EnrichedProfile>> {
  // Implementation...
}
```

3. **Complex Algorithms or Calculations:**
```typescript
/**
 * Mock revenue generation for development.
 *
 * Generates realistic revenue ranges based on Norwegian market data:
 * - Enterprise (1000+ employees): 100M-600M NOK
 * - Medium (100-999 employees): 10M-60M NOK
 * - Small (<100 employees): 1M-11M NOK
 *
 * Based on SSB (Statistics Norway) business data patterns.
 */
function generateMockRevenue(marketSegment: string): number {
  // Implementation...
}
```

4. **Workarounds or Temporary Solutions:**
```typescript
// TODO: Replace with proper type guards once asset-core.ts is refactored
// Currently using runtime checks due to circular dependency issues
if ('data' in asset && typeof asset.data === 'object') {
  return asset as WebSearchAsset;
}
```

### What NOT to Comment

**AVOID these comment patterns:**

```typescript
// ❌ BAD: Obvious comments
const userId = user.id; // Get user ID
if (isLoading) {        // If loading
  return <Loader />;    // Show loader
}

// ❌ BAD: Redundant documentation
/**
 * Sets the loading state to true
 * @param loading - boolean value to set loading state
 */
function setLoading(loading: boolean) {
  setIsLoading(loading);
}

// ❌ BAD: Outdated comments
// This function handles user login (actually handles logout now)
function handleUserLogout() {
  // Implementation...
}
```

### JSDoc for Public APIs

**REQUIRED for all exported functions:**

```typescript
/**
 * Searches for companies using BigQuery with fuzzy matching.
 *
 * Supports searching by:
 * - Company name (fuzzy match)
 * - Industry keywords
 * - Location (city/region)
 * - Revenue range
 *
 * @param searchTerm - Company name or industry keyword
 * @param filters - Optional filtering criteria
 * @param limit - Maximum results to return (default: 50)
 * @returns Promise resolving to search results or error
 *
 * @example
 * ```typescript
 * const results = await searchCompanies("technology", {
 *   location: "Oslo",
 *   minRevenue: 10000000
 * });
 * ```
 */
export async function searchCompanies(
  searchTerm: string,
  filters?: SearchFilters,
  limit: number = 50
): Promise<ActionResult<CompanySearchResult[]>> {
  // Implementation...
}
```

### Component Documentation

**Document component purpose and usage:**

```typescript
/**
 * Displays competitor analysis results with real-time progress updates.
 *
 * Connects to PubSub for live updates and provides:
 * - Overall analysis progress
 * - Individual competitor progress
 * - Export functionality for results
 * - Error handling and retry options
 *
 * @param hubName - PubSub hub identifier for this analysis
 * @param assetId - Optional asset ID for linking results
 * @param onCompletion - Callback when analysis completes
 */
export function AnalysisStatus({ hubName, assetId, onCompletion }: AnalysisStatusProps) {
  // Implementation...
}
```

### Inline Comments for Clarification

**Use for non-obvious code:**

```typescript
export async function searchWebCompetitors(data: CompetitorSearchRequest) {
  // Azure Functions timeout after 5 minutes, but we set 2-minute client timeout
  // to fail fast and fallback to mock data for development
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), API_TIMEOUTS.ANALYSIS);

  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      signal: controller.signal, // Enables timeout cancellation
      // ... rest of config
    });

    // Clear timeout if request completes before timeout
    clearTimeout(timeoutId);

    // Check for both network errors and HTTP error status codes
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

  } catch (error) {
    // AbortError means timeout occurred, not a real network error
    if (error instanceof Error && error.name === 'AbortError') {
      console.log('API timeout - falling back to mock data for development');
      return generateMockCompetitors(data);
    }
    throw error;
  }
}
```

### Comment Maintenance

**Keep comments up-to-date:**

1. **Update comments when changing code**
2. **Remove obsolete comments immediately**
3. **Review TODO comments regularly**
4. **Use descriptive commit messages instead of inline comments for "why" changes were made**

---

## Frontend Development

Our Next.js application follows specific patterns for React components, hooks, and server actions.

### Component Architecture Patterns

#### 1. Feature-Based Component Organization

```typescript
// ✅ GOOD: Feature-specific components in feature folders
src/components/
├── ui/                           # Generic UI components (Button, Card, etc.)
├── company-profiles/             # Company profile feature components
│   ├── company-profile-form.tsx  # Main form component
│   ├── profile-card.tsx          # Display component
│   ├── company-search-results.tsx # Search results
│   └── context.tsx               # Feature context/state
├── competitor-analysis/          # Competitor analysis components
│   ├── interface.tsx            # Main interface (being refactored - too large)
│   ├── analysis-status.tsx      # Real-time status display
│   ├── query-interface.tsx      # Search interface (being refactored)
│   └── message-display.tsx      # Message components
└── assets/                      # Assets feature components
    ├── asset-list.tsx          # List view
    └── canvas-board.tsx        # Visual board
```

#### 2. Component Size and Responsibility

**STRICT RULE: Components must be <400 lines**

```typescript
// ✅ GOOD: Small, focused component with single responsibility
interface CompanyCardProps {
  company: CompanyProfile;
  isSelected?: boolean;
  onSelect?: (company: CompanyProfile) => void;
  onEdit?: (company: CompanyProfile) => void;
  onDelete?: (companyId: string) => void;
}

export function CompanyCard({ company, isSelected, onSelect, onEdit, onDelete }: CompanyCardProps) {
  const { enrichingProfileIds } = useCompanyProfiles();
  const isEnriching = company.enrichmentStatus === 'in_progress';

  return (
    <Card className={`cursor-pointer transition-all ${isSelected ? 'border-green-200' : ''}`}>
      <CardContent>
        <div className="flex justify-between">
          <div>
            <h3>{company.name}</h3>
            <p className="text-sm text-gray-600">{company.industry}</p>
          </div>
          <div className="flex gap-2">
            {onEdit && <Button size="sm" onClick={() => onEdit(company)}>Edit</Button>}
            {onDelete && <Button size="sm" variant="destructive" onClick={() => onDelete(company.id)}>Delete</Button>}
          </div>
        </div>
      </CardContent>
    </Card>
  );
}
```

#### 3. Component Composition Patterns

**Layer components for reusability:**

```typescript
// ✅ GOOD: Compose smaller components
export function CompanyProfilePage() {
  return (
    <div className="space-y-6">
      <CompanySearchSection />
      <CompanyProfileForm />
      <CompanyProfilesList />
    </div>
  );
}

// Each section is a focused component with its own responsibility
function CompanySearchSection() {
  // Only handles company search UI and logic
}

function CompanyProfileForm() {
  // Only handles form creation/editing
}

function CompanyProfilesList() {
  // Only handles displaying the list of profiles
}
```

### React Hooks Patterns

#### 1. Feature-Specific Hook Organization

```typescript
// Hook organization by feature:
src/features/
├── company-profiles/hooks/
│   ├── use-company-search.ts           # Company search logic
│   └── use-company-bigquery-search.ts  # BigQuery-specific search
├── competitor-analysis/hooks/
│   ├── use-pubsub.ts                   # Real-time updates
│   ├── use-analysis-completion.ts     # Analysis state management
│   └── use-competitor-web-search.ts   # Web search logic
├── assets/hooks/
│   └── use-assets.ts                   # Asset CRUD operations
└── shared/hooks/                       # Generic, reusable hooks
    ├── use-async-operation.ts          # Generic async operations
    └── use-form-validation.ts          # Generic form validation
```

#### 2. Async Operation Pattern

**ALWAYS use our shared `useAsyncOperation` hook:**

```typescript
// ✅ GOOD: Use shared async hook to eliminate duplication
import { useAsyncOperation } from '@/shared/hooks/use-async-operation';

export function CompanySearch() {
  const { isLoading, error, execute: searchCompanies } = useAsyncOperation<Company[]>();

  const handleSearch = async (searchTerm: string) => {
    const results = await searchCompanies(async () => {
      const response = await searchCompaniesAPI(searchTerm);
      if (!response.success) {
        throw new Error(response.error);
      }
      return response.data;
    });

    if (results) {
      // Handle successful results
    }
    // Error is automatically handled by the hook
  };

  return (
    <div>
      <SearchInput onSearch={handleSearch} disabled={isLoading} />
      {error && <ErrorMessage error={error} />}
      {isLoading && <LoadingSpinner />}
    </div>
  );
}

// ❌ BAD: Manual loading state management (duplicated code)
export function ManualAsyncComponent() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleOperation = async () => {
    setIsLoading(true);
    setError(null);
    try {
      // operation
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  // This pattern is now eliminated by useAsyncOperation
}
```

#### 3. Context Pattern for Feature State

**Use React Context for feature-wide state:**

```typescript
// ✅ GOOD: Feature-specific context
// src/components/company-profiles/context.tsx
interface CompanyProfilesContextType {
  profiles: CompanyProfile[];
  selectedProfile?: CompanyProfile;
  isLoading: boolean;
  selectProfile: (profile: CompanyProfile) => void;
  enrichingProfileIds: string[];
}

export function CompanyProfilesProvider({ children }: { children: React.ReactNode }) {
  // Context implementation with all company profile state
  return (
    <CompanyProfilesContext.Provider value={contextValue}>
      {children}
    </CompanyProfilesContext.Provider>
  );
}

export function useCompanyProfiles() {
  const context = useContext(CompanyProfilesContext);
  if (!context) {
    throw new Error('useCompanyProfiles must be used within CompanyProfilesProvider');
  }
  return context;
}
```

### Server Actions Pattern

**CRITICAL**: All server-side operations use Next.js Server Actions with our standardized error handling.

#### 1. Server Action Structure

```typescript
// ✅ GOOD: Standard server action pattern
'use server';

import { ActionResult } from '@/shared/utils/error-handler';
import { logger } from '@/shared/utils/logger';
import { revalidatePath } from 'next/cache';

export async function createCompanyProfile(formData: FormData): Promise<ActionResult<CompanyProfile>> {
  try {
    // 1. Extract and validate data
    const data = extractFormData(formData);
    const validation = validateCompanyData(data);

    if (!validation.success) {
      logger.warn('Company profile validation failed', {
        feature: 'company-profiles',
        action: 'create_profile',
        metadata: { validationErrors: validation.error }
      });
      return { success: false, error: validation.error };
    }

    // 2. Perform business logic
    logger.info('Creating company profile', {
      feature: 'company-profiles',
      action: 'create_profile',
      metadata: { companyName: data.name }
    });

    const profile = await companyRepository.create(data);

    // 3. Revalidate relevant paths
    revalidatePath('/company-profiles');

    logger.info('Company profile created successfully', {
      feature: 'company-profiles',
      action: 'create_profile',
      companyId: profile.id
    });

    return { success: true, data: profile };
  } catch (error) {
    logger.error('Failed to create company profile', {
      feature: 'company-profiles',
      action: 'create_profile'
    }, error as Error);

    return {
      success: false,
      error: 'Failed to create company profile. Please try again.'
    };
  }
}
```

#### 2. ActionResult Pattern Usage

**ALWAYS use ActionResult for operations that can fail:**

```typescript
// ✅ GOOD: ActionResult pattern in components
export function CompanyProfileForm() {
  const { execute: createProfile, isLoading, error } = useAsyncOperation<CompanyProfile>();

  const handleSubmit = async (formData: FormData) => {
    const result = await createProfile(async () => {
      const actionResult = await createCompanyProfile(formData);
      if (!actionResult.success) {
        throw new Error(actionResult.error);
      }
      return actionResult.data;
    });

    if (result) {
      // Handle successful creation
      router.push('/company-profiles');
    }
    // Error is automatically handled by useAsyncOperation
  };

  return (
    <form action={handleSubmit}>
      {/* Form fields */}
      <Button type="submit" disabled={isLoading}>
        {isLoading ? 'Creating...' : 'Create Profile'}
      </Button>
      {error && <ErrorMessage error={error} />}
    </form>
  );
}
```

#### 3. Server Action File Organization

```typescript
// Server actions organized by feature:
src/features/
├── company-profiles/
│   └── actions.ts              # Company profile CRUD operations
├── competitor-analysis/
│   └── web-search-actions.ts   # Competitor search operations
├── assets/
│   ├── actions.ts              # Main asset actions (delegator)
│   └── actions/                # Specific action groups
│       ├── crud.ts             # Create, Read, Update, Delete
│       ├── status.ts           # Status management
│       └── competitor-analysis.ts # Competitor-specific actions
└── assistant/
    └── actions.ts              # Assistant thread operations
```

### Real-Time Updates with PubSub

**Our applications use Azure Web PubSub for real-time features:**

```typescript
// ✅ GOOD: PubSub integration pattern
import { usePubSub } from '@/features/competitor-analysis/hooks/use-pubsub';

export function AnalysisStatus({ hubName }: { hubName: string }) {
  const {
    isConnected,
    error,
    messagesByType,
    connectionState
  } = usePubSub(hubName);

  // Real-time updates automatically handled by the hook
  return (
    <div>
      <Badge variant={isConnected ? 'default' : 'destructive'}>
        {isConnected ? 'Connected' : 'Disconnected'}
      </Badge>

      {messagesByType.general.map((message, index) => (
        <MessageDisplay key={index} message={message} />
      ))}

      {Object.entries(messagesByType.competitors).map(([name, messages]) => (
        <CompetitorProgress key={name} competitorName={name} messages={messages} />
      ))}
    </div>
  );
}
```

### UI Component Patterns

#### 1. Shared UI Components

**Use existing UI components from `@/components/ui/`:**

```typescript
// ✅ GOOD: Use existing components
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Input } from '@/components/ui/input';

// ❌ BAD: Don't create custom versions unless absolutely necessary
function CustomButton() { /* ... */ }
```

#### 2. Conditional Rendering Patterns

```typescript
// ✅ GOOD: Clear conditional rendering
export function CompanyCard({ company }: { company: CompanyProfile }) {
  const isEnriching = company.enrichmentStatus === 'in_progress';

  return (
    <Card>
      <CardContent>
        <div className="flex justify-between items-center">
          <h3>{company.name}</h3>

          {isEnriching ? (
            <Badge variant="outline" className="flex items-center gap-1">
              <Loader2 className="h-3 w-3 animate-spin" />
              Enriching...
            </Badge>
          ) : (
            <Badge variant="default">Ready</Badge>
          )}
        </div>
      </CardContent>
    </Card>
  );
}
```

#### 3. Loading States and Error Handling

```typescript
// ✅ GOOD: Consistent loading and error states
export function AssetList({ companyId }: { companyId: string }) {
  const { isLoading, error, execute: loadAssets } = useAsyncOperation<Asset[]>();
  const [assets, setAssets] = useState<Asset[]>([]);

  useEffect(() => {
    loadAssets(async () => {
      const result = await getAssetsByCompany(companyId);
      if (!result.success) throw new Error(result.error);
      return result.data;
    }).then(result => {
      if (result) setAssets(result);
    });
  }, [companyId]);

  if (isLoading) {
    return (
      <Card>
        <CardContent className="flex items-center justify-center py-8">
          <Loader2 className="h-6 w-6 animate-spin mr-2" />
          Loading assets...
        </CardContent>
      </Card>
    );
  }

  if (error) {
    return (
      <Card>
        <CardContent className="py-8">
          <div className="text-center text-red-600">
            <AlertCircle className="h-6 w-6 mx-auto mb-2" />
            {error}
          </div>
        </CardContent>
      </Card>
    );
  }

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {assets.map(asset => (
        <AssetCard key={asset.id} asset={asset} />
      ))}
    </div>
  );
}
```

---

## Backend Development

Our backend architecture covers both Next.js server actions and standalone Node.js functions (Azure Functions, AWS Lambda, etc.).

### Node.js Functions Architecture

#### 1. Project Structure for Node.js Functions

```
src/
├── functions/              # Function handlers (Azure Functions, AWS Lambda, etc.)
│   ├── enrichment/
│   │   ├── index.ts       # Main function handler
│   │   └── function.json  # Function configuration (Azure)
│   ├── competitor-search/
│   │   ├── index.ts       # Search function handler
│   │   └── function.json  # Function configuration
│   └── data-processing/
│       ├── index.ts       # Processing handler
│       └── function.json  # Function configuration
├── features/              # Business domain modules
│   ├── enrichment/
│   │   ├── types/         # Enrichment-specific types
│   │   ├── services/      # Business logic services
│   │   ├── repositories/  # Data access layer
│   │   └── utils/         # Feature-specific utilities
│   ├── competitor-analysis/
│   │   ├── types/         # Analysis types and interfaces
│   │   ├── services/      # Analysis business logic
│   │   ├── repositories/  # Data persistence
│   │   └── utils/         # Analysis utilities
│   └── data-processing/
│       ├── types/         # Data processing types
│       ├── services/      # Processing logic
│       └── utils/         # Processing utilities
├── lib/                   # Infrastructure services
│   ├── databases/         # Database connections
│   │   ├── mongodb.ts     # MongoDB client
│   │   ├── bigquery.ts    # BigQuery client
│   │   └── redis.ts       # Redis client
│   ├── external-apis/     # Third-party integrations
│   │   ├── proff-api.ts   # Proff.no API client
│   │   ├── openai.ts      # OpenAI integration
│   │   └── azure-search.ts # Azure Search client
│   ├── storage/           # File/blob storage
│   │   ├── azure-storage.ts # Azure Blob Storage
│   │   └── gcs.ts         # Google Cloud Storage
│   └── messaging/         # Queues and pub/sub
│       ├── service-bus.ts # Azure Service Bus
│       └── pubsub.ts      # Google Pub/Sub
└── shared/                # Truly generic utilities
    ├── utils/             # Generic utilities
    │   ├── logger.ts      # Structured logging
    │   ├── retry.ts       # Retry mechanisms
    │   ├── formatting.ts  # Data formatting
    │   └── error-handler.ts # Error handling
    ├── constants/         # Application constants
    └── types/             # Global type definitions
```

#### 2. Function Handler Pattern

**Azure Functions Example:**

```typescript
// src/functions/enrichment/index.ts
import { AzureFunction, Context, HttpRequest } from '@azure/functions';
import { enrichCompany } from '@/features/enrichment/services/enrichment-service';
import { logger } from '@/shared/utils/logger';
import { withErrorHandling, ActionResult } from '@/shared/utils/error-handler';
import { API_TIMEOUTS } from '@/shared/constants';

interface EnrichmentRequest {
  companyId: string;
  companyName: string;
  orgNumber?: string;
}

const httpTrigger: AzureFunction = async (context: Context, req: HttpRequest) => {
  // Set function timeout
  context.executionContext.functionTimeoutInMilliseconds = API_TIMEOUTS.ENRICHMENT;

  const result = await withErrorHandling(async (): Promise<ActionResult<any>> => {
    logger.info('Enrichment function triggered', {
      feature: 'enrichment',
      action: 'function_start',
      metadata: { invocationId: context.invocationId }
    });

    // Validate request
    const requestData = req.body as EnrichmentRequest;
    if (!requestData?.companyId || !requestData?.companyName) {
      return {
        success: false,
        error: 'Missing required fields: companyId and companyName'
      };
    }

    // Perform enrichment
    const enrichmentResult = await enrichCompany({
      companyId: requestData.companyId,
      companyName: requestData.companyName,
      orgNumber: requestData.orgNumber
    });

    logger.info('Enrichment function completed', {
      feature: 'enrichment',
      action: 'function_complete',
      metadata: {
        companyId: requestData.companyId,
        success: enrichmentResult.success
      }
    });

    return enrichmentResult;
  });

  // Return appropriate HTTP response
  context.res = {
    status: result.success ? 200 : 400,
    headers: { 'Content-Type': 'application/json' },
    body: result
  };
};

export default httpTrigger;
```

**AWS Lambda Example:**

```typescript
// src/functions/competitor-search/index.ts
import { APIGatewayProxyHandler, APIGatewayProxyResult } from 'aws-lambda';
import { searchCompetitors } from '@/features/competitor-analysis/services/competitor-service';
import { logger } from '@/shared/utils/logger';
import { withErrorHandling } from '@/shared/utils/error-handler';

export const handler: APIGatewayProxyHandler = async (event, context): Promise<APIGatewayProxyResult> => {
  const result = await withErrorHandling(async () => {
    logger.info('Competitor search function triggered', {
      feature: 'competitor-analysis',
      action: 'lambda_start',
      metadata: { requestId: context.awsRequestId }
    });

    const requestData = JSON.parse(event.body || '{}');

    return await searchCompetitors({
      companyProfile: requestData.companyProfile,
      searchCriteria: requestData.searchCriteria,
      maxResults: requestData.maxResults || 10
    });
  });

  return {
    statusCode: result.success ? 200 : 400,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*'
    },
    body: JSON.stringify(result)
  };
};
```

### Database Layer Pattern

**ALWAYS use Repository Pattern with interfaces:**

```typescript
// src/features/enrichment/repositories/company-repository.ts
import { ActionResult } from '@/shared/utils/error-handler';
import { logger } from '@/shared/utils/logger';

export interface ICompanyRepository {
  create(data: CreateCompanyRequest): Promise<Company>;
  findById(id: string): Promise<Company | null>;
  findByOrgNumber(orgNumber: string): Promise<Company | null>;
  update(id: string, updates: Partial<Company>): Promise<Company | null>;
  delete(id: string): Promise<boolean>;
  search(criteria: SearchCriteria): Promise<Company[]>;
}

export class MongoCompanyRepository implements ICompanyRepository {
  constructor(private db: Db) {}

  async create(data: CreateCompanyRequest): Promise<Company> {
    logger.dbStart('create', 'companies', {
      feature: 'enrichment',
      metadata: { companyName: data.name }
    });

    const result = await this.db.collection('companies').insertOne({
      ...data,
      createdAt: new Date(),
      updatedAt: new Date()
    });

    const company = { ...data, id: result.insertedId.toString() };

    logger.dbSuccess('create', 'companies', 1, {
      feature: 'enrichment',
      companyId: company.id
    });

    return company;
  }

  async findById(id: string): Promise<Company | null> {
    logger.dbStart('findById', 'companies', {
      feature: 'enrichment',
      metadata: { companyId: id }
    });

    const result = await this.db.collection('companies').findOne({
      _id: new ObjectId(id)
    });

    if (result) {
      logger.dbSuccess('findById', 'companies', 1, {
        feature: 'enrichment',
        companyId: id
      });
      return { ...result, id: result._id.toString() };
    }

    return null;
  }

  // ... other methods following same pattern
}
```

### Service Layer Pattern

**Business logic services with dependency injection:**

```typescript
// src/features/enrichment/services/enrichment-service.ts
import { ICompanyRepository } from '@/features/enrichment/repositories/company-repository';
import { IEnrichmentAPI } from '@/lib/external-apis/enrichment-api';
import { logger } from '@/shared/utils/logger';
import { withRetry } from '@/shared/utils/retry';
import { ActionResult, withErrorHandling } from '@/shared/utils/error-handler';

export interface EnrichmentRequest {
  companyId: string;
  companyName: string;
  orgNumber?: string;
}

export class EnrichmentService {
  constructor(
    private companyRepo: ICompanyRepository,
    private proffAPI: IEnrichmentAPI,
    private openAIService: IOpenAIService
  ) {}

  async enrichCompany(request: EnrichmentRequest): Promise<ActionResult<EnrichedCompany>> {
    return withErrorHandling(async () => {
      logger.featureUsage('enrichment', 'enrich_company', {
        companyId: request.companyId,
        metadata: { companyName: request.companyName }
      });

      // 1. Fetch existing company data
      const company = await this.companyRepo.findById(request.companyId);
      if (!company) {
        throw new AppError('Company not found', 'COMPANY_NOT_FOUND', 404);
      }

      // 2. Enrich with external data sources (with retry)
      const proffData = await withRetry(async () => {
        return await this.proffAPI.getCompanyData(company.name, request.orgNumber);
      });

      // 3. Generate AI insights
      const aiInsights = await this.openAIService.generateInsights({
        companyName: company.name,
        industry: company.industry,
        externalData: proffData
      });

      // 4. Combine and save enriched data
      const enrichedCompany = {
        ...company,
        ...proffData,
        aiInsights,
        enrichedAt: new Date(),
        enrichmentStatus: 'completed' as const
      };

      await this.companyRepo.update(request.companyId, enrichedCompany);

      logger.info('Company enrichment completed successfully', {
        feature: 'enrichment',
        companyId: request.companyId,
        metadata: {
          dataSources: ['proff', 'openai'],
          enrichmentFields: Object.keys(proffData).length
        }
      });

      return enrichedCompany;
    });
  }
}

// Factory function for dependency injection
export function createEnrichmentService(): EnrichmentService {
  const companyRepo = new MongoCompanyRepository(getDatabase());
  const proffAPI = new ProffAPIClient(process.env.PROFF_API_KEY);
  const openAIService = new OpenAIService(process.env.OPENAI_API_KEY);

  return new EnrichmentService(companyRepo, proffAPI, openAIService);
}

// Main export for function handlers
export async function enrichCompany(request: EnrichmentRequest): Promise<ActionResult<EnrichedCompany>> {
  const service = createEnrichmentService();
  return await service.enrichCompany(request);
}
```

### External API Integration Pattern

```typescript
// src/lib/external-apis/proff-api.ts
import { withRetry } from '@/shared/utils/retry';
import { logger } from '@/shared/utils/logger';
import { API_TIMEOUTS } from '@/shared/constants';

export interface IEnrichmentAPI {
  getCompanyData(companyName: string, orgNumber?: string): Promise<EnrichmentData>;
}

export class ProffAPIClient implements IEnrichmentAPI {
  constructor(private apiKey: string) {}

  async getCompanyData(companyName: string, orgNumber?: string): Promise<EnrichmentData> {
    return withRetry(async () => {
      logger.apiStart('/company-data', 'GET', {
        feature: 'enrichment',
        metadata: { companyName, orgNumber }
      });

      const startTime = Date.now();

      const response = await fetch(`${this.baseUrl}/company-data`, {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        },
        signal: AbortSignal.timeout(API_TIMEOUTS.ENRICHMENT)
      });

      const duration = Date.now() - startTime;

      if (!response.ok) {
        logger.apiError('/company-data', new Error(`HTTP ${response.status}`), {
          feature: 'enrichment',
          metadata: { companyName, statusCode: response.status }
        });
        throw new Error(`API error: ${response.status} ${response.statusText}`);
      }

      const data = await response.json();

      logger.apiSuccess('/company-data', duration, {
        feature: 'enrichment',
        metadata: { companyName, dataFields: Object.keys(data).length }
      });

      return this.transformResponse(data);
    });
  }

  private transformResponse(rawData: any): EnrichmentData {
    // Transform external API response to internal format
    return {
      revenue: rawData.omsetning,
      employees: rawData.ansatte,
      industry: rawData.bransje,
      // ... other transformations
    };
  }
}
```

---

## Error Handling & Resilience

### ActionResult Pattern

**Use this pattern for all operations that can fail:**

```typescript
// shared/utils/error-handler.ts
export type ActionResult<T> = {
  success: true;
  data: T;
} | {
  success: false;
  error: string;
};

export async function withErrorHandling<T>(
  operation: () => Promise<T>,
  errorMessage?: string
): Promise<ActionResult<T>> {
  try {
    const data = await operation();
    return { success: true, data };
  } catch (error) {
    console.error('Operation failed:', error);
    return {
      success: false,
      error: errorMessage || (error instanceof Error ? error.message : 'Operation failed')
    };
  }
}
```

### Retry Pattern

**Use for external API calls and unreliable operations:**

```typescript
// shared/utils/retry.ts
import { withRetry } from '@/shared/utils/retry';

// Usage in service
async function fetchCompanyData(companyName: string): Promise<CompanyData> {
  return withRetry(
    () => externalAPI.getCompanyData(companyName),
    {
      maxAttempts: 3,
      baseDelayMs: 1000,
      maxDelayMs: 10000
    }
  );
}
```

### Application Errors

**Create typed error classes:**

```typescript
// shared/utils/error-handler.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// Usage
throw new AppError('Invalid company data', 'INVALID_COMPANY_DATA', 400);
```

---

## Performance Guidelines

### Database Optimization

```typescript
// ✅ Use appropriate indexes and projections
const companies = await db.collection('companies')
  .find({ industry: 'technology' })
  .project({ name: 1, email: 1, revenue: 1 })
  .limit(50)
  .toArray();
```

### Frontend Performance

```typescript
// ✅ Use React.memo for expensive components
export const ExpensiveComponent = React.memo(({ data }: { data: ComplexData }) => {
  const processedData = useMemo(() => processComplexData(data), [data]);

  return <div>{/* Render processed data */}</div>;
});

// ✅ Use useCallback for event handlers
const handleSubmit = useCallback(async (formData: FormData) => {
  await submitForm(formData);
}, []);
```

---

## Implementation Checklist

When creating new features or refactoring existing code:

### Architecture
- [ ] Feature is self-contained in its own folder
- [ ] Dependencies are injected, not hard-coded
- [ ] Interfaces are defined for external dependencies
- [ ] Error handling uses ActionResult pattern

### Code Quality
- [ ] File is under 400 lines
- [ ] Functions are under 20 lines
- [ ] No magic numbers (use named constants)
- [ ] Clear, descriptive naming
- [ ] Type safety (no `any` types)

### Frontend (if applicable)
- [ ] Components have single responsibility
- [ ] Server actions handle errors properly
- [ ] Loading and error states are handled
- [ ] Memoization used where appropriate

### Backend (if applicable)
- [ ] Repository pattern for data access
- [ ] Service layer for business logic
- [ ] Retry logic for external API calls
- [ ] Comprehensive error logging


---

## Migration Guidelines

When working with existing code that doesn't follow these guidelines:

1. **Incremental Refactoring**: Don't rewrite everything at once
2. **Extract First**: Move functionality to new structure before changing logic
3. **Maintain Compatibility**: Ensure existing functionality continues to work
4. **Validate Changes**: Ensure functionality works before refactoring
5. **Document Changes**: Update documentation and comments

---

This document should be imported into your IDE's AI agent configuration to ensure consistent code generation and suggestions across all team members and projects.
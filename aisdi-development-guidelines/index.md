---
title: Development Guidelines - KNOBO Code Standards
layout: default
---

# Development Guidelines - KNOBO Code Standards

**Version**: 1.1
**Last Updated**: 2025-01-27
**Scope**: Next.js Web Applications & Node.js Functions

This document defines our unified coding standards, architecture patterns, and best practices across all KNOBO projects to ensure consistency, maintainability, and scalability.

## Table of Contents

1. [Core Principles](#core-principles)
2. [AI Assistant Guidelines](#ai-assistant-guidelines)
3. [Project Structure](#project-structure)
4. [Code Quality Standards](#code-quality-standards)
5. [Frontend Development](#frontend-development)
6. [Backend Development](#backend-development)
7. [Error Handling & Resilience](#error-handling--resilience)
8. [Implementation Checklist](#implementation-checklist)

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

### Code Reuse Priorities

**MANDATORY ORDER** - Check these locations before creating new code:

1. **`src/shared/`** - Generic utilities, hooks, constants
2. **`src/features/*/`** - Feature-specific types, components, services
3. **`src/lib/`** - Infrastructure services (auth, database, external APIs)
4. **`src/components/ui/`** - Base UI components

### Before Creating New Code

**ASK THESE QUESTIONS:**

- ✅ "Does a similar type/interface already exist that I can extend?"
- ✅ "Is there an existing utility function that does something similar?"
- ✅ "Can I reuse an existing component and just modify props?"
- ✅ "Is there already a hook that handles similar state/logic?"

---

## Project Structure

### Next.js Web Applications

```
src/
├── app/                    # Next.js App Router pages
├── features/              # Business domain modules
│   └── {feature-name}/
│       ├── actions.ts     # Server actions
│       ├── types/         # Feature-specific types
│       ├── hooks/         # Feature-specific hooks
│       └── utils/         # Feature-specific utilities
├── components/            # UI components organized by feature
│   ├── ui/               # Generic UI components
│   └── {feature-name}/   # Feature-specific components
├── lib/                  # Infrastructure & shared services
└── shared/               # Truly generic utilities
```

---

## Code Quality Standards

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

// ❌ Unclear abbreviations
const getUsrById = (id: string) => { /* ... */ };
```

---

## Frontend Development

### Component Architecture Patterns

**STRICT RULE: Components must be <400 lines**

```typescript
// ✅ GOOD: Small, focused component
export function CompanyCard({ company, onSelect }: CompanyCardProps) {
  return (
    <Card>
      <CardContent>
        <h3>{company.name}</h3>
        <p>{company.industry}</p>
      </CardContent>
    </Card>
  );
}
```

### React Hooks Patterns

**ALWAYS use our shared `useAsyncOperation` hook:**

```typescript
import { useAsyncOperation } from '@/shared/hooks/use-async-operation';

export function CompanySearch() {
  const { isLoading, error, execute: searchCompanies } = useAsyncOperation<Company[]>();

  // Implementation...
}
```

---

## Backend Development

### Server Actions Pattern

```typescript
'use server';

export async function createCompanyProfile(formData: FormData): Promise<ActionResult<CompanyProfile>> {
  try {
    const validation = validateCompanyData(data);
    if (!validation.success) {
      return { success: false, error: validation.error };
    }

    const profile = await companyRepository.create(data);
    revalidatePath('/company-profiles');

    return { success: true, data: profile };
  } catch (error) {
    return { success: false, error: 'Failed to create company profile' };
  }
}
```

---

## Error Handling & Resilience

### ActionResult Pattern

```typescript
export type ActionResult<T> = {
  success: true;
  data: T;
} | {
  success: false;
  error: string;
};
```

---

## Implementation Checklist

### Architecture
- [ ] Feature is self-contained in its own folder
- [ ] Dependencies are injected, not hard-coded
- [ ] Error handling uses ActionResult pattern

### Code Quality
- [ ] File is under 400 lines
- [ ] Functions are under 20 lines
- [ ] Clear, descriptive naming
- [ ] Type safety (no `any` types)

---

*Development Guidelines | Team: KNOBO*
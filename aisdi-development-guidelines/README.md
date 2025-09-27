# AISDI Development Guidelines

Code standards and best practices for the AISDI development team.

## Project Structure

Maintain consistent project organization across all AISDI projects:

```
src/
├── components/          # Reusable UI components
├── pages/              # Page components
├── services/           # API and business logic
├── utils/              # Helper functions
├── types/              # TypeScript definitions
└── styles/             # Global styles and themes
```

## Code Quality Standards

All code must meet these quality requirements:

- ✅ TypeScript for type safety
- ✅ ESLint and Prettier for consistent formatting
- ✅ 100% test coverage for critical business logic
- ✅ Meaningful variable and function names
- ✅ Comprehensive documentation for public APIs

## Git Workflow

### ✅ Do
- Use feature branches
- Write descriptive commit messages
- Squash commits before merge
- Require PR reviews

### ❌ Don't
- Commit directly to main
- Use vague commit messages
- Skip code reviews
- Leave WIP commits in history

## Feature Development Checklist

Use this checklist for every feature:

- [ ] Feature branch created from main
- [ ] Code follows team standards
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] PR created with detailed description
- [ ] Code reviewed by team member
- [ ] All CI checks passing

---

*AISDI Development Guidelines | Team: AISDI*
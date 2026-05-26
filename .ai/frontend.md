---
description: Specific instructions for Angular frontend development. Activates when working on templates, styles, or TypeScript components.
globs: frontend/**/src/**/*.{ts,html,css,scss}
alwaysApply: false
---

# Angular Frontend Architecture Rules

## 1. Component Design
- **Standalone First:** Always create standalone components (`standalone: true`) unless explicitly working inside an older legacy module.
- **Change Detection:** Prefer `ChangeDetectionStrategy.OnPush` to optimize rendering performance.
- **State Management:** Prioritize Angular **Signals** (`signal()`, `computed()`) for local component state over complex RxJS flows where applicable.

## 2. Code Consistency
- Write strict TypeScript. Avoid `any`. Define strong interfaces for API response bodies.
- Follow the Angular coding style guide for file naming: `name.component.ts`, `name.service.ts`.
- Keep templates clean. Extract complex logical structures or repeating UI elements into sub-components.

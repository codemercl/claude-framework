# Skill: Bus Domain Scaffold

Create a complete bus/ domain module with all standard directories and files.

## When to use

When adding a new feature domain to the application.

## Directory Structure

```
bus/{domain}/
├── actions.ts
├── reducer.ts
├── types/
│   ├── actions.ts
│   ├── reducer.ts
│   └── index.ts          (shared domain types)
├── components/            (smart UI with hooks)
├── elements/              (dumb UI, props only)
├── hooks/                 (useQuery*, useMutation*, custom hooks)
├── helpers/               (domain utility functions)
└── loaders/               (server-side data loading)
```

## Steps

1. Create directory structure
2. Create types/reducer.ts with initial state type
3. Create types/actions.ts with action constants and union type
4. Create reducer.ts using `redux-module` skill
5. Create actions.ts using `redux-module` skill
6. Register reducer in `init/rootReducer.ts`:
   ```typescript
   import { {domain}Reducer } from '../bus/{domain}/reducer';

   // Add to combineReducers:
   {domain}: {domain}Reducer,
   ```
7. Add selector in `core/helpers/selectors.ts`:
   ```typescript
   export const {domain}Selector = (state: RootStateType): {Domain}StateType => state.{domain};
   ```

## When NOT to create a full domain

- If the feature only needs GraphQL data (no Redux state) → just create hooks/ in existing domain
- If it's shared across domains → put in core/
- If it's theme-specific → put in platform/

## Integration checklist

- [ ] Reducer registered in rootReducer.ts
- [ ] Selector added to selectors.ts
- [ ] No cross-domain imports planned
- [ ] Types follow naming convention ({Domain}StateType, {Domain}ActionType)

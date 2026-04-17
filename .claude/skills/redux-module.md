# Skill: Redux Module

Create a complete Redux module (actions + reducer + types) for a bus domain.

## When to use

When a bus domain needs Redux state management.

## File Structure

```
bus/{domain}/
├── actions.ts
├── reducer.ts
└── types/
    ├── actions.ts
    └── reducer.ts
```

## types/actions.ts

```typescript
export const {DOMAIN}_ACTION_ONE = '{DOMAIN}_ACTION_ONE';
type ActionOneType = {
  type: typeof {DOMAIN}_ACTION_ONE;
  payload: {PayloadType};
};

export const {DOMAIN}_ACTION_TWO = '{DOMAIN}_ACTION_TWO';
type ActionTwoType = {
  type: typeof {DOMAIN}_ACTION_TWO;
  payload: {PayloadType};
};

export type {Domain}ActionType =
  | ActionOneType
  | ActionTwoType;
```

## types/reducer.ts

```typescript
export type {Domain}StateType = {
  isActive: boolean;
  data: {DataType} | null;
  // domain-specific state fields
};
```

## actions.ts

```typescript
// Other
import {
  {DOMAIN}_ACTION_ONE,
  {DOMAIN}_ACTION_TWO,
  {Domain}ActionType,
} from './types/actions';

export const actionOne = (payload: {PayloadType}): {Domain}ActionType => ({
  type: {DOMAIN}_ACTION_ONE,
  payload,
});

export const actionTwo = (payload: {PayloadType}): {Domain}ActionType => ({
  type: {DOMAIN}_ACTION_TWO,
  payload,
});
```

## reducer.ts

```typescript
// Other
import {
  {DOMAIN}_ACTION_ONE,
  {DOMAIN}_ACTION_TWO,
  {Domain}ActionType,
} from './types/actions';
import { {Domain}StateType } from './types/reducer';

export const initial{Domain}State: {Domain}StateType = {
  isActive: false,
  data: null,
};

export const {domain}Reducer = (
  state: {Domain}StateType = initial{Domain}State,
  action: {Domain}ActionType,
): {Domain}StateType => {
  switch (action.type) {
    case {DOMAIN}_ACTION_ONE:
      return {
        ...state,
        // update state
      };
    case {DOMAIN}_ACTION_TWO:
      return {
        ...state,
        // update state
      };
    default: {
      const exhaustiveCheck: never = action;

      return state;
    }
  }
};
```

## Integration

After creating the module, register it in:
- `init/rootReducer.ts` — add to combined reducers
- `core/helpers/selectors.ts` — add selector if needed

## Rules

- Action constants: `SCREAMING_SNAKE_CASE` with domain prefix
- Reducer must be pure — no side effects
- State must be serializable — no Date objects, no functions
- Exhaustive switch with `never` check in default
- All types end with `Type` suffix

---
globs: "**/reducer.ts,**/actions.ts,**/saga.ts,**/sagas/**,**/types/actions.ts,**/types/reducer.ts"
---

# Redux Conventions

## Module Structure

```
bus/{domain}/
├── actions.ts           — action creators returning typed actions
├── reducer.ts           — pure reducer with exhaustive switch
├── types/
│   ├── actions.ts       — action constants + action type definitions + union type
│   └── reducer.ts       — state type definition
└── saga.ts / sagas/     — side effects with redux-saga
```

## Action Types (types/actions.ts)

```typescript
export const PROFILE_SET_DATA = 'PROFILE_SET_DATA';
type SetDataActionType = {
  type: typeof PROFILE_SET_DATA;
  payload: ProfileDataType;
};

export type ProfileActionType = SetDataActionType | OtherActionType;
```

## Reducer

- Traditional Redux — NO Redux Toolkit
- Must be pure: no side effects, API calls, timers, direct mutations
- State must be serializable: timestamps or ISO strings, never Date objects, no functions/classes
- Exhaustive switch with `never` check: `default: { const x: never = action; return state; }`
- Guard: `if (!('type' in action)) return state;`
- Immutable updates via spread: `{ ...state, [key]: newValue }`

## Selectors

- Custom selector hooks: `const { data } = useSelector(profileSelector)`
- Memoized selectors with reselect: `createSelector` for derived data
- Selectors defined in `core/helpers/selectors.ts`

## Sagas

- Workers must have try-catch and return `SagaIterator`
- Use correct effects: `yield call()`, `yield put()`, `yield takeLatest()`, `yield delay()`, `yield all()`
- Side effects (API calls, timers) belong ONLY in sagas, never in reducers

## Forbidden

- Redux Toolkit (RTK) — prohibited
- Dispatching in render phase
- Non-serializable payloads (Date, Map, Set, class instances)
- Side effects in reducers
- Direct state mutation

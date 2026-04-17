---
globs: "**/*.graphql,**/hooks/useQuery*/**,**/hooks/useMutation*/**"
---

# GraphQL Conventions

## Operation Naming

- Queries: `QUERY_DOMAIN_ENTITY` — VerbNoun style: `QUERY_PROFILE_SEARCH_PROFILES`
- Mutations: `MUTATION_DOMAIN_ACTION` — `MUTATION_CUSTOMER_REGISTER`
- Fragments: entity-based, reusable: `fragment StoryFragment on RbbStory { ... }`

## Entity Naming Consistency

Entity name must match across ALL layers — change one, change all in same MR:
- GraphQL file: `queryUserProfile.graphql`
- Operation name: `QUERY_USER_PROFILE`
- Generated type import: `QUERY_USER_PROFILE as UserProfileType`
- Variables type import: `QUERY_USER_PROFILEVariables as UserProfileVariablesType`
- Hook: `useQueryUserProfile`
- Directory: `useQueryUserProfile/`

## Generated Types

- Types ALWAYS imported from `__generated__/` — never create manual GraphQL types
- Import pattern with aliasing:
  ```typescript
  import {
    QUERY_USER_PROFILE as UserProfileType,
    QUERY_USER_PROFILEVariables as UserProfileVariablesType,
  } from './gql/__generated__/QUERY_USER_PROFILE';
  ```

## Error Handling

- Always log errors: `logger.client.push({ name: 'hookName error', error })`
- User-facing errors: `prepare.error.message(error)`
- Error code mapping: `ErrorCodeType` from `core/types/dictionaries/errorCode.ts` (UNAUTHENTICATED, LACK_CREDITS, etc.)

## Apollo Cache

- Use `InMemoryCache()` with `typePolicies` for custom cache behavior
- Mutation cache updates via `refetchQueries` or `update` function
- SSR queries: `initializeApp` + `collectQueriesExecutionDurations` wrapper for performance monitoring

## Forbidden

- No inline GraphQL strings — always separate `.graphql` files
- No manual type definitions for GraphQL — always `__generated__/`
- No custom error handling — use existing `logger` and `prepare.error` helpers

## After changes

Run `yarn apollo:generate-types` after changing any `.graphql` file.

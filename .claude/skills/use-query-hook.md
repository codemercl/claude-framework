# Skill: useQuery Hook

Create an Apollo Client query hook with proper typing, error logging, and colocated GraphQL.

## When to use

When adding a new GraphQL query to fetch data.

## Naming Convention

Hook name determines ALL other names:
- Hook: `useQuery{Entity}` (e.g. `useQueryProfileSearch`)
- Directory: `useQueryProfileSearch/`
- GraphQL file: `queryProfileSearch.graphql`
- Operation: `QUERY_PROFILE_SEARCH`
- Generated type: `QUERY_PROFILE_SEARCH` as `ProfileSearchType`
- Variables type: `QUERY_PROFILE_SEARCHVariables` as `ProfileSearchVariablesType`

## File Structure

```
hooks/useQuery{Entity}/
├── index.ts
└── gql/
    ├── query{Entity}.graphql
    └── __generated__/          (auto-generated, don't create)
        └── QUERY_{ENTITY}.ts
```

## gql/query{Entity}.graphql

```graphql
query QUERY_{ENTITY}($input: {InputType}!) {
  {fieldName}(input: $input) {
    # fields from schema.graphql
  }
}
```

Without variables:
```graphql
query QUERY_{ENTITY} {
  {fieldName} {
    # fields
  }
}
```

## index.ts (with variables)

```typescript
// Core
import { useQuery, ApolloError } from '@apollo/client';

// Other
import {
  QUERY_{ENTITY} as {Entity}Type,
  QUERY_{ENTITY}Variables as {Entity}VariablesType,
} from './gql/__generated__/QUERY_{ENTITY}';
import query{Entity} from './gql/query{Entity}.graphql';
import { logger } from '../../../../core/helpers/logger';

type ReturnedType = {
  data?: {Entity}Type;
  isLoading: boolean;
  error?: ApolloError;
};

export const useQuery{Entity} = (variables: {Entity}VariablesType): ReturnedType => {
  const { data, loading, error } = useQuery<{Entity}Type, {Entity}VariablesType>(
    query{Entity},
    {
      variables,
      fetchPolicy: 'cache-and-network',
    },
  );

  if (!loading && error) {
    logger.client.push({
      name: 'useQuery{Entity} hook error',
      error,
    });
  }

  return {
    data,
    isLoading: loading,
    error,
  };
};
```

## index.ts (without variables)

```typescript
// Core
import { useQuery, ApolloError } from '@apollo/client';

// Other
import {
  QUERY_{ENTITY} as {Entity}Type,
} from './gql/__generated__/QUERY_{ENTITY}';
import query{Entity} from './gql/query{Entity}.graphql';
import { logger } from '../../../../core/helpers/logger';

type ReturnedType = {
  data?: {Entity}Type;
  isLoading: boolean;
  error?: ApolloError;
};

export const useQuery{Entity} = (): ReturnedType => {
  const { data, loading, error } = useQuery<{Entity}Type>(query{Entity});

  if (!loading && error) {
    logger.client.push({
      name: 'useQuery{Entity} hook error',
      error,
    });
  }

  return {
    data,
    isLoading: loading,
    error,
  };
};
```

## Common options

- `skip: !isAuthorized` — skip query if user not logged in
- `fetchPolicy: 'network-only'` — always fetch from server
- `pollInterval: 1000` — poll every second
- `notifyOnNetworkStatusChange: true` — re-render on refetch

## After creation

Run `yarn apollo:generate-types` to generate `__generated__/` types.

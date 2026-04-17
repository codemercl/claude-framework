---
name: new-hook
description: Scaffold a new GraphQL hook (useQuery or useMutation) with gql/ directory, following project conventions. Usage: /new-hook <domain> <hookName> (e.g. /new-hook profile useQueryProfileSearch)
---

Create a new GraphQL hook in `front/src/bus/$ARGUMENTS`.

Parse arguments: first arg is domain (e.g. `profile`), second is hookName (e.g. `useQueryProfileSearch` or `useMutationAddFavorite`).

## Steps

1. Determine hook type from name prefix: `useQuery` → query, `useMutation` → mutation.

2. Derive naming from hookName:
   - hookName: `useQueryProfileSearch` or `useMutationAddFavorite`
   - operationType: `query` or `mutation`
   - operationName: strip `useQuery`/`useMutation` prefix → `ProfileSearch` or `AddFavorite`
   - graphqlOperationName: `QUERY_PROFILE_SEARCH` or `MUTATION_ADD_FAVORITE` (SCREAMING_SNAKE_CASE)
   - graphqlFileName: `queryProfileSearch.graphql` or `mutationAddFavorite.graphql`

3. Create directory: `bus/{domain}/hooks/{hookName}/`

4. Create `gql/{graphqlFileName}`:

For query:
```graphql
query {graphqlOperationName} {
  # TODO: add fields from schema.graphql
}
```

For mutation:
```graphql
mutation {graphqlOperationName}($input: TODO!) {
  # TODO: add fields from schema.graphql
}
```

5. Create `index.ts`:

For useQuery:
```typescript
// Core
import { useQuery } from '@apollo/client';

// Other
import {
  {graphqlOperationName} as {OperationName}Type,
} from './gql/__generated__/{graphqlOperationName}';
import {graphqlFileNameNoExt} from './gql/{graphqlFileName}';
import { logger } from '../../../../core/helpers/logger';

type ReturnedType = {
  data?: {OperationName}Type;
  isLoading: boolean;
};

export const {hookName} = (): ReturnedType => {
  const { data, loading, error } = useQuery<{OperationName}Type>(
    {graphqlFileNameNoExt},
  );

  if (!loading && error) {
    logger.client.push({
      name: '{hookName} hook error',
      error,
    });
  }

  return {
    data,
    isLoading: loading,
  };
};
```

For useMutation:
```typescript
// Core
import { useMutation, FetchResult } from '@apollo/client';

// Other
import {
  {graphqlOperationName} as {OperationName}Type,
  {graphqlOperationName}Variables as {OperationName}VariablesType,
} from './gql/__generated__/{graphqlOperationName}';
import {graphqlFileNameNoExt} from './gql/{graphqlFileName}';
import { logger } from '../../../../core/helpers/logger';

type ReturnedType = {
  submit: (variables: {OperationName}VariablesType) => Promise<FetchResult<{OperationName}Type>>;
  isLoading: boolean;
  data?: {OperationName}Type | null;
};

export const {hookName} = (): ReturnedType => {
  const [submitFn, { loading, data, error }] = useMutation<
    {OperationName}Type,
    {OperationName}VariablesType
  >({graphqlFileNameNoExt});

  if (!loading && error) {
    logger.client.push({
      name: '{hookName} hook error',
      error,
    });
  }

  const submit = (
    variables: {OperationName}VariablesType,
  ): Promise<FetchResult<{OperationName}Type>> => submitFn({ variables });

  return {
    submit,
    isLoading: loading,
    data,
  };
};
```

6. Remind user to:
   - Fill in the GraphQL operation fields from `front/schema.graphql`
   - Run `yarn apollo:generate-types` to generate `__generated__/` types
   - Adjust `ReturnedType` after types are generated

## Rules
- Entity name consistent across: graphql file, operation name, hook name, directory
- Types imported from `__generated__/` only
- Error logging with `logger.client.push`
- Named export
- Return type always `ReturnedType`

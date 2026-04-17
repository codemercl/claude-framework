# Skill: useMutation Hook

Create an Apollo Client mutation hook with proper typing, error logging, and submit wrapper.

## When to use

When adding a new GraphQL mutation.

## Naming Convention

- Hook: `useMutation{Action}` (e.g. `useMutationAddFavorite`)
- Directory: `useMutationAddFavorite/`
- GraphQL file: `mutationAddFavorite.graphql`
- Operation: `MUTATION_{DOMAIN}_{ACTION}`
- Generated type: `MUTATION_{DOMAIN}_{ACTION}` as `{Action}Type`
- Variables type: `MUTATION_{DOMAIN}_{ACTION}Variables` as `{Action}VariablesType`

## File Structure

```
hooks/useMutation{Action}/
├── index.ts
└── gql/
    ├── mutation{Action}.graphql
    └── __generated__/
        └── MUTATION_{DOMAIN}_{ACTION}.ts
```

## gql/mutation{Action}.graphql

```graphql
mutation MUTATION_{DOMAIN}_{ACTION}($input: {InputType}!) {
  {mutationField}(input: $input) {
    isSuccess
    error {
      reason
      data {
        # error-specific fields
      }
    }
  }
}
```

## index.ts

```typescript
// Core
import { useMutation, ApolloError, FetchResult } from '@apollo/client';

// Other
import {
  MUTATION_{DOMAIN}_{ACTION} as {Action}Type,
  MUTATION_{DOMAIN}_{ACTION}Variables as {Action}VariablesType,
} from './gql/__generated__/MUTATION_{DOMAIN}_{ACTION}';
import mutation{Action} from './gql/mutation{Action}.graphql';
import { logger } from '../../../../core/helpers/logger';

type ReturnedType = {
  submit: (variables: {Action}VariablesType) => Promise<FetchResult<{Action}Type>>;
  isLoading: boolean;
  data?: {Action}Type | null;
  error?: ApolloError;
};

export const useMutation{Action} = (): ReturnedType => {
  const [submitFn, { loading, data, error }] = useMutation<
    {Action}Type,
    {Action}VariablesType
  >(mutation{Action});

  if (!loading && error) {
    logger.client.push({
      name: 'useMutation{Action} hook error',
      error,
    });
  }

  const submit = (
    variables: {Action}VariablesType,
  ): Promise<FetchResult<{Action}Type>> => submitFn({ variables });

  return {
    submit,
    isLoading: loading,
    data,
    error,
  };
};
```

## With refetchQueries

When mutation should refresh cached data:

```typescript
const [submitFn, { loading, data, error }] = useMutation<
  {Action}Type,
  {Action}VariablesType
>(mutation{Action}, {
  refetchQueries: [
    { query: queryToRefresh },
  ],
  awaitRefetchQueries: true,
});
```

## After creation

Run `yarn apollo:generate-types` to generate `__generated__/` types.

---
globs: "front/src/pages/**"
---

# Next.js Page Conventions

## Pages Router

This project uses **Pages Router** (not App Router). The following are FORBIDDEN: App Router, Server Components, Server Actions, `getStaticProps`.

## Page Structure

- Type: `NextPage<PropsType>` with explicit `PropsType` including `children?: never`
- Return: `ReactElement`
- Export: **default export** (only place in the project where default export is used)
- Data fetching: `getServerSideProps` with `initializeApp` + `collectQueriesExecutionDurations`
- Return props always include: `initialApolloState`, `initialReduxState`, `footer`

## SSR Data Fetching

```typescript
export const getServerSideProps: GetServerSideProps = async (context) => {
  const { initialApolloState, initialReduxState, footer } = await initializeApp({
    context,
    loadGraphqlQueries: (execute) => [
      execute<VariablesType>({
        query: queryName,
        variables: { /* ... */ },
        fetchPolicy: 'network-only',
      }),
    ],
  });
  return { props: { footer, initialApolloState, initialReduxState } };
};
```

## Auth-gated Pages

```typescript
const { isAuthorized } = await userCookieChecker(context) || { isAuthorized: false };
```

## File Naming

- kebab-case for multi-word routes: `special-offer.tsx`
- Dynamic segments: `[slug]/index.tsx`, `[id].tsx`

## SEO

- Use `BreadcrumbsComponent` and `HeadlineComponent` from `bus/seo/components/`
- Platform hero: `platformComponents.hero({ type: 'default', variant: 'secondary' })`

## Redirects & Error Pages

- Server redirect: `serverSideRedirect(context, path, statusCode)`
- 404: return `{ notFound: true }` from getServerSideProps
- Route validation: `getRbbPageFromSlug()` for dynamic pages

## Client-only Components

```typescript
const HeavyComponent = dynamic(
  () => import('../../bus/{domain}/components/{component}'),
  { ssr: false },
);
```

## Pages can import from ALL layers (only layer with this privilege).

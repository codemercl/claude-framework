# Skill: Next.js Page Scaffold

Create a page with getServerSideProps, initializeApp, and standard layout.

## When to use

When creating a new route/page.

## File Location

`front/src/pages/{route}.tsx` or `front/src/pages/{route}/index.tsx`

## Template (authenticated page)

```typescript
// Core
import React, { ReactElement } from 'react';
import { GetServerSideProps, NextPage } from 'next';

// Platform
import { platformComponents } from '@platform/components';

// Templates
import { AlertTemplate } from '../../core/templates/alert';
import { ModalTemplate } from '../../core/templates/modal';

// Views
import { HeaderView } from '../../core/views/header';

// Components
import { FooterComponent } from '../../core/components/footer';
import { BreadcrumbsComponent } from '../../bus/seo/components/breadcrumbs';
import { HeadlineComponent } from '../../bus/seo/components/headline';

// Other
import { initializeApp } from '../../init/initializeApp';
import { userCookieChecker } from '../../core/helpers/userCookieChecker';

export const getServerSideProps: GetServerSideProps = async (context) => {
  const { isAuthorized } = await userCookieChecker(context) || { isAuthorized: false };

  const { initialApolloState, initialReduxState, footer } = await initializeApp({
    context,
    loadGraphqlQueries: (execute) => [
      // Add SSR queries here
    ],
  });

  return {
    props: {
      footer,
      initialApolloState,
      initialReduxState,
    },
  };
};

type PropsType = {
  footer: string | null;
  children?: never;
}

const {PageName}: NextPage<PropsType> = ({ footer }: PropsType): ReactElement => {
  const heroJSX = platformComponents.hero({
    type: 'default',
    variant: 'secondary',
    basePath: '/static/styles',
    children: [
      <BreadcrumbsComponent key='breadcrumbs' />,
      <HeadlineComponent key='headline' />,
    ],
  });

  const footerJSX = footer && (
    <FooterComponent html={footer} />
  );

  return (
    <>
      <HeaderView>
        {platformComponents.header({ activeRoute: '{routeName}' })}
      </HeaderView>
      {heroJSX}
      <AlertTemplate type='{alertType}' />
      {/* Page content here */}
      {footerJSX}
      <ModalTemplate type='{modalType}' />
    </>
  );
};

export default {PageName};
```

## Template (public page, no auth check)

Same but without `userCookieChecker`:

```typescript
export const getServerSideProps: GetServerSideProps = async (context) => {
  const { initialApolloState, initialReduxState, footer } = await initializeApp({
    context,
    loadGraphqlQueries: (execute) => [],
  });

  return {
    props: {
      footer,
      initialApolloState,
      initialReduxState,
    },
  };
};
```

## With SSR GraphQL queries

```typescript
import queryArticle from '../../bus/blog/hooks/useQueryArticle/gql/queryArticle.graphql';
import { QUERY_ARTICLEVariables as ArticleVariablesType } from '../../bus/blog/hooks/useQueryArticle/gql/__generated__/QUERY_ARTICLE';

export const getServerSideProps: GetServerSideProps = async (context) => {
  const { initialApolloState, initialReduxState, footer } = await initializeApp({
    context,
    loadGraphqlQueries: (execute) => [
      execute<ArticleVariablesType>({
        query: queryArticle,
        variables: { alias: String(context.params?.alias || '') },
        fetchPolicy: 'network-only',
      }),
    ],
  });

  return {
    props: {
      footer,
      initialApolloState,
      initialReduxState,
    },
  };
};
```

## With dynamic imports (client-only components)

```typescript
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(
  () => import('../../bus/{domain}/components/{component}'),
  { ssr: false },
);
```

## Rules

- ALWAYS default export the page component
- ALWAYS use `GetServerSideProps` (not `getStaticProps`)
- ALWAYS return `initialApolloState`, `initialReduxState`, `footer`
- Use `NextPage<PropsType>` type
- Import paths: adjust `../../` depth based on page location

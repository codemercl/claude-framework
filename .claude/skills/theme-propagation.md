# Skill: Theme Propagation

Add new text keys, routes, or config values across all 17 themes.

## When to use

When a new feature needs theme-specific text, routes, or configuration.

## Theme List (all 17)

```
bridesBay
bridesDating
bridesStars
datingLadies
goDateFast
jumpForLove
mariaDating
meetWife
natalyDate
planetOfBrides
primeDating
ruBrides
uaBrides
vavaDating
yourBrides
```

## Adding new text keys

1. Add the key to the type: `platform/text/types.ts`
   ```typescript
   export type TextType = {
     // existing keys...
     NEW_KEY: string;
   };
   ```

2. Add value to EACH theme file: `platform/text/{theme}.ts`
   ```typescript
   export const siteText: TextType = {
     // existing...
     NEW_KEY: 'Theme-specific value',
   };
   ```

3. Use in code:
   ```typescript
   import { getText } from '@platform/text';
   const text = getText('NEW_KEY');
   ```

## Adding new routes

1. Add route type to: `platform/routes/types.ts`

2. Create route file for EACH theme: `platform/routes/{theme}/{routeName}.ts`
   ```typescript
   import { sitePaths } from '@platform/routes/site';
   import { RouteType } from '../types';
   import { SeoMetaScopeDictionary } from '../../../__generated__/globalTypes';
   import { RoutePrivacyDictionary } from '../types/dictionaries';

   export const {routeName}: RouteType = {
     path: sitePaths.{routeName},
     page: '/{routeName}',
     isReact: true,
     alias: '{routeName}',
     breadcrumbs: ['homepage', '{routeName}'],
     scope: SeoMetaScopeDictionary.core,
     isPaginatorPage: false,
     meta: {
       privacy: RoutePrivacyDictionary.customer,
       redirectTo: sitePaths.homepage,
     },
   };
   ```

3. Add path to `platform/routes/site.ts`:
   ```typescript
   export const sitePaths = {
     // existing...
     {routeName}: '/{route-path}',
   };
   ```

## Workflow

This is mechanical work across 17 files. When writing the plan:
1. Define the key/value once in the plan
2. List all 17 theme files that need updating
3. Specify theme-specific values where they differ (usually site name)
4. For most keys, all themes use the same value — note this in the plan

## Common patterns for theme-specific values

- Site name: `J4L`, `RuBrides`, `BridesBay`, etc. — use `SITE_NAME_SHORT` from existing text
- Links: usually same path, different domain
- Slogans/marketing copy: often unique per theme
- Feature flags: sometimes different per theme

# Code style

# Table of Contents

1. [Libraries/packages](#libraries)
2. [Folder and files organization](#organization)
3. [Naming conventions](#naming-conventions)
4. [Typescript](#typescript)
5. [React](#react)
6. [CSS & UI library](#css--ui-library)
7. [ESLint and Prettier](#eslint-and-prettier)
8. [Accessibility](#accessibility)
9. [Translations](#translations)

## [Libraries/packages](#libraries)

- Adding a new library is a collective decision.
- Prefer fixing a library version:

```json
// not recommended
"react": "^18.2.0"

// recommended
"react": "18.2.0"
```

## [Folder and files organization](#organization)

- It's preferable to have as flat folder structure as possible.
- Avoid `index` files (except for libraries).
- Folders must be named in PascalCase (e.g. `RetailerDetails`).
- Component files must be named in PascalCase (e.g. `NewComponent.tsx`).
- Component style file must be named `Component.styles.ts`.
- Component hooks file must be named `Component.hooks.ts`.
- Component utils file must be named `Component.utils.ts`.
- No cross dependencies between modules (e.g. module `A` mustn't import files
  from module `B`).
- Cross module dependencies should be moved to the `shared` folder inside an
  app. `Shared` is comprised of such folders as `constants`, `types`,
  `components`, `utils`, `styles`, etc.
- Cross product dependencies should be moved to `libs`.

### Prefer named export

```ts
// not recommended
export default () => {};
```

```ts
// recommended
export const MyComponent = () => {};
export const useHook = () => {};
```

Why? There are many reasons in different areas. TL DR: It reduces amount of
typos, enables more feautures by IDE, static analyzers and other “dumb” tools.
Some thoughts are described
[here](https://basarat.gitbook.io/typescript/main-1/defaultisbad). Default
export is alive. But how? Sometimes, there are a lot of files which has a
default export. Don’t change it. Leave as it is. It’s better to have the same
approach in a scoped folder.

But default exports has to be named. See how:

```ts
// not recommended
export default () => {};
```

```ts
// recommended
const useAutocomplete = () => {};
export default useAutocomplete;
```

### Importing stuff

There is a three kinds of import: from other package, from current package and
from current module.

General principe — import should be as shallow as possible. You should use only
public imports of given module and avoid getting anything from depths of a
module.

```ts
// not recommended
import { Module } from "@package/src";
import { OtherModule } from "@package/src/Components";
import { Submodule } from "@package/src/Components/Module";
```

```ts
// recommended
import { Module, OtherModule, Submodule } from "@package";
```

Importing from current package usually looks the same, but there is an important
caveat: you shouldn’t import from current module by a root path, as it will
represent a dependency loop, which will not always be caught by a linter.
Usually it’s good to avoid relative parent imports, but here it’s unavoidable.

Because of that, we don’t use root imports when importing from current package,
even if there is a root export.

```ts
// not recommended
// In Components/SomeComponent
import { Module } from "../../modules";
import { Component } from "@self";
import { FancyComponent } from "@self/Components";
```

```ts
// recommended
// In Components/SomeComponent
import { Module } from "@self/modules";
import { Component } from "..";
import { FancyComponent } from "..";
```

Importing from current module is usually simple, just use relative imports.

```ts
// not recommended
import { Submodule } from "@root/modules/Module/Submodule";
```

```ts
// recommended
import { Submodule } from "./Submodule";
```

## [Naming conventions](#naming-conventions)

- Enum names are written in `UpperCamelCase`. Enum fields are written in
  `CONSTANT_CASE`. Enum values are specified in `CONSTANT_CASE`.

```ts
// recommended
enum ProductChoices {
  FACTORING = "FACTORING",
  PAY_FOR_PLATFORMS = "PAY_FOR_PLATFORMS",
  PAY_FOR_TEMPS = "PAY_FOR_TEMPS",
  CONNECT = "CONNECT",
  DEBTOR_PORTAL = "DEBTOR_PORTAL",
}
```

- Variables and function names are written in `camelCase`.
- Constants (non-changing variables) are written in `CONSTANT_CASE`.
- Component props are written in `IComponentProps` way.

## [Typescript](#typescript)

- Follow the rules set on the linter level.
- Use generated types as much as possible.
- Use utility types (`Omit`, `Pick`, `Required`, `NonNullable`, etc.) instead of
  redeclaring types.
- Avoid using **`any`**.
- Prefer using interfaces to describe component props.
- Prefer string enums to numeric:

```ts
// not recommended
enum Direction {
  UP,
  DOWN,
  LEFT,
  RIGHT,
}

// recommended
enum Direction {
  UP = "up",
  DOWN = "down",
  LEFT = "left",
  RIGHT = "right",
}
```

- Recommendation: Prefer destructuring props in function declaration:

```ts
// not recommended
export const Component = (props: ComponentProps) => {};

// recommended
export const Component = (props: ComponentProps) => {
  const { value, label } = props;
};

// recommended
export const Component = ({ value, label }: ComponentProps) => {};
```

- Prefer setting default params to if/else checks.
- Use `null coalescing operator` instead of checking for
  `variable !== undefined && variable !== null`.
- Use `optional chaining` instead of checking existence of an
  object/array/function. E.g. `onChange?.()`, `items?.[0]`,
  `accounts?.pageInfo`.
- Avoid abbreviations (e.g. `br`, `opt`).
- Prefer using [jsdoc](https://jsdoc.app/) with jsdoc tags (`@description`,
  `@example`, `@default`, etc.) instead of plain comments whenever describing
  types.

### Type Narrowing

Sometimes you have a data structure which significantly differs due to some
conditions. For example, SuccessResponse and ErrorResponse are different by
nature but this is one object.

To achieve that Typescript has
[this feature](https://www.typescriptlang.org/docs/handbook/2/narrowing.html).
Unfortunately, it works weird sometimes.

Let’s have a simple example: Note: Destructuring breaks Typescript Narrowing

```ts
type SuccessResponse = {
  data: unknown; // it can be any type or passed using Generics
  isSuccess: true;
  isError: false;
};

type ErrorResponse = {
  error: Error;
  isSuccess: false;
  isError: true;
};

type Response = SuccessResponse | ErrorResponse;

const getResponse = (response: Response) => {
  // Note: you can't do destructuring of response, TS won't get it.
  if (response.isError) {
    console.log(response.error); // error here is Error
    // response.data is undefined
  }
  if (response.isSuccess) {
    // response.error is undefined
    console.log(response.data); // data is defined
  }
};
```

### Interface Narrowing

Simple example:

```ts
interface Person {
  kind: "business" | "academic";
  name: string;
  age: number;
}

interface BusinessPerson extends Person {
  kind: "business";
  sallary: number;
}

interface AcademicPerson extends Person {
  kind: "academic";
  publication: string;
}

type Human = BusinessPerson | AcademicPerson;

const logHuman = (human: Human) => {
  if (human.kind === "academic") {
    console.log(human); // type of human is AcademicPerson,
  }
  if (human.kind === "business") {
    console.log(human); //type of human is BusinessPerson
  }
};
```

### Types vs Interfaces

Prefer `interface` when possible. Use `type` for unions and smaller stuff.

```ts
// recommended
type MyValue = number | string;

type UserInfo = {
  name: string;
  avatar: string;
};

interface HexFieldProps extends TextFieldProps {
  hex: string;
}
```

Good (Work faster):

```ts
interface Entity {
  id: string;
  createdAt: Date;
}

interface User extends Entity {
  name: string;
  avatar: string;
}

interface FullUser extends User {
  jobTitle: string;
  about: string;
  projects: Project[];
}
```

Bad (works slower)

```ts
// not recommended
type Entity = {
  id: string;
  createdAt: Date;
};

type User = Entity & {
  name: string;
  avatar: string;
};

type FullUser = User & {
  jobTitle: string;
  about: string;
  projects: Project[];
};
```

## [React](#react)

- Use named export for components, utility functions and constants.
- Use default export for route files (for lazy loading).
- Return ternary operator instead of fragment wrapping a couple of conditions.
- Avoid using `index` as a key in loops.

## [CSS & UI library](#css)

- Use [MaterialUI](https://mui.com/material-ui/getting-started/) components as
  much as possible.
- UI component must be considered a closed system. UI component can change its
  internals at any moment. Do not try to change deep component styles, it makes
  hard to introduce any new changes. Do not import files from inside a UI
  component:

```js
// not recommended
import { ControlledCheckbox } from "@ui/checkbox/ControlledCheckbox";

// recommended
import { ControlledCheckbox } from "@ui/checkbox";
```

- Use tags for styling. Make all references explicit:

````ts
// recommended
export const ComponentStyles = styles.div`
  span {
    // styles here
  }
`;



- Split component styles in case a lot of styles depend on one variable:

```ts
// recommended
export const SwitchLabel = styled.label<{ isTextMode?: boolean}>`

 ${({ isTextMode }) =>
    isTextMode
      ? css`
          // all styles for text mode
      : css`
          // all styles for non text mode
      `}
````

## [ESLint and Prettier](#eslint)

Golden rule: master brach should be free of ESLint and Prettier errors, it
should be impossible on CI level to merge branch with these.

There should be no warnings. It’s either a `error` or `off`.

It’s preferable to use recommended configs, such as `eslint/recommeneded`,
`typescript-eslint/recommended`, etc. It may me possible, that you will need a
prettier config to turn off some conflicting rules.

You should not EVER disable warning from `react-hooks`, `typescript` or `import`
plugins.

You may safely turn off all `jsx-a11y` rules.

## [Accessibility](#accessibility)

Care about accessibility.

- Use semantic HTML whenever possible.
- Use aria attributes when it's needed (but do not overuse).
- Create keyboard navigable interface.
- Add focus styles for keyboard navigation.
- UI component must be accessible.

## [Translations](#translations)

- Always use translation keys instead of plain text. We support
  internationalization.
- Each module should use its own translation keys. Prefer distinguishing
  translation keys following the pattern `${moduleName}.${description}`:

```json
"backoffice": {
      countries: 'Countries',
      country: 'Country',
      countryCode: 'Country Code',
      organization: 'Organization',
      languages: 'Languages',
      alternateCodes: 'Alternate PS Codes',
      languageCode: 'Language Code',
      currencies: 'Currencies',
      defaultCurrency: 'Default Currency',
      status: 'Status',
      basicDetails: 'Basic Details',
      longitude: 'Longitude',
      latitude: 'Latitude',
      basicInfo: 'Basic Info',
      contactNames: 'Contact Names',
      closed: 'Closed',
      open: 'Open',
      overview: 'Overview',
      socialMedia: 'Social Media',
  }
```

- make translation keys descriptive. Prefer `retailers.addNewRetailer` to
  `retailers.action`.

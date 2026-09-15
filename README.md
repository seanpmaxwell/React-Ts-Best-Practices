# ⚛️ React + TypeScript Best Practices
[![GitHub stars](https://img.shields.io/github/stars/seanpmaxwell/React-Ts-Best-Practices?style=flat-square)](https://github.com/seanpmaxwell/React-Ts-Best-Practices/stargazers)

This guide covers React-specific habits that pair well with TypeScript. It assumes you already follow the [TypeScript best practices](https://github.com/seanpmaxwell/Typescript-Best-Practices) document; anything covered there isn't repeated here.

Some of these are React fundamentals, others are house conventions. If your framework has its own rules (file-based routing, Server Components, etc.), follow those where they differ.

## Table of contents
- [Project structure](#project-structure)
  - [Overview summary](#project-structure-overview)
  - [Overview in detail](#project-structure-structuring)
  - [Example layout](#project-structure-example)
- [Function components](#function-components)
  - [Declaring components](#function-components-declaring)
  - [Organizing component code](#function-components-organization)
  - [Working with props](#function-components-props)
- [The Container/Presenter pattern and state management](#container-presenter-state-management)
  - [Container/Presenter pattern](#container-presenter-pattern)
  - [`useState` vs `useSetState`](#state-management-usestate)
- [Misc styling rules](#misc-styling-rules)
  - [Styling the UI](#misc-styling-ui)
  - [Callback parameter names](#misc-styling-callbacks)
  - [Other rules](#misc-styling-other)

<br/><b>***</b><br/>

## Project structure <a name="project-structure"></a>

### Overview summary <a name="project-structure-overview"></a>
```yml
- public/
- src/
  - assets/
  - _common/
  - components/
    - _common/
    - pages/
    - App.test.tsx
    - App.tsx
    - index.css
  - domains/
  - infra/
  - main.tsx
- .env
- package.json
- tsconfig.json
```

### Overview in detail <a name="project-structure-structuring"></a>
- Name files and folders after the React component they represent. A single-file component's file name should match the component name. Use `.tsx` only for files that contain JSX; plain helpers go in `.ts`.
- `main.tsx` is the entry point; `index.ts` is reserved for barrel files, same as the TypeScript guide.
- For scalable React apps, I like **domain-based** architecture. See the [architecture section](https://github.com/seanpmaxwell/Typescript-Best-Practices/blob/main/README.md#architecture) of the TypeScript guide for the trade-offs.
- Naming follows the TypeScript guide on both sides of the stack: **Service** means business logic, **Api** means the client-side boundary that talks to the server. So a domain has `User.ts` (model/types), `UserService.ts` (business rules and workflows), and `UserApi.ts` (HTTP requests and responses).
- Logic tied to a domain (i.e. is this user an `Admin`) goes under `src/domains/`. Client-side checks like that only shape the UI; the server still has to enforce authorization.
- Logic that only exists to make the UI work belongs with the component that uses it. Hooks are React-specific, so a hook used by one component stays in that component's folder, and only broadly reusable hooks go in `components/_common/hooks/`.
- Keep dependencies flowing one way: component or hook → service → api → shared HTTP client. Skip a layer when it adds nothing; a page that just loads a list can call `UserApi` directly. Services should never import components or hooks.
- Anything bundled for the browser is public. Putting a value in `.env` doesn't make it a secret.

### Example `src/` folder layout with domain-based architecture <a name="project-structure-example"></a>
```yml
- assets/
- _common/
  - constants/
    - EnvVars.ts
    - Paths.ts <-- Keep all navigation paths in one place
  - types/
  - utils/
- components/
  - _common/
    - ui/  <-- group by purpose, not size labels like sm/md/lg
      - buttons/
      - dialogs/
    - hooks/
      - usePageTitle.ts
    - styles/
      - Colors.ts
      - BoxStyles.ts
  - pages/
    - Home/ (https://my-site.com/home)
      - Home.tsx
      - Home.test.tsx
    - Account/ (https://my-site.com/account)
      - UpdatePaymentForm/
        - _local/
          - usePaymentForm.ts
        - UpdatePaymentForm.tsx
        - UpdatePaymentForm.test.tsx
      - Account.tsx  // imports <UpdatePaymentForm/>
      - Account.test.tsx
    - Posts/ (https://my-site.com/posts)
      - _common/
        - types.ts // shared across View/Edit/New
        - components/
          - PostForm.tsx  // shared between New and Edit
      - Edit/ (https://my-site.com/posts/:id/edit)
        - Edit.test.tsx
        - Edit.tsx
      - New/ (https://my-site.com/posts/new)
        - New.test.tsx
        - New.tsx
      - View/ (https://my-site.com/posts/:id)
        - View.test.tsx
        - View.tsx  // displays a specific post
      - Posts.tsx  // shows <PostsTable/> when no post is selected
      - Posts.css
- domains/
  - _common/
    - constants/
      - ApiPaths.ts <-- API endpoint paths, separate from navigation paths
    - types/
      - Entity.ts <-- Parent model interface
  - users/
    - User.ts // model-layer
    - UserService.ts <-- business logic, counterpart to the back-end services layer
    - UserApi.ts <-- HTTP requests and responses
  - posts/
    - Post.ts
    - PostService.ts
    - PostApi.ts
- infra/
  - http/
    - setup-axios.ts
    - index.ts <-- used by the Api layer
- main.tsx
```

Snippets below use `@src/` as an alias for `src/`. Configure it in TypeScript and your build/test tools.

<br/><b>***</b><br/>

## Function components <a name="function-components"></a>

### Declaring components <a name="function-components-declaring"></a>
- Use PascalCase names and prefer function declarations over classes or arrow functions, mainly for consistency with the TypeScript guide. Parents can precede children with any declaration style (JSX resolves a child when the parent renders, after the module has loaded), and arrow functions assigned to variables get proper names in stack traces, so this is about consistency rather than a technical requirement.
- Define parent components first and declare children beneath them in the same file to keep logic top-down. Never define a component *inside* another component: that creates a new component type every render, which remounts the subtree and loses its state.
- Always type props. Define an interface (e.g. `IProps`) in the `Types` region for complex components. Use a more specific name like `ILoginFormProps` when the file has several components or the component is shared.
- Let TypeScript infer the return type. Components can return `null`, strings, arrays, and other renderable content, so `JSX.Element` isn't accurate. If you want an explicit type, import `ReactNode` or `ReactElement` from React.

### Organizing component code <a name="function-components-organization"></a>
- Keep static values outside component functions under the `Constants` region. It avoids re-creating them every render, but the bigger win is making it obvious they don't depend on props or state. Never put mutable per-user state at module scope; it's shared by every instance.
- Extract long helpers that don't depend on props or state and place them in the `Functions` region.
- Keep rendering pure. Requests, storage writes, and other side effects go in event handlers or effects. If a value can be derived from props or state, derive it during render instead of storing a second copy.
- Follow the Rules of Hooks: call hooks at the top level, not inside conditions or loops, and keep effect dependencies honest.
- Place layout logic for sibling components in their parent so positioning and interactions are visible in one place.
- Prefer whitespace plus short comments to separate hook calls, derived values, handlers, and the returned JSX.
- Keep the default export at the bottom. When the default export is a function component, start its comment with `Default component ...` so readers can identify it quickly.
- Avoid giant `return` statements. Create child components for related DOM blocks rather than declaring many JSX variables above the return. Snippet 2 shows how composing smaller components keeps the parent lean.
- Use function declarations at the top level and arrow functions only for inline callbacks inside JSX.
- Don't reach for `useCallback`/`useMemo` by default. They matter when function or object identity matters (e.g. props to a memoized child). Write clear code first and measure before adding them.

#### Snippet 2 – parent vs child composition

```tsx
// Bad: logic split across temporary JSX variables
function Parent() {
  const posts: string[] = [];
  const name = '';

  let child = null;
  if (something) {
    child = (
      <Box mb={2}>
        Name: {name} Posts: {posts.length}
      </Box>
    );
  } else {
    child = (
      <Box {...otherProps}>
        Foo: {name} Bar: {posts.length}
      </Box>
    );
  }

  // A JSX value is rendered as {child}, never as <Child/>.
  return (
    <Box>
      {child}
      <SomeOtherChild />
    </Box>
  );
}

export default Parent;
```

```tsx
// Good: extract child components
import Box, { BoxProps } from '@mui/material/Box';

interface IChildProps extends BoxProps {
  name?: string;
  posts?: string[];
}

/** Default component. Display a list of <Child/> elements. */
function Parent() {
  return (
    <Box>
      {something ? (
        <Child1 mb={1} name={name} posts={posts} />
      ) : (
        <Child2 name={name} posts={posts} />
      )}
      <SomeOtherChild />
    </Box>
  );
}

/** Display a child's name and number of posts. */
function Child1(props: IChildProps) {
  const { name = '', posts = [], ...otherProps } = props;
  return (
    <Box {...otherProps}>
      Name: {name} Posts: {posts.length}
    </Box>
  );
}

/** Lorum Ipsum. */
function Child2(props: IChildProps) {
  const { name = '', posts = [], ...otherProps } = props;
  return (
    <Box {...otherProps}>
      Foo: {name} Bar: {posts.length}
    </Box>
  );
}

export default Parent;
```

### Working with props <a name="functional-components-props"></a>
- Destructure props at the top of the component. It's easier to set defaults and spot unused values, and wrapper components can mirror the child's props by reusing the same names.
- In wrapper components, separate the props you handle from the ones you forward, and don't forward internal-only props to DOM elements. Be careful with styling props: MUI's `sx` can be an object, array, or function, so spreading it as an object isn't reliable.
- Snippet 3 shows how to extract props, separate regions, and organize hooks.

#### Snippet 3 – component layout template

```tsx
// LoginForm.tsx
import type { FormEvent } from 'react';
import { useNavigate } from 'react-router';
import { useSetState } from 'react-use';
import Box, { BoxProps } from '@mui/material/Box';
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';

import Paths from '@src/_common/constants/Paths';
import AuthService from '@src/domains/auth/AuthService';
import Indicator from '@src/components/_common/ui/Indicator';

/******************************************************************************
                               Components
******************************************************************************/

/**
 * Default component. Log in a user.
 */
function LoginForm(props: BoxProps<'form'>) {
  const { sx, ...otherProps } = props;
  const navigate = useNavigate();

  // Init state
  const [state, setState] = useSetState({
    username: '',
    password: '',
    error: '',
    isLoading: false,
  });

  // Submit. HTTP details live in AuthService/AuthApi, never in a component.
  const submit = async (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    if (state.isLoading) return;
    setState({ isLoading: true, error: '' });
    try {
      await AuthService.login({
        username: state.username.trim(),
        password: state.password, // never silently alter a password
      });
      navigate(Paths.ACCOUNT);
    } catch (err) {
      setState({ isLoading: false, error: getErrorMessage(err) });
    }
  };

  // Return
  return (
    <Box
      component="form"
      onSubmit={submit}
      sx={[{ position: 'relative' }, ...(Array.isArray(sx) ? sx : [sx])]}
      {...otherProps}
    >
      {/* Indicator */}
      {state.isLoading && <Indicator />}
      {state.error && <p role="alert">{state.error}</p>}

      {/* Input Fields */}
      <TextField
        label="Username"
        value={state.username}
        onChange={e => setState({ username: e.currentTarget.value })}
      />
      <TextField
        label="Password"
        type="password"
        value={state.password}
        onChange={e => setState({ password: e.currentTarget.value })}
      />

      {/* Action Buttons */}
      <Button type="button" color="error" onClick={() => navigate(Paths.HOME)}>
        Cancel
      </Button>
      <Button type="submit" color="primary" disabled={state.isLoading}>
        Login
      </Button>
    </Box>
  );
}

/******************************************************************************
                               Functions
******************************************************************************/

function getErrorMessage(err: unknown): string {
  return err instanceof Error ? err.message : 'Unable to log in.';
}

/******************************************************************************
                               Export default
******************************************************************************/

export default LoginForm;
```

Using a real `<form>` with `type="submit"` means the keyboard works too, and the Cancel button needs `type="button"` so it doesn't submit. React 19's `useActionState` can handle the pending/error state for you, but the explicit handler works on any version.

<br/><b>***</b><br/>

## The Container/Presenter pattern and state management <a name="container-presenter-state-management"></a>

If a component is complex, separate data-fetching and state logic (containers) from rendering logic (presenters). This keeps components testable and reusable. For state itself, combine `useState`, `useContext` (and sometimes a third-party store such as Redux) using the lightest tool that satisfies the data flow you need. Inside containers, values shared across _multiple sub-trees_ go in context or a store; for the rest, prop-drilling a few levels is fine. Keep state as close as possible to the components that need it.

### Container/Presenter pattern <a name="container-presenter-pattern"></a>
- A **Container** component owns state, side effects, and data fetching. It passes data and callbacks down as props. The page component usually plays this role; it doesn't need a `Container` suffix.
- A **Presenter** component is mostly visual: it receives props and renders UI with no knowledge of where data comes from. If a presenter needs its own state, it should only be display state (which tab is open, is a section expanded).
- This separation makes presenters easy to test in isolation (just pass props) and lets you swap data sources in the container without touching the UI. A custom hook is another way to pull the workflow out of the component.

```tsx
// Users.tsx – container
function Users() {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    let isActive = true;
    UserApi.fetchAll().then(users => {
      if (isActive) setUsers(users); // ignore late results after unmount
    });
    return () => { isActive = false; };
  }, []);

  return <UsersList users={users} />;
}

export default Users;
```

This is a plain read with no business workflow, so the container calls `UserApi` directly. Loading and error states are omitted for brevity; in a real app a router loader or query library usually handles those, along with caching and refetching, so you don't rebuild them in every container.

```tsx
// UsersList.tsx – presenter
interface IUsersListProps {
  users: User[];
}

function UsersList({ users }: IUsersListProps) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{getDisplayName(user)}</li>
      ))}
    </ul>
  );
}

// Lives here because it's only relevant to how users are displayed.
function getDisplayName(user: User): string {
  const { firstName, lastName } = user;
  return lastName ? `${firstName} ${lastName}` : firstName;
}

export default UsersList;
```

The helper leaves the user's capitalization alone rather than recasing their name.

### `useState` vs `useSetState` <a name="state-management-usestate"></a>
- `useState` is fine for components with one or two state variables. It *replaces* the value, so when the state is an object you have to spread the previous value yourself: `setForm(prev => ({ ...prev, name }))`.
- As state grows, switch to a hook such as [`useSetState`](https://github.com/streamich/react-use/blob/master/src/useSetState.ts) from `react-use`. It shallow-merges the patch you pass in, so `setState({ name })` keeps the other fields. Nested objects are replaced, not merged.
- Grouping state into one object keeps every value prefixed with `state` and managed by a single updater, which reads well.
- Neither hook returns a reset function. Build one from a defaults function: `const resetState = () => setState(Defaults());`. Because it merges, it only resets the keys you pass. That's handy in modal flows where you need to restore defaults.
- Don't store what you can derive. A "field is required" error can be computed from the field's value instead of kept in a synced boolean, and a single `status: 'idle' | 'loading' | 'error'` beats several booleans that can contradict each other.
- Context distributes a value; it doesn't own state. Changing it re-renders every consumer, so split unrelated concerns rather than making one giant provider. Calling the same custom hook in two components does *not* share state unless the hook reads from a provider or store.

<br/><b>***</b><br/>

## Misc styling rules <a name="misc-styling-rules"></a>

These items may not be enforced by the linter but they help keep React + TS projects readable.

### Styling the UI <a name="misc-styling-ui"></a>
- Keep color tokens in `src/components/_common/styles/Colors.ts` instead of hardcoding hex strings in JSX. Group base colors, then expose them through semantic buckets so updates stay centralized.
- If your component library already has a theme (MUI's palette, for example), use that as the source of truth rather than maintaining a competing color system. Semantic names don't guarantee accessible contrast, so check the real combinations.

#### Snippet 4 – color tokens

```ts
// src/components/_common/styles/Colors.ts
const Base = {
  Grey: {
    UltraLight: '#f2f2f2',
    Lighter: '#e5e5e5',
    Light: '#d3d3d3',
    Default: '#808080',
    Dark: '#a9a9a9',
    Darker: '#404040',
    UltraDark: '#0c0c0c',
  },
  Red: {
    Default: '#ff0000',
    Dark: '#8b0000',
  },
  White: {
    Default: '#ffffff',
  },
} as const;

export default {
  Background: {
    Default: Base.Grey.Default,
    White: Base.White.Default,
    Hover: Base.Grey.Light,
  },
  Border: Base.Grey.Dark,
  Text: {
    Error: {
      Default: Base.Red.Default,
      Hover: Base.Red.Dark,
    },
  },
} as const;
```

#### Snippet 5 – using shared colors

```tsx
import Colors from '@src/components/_common/styles/Colors';

function Foo() {
  return (
    <div>
      <div
        style={{
          marginBottom: 16,
          fontSize: 12,
          backgroundColor: Colors.Background.Default, // never '#808080'
        }}
      >
        Hello
      </div>

      {/* A button, not a clickable div: it gets keyboard and focus behavior for free. */}
      <button
        type="button"
        style={{ padding: 8 }}
        onClick={() => alert('How are you?')}
      >
        How are you?
      </button>
    </div>
  );
}
```

Inline styles are fine for small dynamic values like these. Reusable styles, hover/focus states, and responsive layouts belong in CSS, CSS Modules, or your theme.

### Callback parameter names <a name="misc-styling-callbacks"></a>
- Give parameters meaningful names in general, but for one-line inline JSX callbacks a short placeholder is fine: `v` for a value, `e` for an event, `err` for an error. Just don't confuse an event with its value; a DOM `onChange` gives you an event, and the string is `e.currentTarget.value`.
- For custom inputs, a value-based `onChange(value: string)` keeps the parent independent of DOM event details.

#### Snippet 6

```tsx
function Parent() {
  const [state, setState] = useSetState({ name: '', email: '' });
  const isMissingRequired = !state.name.trim() || !state.email.trim();

  return (
    <form
      onSubmit={e => {
        e.preventDefault();
        if (!isMissingRequired) {/* some API call, trim values here */}
      }}
    >
      <CustomInput
        label="Name"
        value={state.name}
        isRequired
        onChange={v => setState({ name: v })}
      />
      <CustomInput
        label="Email"
        value={state.email}
        isRequired
        onChange={v => setState({ email: v })}
      />
      <button type="submit">Submit</button>
    </form>
  );
}

function CustomInput(props: {
  label: string;
  value: string;
  isRequired?: boolean;
  onChange: (value: string) => void;
}) {
  const { label, value, isRequired = false, onChange } = props;
  const isMissing = isRequired && !value.trim(); // derived, not stored
  return (
    <label>
      {label}
      <input
        type="text"
        required={isRequired}
        value={value}
        onChange={e => onChange(e.currentTarget.value)}
      />
      <div>{isMissing ? `${label} is required` : ''}</div>
    </label>
  );
}
```

A few things worth noticing: the error is derived from the value instead of stored as a separate boolean, the input isn't trimmed on every keystroke (that fights the user while typing spaces; trim at submit instead), and the submit button stays enabled so the user can attempt a submit and see what's wrong. A disabled button can't even receive keyboard focus.

### Other rules <a name="misc-styling-other"></a>
- Use single quotes for standard JS/TS code and double quotes for JSX attributes. Let Prettier enforce it (`"singleQuote": true, "jsxSingleQuote": false`).
- Use stable IDs from the data as list keys. Array indexes break when items are inserted, removed, or reordered, and `useId` is for accessibility relationships, not list keys.
- Associate labels with inputs, use `type="button"` for non-submit buttons inside forms, and don't rely on color alone to communicate loading or error states.
- Test what the user can observe (labels, roles, interactions, empty and failure states), not internal state variables. A refactor shouldn't break a test if the behavior didn't change.

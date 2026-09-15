# ⚛️ React + TypeScript Best Practices

[![GitHub stars](https://img.shields.io/github/stars/seanpmaxwell/React-Ts-Best-Practices?style=flat-square)](https://github.com/seanpmaxwell/React-Ts-Best-Practices/stargazers)

Practical patterns for building React applications that are easier to read, test, and maintain.

This guide builds on the [TypeScript best practices](https://github.com/seanpmaxwell/Typescript-Best-Practices) document. It focuses on React-specific decisions rather than repeating the general TypeScript conventions.

Some recommendations are React fundamentals; others are house conventions. Use the conventions that help your team, and follow your framework’s requirements where they differ.

The examples focus on client-side function components. Framework features such as file-based routing and Server Components may call for a different project layout or data-loading approach.

## 📚 Table of contents

- [🗂️ Project structure](#project-structure)
  - [Overview](#project-structure-overview)
  - [Where code belongs](#project-structure-where-code-belongs)
  - [Example layout](#project-structure-example)
- [🧩 Function components](#function-components)
  - [Declaring components](#function-components-declaring)
  - [Organizing component code](#function-components-organization)
  - [Working with props](#function-components-props)
- [🔄 Containers and state management](#containers-and-state-management)
  - [The Container/Presenter pattern](#container-presenter-pattern)
  - [Choosing and updating state](#choosing-and-updating-state)
- [🎨 Styling and everyday conventions](#styling-and-everyday-conventions)
  - [Styling the UI](#styling-the-ui)
  - [Callback parameter names](#callback-parameter-names)
  - [Other conventions](#other-conventions)

---

<a id="project-structure"></a>

## 🗂️ Project structure

A useful project structure answers two questions:

- Where should new code go?
- Where would someone else expect to find it?

Start small. Add folders when they clarify responsibilities—not just because a diagram includes them.

<a id="project-structure-overview"></a>

### Overview

For a client-side application, I usually start with a layout like this:

```text
public/
src/
├── assets/
├── _common/
├── components/
│   ├── _common/
│   ├── pages/
│   ├── App.test.tsx
│   ├── App.tsx
│   └── index.css
├── domains/
├── infra/
└── main.tsx
.env
package.json
tsconfig.json
```

| Location | Purpose |
| --- | --- |
| `public/` | Static files served directly, according to your build tool’s conventions. |
| `src/assets/` | Assets imported by application code. |
| `src/_common/` | Shared code that is not tied to React or one domain. |
| `src/components/` | Components, pages, and React-specific UI logic. |
| `src/domains/` | Domain models, business services, and domain-specific API clients. |
| `src/infra/` | Shared integrations, such as an HTTP client or browser-storage adapter. |
| `src/main.tsx` | The application entry point. |

I use `main.tsx` for the application entry point and reserve `index.ts` for barrel exports.

This is a house convention, not a React requirement. If your framework defines its own entry points or routing folders, follow those conventions instead.

Values embedded in a client bundle are public. Putting a value in `.env` does not make it secret once the build exposes it to browser code.

<a id="project-structure-where-code-belongs"></a>

### Where code belongs

#### Name components and their files consistently

Use PascalCase for component names, and match the filename to the main component it exports:

```text
Account/
├── Account.tsx
└── Account.test.tsx
```

A small component can live in a single file. Give it a folder when it gains related tests, styles, hooks, or supporting components.

Use `.tsx` when a file contains JSX. Ordinary TypeScript helpers usually belong in `.ts` files.

#### Organize larger applications by domain

For applications with several business areas, I prefer domain-based organization. See the [architecture section](https://github.com/seanpmaxwell/Typescript-Best-Practices/blob/main/README.md#architecture) of the TypeScript guide for the trade-offs.

Within a domain, I use these names:

| File | Responsibility |
| --- | --- |
| `User.ts` | User-related types and model helpers. |
| `UserService.ts` | User-related business rules and workflows. |
| `UserApi.ts` | User-related HTTP requests and response handling. |

This keeps **Service** associated with business logic on both the frontend and backend. **Api** identifies the client-side boundary that communicates with the server.

For example:

- Checking whether a user is eligible for an action belongs in domain logic.
- Choosing how to display that eligibility belongs in the UI.
- Coordinating an account-update workflow belongs in `UserService`.
- Sending the account-update request belongs in `UserApi`.

Client-side permission checks help control the UI; the server must still enforce authorization independently.

#### Keep React-specific logic near its consumers

A hook does not automatically belong in a global hooks folder.

Use its scope to decide where it belongs:

- A hook used by one component stays near that component.
- A hook shared by related components belongs in their nearest shared folder.
- A broadly reusable UI hook can live in `components/_common/hooks/`.

Name custom hooks with a `use` prefix, such as `usePageTitle` or `usePaymentForm`.

Keep reusable business rules independent of React where practical. A hook can coordinate those rules with component state without moving the rules themselves into the UI layer.

#### Keep dependencies flowing in a clear direction

A business workflow typically follows this path:

```text
Component or custom hook
        ↓
Domain service
        ↓
Domain API client
        ↓
Shared HTTP client
```

Each part has a different job:

- The **component or hook** handles React state, navigation, and UI interactions.
- The **service** applies business rules and coordinates the workflow.
- The **API client** handles endpoints, request payloads, and responses.
- The **HTTP client** handles shared transport configuration.

Not every request needs every layer.

A container that simply loads users can call `UserApi.fetchAll()` directly. A workflow involving business validation, transformations, or several requests belongs in `UserService`.

Avoid adding a service wrapper that only forwards a call without providing a useful boundary.

Domain services should not depend on components or React hooks. Keep UI-specific behavior, such as navigation and opening dialogs, in the component or hook that owns it.

<a id="project-structure-example"></a>

### Example layout

Here is a more developed `src/` layout:

```text
src/
├── assets/
├── _common/
│   ├── constants/
│   │   ├── EnvVars.ts
│   │   └── Paths.ts
│   ├── types/
│   └── utils/
├── components/
│   ├── _common/
│   │   ├── hooks/
│   │   │   └── usePageTitle.ts
│   │   ├── styles/
│   │   │   ├── Colors.ts
│   │   │   └── BoxStyles.ts
│   │   └── ui/
│   │       ├── buttons/
│   │       ├── dialogs/
│   │       └── layout/
│   ├── pages/
│   │   ├── Home/                         ← /home
│   │   │   ├── Home.tsx
│   │   │   └── Home.test.tsx
│   │   ├── Account/                      ← /account
│   │   │   ├── UpdatePaymentForm/
│   │   │   │   ├── _local/
│   │   │   │   │   └── usePaymentForm.ts
│   │   │   │   ├── UpdatePaymentForm.tsx
│   │   │   │   └── UpdatePaymentForm.test.tsx
│   │   │   ├── Account.tsx
│   │   │   └── Account.test.tsx
│   │   ├── Users/                        ← /users
│   │   │   ├── UsersContainer.tsx
│   │   │   ├── UsersList.tsx
│   │   │   └── UsersList.test.tsx
│   │   └── Posts/                        ← /posts
│   │       ├── _common/
│   │       │   ├── types/
│   │       │   │   └── post-form-types.ts
│   │       │   └── ui/
│   │       │       └── PostForm.tsx
│   │       ├── Edit/                     ← /posts/:id/edit
│   │       │   ├── Edit.tsx
│   │       │   └── Edit.test.tsx
│   │       ├── New/                      ← /posts/new
│   │       │   ├── New.tsx
│   │       │   └── New.test.tsx
│   │       ├── View/                     ← /posts/:id
│   │       │   ├── View.tsx
│   │       │   └── View.test.tsx
│   │       ├── Posts.tsx
│   │       └── Posts.css
│   ├── App.tsx
│   ├── App.test.tsx
│   └── index.css
├── domains/
│   ├── _common/
│   │   ├── constants/
│   │   │   └── ApiPaths.ts
│   │   └── types/
│   │       └── Entity.ts
│   ├── auth/
│   │   ├── AuthService.ts
│   │   └── AuthApi.ts
│   ├── users/
│   │   ├── User.ts
│   │   ├── UserService.ts
│   │   └── UserApi.ts
│   ├── payments/
│   │   ├── PaymentService.ts
│   │   └── PaymentApi.ts
│   └── posts/
│       ├── Post.ts
│       ├── PostService.ts
│       └── PostApi.ts
├── infra/
│   └── http/
│       ├── setup-axios.ts
│       └── index.ts
└── main.tsx
```

A few details worth noting:

- `Account.tsx` composes `UpdatePaymentForm`.
- `PostForm.tsx` is shared by the New and Edit pages.
- Payment rules and workflows live in `PaymentService`; HTTP requests live in `PaymentApi`.
- `usePaymentForm` connects the payment workflow to React state and form behavior.
- UI folders are grouped by purpose rather than vague size labels such as `sm`, `md`, and `lg`.
- Shared navigation paths and API endpoint paths have separate, clearly named homes.

The examples below use `@src/` as an alias for `src/`. Configure that alias in both TypeScript and the relevant build or test tools.

Application-specific modules such as `AuthService`, `UserApi`, and `Paths` are project code, not React APIs.

---

<a id="function-components"></a>

## 🧩 Function components

Function components are the default choice for new React code. They work naturally with hooks and keep most component logic in one place.

“Function component” describes how the component is declared. It does not mean the entire application follows functional programming.

<a id="function-components-declaring"></a>

### Declaring components

#### Use PascalCase names

React treats capitalized JSX names as component references:

```tsx
<UserProfile />
```

Use descriptive names that explain what the component represents.

#### Prefer function declarations for top-level components

I prefer:

```tsx
/**
 * Display a welcome message.
 */
function WelcomeMessage() {
  return <p>Welcome back.</p>;
}

export default WelcomeMessage;
```

You can put parent components before their supporting children with any declaration style: JSX only looks up a child component when the parent renders, after the module has finished loading. I prefer function declarations for consistency with the TypeScript guide.

Named arrow functions are valid too and generally receive useful names in stack traces. This preference is about organization—not a React requirement or an inherent performance advantage.

Classes remain supported, but function components are the usual choice for new components. Some specialized patterns, such as implementing an error boundary directly, may still involve a class or a supporting library.

#### Declare child components at module scope

When small components share a file, place the parent first and its supporting children below it.

Do not define a component inside another component merely to keep it nearby. That creates a new component type on each render and can cause its subtree to remount and lose local state.

Keep the definition outside and pass the values it needs as props.

#### Type the props

Use a descriptive props type, such as `UserProfileProps`, rather than a generic name when the component is shared or the file contains several components.

```tsx
interface WelcomeMessageProps {
  name: string;
}

/**
 * Welcome a user by name.
 */
function WelcomeMessage(props: WelcomeMessageProps) {
  const { name } = props;

  return <p>Welcome back, {name}.</p>;
}

export default WelcomeMessage;
```

For larger files, place props interfaces in the **Types** region.

An `I` prefix is optional. Use it only if it helps distinguish the interface from a related value or class.

#### Let TypeScript infer the return type by default

Components do not always return a JSX element. They can also return `null`, text, numbers, arrays of renderable values, and other supported React content.

Return-type inference usually provides the clearest signature.

If you want an explicit restriction:

- `ReactElement | null` describes an element or no rendered output.
- `ReactNode` describes a broader range of renderable content.

Use type imports from React when needed. Do not assume the global `JSX` namespace is available in every React typing configuration.

Keep ordinary client components synchronous. Async component support depends on the rendering environment and framework.

<a id="function-components-organization"></a>

### Organizing component code

Keep the component easy to scan from top to bottom:

1. Destructure props.
2. Call hooks.
3. Compute derived values.
4. Define event handlers.
5. Return the UI.

This is a reading convention, not a rigid template. Hooks still need to follow their rules regardless of where a section looks best.

#### Keep shared constants and independent helpers outside

Move fixed configuration and helpers that do not depend on a component instance to module scope.

This avoids rebuilding them during each render, but the bigger benefit is making their independence clear.

Be selective:

- Values based on current props or state usually belong inside the component.
- A helper can stay inside when it needs those values, or accept them as explicit arguments.
- Do not move mutable per-user state to module scope. Module values are shared between component instances and may also be shared across server requests.

Creating a small object or callback during rendering is usually inexpensive. Do not complicate the design just to avoid every allocation.

#### Keep rendering pure

Rendering should calculate what the UI looks like—not send requests, write to storage, or mutate shared data.

Use event handlers for work caused by a user action. Use effects when the component needs to synchronize with an external system.

If a value can be calculated from existing props or state, calculate it during rendering instead of storing a second copy and synchronizing it with an effect.

#### Follow the Rules of Hooks

Call hooks such as `useState` and `useEffect` at the top level of a component or custom hook, not inside conditions, loops, or event handlers.

Put conditional behavior inside the hook’s callback when appropriate.

Keep effect dependencies accurate. Do not omit a dependency just to stop an effect from running.

#### Keep sibling layout in the parent

The parent should usually control spacing and positioning between its children.

That makes the relationship between sibling components visible in one place instead of spreading it across unrelated margins and callbacks.

#### Extract meaningful components, not every fragment

A child component is useful when a block has a clear responsibility, its own behavior, or enough markup to distract from the parent.

Short JSX variables are also fine:

```tsx
/**
 * Display a short greeting.
 */
function Greeting() {
  const message = <p>Welcome back.</p>;

  return <section>{message}</section>;
}
```

A JSX value is rendered with `{message}`. It is not a component function, so it should not be rendered as `<Message />`.

For larger blocks, composition usually reads better:

```tsx
import Box from '@mui/material/Box';

interface PostSummary {
  id: string;
  title: string;
}

interface SummaryProps {
  name: string;
  posts: readonly PostSummary[];
}

interface UserOverviewProps extends SummaryProps {
  isCompact?: boolean;
}

/**
 * Default component. Display a user's post summary.
 */
function UserOverview(props: UserOverviewProps) {
  const { name, posts, isCompact = false } = props;

  return (
    <Box sx={{ display: 'grid', gap: 2 }}>
      {isCompact ? (
        <CompactSummary name={name} posts={posts} />
      ) : (
        <DetailedSummary name={name} posts={posts} />
      )}
    </Box>
  );
}

/**
 * Display a user's name and post count.
 */
function CompactSummary(props: SummaryProps) {
  const { name, posts } = props;

  return (
    <p>
      {name}: {posts.length} posts
    </p>
  );
}

/**
 * Display a user's name and individual posts.
 */
function DetailedSummary(props: SummaryProps) {
  const { name, posts } = props;

  return (
    <section>
      <h2>{name}'s posts</h2>
      {posts.length === 0 ? (
        <p>No posts yet.</p>
      ) : (
        <ul>
          {posts.map(post => (
            <li key={post.id}>{post.title}</li>
          ))}
        </ul>
      )}
    </section>
  );
}

export default UserOverview;
```

#### Use memoization when it solves a problem

Do not wrap every handler in `useCallback`.

It is useful when function identity matters—for example, when passing a callback to a memoized child whose unnecessary renders are expensive.

Likewise, use `useMemo` for calculations or identities that benefit from caching, not as a requirement for ordinary derived values.

Start with clear code. Measure before adding performance-oriented complexity.

#### Keep the public component easy to find

I keep the default export at the bottom and start the main component’s documentation with `Default component.`

That is a house convention. Named exports are also valid; consistency matters more than either style. A default export also works directly with `React.lazy()`, which expects the imported module’s default export to be a component; named exports need a small wrapper.

<a id="function-components-props"></a>

### Working with props

Destructure props near the top of the component. This makes its inputs, defaults, and forwarded properties easy to see.

For wrapper components:

- Separate the props your wrapper handles from those it forwards.
- Do not forward internal-only props to DOM elements.
- Decide which underlying behavior callers may override.
- Treat props and their nested values as inputs, not mutable state.

Be especially careful with styling props. Material UI’s `sx` accepts objects, arrays, and functions, so spreading it as if it were always an object is not reliable.

#### Example: a login form

This example uses:

- React Router (v7) for navigation.
- Material UI for controls.
- An application `AuthService.login()` method that accepts credentials, resolves when the login workflow succeeds, and rejects on failure.
- A `Paths` module containing `HOME` and `ACCOUNT` route strings.

The responsibilities are separated:

- `LoginForm` handles form state, feedback, and navigation.
- `AuthService` owns the authentication workflow.
- `AuthApi` handles the HTTP request and response used by that workflow.

```tsx
// LoginForm.tsx
import { useId, useState } from 'react';
import type { FormEvent } from 'react';
import { useNavigate } from 'react-router';
import Alert from '@mui/material/Alert';
import Box from '@mui/material/Box';
import type { BoxProps } from '@mui/material/Box';
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';

import Paths from '@src/_common/constants/Paths';
import AuthService from '@src/domains/auth/AuthService';

// ========================================================================= //
//                                   TYPES                                   //
// ========================================================================= //

type LoginFormProps = Omit<
  BoxProps<'form'>,
  'children' | 'component' | 'onSubmit'
>;

interface LoginFields {
  username: string;
  password: string;
}

type SubmitState =
  | { status: 'idle' }
  | { status: 'submitting' }
  | { status: 'error'; message: string };

// ========================================================================= //
//                                COMPONENTS                                 //
// ========================================================================= //

/**
 * Default component. Let a user log in.
 */
function LoginForm(props: LoginFormProps) {
  const { sx = [], ...boxProps } = props;

  // Hooks.
  const id = useId();
  const navigate = useNavigate();
  const [fields, setFields] = useState<LoginFields>({
    username: '',
    password: '',
  });
  const [submitState, setSubmitState] = useState<SubmitState>({
    status: 'idle',
  });

  // Derived values.
  const isSubmitting = submitState.status === 'submitting';

  // Event handlers.
  const handleSubmit = async (
    event: FormEvent<HTMLFormElement>,
  ): Promise<void> => {
    event.preventDefault();

    if (isSubmitting) return;

    const username = fields.username.trim();

    if (!username || !fields.password) {
      setSubmitState({
        status: 'error',
        message: 'Enter your username and password.',
      });
      return;
    }

    setSubmitState({ status: 'submitting' });

    try {
      await AuthService.login({
        username,
        password: fields.password,
      });

      setSubmitState({ status: 'idle' });
      navigate(Paths.ACCOUNT);
    } catch {
      setSubmitState({
        status: 'error',
        message: 'Unable to log in. Check your details or try again.',
      });
    }
  };

  // UI.
  return (
    <Box
      {...boxProps}
      component="form"
      aria-busy={isSubmitting}
      onSubmit={event => {
        void handleSubmit(event);
      }}
      sx={[
        { display: 'grid', gap: 2 },
        ...(Array.isArray(sx) ? sx : [sx]),
      ]}
    >
      {submitState.status === 'error' && (
        <Alert severity="error">{submitState.message}</Alert>
      )}

      <TextField
        id={`${id}-username`}
        name="username"
        label="Username"
        autoComplete="username"
        required
        disabled={isSubmitting}
        value={fields.username}
        onChange={event => {
          const username = event.currentTarget.value;
          setFields(previous => ({ ...previous, username }));
        }}
      />

      <TextField
        id={`${id}-password`}
        name="password"
        label="Password"
        type="password"
        autoComplete="current-password"
        required
        disabled={isSubmitting}
        value={fields.password}
        onChange={event => {
          const password = event.currentTarget.value;
          setFields(previous => ({ ...previous, password }));
        }}
      />

      <Box sx={{ display: 'flex', gap: 1 }}>
        <Button
          type="button"
          disabled={isSubmitting}
          onClick={() => navigate(Paths.HOME)}
        >
          Cancel
        </Button>

        <Button
          type="submit"
          variant="contained"
          disabled={isSubmitting}
        >
          {isSubmitting ? 'Logging in…' : 'Log in'}
        </Button>
      </Box>
    </Box>
  );
}

// ========================================================================= //
//                                  EXPORT                                   //
// ========================================================================= //

export default LoginForm;
```

A few details matter here:

- The component must render inside the router that supplies `useNavigate`.
- Submitting the form works through the button or the keyboard.
- DOM change handlers read `event.currentTarget.value`.
- The input value is read before entering a state-updater callback.
- State updates based on existing state use functional updaters.
- Both success and failure leave the submitting state.
- The password is not trimmed or otherwise silently changed.
- The `sx` array preserves support for caller-supplied objects, arrays, and theme functions.

Keeping HTTP details out of components is an architectural preference, not a React restriction. It gives the UI a smaller API and keeps request handling reusable and independently testable.

In React 19, form actions and `useActionState` can manage the pending and error states for a submission like this. The example uses an explicit `onSubmit` handler so it also works with earlier versions of React and keeps each state transition visible.

<a id="function-components-form-example"></a>

#### Example: a form with controlled inputs

This form keeps the field values in the parent. Each input owns only its local display state: whether the user has left the field or tried to submit the form.

Required-field errors are derived from the values rather than stored as duplicate parent state.

```tsx
// ContactForm.tsx
import { useId, useState } from 'react';
import type { FormEvent } from 'react';

interface ContactFields {
  name: string;
  email: string;
}

interface ContactFormProps {
  onSubmit: (values: ContactFields) => void;
}

interface TextInputProps {
  label: string;
  value: string;
  type?: 'text' | 'email';
  isRequired?: boolean;
  onChange: (value: string) => void;
}

/**
 * Default component. Collect a name and email address.
 */
function ContactForm(props: ContactFormProps) {
  const { onSubmit } = props;

  const [state, setState] = useState<ContactFields>({
    name: '',
    email: '',
  });

  const isMissingRequiredValue =
    !state.name.trim() || !state.email.trim();

  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    if (isMissingRequiredValue) return;

    onSubmit({
      name: state.name.trim(),
      email: state.email.trim(),
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <TextInput
        label="Name"
        value={state.name}
        isRequired
        onChange={value => {
          setState(previous => ({ ...previous, name: value }));
        }}
      />

      <TextInput
        label="Email"
        type="email"
        value={state.email}
        isRequired
        onChange={value => {
          setState(previous => ({ ...previous, email: value }));
        }}
      />

      <button type="submit">
        Submit
      </button>
    </form>
  );
}

/**
 * Display a labeled input and its required-field message.
 */
function TextInput(props: TextInputProps) {
  const {
    label,
    value,
    type = 'text',
    isRequired = false,
    onChange,
  } = props;

  const id = useId();
  const [isTouched, setIsTouched] = useState(false);

  const isMissing = isRequired && value.trim().length === 0;
  const isErrorVisible = isTouched && isMissing;

  return (
    <div>
      <label htmlFor={id}>{label}</label>

      <input
        id={id}
        type={type}
        required={isRequired}
        pattern={isRequired ? '.*\\S.*' : undefined}
        value={value}
        aria-invalid={isErrorVisible || undefined}
        aria-describedby={isErrorVisible ? `${id}-error` : undefined}
        onBlur={() => setIsTouched(true)}
        onInvalid={() => setIsTouched(true)}
        onChange={event => onChange(event.currentTarget.value)}
      />

      {isErrorVisible && (
        <p id={`${id}-error`}>{label} is required.</p>
      )}
    </div>
  );
}

export default ContactForm;
```

The submit button stays enabled. Disabling it until the form is valid hides why submission isn’t possible, and a disabled button can’t receive keyboard focus. Instead, a submit attempt fires the `invalid` event on each field that fails native validation (including whitespace-only values, via `pattern`), which reveals that field’s error message.

The native email input also provides browser email-format validation during form submission. It does not prove that an address exists, and client-side validation does not replace server-side validation.

Notice that the input does not trim the value on every keystroke. Doing that can interfere with typing spaces and moving the cursor.

Normalize values at a deliberate boundary, such as submission, and only when the field’s rules allow it.

The parent supplies `onSubmit`, so this form does not need to know which service or API client handles the submitted data. An asynchronous workflow should also provide appropriate pending and error feedback.

---

<a id="containers-and-state-management"></a>

## 🔄 Containers and state management

Keep state as close as possible to the components that need it.

When a component starts mixing several responsibilities—loading data, coordinating workflows, and rendering a large UI—consider separating those responsibilities.

Do not create extra layers just to satisfy a pattern. A small component can reasonably handle its own state and rendering.

<a id="container-presenter-pattern"></a>

### The Container/Presenter pattern

The pattern separates two responsibilities:

| Role | Responsibility |
| --- | --- |
| **Container** | Coordinates data loading, application state, and interactions with domain services or API clients. |
| **Presenter** | Receives data and callbacks, then renders the UI. |

Presenters do not need to be completely stateless. They can own local display behavior, such as whether a section is expanded or which tab is selected.

The useful boundary is whether the component needs to know about the application workflow or where its data comes from.

Custom hooks are another way to separate these concerns. You do not need a separate container file for every component.

#### Example: loading and displaying users

The examples use this domain type:

```ts
// domains/users/User.ts
export interface User {
  id: string;
  firstName: string;
  lastName?: string | null;
}
```

The application’s `UserApi.fetchAll({ signal })` returns a `Promise<User[]>` and forwards the abort signal to its HTTP client.

This is a straightforward read with no additional business workflow, so the container calls `UserApi` directly.

The API client is also the boundary for checking external response data. A TypeScript return annotation alone does not validate JSON at runtime.

##### Container

```tsx
// UsersContainer.tsx
import { useEffect, useState } from 'react';

import type { User } from '@src/domains/users/User';
import UserApi from '@src/domains/users/UserApi';

import UsersList from './UsersList';

type UsersState =
  | { status: 'loading' }
  | { status: 'success'; users: User[] }
  | { status: 'error'; message: string };

/**
 * Default component. Load users and handle request states.
 */
function UsersContainer() {
  const [state, setState] = useState<UsersState>({
    status: 'loading',
  });

  useEffect(() => {
    const controller = new AbortController();
    let isActive = true;

    const loadUsers = async (): Promise<void> => {
      try {
        const users = await UserApi.fetchAll({
          signal: controller.signal,
        });

        if (isActive) {
          setState({ status: 'success', users });
        }
      } catch {
        if (isActive) {
          setState({
            status: 'error',
            message: 'Unable to load users. Please try again later.',
          });
        }
      }
    };

    void loadUsers();

    return () => {
      isActive = false;
      controller.abort();
    };
  }, []);

  if (state.status === 'loading') {
    return <p role="status">Loading users…</p>;
  }

  if (state.status === 'error') {
    return <p role="alert">{state.message}</p>;
  }

  return <UsersList users={state.users} />;
}

export default UsersContainer;
```

The cleanup requests cancellation and prevents late results from updating this effect’s state.

React Strict Mode may run an extra effect setup-and-cleanup cycle during development. Cleanup should make that safe rather than trying to suppress it.

This example has no changing request inputs. If the request depends on a user ID, filter, or other reactive value, include it in the effect’s dependencies and handle the loading transition when it changes.

##### Presenter

```tsx
// UsersList.tsx
import type { User } from '@src/domains/users/User';

interface UsersListProps {
  users: readonly User[];
}

/**
 * Default component. Display a list of users.
 */
function UsersList(props: UsersListProps) {
  const { users } = props;

  if (users.length === 0) {
    return <p>No users yet.</p>;
  }

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{getUserDisplayName(user)}</li>
      ))}
    </ul>
  );
}

/**
 * Combine the user's name fields for display.
 *
 * Used by {@link UsersList}.
 *
 * @private
 */
function getUserDisplayName(user: User): string {
  const name = `${user.firstName} ${user.lastName ?? ''}`.trim();
  return name || 'Unnamed user';
}

export default UsersList;
```

The presenter does not know how users are fetched. Its formatting helper stays nearby because it is specific to this display.

The helper also preserves the user’s chosen capitalization rather than automatically recasing their name.

#### Example: moving the workflow into a custom hook

A custom hook can replace the container file. The hook owns the loading workflow, and the component decides what to render for each state:

```ts
// Users/_local/useUsers.ts
import { useEffect, useState } from 'react';

import type { User } from '@src/domains/users/User';
import UserApi from '@src/domains/users/UserApi';

type UsersState =
  | { status: 'loading' }
  | { status: 'success'; users: User[] }
  | { status: 'error'; message: string };

/**
 * Load users and track the request state.
 */
function useUsers(): UsersState {
  const [state, setState] = useState<UsersState>({
    status: 'loading',
  });

  useEffect(() => {
    const controller = new AbortController();
    let isActive = true;

    const loadUsers = async (): Promise<void> => {
      try {
        const users = await UserApi.fetchAll({
          signal: controller.signal,
        });

        if (isActive) {
          setState({ status: 'success', users });
        }
      } catch {
        if (isActive) {
          setState({
            status: 'error',
            message: 'Unable to load users. Please try again later.',
          });
        }
      }
    };

    void loadUsers();

    return () => {
      isActive = false;
      controller.abort();
    };
  }, []);

  return state;
}

export default useUsers;
```

```tsx
// Users/Users.tsx
import useUsers from './_local/useUsers';
import UsersList from './UsersList';

/**
 * Default component. Display users and their request states.
 */
function Users() {
  const state = useUsers();

  if (state.status === 'loading') {
    return <p role="status">Loading users…</p>;
  }

  if (state.status === 'error') {
    return <p role="alert">{state.message}</p>;
  }

  return <UsersList users={state.users} />;
}

export default Users;
```

The hook lives in the page’s `_local/` folder because only this page uses it. Move it to a shared folder once other components need the same workflow, and remember that each component calling `useUsers` still gets its own request and state.

#### Prefer existing data-loading tools when they fit

An effect-based example makes the lifecycle explicit, but larger applications often benefit from a router loader or a query library.

Those tools can handle concerns such as:

- Caching and request deduplication.
- Refetching and retries.
- Loading and error states.
- Keeping server data fresh.

Avoid rebuilding those features separately in every container.

A loader or query function can call the same `UserApi` module. If loading requires a business workflow, it can call `UserService` instead.

<a id="choosing-and-updating-state"></a>

### Choosing and updating state

Choose state tools based on how values relate and how they are shared—not on the number of `useState` calls.

| Situation | A useful starting point |
| --- | --- |
| A value used by one component | `useState`. |
| Several independent local values | Separate `useState` calls. |
| Related fields that often change together | An object in `useState` or a shallow-merge state hook. |
| Several coordinated state transitions | `useReducer`. |
| State shared by a few nearby components | Lift it to their nearest common parent and pass props. |
| State needed across many nested branches | Context combined with state or a reducer. |
| Complex shared client state | A dedicated store, such as Redux, when its capabilities are useful. |
| Cached server data | A framework data layer, router loader, or query library. |

A few levels of prop passing are not automatically a problem. Context becomes useful when passing the same values through unrelated intermediate components makes the code harder to follow.

Context distributes a value; it does not store or update state on its own. A provider usually gets its state from hooks or an external store.

Changing a context value updates its consumers. Split unrelated concerns rather than putting every changing field into one large provider.

Also, calling the same custom hook in two components does not automatically share state. Each call gets its own hook state unless the hook connects to a shared provider or store.

#### `useState` replaces values

When state is an object, `useState` does not merge a partial update for you.

Preserve the other fields explicitly:

```tsx
setForm(previous => ({
  ...previous,
  name: value,
}));
```

Use a functional updater when the next value depends on the previous one.

#### `useSetState` shallow-merges updates

A hook such as [`react-use`’s `useSetState`](https://github.com/streamich/react-use/blob/master/src/useSetState.ts) can make related form fields less repetitive.

It merges the supplied patch into the current object:

```tsx
setState({ name: value });
```

That merge is **shallow**. Updating a nested object replaces that nested property unless you merge it explicitly.

This is an optional convenience—not a required upgrade once a component has more than two state variables.

#### Reset behavior depends on the hook

React’s `useState` does not return a reset function. The linked `react-use` implementation of `useSetState` returns two items: the state and its updater.

If you want a reset operation, define it or choose a hook that explicitly provides one.

For a fixed-shape state object:

```tsx
import { useSetState } from 'react-use';

interface DraftState {
  name: string;
  email: string;
}

const DraftDefaults = (): DraftState => ({
  name: '',
  email: '',
});

/**
 * Default component. Edit a contact draft and restore its defaults.
 */
function ContactDraft() {
  const [state, setState] = useSetState<DraftState>(DraftDefaults());

  const resetState = () => setState(DraftDefaults());

  return (
    <section aria-label="Contact draft">
      <label>
        Name
        <input
          value={state.name}
          onChange={event => {
            setState({ name: event.currentTarget.value });
          }}
        />
      </label>

      <label>
        Email
        <input
          type="email"
          value={state.email}
          onChange={event => {
            setState({ email: event.currentTarget.value });
          }}
        />
      </label>

      <button type="button" onClick={resetState}>
        Reset
      </button>
    </section>
  );
}

export default ContactDraft;
```

Because this reset uses a merge, it only resets the supplied fields. It does not remove additional properties.

If resetting must replace the entire object or remove keys, use replacement semantics with `useState`, a reducer, or a hook with an explicit reset operation.

#### Keep state minimal

Avoid storing values that can be derived from existing state.

For example, a required-field error can often be calculated from the current field value instead of keeping a separate error boolean synchronized with it.

For asynchronous work, a status such as `'idle'`, `'loading'`, `'success'`, or `'error'` can be clearer than several booleans that allow contradictory combinations.

---

<a id="styling-and-everyday-conventions"></a>

## 🎨 Styling and everyday conventions

Small conventions help a codebase feel consistent. They are most useful when they remove decisions without making ordinary work harder.

<a id="styling-the-ui"></a>

### Styling the UI

#### Use shared design tokens

Keep reusable colors in a theme or a shared token module instead of scattering color literals across components.

Prefer semantic names that explain the role of a color:

- Background.
- Surface.
- Border.
- Primary text.
- Muted text.
- Error text.

For example:

```ts
// src/components/_common/styles/Colors.ts

const Base = {
  Grey: {
    LIGHTEST: '#f9fafb',
    LIGHT: '#e5e7eb',
    MEDIUM: '#6b7280',
    DARK: '#111827',
  },
  Red: {
    DEFAULT: '#b91c1c',
    DARK: '#991b1b',
  },
  WHITE: '#ffffff',
} as const;

const Colors = {
  Background: {
    DEFAULT: Base.Grey.LIGHTEST,
    SURFACE: Base.WHITE,
    SELECTED: Base.Grey.LIGHT,
  },
  BORDER: Base.Grey.MEDIUM,
  Text: {
    DEFAULT: Base.Grey.DARK,
    MUTED: Base.Grey.MEDIUM,
    ERROR: Base.Red.DEFAULT,
  },
} as const;

export default Colors;
```

Here is a small component that uses those tokens. Its fixed layout lives in a CSS Module, and the inline style only carries token values, including one that changes with the component’s state:

```css
/* UserCard.module.css */
.card {
  padding: 8px 16px;
  border: 1px solid;
  border-radius: 4px;
  text-align: left;
}
```

```tsx
// UserCard.tsx
import Colors from '@src/components/_common/styles/Colors';

import styles from './UserCard.module.css';

interface UserCardProps {
  name: string;
  isSelected: boolean;
  onSelect: () => void;
}

/**
 * Default component. Display a user that can be selected.
 */
function UserCard(props: UserCardProps) {
  const { name, isSelected, onSelect } = props;

  return (
    <button
      type="button"
      className={styles.card}
      aria-pressed={isSelected}
      style={{
        backgroundColor: isSelected
          ? Colors.Background.SELECTED
          : Colors.Background.SURFACE,
        borderColor: Colors.BORDER,
        color: Colors.Text.DEFAULT,
      }}
      onClick={onSelect}
    >
      {name}
    </button>
  );
}

export default UserCard;
```

The tokens are applied inline because they live in TypeScript. If most of your styling lives in CSS, expose the tokens as CSS custom properties so stylesheets can also use them for hover and focus states.

If a component library already provides a theme, use that as the source of truth rather than maintaining a competing color system. For Material UI, that often means using the theme’s palette and semantic `sx` values.

A semantic token name does not guarantee accessible contrast. Check the actual foreground/background combinations, including hover, focus, disabled, and dark-mode states.

#### Choose the right styling tool

Inline styles are fine for small, dynamic values.

For reusable styles, responsive layouts, hover states, and focus states, use your project’s CSS, CSS Modules, theme, or styling system.

Avoid introducing several styling approaches without a clear reason.

#### Use interactive elements for interactions

Use a button for an action and a link for navigation.

A clickable `<div>` does not automatically provide keyboard interaction, focus behavior, or button semantics.

Prefer native elements before rebuilding their behavior yourself.

<a id="callback-parameter-names"></a>

### Callback parameter names

Use names that make the callback’s input obvious:

| Input | Useful name |
| --- | --- |
| A DOM or React event | `event` |
| A field’s new value | `value` |
| An error object | `error` or `err` |
| A boolean validation result | `isInvalid` |
| An item from a collection | Its domain name, such as `user` or `post` |

Short names such as `v` are acceptable in tiny callbacks when the meaning is obvious. Do not shorten a name if it makes readers inspect the component’s type just to understand the argument.

Be especially careful not to confuse an input event with the input’s value:

```tsx
onChange={event => {
  onValueChange(event.currentTarget.value);
}}
```

For custom components, a value-based callback can keep the parent independent of DOM event details.

For a complete form that uses value-based callbacks, see [Example: a form with controlled inputs](#function-components-form-example).

If a custom callback needs to report a validation result, use an explicit contract:

```ts
onChange: (value: string, isInvalid: boolean) => void;
```

Do not make the boolean optional unless `undefined` has a meaningful role.

<a id="other-conventions"></a>

### Other conventions

#### Keep formatting automatic

I use single quotes in JavaScript and TypeScript, and double quotes for JSX attributes.

With Prettier:

```json
{
  "singleQuote": true,
  "jsxSingleQuote": false
}
```

Let the formatter handle these decisions so code reviews can focus on behavior.

#### Use stable keys

For dynamic lists, use stable identifiers from the data:

```tsx
users.map(user => <UserRow key={user.id} user={user} />)
```

Do not generate a fresh key during each render.

Array indexes are a poor fit when items can be inserted, removed, or reordered. `useId` is useful for accessibility relationships, not list keys.

#### Make forms and actions accessible

- Associate labels with their inputs.
- Use `type="button"` for non-submit buttons inside forms.
- Make loading and error states understandable without relying only on color.
- Preserve visible keyboard focus.
- Use meaningful alternative text for informative images and empty alternative text for decorative ones.

#### Test what the user can observe

Test rendering, interactions, keyboard submission, loading states, empty states, and failure paths.

Avoid tying every test to internal state variables or implementation details. A refactor should not break a test when the user-visible behavior is unchanged.

For example, with Vitest and React Testing Library, a test for the [contact form](#function-components-form-example) can interact with it the way a user would:

```tsx
// ContactForm.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { expect, test, vi } from 'vitest';

import ContactForm from './ContactForm';

test('submits trimmed values', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();

  render(<ContactForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText('Name'), '  Ada Lovelace ');
  await user.type(screen.getByLabelText('Email'), 'ada@example.com');
  await user.click(screen.getByRole('button', { name: 'Submit' }));

  expect(onSubmit).toHaveBeenCalledWith({
    name: 'Ada Lovelace',
    email: 'ada@example.com',
  });
});
```

The test finds elements by their labels and roles, so it keeps passing if the form’s internal state changes shape.

The overall goal is the same as in the TypeScript guide: **clear responsibilities, predictable structure, and code that explains itself without making readers work too hard.**

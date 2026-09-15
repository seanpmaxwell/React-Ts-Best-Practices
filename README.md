Same treatment as before: every code sample, table, link, and anchor is preserved; I cut restatements, hedges, and connective filler. Should land around 60% of the original.

```md
# ⚛️ React + TypeScript Best Practices

[![GitHub stars](https://img.shields.io/github/stars/seanpmaxwell/React-Ts-Best-Practices?style=flat-square)](https://github.com/seanpmaxwell/React-Ts-Best-Practices/stargazers)

Practical patterns for React applications that are easier to read, test, and maintain.

This guide builds on the [TypeScript best practices](https://github.com/seanpmaxwell/Typescript-Best-Practices) document and covers React-specific decisions only. Some recommendations are React fundamentals; others are house conventions. Use what helps your team, and follow your framework's requirements where they differ.

Examples focus on client-side function components. File-based routing and Server Components may call for a different layout or data-loading approach.

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

A good structure answers two questions: where should new code go, and where would someone else expect to find it? Start small; add folders when they clarify responsibilities.

<a id="project-structure-overview"></a>

### Overview

My usual client-side starting layout:

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
| `public/` | Static files served directly, per your build tool. |
| `src/assets/` | Assets imported by application code. |
| `src/_common/` | Shared code not tied to React or one domain. |
| `src/components/` | Components, pages, and React-specific UI logic. |
| `src/domains/` | Domain models, business services, and domain API clients. |
| `src/infra/` | Shared integrations, such as an HTTP client or storage adapter. |
| `src/main.tsx` | The application entry point. |

`main.tsx` is the entry point; `index.ts` is reserved for barrels. This is a house convention—if your framework defines its own entry points or routing folders, follow those.

Anything in a client bundle is public. `.env` does not make a value secret once the build exposes it to the browser.

<a id="project-structure-where-code-belongs"></a>

### Where code belongs

#### Name components and their files consistently

PascalCase for components; match the filename to the main export:

```text
Account/
├── Account.tsx
└── Account.test.tsx
```

A small component can be a single file. Give it a folder when it gains tests, styles, hooks, or supporting components. Use `.tsx` for files with JSX; plain helpers belong in `.ts`.

#### Organize larger applications by domain

For apps with several business areas, I prefer domain-based organization. See the [architecture section](https://github.com/seanpmaxwell/Typescript-Best-Practices/blob/main/README.md#architecture) of the TypeScript guide for trade-offs.

Within a domain:

| File | Responsibility |
| --- | --- |
| `User.ts` | User types and model helpers. |
| `UserService.ts` | User business rules and workflows. |
| `UserApi.ts` | User HTTP requests and response handling. |

**Service** means business logic on both frontend and backend; **Api** is the client-side boundary with the server. For example:

- Checking whether a user is eligible for an action → domain logic.
- Deciding how to display that eligibility → UI.
- Coordinating an account-update workflow → `UserService`.
- Sending the account-update request → `UserApi`.

Client-side permission checks shape the UI; the server must still enforce authorization.

#### Keep React-specific logic near its consumers

A hook does not automatically belong in a global hooks folder. Decide by scope:

- Used by one component → stays near that component.
- Shared by related components → their nearest shared folder.
- Broadly reusable UI hook → `components/_common/hooks/`.

Prefix custom hooks with `use`, e.g. `usePageTitle` or `usePaymentForm`.

Keep reusable business rules independent of React where practical. A hook can coordinate those rules with component state without moving them into the UI layer.

#### Keep dependencies flowing in one direction

A business workflow typically follows:

```text
Component or custom hook
        ↓
Domain service
        ↓
Domain API client
        ↓
Shared HTTP client
```

- **Component or hook:** React state, navigation, UI interactions.
- **Service:** Business rules and workflow coordination.
- **API client:** Endpoints, payloads, responses.
- **HTTP client:** Shared transport configuration.

Not every request needs every layer. A container that just loads users can call `UserApi.fetchAll()` directly; a workflow with validation, transformation, or several requests belongs in `UserService`. Don't add a service that merely forwards a call.

Domain services must not depend on components or hooks. Navigation, dialogs, and other UI behavior stay in the component or hook that owns them.

<a id="project-structure-example"></a>

### Example layout

A more developed `src/`:

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

Notes:

- `Account.tsx` composes `UpdatePaymentForm`.
- `PostForm.tsx` is shared by New and Edit.
- Payment rules live in `PaymentService`; requests in `PaymentApi`.
- `usePaymentForm` connects the payment workflow to React state.
- UI folders are grouped by purpose, not size labels like `sm`/`md`/`lg`.
- Navigation paths and API paths have separate, named homes.

Examples below use `@src/` as an alias for `src/`; configure it in TypeScript and your build/test tools. `AuthService`, `UserApi`, `Paths`, etc. are project code, not React APIs.

---

<a id="function-components"></a>

## 🧩 Function components

Function components are the default for new React code. "Function component" describes the declaration style, not a commitment to functional programming.

<a id="function-components-declaring"></a>

### Declaring components

#### Use PascalCase names

React treats capitalized JSX names as component references:

```tsx
<UserProfile />
```

Use descriptive names.

#### Prefer function declarations for top-level components

```tsx
/**
 * Display a welcome message.
 */
function WelcomeMessage() {
  return <p>Welcome back.</p>;
}

export default WelcomeMessage;
```

Any declaration style lets parents precede children—JSX resolves a child when the parent renders, after the module loads. I use declarations for consistency with the TypeScript guide. Named arrow functions are valid and get useful stack-trace names; this is organizational preference, not a React requirement or performance win.

Class components remain supported, but functions are the norm. Some patterns, such as a hand-rolled error boundary, may still need a class or library.

#### Declare child components at module scope

When small components share a file, put the parent first and children below.

Never define a component inside another just to keep it nearby. That creates a new component type each render, remounting the subtree and losing local state. Define it outside and pass what it needs as props.

#### Type the props

Use a descriptive props type like `UserProfileProps` when the component is shared or the file has several components.

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

In larger files, props interfaces go in the **Types** region. An `I` prefix is optional—use it only to distinguish from a related value or class.

#### Let TypeScript infer the return type

Components can return `null`, text, numbers, arrays, and other renderable content, so inference usually gives the clearest signature. For an explicit restriction: `ReactElement | null` for an element or nothing; `ReactNode` for broader content. Import types from React—do not assume a global `JSX` namespace.

Keep ordinary client components synchronous; async support depends on the framework.

<a id="function-components-organization"></a>

### Organizing component code

Top to bottom:

1. Destructure props.
2. Call hooks.
3. Compute derived values.
4. Define event handlers.
5. Return the UI.

This is a reading convention, not a template. Hooks still follow their rules regardless of where a section looks best.

#### Keep shared constants and independent helpers outside

Move fixed configuration and instance-independent helpers to module scope. It avoids rebuilding them per render, but mainly it makes their independence visible.

Be selective:

- Values based on props or state belong inside.
- A helper that needs those values stays inside or takes them as arguments.
- Never put mutable per-user state at module scope—it is shared across instances and possibly across server requests.

Small allocations during render are cheap. Don't contort the design to avoid them.

#### Keep rendering pure

Rendering computes the UI; it does not send requests, write to storage, or mutate shared data. Use event handlers for user-triggered work and effects for syncing with external systems.

If a value can be derived from props or state, derive it during render instead of storing a copy and syncing it with an effect.

#### Follow the Rules of Hooks

Call `useState`, `useEffect`, etc. at the top level of a component or custom hook—not in conditions, loops, or handlers. Put conditional logic inside the hook's callback. Keep dependencies accurate; never omit one just to stop an effect from running.

#### Keep sibling layout in the parent

The parent controls spacing and positioning between children, so the relationship is visible in one place rather than scattered across margins.

#### Extract meaningful components, not every fragment

Extract a child when a block has a clear responsibility, its own behavior, or enough markup to distract. Short JSX variables are fine too:

```tsx
/**
 * Display a short greeting.
 */
function Greeting() {
  const message = <p>Welcome back.</p>;

  return <section>{message}</section>;
}
```

A JSX value renders as `{message}`; it is not a component and should not be rendered as `<Message />`.

For larger blocks, composition reads better:

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

#### Memoize when it solves a problem

Don't wrap every handler in `useCallback`. It matters when function identity matters—e.g. passing a callback to a memoized child whose re-renders are expensive. Same for `useMemo`: use it for calculations or identities that benefit from caching, not for every derived value. Write clear code first; measure before adding complexity.

#### Keep the public component easy to find

I put the default export at the bottom and start the main component's doc comment with `Default component.` Named exports are also valid; consistency matters more. Default exports also work directly with `React.lazy()`, which expects a default-exported component; named exports need a small wrapper.

<a id="function-components-props"></a>

### Working with props

Destructure props at the top so inputs, defaults, and forwarded properties are visible.

For wrappers:

- Separate props you handle from props you forward.
- Don't forward internal-only props to DOM elements.
- Decide which underlying behavior callers may override.
- Treat props and their nested values as inputs, not mutable state.

Be careful with styling props. Material UI's `sx` accepts objects, arrays, and functions, so spreading it as an object is unreliable.

#### Example: a login form

Uses React Router (v7), Material UI, an `AuthService.login()` that resolves on success and rejects on failure, and a `Paths` module with `HOME` and `ACCOUNT`.

- `LoginForm` handles form state, feedback, and navigation.
- `AuthService` owns the authentication workflow.
- `AuthApi` handles the HTTP request and response.

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

Details:

- Must render inside the router that supplies `useNavigate`.
- Submission works via button or keyboard.
- DOM handlers read `event.currentTarget.value`.
- The input value is read before entering a state-updater callback.
- Updates based on existing state use functional updaters.
- Both success and failure leave the submitting state.
- The password is never trimmed or silently changed.
- The `sx` array preserves caller-supplied objects, arrays, and theme functions.

Keeping HTTP out of components is an architectural preference. It gives the UI a smaller API and keeps request handling reusable and testable.

React 19's form actions and `useActionState` can manage pending/error states for this. The explicit `onSubmit` works with earlier versions and keeps each transition visible.

<a id="function-components-form-example"></a>

#### Example: a form with controlled inputs

Field values live in the parent. Each input owns only its display state: whether the user has left the field or tried to submit. Required-field errors are derived, not stored.

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

The submit button stays enabled. Disabling it hides why submission is blocked and removes it from keyboard focus. Instead, a submit attempt fires `invalid` on each failing field (including whitespace-only values, via `pattern`), which reveals that field's error.

The native email input adds browser format validation on submit. It does not prove the address exists, and client validation never replaces server validation.

The input does not trim on every keystroke—that interferes with typing spaces and cursor movement. Normalize at a deliberate boundary, such as submission, when the field's rules allow.

The parent supplies `onSubmit`, so the form does not know which service handles the data. An async workflow should also provide pending and error feedback.

---

<a id="containers-and-state-management"></a>

## 🔄 Containers and state management

Keep state as close as possible to the components that need it. When one component mixes data loading, workflow coordination, and a large UI, consider splitting those responsibilities—but don't add layers just to satisfy a pattern. A small component can own its own state and rendering.

<a id="container-presenter-pattern"></a>

### The Container/Presenter pattern

| Role | Responsibility |
| --- | --- |
| **Container** | Coordinates data loading, application state, and calls to services or API clients. |
| **Presenter** | Receives data and callbacks; renders the UI. |

Presenters need not be stateless—they can own display behavior such as which tab is selected. The real boundary is whether the component needs to know about the workflow or where data comes from.

Custom hooks are another way to separate these concerns; not every component needs a container file.

#### Example: loading and displaying users

Domain type:

```ts
// domains/users/User.ts
export interface User {
  id: string;
  firstName: string;
  lastName?: string | null;
}
```

`UserApi.fetchAll({ signal })` returns `Promise<User[]>` and forwards the abort signal. This is a plain read with no workflow, so the container calls `UserApi` directly. The API client is also where external response data gets checked—a return annotation does not validate JSON at runtime.

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

Cleanup cancels the request and blocks late results. Strict Mode may run an extra setup/cleanup cycle in development; cleanup should make that safe rather than suppress it.

This request has no changing inputs. If it depends on an ID, filter, or other reactive value, add it to the dependencies and handle the loading transition on change.

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

The presenter doesn't know how users are fetched. Its formatting helper stays nearby because it is specific to this display, and it preserves the user's capitalization rather than recasing.

#### Example: moving the workflow into a custom hook

The hook owns loading; the component decides what to render:

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

The hook lives in `_local/` because only this page uses it. Move it when others need it—and remember each caller of `useUsers` gets its own request and state.

#### Prefer existing data-loading tools when they fit

Effects make the lifecycle explicit, but larger apps benefit from a router loader or query library, which handle caching and deduplication, refetching and retries, loading/error states, and freshness. Don't rebuild those per container. A loader or query can call the same `UserApi`, or `UserService` when a workflow is involved.

<a id="choosing-and-updating-state"></a>

### Choosing and updating state

Choose by how values relate and how they're shared—not by the count of `useState` calls.

| Situation | A useful starting point |
| --- | --- |
| A value used by one component | `useState`. |
| Several independent local values | Separate `useState` calls. |
| Related fields that often change together | An object in `useState` or a shallow-merge hook. |
| Several coordinated transitions | `useReducer`. |
| State shared by a few nearby components | Lift to the nearest common parent. |
| State needed across many nested branches | Context plus state or a reducer. |
| Complex shared client state | A dedicated store such as Redux. |
| Cached server data | Framework data layer, router loader, or query library. |

A few levels of prop passing are fine. Context earns its place when threading the same values through unrelated intermediates hurts readability.

Context distributes a value; it does not store or update state. Changing it re-renders all consumers, so split unrelated concerns rather than one giant provider. Calling the same custom hook in two components does not share state unless the hook connects to a provider or store.

#### `useState` replaces values

For object state, `useState` does not merge. Preserve other fields explicitly, and use a functional updater when the next value depends on the previous:

```tsx
setForm(previous => ({
  ...previous,
  name: value,
}));
```

#### `useSetState` shallow-merges updates

A hook like [`react-use`'s `useSetState`](https://github.com/streamich/react-use/blob/master/src/useSetState.ts) merges a patch into the current object:

```tsx
setState({ name: value });
```

The merge is **shallow**—nested objects are replaced unless merged explicitly. This is an optional convenience, not a required upgrade past two state variables.

#### Reset behavior depends on the hook

Neither `useState` nor `react-use`'s `useSetState` returns a reset function; the latter returns only state and updater. Define your own or pick a hook that provides one.

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

Because this reset merges, it resets only the supplied fields and removes nothing. To replace the whole object or drop keys, use `useState`, a reducer, or a hook with explicit reset.

#### Keep state minimal

Don't store what can be derived. A required-field error can be computed from the field value rather than kept in a synced boolean. For async work, a status like `'idle' | 'loading' | 'success' | 'error'` beats several booleans that permit contradictory combinations.

---

<a id="styling-and-everyday-conventions"></a>

## 🎨 Styling and everyday conventions

Small conventions make a codebase feel consistent. They're most useful when they remove decisions without adding friction.

<a id="styling-the-ui"></a>

### Styling the UI

#### Use shared design tokens

Keep reusable colors in a theme or token module, not scattered literals. Prefer semantic names by role: background, surface, border, primary text, muted text, error text.

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

A component using those tokens—fixed layout in a CSS Module, inline style carrying only token values, one of which depends on state:

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

Tokens are inline because they live in TypeScript. If most styling is CSS, expose them as custom properties so stylesheets can use them for hover and focus.

If a component library provides a theme, use it as the source of truth—for Material UI, the palette and semantic `sx` values—rather than a competing system. Semantic names don't guarantee accessible contrast; check real combinations including hover, focus, disabled, and dark mode.

#### Choose the right styling tool

Inline styles for small dynamic values. CSS, CSS Modules, theme, or your styling system for reusable styles, responsive layouts, hover, and focus. Don't mix several approaches without reason.

#### Use interactive elements for interactions

Button for actions, link for navigation. A clickable `<div>` provides no keyboard interaction, focus behavior, or semantics. Prefer native elements before rebuilding them.

<a id="callback-parameter-names"></a>

### Callback parameter names

| Input | Useful name |
| --- | --- |
| A DOM or React event | `event` |
| A field's new value | `value` |
| An error object | `error` or `err` |
| A boolean validation result | `isInvalid` |
| An item from a collection | Its domain name, such as `user` or `post` |

Short names like `v` are fine in tiny callbacks when obvious. Don't shorten if readers must inspect the type to understand the argument.

Don't confuse an event with its value:

```tsx
onChange={event => {
  onValueChange(event.currentTarget.value);
}}
```

For custom components, value-based callbacks keep the parent independent of DOM events. See [Example: a form with controlled inputs](#function-components-form-example).

To report a validation result, use an explicit contract:

```ts
onChange: (value: string, isInvalid: boolean) => void;
```

Don't make the boolean optional unless `undefined` means something.

<a id="other-conventions"></a>

### Other conventions

#### Keep formatting automatic

Single quotes in JS/TS, double quotes in JSX attributes. With Prettier:

```json
{
  "singleQuote": true,
  "jsxSingleQuote": false
}
```

Let the formatter decide so reviews focus on behavior.

#### Use stable keys

Use stable identifiers from the data:

```tsx
users.map(user => <UserRow key={user.id} user={user} />)
```

Never generate a fresh key per render. Indexes break when items are inserted, removed, or reordered. `useId` is for accessibility relationships, not list keys.

#### Make forms and actions accessible

- Associate labels with inputs.
- `type="button"` for non-submit buttons in forms.
- Loading and error states understandable without color alone.
- Preserve visible keyboard focus.
- Meaningful alt text for informative images; empty alt for decorative ones.

#### Test what the user can observe

Test rendering, interactions, keyboard submission, loading, empty, and failure states. Don't tie tests to internal state or implementation details; a refactor with unchanged user-visible behavior shouldn't break them.

With Vitest and React Testing Library, a test for the [contact form](#function-components-form-example) interacts like a user:

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

Finding elements by label and role keeps the test passing if internal state changes shape.

The goal matches the TypeScript guide: **clear responsibilities, predictable structure, and code that explains itself without making readers work too hard.**
```

One factual check while I was at it: the `useSetState` source confirms it returns exactly `[state, setState]` with an `Object.assign` merge, so the "no reset function" and "shallow merge" claims in your doc are accurate.

Same offer as before—if you want it even shorter, the biggest remaining chunks are the duplicated `useEffect` body in `UsersContainer` and `useUsers` (you could show the hook only and reference it from the container), and the `ContactDraft` example. Want me to consolidate those?

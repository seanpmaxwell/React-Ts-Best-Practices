## React + TypeScript Best Practices
[![GitHub stars](https://img.shields.io/github/stars/seanpmaxwell/React-Ts-Best-Practices?style=flat-square)](https://github.com/seanpmaxwell/React-Ts-Best-Practices/stargazers)

This guide focuses on React-specific habits that pair well with TypeScript. It assumes you already follow the [TypeScript best practices](https://github.com/seanpmaxwell/Typescript-Best-Practices) document; concepts already covered there are not repeated here.

## Table of contents
- [Project structure](#project-structure)
  - [Overview summary](#project-structure-overview)
  - [Overview in detail](#project-structure-structuring)
  - [Example layout](#project-structure-example)
- [Functional components](#functional-components)
  - [Declaring components](#functional-components-declaring)
  - [Organizing component code](#functional-components-organization)
  - [Working with props](#functional-components-props)
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
    - common/
    - pages/
    - App.test.tsx
    - App.tsx
    - index.css
    - index.tsx
  - domains/
  - infra/
- .env
- package.json
- tsconfig.json
```

### Overview in detail <a name="project-structure-structuring"></a>
- Name files and folders after the React component they represent. File names of single-file components should share the component name.
- For scalable React apps, I like to use **domain-based** architecture, see the [architecture section](https://github.com/seanpmaxwell/Typescript-Best-Practices/blob/main/README.md#architecture) of the TypeScript-Best-Practices readme for more details about architecture choices.
- In the `server/`, the _Services_ layer usually refers to business logic while in the client, _services_ usually refers to integration-logic (handling API calls). So on the client-side, I've decided to place the business-logic in namespace-object scripts using the name `"Domain Name"+Ops.ts` (i.e. `UserOps.ts`).
- Application logic related to a specific domain (i.e. is user an `Admin` user-type) should go under `src/domains/`. Logic specifically for UI implementation details should go in the component folder where it's used. For example, hooks are React specific functions for handling UI updates, so we put them under `src/components/common`

### Example `src/` folder layout with domain-based architecture <a name="project-structure-example"></a>
```yml
- assets/
- _common/
  - constants/
    - EnvVars.ts
  - types/
  - utils/
- components/
  - _common/
    - ui/
      - lg/
      - md/
      - sm/
        - buttons.tsx
    - hooks/
      - yourCustomHook.ts
    - styles/
      - Colors.ts
      - BoxStyles.ts
  - pages/
    - Home/ (https://my-site.com/home)
      - Home.tsx
      - Home.test.tsx
    - Account/ (https://my-site.com/account)
      - UpdatePaymentForm/
        - ValidatePaymentInfo.tsx
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
      - Paths.ts <-- Keep all paths in one place
    - types/
      - Entity.ts <-- Parent model interface
  - users/
    - UserOps.ts <-- business logic / counter-part to our services-layer in the back-end
    - UserService.ts
    - User.ts // model-layer
  - posts/
    - PostOps.ts
    - PostService.ts
    - Post.ts
- infra/
  - http/
    - setup-axios.ts
    - index.ts <-- used by the services layer
```

<br/><b>***</b><br/>

## Functional components <a name="functional-components"></a>

### Declaring components <a name="functional-components-declaring"></a>
- Use PascalCase component names and prefer function declarations over classes or arrow functions so components are hoisted. function-declarations are also better for stack tracing errors.
- Define parent components first and declare children beneath them in the same file to keep logic top-down.
- Always type props. Define an interface (e.g. `IProps`) in the `Types` region for complex components. 
- You do not need to specify a return type for component functions because React components always return `JSX.Element`.

### Organizing component code <a name="functional-components-organization"></a>
- Keep static values outside component functions under the `Constants` region; otherwise, they are reinitialized on every render.
- Extract long, reusable helpers out of the component body and place them in the `Functions` region so they are not recreated on each render.
- Place layout logic for sibling components in their parent so that positioning and interactions are visible in one place.
- Prefer whitespace plus short comments to separate hook calls, DOM chunks, and initialization inside a function component.
- Keep the default export at the bottom. When the default export is a function component, start its comment with `Default component ...` so readers can identify it quickly without scrolling.
- Avoid giant `return` statements. Create child components for related DOM blocks rather than declaring many JSX variables above the return. Snippet 2 illustrates how composing smaller components keeps the parent lean.
- Follow the convention of function declarations at the top level and arrow functions only for inline callbacks inside JSX.

#### Snippet 2 – parent vs child composition

```tsx
// Bad: logic split across temporary JSX variables
function Parent() {
  const posts: string[] = [];
  const name = '';

  let Child = null;
  if (something) {
    Child = (
      <Box mb={2}>
        Name: {name ?? ''} Posts: {posts?.length ?? 0}
      </Box>
    );
  } else {
    Child = (
      <Box {...otherProps}>
        Foo: {name} Bar: {posts.length}
      </Box>
    );
  }

  return (
    <Box>
      <Child />
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
- Destructure props at the top of the component. It is easier to set defaults and identify unused values, and wrapper components can mirror the child props by reusing the same names.
- Snippet 3 shows how to extract props, separate regions, and organize hooks.

#### Snippet 3 – component layout template

```tsx
// LoginForm.tsx
import { useCallback } from 'react';
import axios from 'axios';
import Box, { BoxProps } from '@mui/material/Box';
import Button from '@mui/material/Button';

import Indicator from 'components/md/Indicator';

/******************************************************************************
                               Components
******************************************************************************/

/**
 * Default component. Login a user.
 */
function LoginForm(props: BoxProps) {
  const { sx, ...otherProps } = props,
    navigate = useNavigate();

  // Init state
  const [state, setState, resetState] = useSetState({
    username: '',
    password: '',
    isLoading: false,
  });

  // Call the "submit" API. Just an example, you should never call axios
  // directly in a component.
  const submit = useCallback(async () => {
    setState({ isLoading: true });
    const resp = await axios.post({
      username: state.username,
      password: state.password,
    });
    const isSuccess = _someLongComplexFn(resp);
    setState({ isLoading: false });
    if (isSuccess) {
      navigate('/account');
    }
  }, [setState, state.username, state.password, navigate]);

  // Return
  return (
    <Box
      sx={{
        position: 'relative',
        ...sx,
      }}
      {...otherProps}
    >
      {/* Indicator */}
      {state.isLoading && <Indicator />}

      {/* Input Fields */}
      <TextField
        type="text"
        value={state.username}
        onChange={v => setState({ username: v.currentTargetValue })}
      />
      <TextField
        type="password"
        value={state.password}
        onChange={v => setState({ password: v.currentTargetValue })}
      />

      {/* Action Buttons */}
      <Button color="error" onClick={() => navigate('/home')}>
        Cancel
      </Button>
      <Button color="primary" onClick={() => submit()}>
        Login
      </Button>
    </Box>
  );
}

/******************************************************************************
                               Functions
******************************************************************************/

function _someLongComplexFn(data: AxiosResponse<unknown>): boolean {
  // ...do stuff
  return true;
}

/******************************************************************************
                               Export default
******************************************************************************/

export default LoginForm;
```

<br/><b>***</b><br/>

## The Container/Presenter pattern and state management <a name="container-presenter-state-management"></a>

If a component is complex, separate data-fetching and state logic (containers) from rendering logic (presenters). This keeps components testable and reusable. For state itself, combine `useState`, `useContext` (and sometimes a third-party library such as Redux) using the lightest tool that satisfies the data flow you need. Inside of containers, for values that need to be shared amongst _multiple sub-trees_, place those in a state-management tools like useContext or redux, for the rest just use prop-drilling. 

### Container/Presenter pattern <a name="container-presenter-pattern"></a>
- A **Container** component owns state, side effects, and data fetching. It passes data and callbacks down as props.
- A **Presenter** component is purely visual—it receives props and renders UI with no direct knowledge of where data comes from.
- This separation makes presenters easy to test in isolation (just pass props) and lets you swap data sources in the container without touching the UI. If a presenter needs to contain a separate `state` or perform logic, it should only be relevant to what's needed for displaying that presenter's content. 

```tsx
// UsersContainer.tsx – container
function UsersContainer() {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    UserService.fetchAll().then(setUsers);
  }, []);

  return <UsersList users={users} />;
}

export default UsersContainer;
```

```tsx
// UsersList.tsx – presenter
interface IUsersListProps {
  users: User[];
}

function UsersList({ users }: IUsersListProps) {
  return (
    <ul>
      {users.map(u => (
        <li key={u.id}>{renderUserLink(u.name)}</li>
      ))}
    </ul>
  );
}

// This function does here, because it's only relevant to how we "display" users
function renderUserLink(user: User): string {
  const { firstName, lastName } = user;
  const lastName = !!lastName ? ' ' + lastName : '';
  return capitolize(firstName + lastName)
}

export default UsersList;
```

### `useState` vs `useSetState` <a name="state-management-usestate"></a>
- `useState` is fine for components with one or two state variables. As state grows, switch to a custom hook (such as [`useSetState`](https://github.com/streamich/react-use/blob/master/src/useSetState.ts)) that manages a single state object.
- Grouping state into one object keeps every state value prefixed with `state` and managed by a single updater, improving readability.
- Hooks such as `resetState` are especially helpful in modal flows or anywhere you need a quick way to restore defaults.

<br/><b>***</b><br/>

## Misc styling rules <a name="misc-styling-rules"></a>

These items may not be enforced by the linter but they help keep React + TS projects readable.

### Styling the UI <a name="misc-styling-ui"></a>
- Keep color tokens in `src/common/styles/Colors.ts` instead of hardcoding hex strings in JSX. Group base colors, then expose them through semantic buckets so updates stay centralized.

#### Snippet 4 – color tokens

```ts
// src/common/styles/Colors.ts
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
};

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
};
```

#### Snippet 5 – using shared colors

```tsx
import Colors from '@src/common/styles/Colors';

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

      <div
        id="how-are-you-ele"
        style={{ padding: 8 }}
        onClick={() => alert('How are you?')}
      >
        How are you?
      </div>
    </div>
  );
}
```

### Callback parameter names <a name="misc-styling-callbacks"></a>
- Give parameters meaningful names in general, but for simple inline JSX callbacks you can use a short placeholder like `v` for `value` or `err` for `error`  when it only takes one line of logic. This keeps JSX uncluttered while still distinguishing callback data from other variables.

#### Snippet 6

```tsx
function Parent() {
  const [state, setState] = useSetState({
    name: '',
    nameError: false,
    email: '',
    emailError: false,
  });

  return (
    <div>
      <CustomInput
        value={state.name}
        isRequired={true}
        onChange={(v, err) => setState({ name: v, nameError: err })}
      />
      <CustomInput
        value={state.email}
        isRequired={true}
        onChange={(v, err) => setState({ email: v, emailError: err })}
      />
      <button
        disabled={state.nameError || state.emailError}
        onClick={() => {/* some API call */}}
      >
        Submit
      </button>
    </div>
  );
}

function CustomInput(props: {
  value: string;
  isRequired: boolean;
  onChange: (value: string, error?: boolean) => void;
}) {
  const { value, isRequired, onChange } = props;
  return (
    <div>
      <input
        type="text"
        value={value}
        onChange={v => onChange(v.trim(), isRequired && !v)}
      />
      <div>{!value && isRequired ? 'Value is required' : ''}</div>
    </div>
  );
}
```

### Other rules <a name="misc-styling-other"></a>
- Use single quotes for standard JS/TS code and double quotes for JSX attributes. Configure the linter to enforce this consistently.

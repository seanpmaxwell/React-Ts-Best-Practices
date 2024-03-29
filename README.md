# React-Ts-Best-Practices

Documentation for best practices to use with React with Typescript. Note that this documentation builds off of <a href="https://github.com/seanpmaxwell/Typescript-Best-Practices">Typescript best practices</a>


## Structuring your app

- Use functions for declaring components. Procedural/functional programming is the dominant trend in JavaScript and also makes it easier to decouple your DOM elements.
- Use function-declarations with a top-down approach for components that have child-components.
- The backbone of your React app should be under a src/pages/ folder. Create pages using declaration scripts (see link above).


## Structuring Pages/Components

- Don't separate things by putting logic/dom/styling into different files. This leads to a lot of spaghetti code and having to search for things. Instead make rich use of JSX elements and isolate the logic/dom/styling into individual JSX elements. For example, say you have a form which has multiple inputs and different buttons which do different API calls. Isolate the inputs and button that are part of one API call into one child element, and put the other inputs/button into another child elements. Then use the parent elements for creating the arrangement and spacing between the two child elements.
```typescript
// SignupPage.tsx

import { useState, CSSProperties } from 'react';
import useQuery from 'some-http-hook-lib';

import Spinner from '../shared-components/Spinner';
import { useSetState } from '../custom-hooks/useSetState';


// **** Types **** //

interface IChildProps {
  style?: CSSProperties;
}


// **** Functional Components **** //

/**
 * The signup page at 'host:port/signup'
 */
function SignupPage() {
  const [ showSpinner, setShowSpinner] = useState(false);
  return (
    <div style={{position: 'relative'}}>
       <Spinner show={showSpinner}/>
       <SignupLocalForm style={{ marginBottom: 16 }} />
       <GoogleBtnWrapper style={{ marginBottom: 32 }}/>
    </div>
  );
}

/**
 * Loading indicator element
 */
function SignupLocalForm(props: IChildProps) {
   const [ state, setState ] = useSetState({ name: '', email: '' }),
    request = useQuery(Paths.SignupLocally);
   // Return
   return (
      <div {...props}>
        <input onChange={e = setState({ name: e.currentTarget.value }})/>
        <input onChange={e = setState({ name: e.currentTarget.value }})/>
        <button onClick={() => req.refetch()}
      </div>
   );
}

/**
 * Signup With Google
 */
function GoogleBtnWrapper(props: IChildProps) {
  const req = useQuery(Paths.SignupWithGoogle);
  return (
    <div {...props}>
       <button onClick={() => req.refetch()}/>
    </div>
  );
}


// **** Export default **** //

export default SignupPage;
```

- Don't need to wrap DOM elements in parenthesis for `&&`. The proceeding boolean statement should be wrapped though.
```
// Don't do
{isLoading && (
  <div>
    <Indicator allPage />
  </div>
)}

// Do do
{isLoading && 
  <div>
    <Indicator allPage />
  </div>
}

// Parenthesis are fine for ternary statements though
{(isError && !!errMsg) ? (
  <div>
    {errMsg}
  </div>
) : (
  <div>
    Foo Bar
  </div>
)}
```

## Functions
- Add return types for functions you create that are used elsewhere in your code. JSX.Elements are exceptions to this though because it's obvious what's being returned.
- Referring to Typescript-best-practices, you should generally not put spaces in functions. But for JSX elements (which can get really large), we can put spaces in between some of the Html elements and blocks of useEffect code.

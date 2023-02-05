# React-Ts-Best-Practices

Documentation for best practices to use with React with Typescript. Note that this documentation builds off of <a href="https://github.com/seanpmaxwell/Typescript-Best-Practices">Typescript best practices</a>


## Structuring your app

- Use functions for declaring components. Procedural/functional programming is the dominant trend in JavaScript and also makes it easier to decouple your DOM elements.
- The backbone of your React app should be under a src/pages/ folder. Create pages using declaration scripts (see link above).

## Structuring Pages/Components
- Don't separate things by putting logic/dom/styling into different files. This leads to a lot of spaghetti code and having to search for things. Instead make rich use of JSX elements and isolate the logic/dom/styling into different JSX elements.
- For example say you have a form which has multiple inputs and different buttons which do different API calls. Isolate the inputs and button that are part of one API call into one child element, and put the other inputs/button into another child elements. Then use the parent elements for creating the arrangement and spacing between the two child elements.
```
// pick up here, create an example
```

# Under the Hood of React: What's Happening Behind the Scenes?

Have you ever been curious about how React works behind the scenes?

In this article, we will take you on an exciting journey to explore the internal structure of React and build a simple, initial version of it from scratch. This is an opportunity to gain a deep and practical understanding of concepts that every React developer should know: from the **virtual DOM** to **state management** and **lifecycle methods**.

We will proceed with this article step by step. First, we start with the basic structure of **Mini React**, which includes fundamental functions like `createElement`, `render`, `useState`, and `useEffect`. Then, we will examine and implement each of these functions in detail. Finally, by building a simple and practical application, you will see the real functionality of these concepts and understand how React works behind the scenes.

> In this section, we examine a simple implementation of React. Note that this is just an example to understand the basic principles, and for simplicity, we have omitted complex structures like LinkedList that React uses for internal component management. In this implementation, we only focus on the top-level rendering, and in practice, React is more complex than what is shown here.

# Internal Structure and Simple Implementation of React

In this article, we will get to know the internal structure of React step by step and implement a simple version of it. Our goal is to better understand concepts such as the virtual DOM, state management, and lifecycle methods.

We start by creating the basic skeleton for Mini React and then implement key functions like `createElement`, `render`, `useState`, and `useEffect`. After that, we build a simple application to observe the functionality of these concepts.

## Basic React Code

```javascript
const React = {
  Component: undefined,
  version: "1.0.0",
  index: 0,
  hooks: [],
  effects: [],
  container: undefined,
  createElement() {},
  render() {},
  useState() {},
  useEffect() {},
};
```

> In this article, for the sake of code integrity, features are defined within a simple object; whereas in React, these features might be implemented as modules or other concepts.

Here, you see a basic structure of React. This code includes various variables that are generally used to implement different functions. Each of these functions will be implemented in real projects.

- `Component`: An object that represents a React component and is used in the rendering process.
- `version`: The version of React that is used at runtime.
- `index`: An index to track the order of hook execution (such as useState).
- `hooks`: An array to store states and values related to hooks.
- `effects`: A list of side effects that are executed after rendering (such as useEffect).
- `container`: The DOM element where the React application is mounted.

The functions `createElement`, `render`, `useState`, and `useEffect` are simply defined and will be implemented in the next steps.

## Defining the ReactElement Structure

To model JSX elements in our simplified version of React, we first define a TypeScript type that specifies the structure of each element. This type indicates that an element can be an HTML tag or a functional component, and it can also include children that are themselves React elements, text, or numbers.

```typescript
type ReactChild = ReactElement | string | number;

type ReactElement = {
  type: string | (() => ReactElement);
  props: {
    [key: string]: unknown;
    children?: ReactChild[];
  };
};
```

In this definition:

- `type`: Can be an HTML tag like "div" or a function that returns a ReactElement.
- `props`: Properties that are given to the element, including children.
- `children`: An array of other elements, or text and numbers used as the element's internal content.

## Implementing the createElement Function

The `createElement` function is responsible for creating an object of type `ReactElement`, which is exactly what React does when compiling JSX. This function takes the element type, properties (props), and children, and returns them in a standard object format.

```typescript
function createElement(
  type: string | (() => ReactElement),
  props: Record<string, unknown> | null,
  ...children: ReactChild[]
): ReactElement {
  return {
    type,
    props: {
      ...props,
      children,
    },
  };
}
```

In this implementation:

- `type`: Can be an HTML tag like "div" or a functional component.
- `props`: An object of properties assigned to the element.
- `children`: A list of the element's children, which may include text, numbers, or other elements.

> Note that if you want to use JSX, you would need a separate article. In this article, we have tried to demonstrate without JSX and provide concise and useful explanations.

## Detailed Explanation of the Render Code

Here, the code for the `render` function, which is one of the main components for displaying components, is explained. This function is responsible for taking a component as input, creating a DOM structure for it, and then adding it to the web page.

### What is the render Function?

The `render` function is responsible for rendering components. This function first receives a functional component, then calls it and creates a DOM element. Finally, this element is added to the web page.

### How Each Part of the Code Works

1. **Resetting the Counter**: At the beginning of each render, the counter (`this.index`) is reset to zero. This helps ensure that each hook (like `useState` or `useEffect`) has a specific position and that their execution order is maintained across consecutive renders.
2. **Calling the Component**: At this stage, the functional component provided as input to the function is called. The output of this call is an object of type `ReactElement` that includes the component's type (`type`) and its properties (`props`).
3. **Creating the DOM Element**: Using the information in the `ReactElement` object, a new DOM element of the type specified in `component.type` is created. This step is the initial foundation for creating the final user interface structure in the browser.
4. **Applying Props**: At this stage, all component `props` (such as `style` or `children`) are added to the DOM element. If the `style` property exists, these styles are directly applied to the DOM element, and other `props` are transferred to the DOM element in the same manner.
5. **Adding Children**: If the component has children (which can include text, numbers, or other components), they are added as internal content to the DOM element. If the children include components, they are first converted to DOM elements and placed inside the parent element.
6. **Replacing the Element**: After building and configuring the new element, by calling `container.replaceChildren(dom)`, the previous element in the `container` is replaced with the new element. This method is significantly more efficient than completely removing the content and adding it again.
7. **Executing Effects**: After the rendering process is complete, all registered effects (side effects) are executed. These can include updating state, sending network requests, or any other side operations.

```javascript
function render(FC: () => ReactElement, container: HTMLElement) {
  this.index = 0;
  this.Component = FC;
  const component = this.Component();
  const dom = document.createElement(component.type);

  for (const prop in component.props) {
    if (prop === "style") {
      Object.assign(dom.style, component.props[prop]);
    } else {
      const children = Array.isArray(component.props.children)
        ? component.props.children
        : [component.props.children];
      children.forEach((child) => {
        dom.appendChild(document.createTextNode(child));
      });
    }
  }

  container.replaceChildren(dom);

  this.effects.forEach((effect) => {
    effect();
  });

  this.effects = [];
}
```

This code simply and effectively adds a functional component to the DOM and, by using a counter to track the position of hooks, ensures the order of their execution in consecutive renders. Thus, while ensuring the stability of changes, it provides better performance compared to common methods of removing and adding content in the DOM.

## The useState Function

The `useState` function is used to manage state in components. This function stores the state and provides for its update.

```javascript
function useState(initialValue: unknown) {
  const index = this.index;

  if (this.hooks[index] === undefined) {
    this.hooks[index] = initialValue;
  }

  const setState = (newState: unknown) => {
    if (this.hooks[index] !== newState) {
      this.hooks[index] = newState;
      this.render(this.Component, document.body);
    }
  };

  const state = this.hooks[index];

  this.index++;

  return [state, setState];
}
```

This code explains the functionality of the `useState` function:

- First, an `index` is taken from `this.index` to determine the current position in the `this.hooks` array.
- If the state is not defined at the current position, the initial value (initialValue) passed to the function is assigned to it.
- Then, a `setState` function is defined to update the state. If the new value is different from the current state, the state is updated, and the `render` function is executed again for re-rendering.
- Finally, the current state is retrieved from `this.hooks[index]`, and `this.index` is incremented for use in subsequent calls.

## A Simple Component with useState and useEffect

In this example, we define a functional component that manages the _count_ value with `useState` and reports its changes in the console with `useEffect`.

```javascript
function App() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    console.log("App mounted");
    setCount(1);
  }, []);

  React.useEffect(() => {
    console.log(`Count changed to ${count}`);
  }, [count]);

  return React.createElement(
    "div",
    {
      style: {
        color: "red",
        fontSize: "20px",
        fontWeight: "bold",
      },
    },
    `Hello Baarg! count: ${count}`
  );
}

React.render(App, document.body);
```

### Explanations

- **useState(0)**: Creates a state with an initial value of `0`.
- **useEffect(() => {…}, [])**: Runs only on mount and changes `count` from zero to one.
- **useEffect(() => {…}, [count])**: Logs the new value of `count` in the console every time it changes.
- `React.createElement` creates a `div` element with red style and bold text that displays the value of `count`.
- `React.render` adds the component's output to `document.body`.

### Output

In the browser console:

```plaintext
App mounted
Count changed to 0
Count changed to 1
```

In the DOM:

```plaintext
Hello Baarg! count: 1
```

## Final Mini React Code with App Component

The complete and final code of the Mini React system along with the implementation of the `App` function is as follows:

```javascript
type ReactChild = ReactElement | string | number;

type ReactElement = {
  type: string | (() => ReactElement),
  props: {
    [key: string]: unknown,
    children?: ReactChild[],
  },
};

const React = {
  Component: undefined,
  version: "1.0.0",
  index: 0,
  hooks: [],
  effects: [],
  container: undefined,

  createElement(
    type: string | (() => ReactElement),
    props: Record | null,
    ...children: ReactElement[]
  ): ReactElement {
    return {
      type,
      props: {
        ...props,
        children,
      },
    };
  },

  render(FC: () => ReactElement, container: HTMLElement) {
    this.index = 0;
    this.Component = FC;
    const component = this.Component();
    const dom = document.createElement(component.type);

    for (const prop in component.props) {
      if (prop === "style") {
        Object.assign(dom.style, component.props[prop]);
      } else {
        const children = Array.isArray(component.props.children)
          ? component.props.children
          : [component.props.children];
        children.forEach((child) => {
          dom.appendChild(document.createTextNode(child));
        });
      }
    }

    container.replaceChildren(dom);

    this.effects.forEach((effect) => {
      effect();
    });

    this.effects = [];
  },

  useState(initialValue: unknown) {
    const index = this.index;

    if (this.hooks[index] === undefined) {
      this.hooks[index] = initialValue;
    }

    const setState = (newState: unknown) => {
      if (this.hooks[index] !== newState) {
        this.hooks[index] = newState;
        this.render(this.Component, document.body);
      }
    };

    const state = this.hooks[index];

    this.index++;

    return [state, setState];
  },
  useEffect(callback: () => void, dependencies: unknown[]) {
    const index = this.index;
    const prevDeps = this.hooks[index];
    const hasChanged =
      !prevDeps || dependencies.some((dep, i) => !Object.is(dep, prevDeps[i]));

    if (hasChanged) {
      this.hooks[index] = dependencies;
      this.effects.push(callback);
    }

    this.index++;
  },
};
function App() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    console.log("App mounted");
    setCount(1);
  }, []);

  React.useEffect(() => {
    console.log(`Count changed to ${count}`);
  }, [count]);

  return React.createElement(
    "div",
    {
      style: {
        color: "red",
        fontSize: "20px",
        fontWeight: "bold",
      },
    },
    `Hello Baarg! count: ${count}`
  );
}

React.render(App, document.body);
```

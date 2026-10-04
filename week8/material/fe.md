# Understanding `useContext`

In React development, managing state and passing data between components can become cumbersome, especially when deeply nested components need access to the same data. A common issue in such cases is "prop drilling." Let’s explore what prop drilling is, how it can be solved using `useContext`, and provide both simple and more complex examples of how context can enhance a React project.

---

### What is Prop Drilling?

Prop drilling happens when you pass props down through several levels of components, even if only a few deeply nested components need the data. This can lead to repetitive, error-prone code and make it harder to maintain or scale an application.

For example, if you need to pass user information from a top-level component (like `App.js`) down to a deeply nested component (like `Profile.js`), you might find yourself passing the user data through several intermediate components that don’t even use it, just so that `Profile.js` can access it. This is what we call "prop drilling."

Here’s an example of prop drilling:

```javascript
function App() {
  const user = { name: 'John' };
  return <Parent user={user} />;
}

function Parent({ user }) {
  return <Child user={user} />;
}

function Child({ user }) {
  return <Profile user={user} />;
}

function Profile({ user }) {
  return <div>Hello, {user.name}</div>;
}
```

In this example, the `user` prop gets passed from `App` to `Parent`, then to `Child`, and finally to `Profile`, even though only `Profile` needs it. As your app grows, prop drilling can lead to cluttered and difficult-to-manage code.


### When Prop Drilling Is OK

Before jumping to `useContext`, it’s important to note that prop drilling isn’t always a problem. If you only need to pass data through a shallow component tree (e.g., one or two levels deep), regular prop passing is often simpler and clearer. Overusing context can make it harder to trace where data is coming from, leading to increased complexity.

**Use props when:**
- The component hierarchy is shallow.
- The data being passed is specific to a small part of the app.
- It’s more intuitive to pass data explicitly.

### Solving Prop Drilling with `useContext`

When you need to pass data deeply, React’s `useContext` hook allows you to share state more efficiently by creating a "context." Context lets you provide data to components at any depth in your component tree without explicitly passing props down through every level.

---

### Common Use Cases for Context

1. **Theming**: Manage light/dark mode in your app, as shown in the example below.
2. **Authentication**: Share user authentication data across multiple components.
3. **Global Settings**: Store and share configuration or settings throughout the app.
4. **Language/Localization**: Pass the app's current language or translations to components.
5. **User Preferences**: Persist user settings like layout preferences or notification preferences.

---

### Three Steps to Use `useContext`

Let’s break down the process of using `useContext` to replace prop drilling with context, using the code example below.

#### 1. **Create a Context**

The first step is to create a context that will hold the shared data (in this case, the app’s theme). This context serves as the shared source of truth that any component can subscribe to.

```javascript
// Context/ThemeContext.js
import { createContext } from 'react';

const ThemeContext = createContext('light'); // 'light' is the default value
export default ThemeContext;
```

#### 2. **Provide the Context in a Parent Component**

Next, we need a provider to wrap the components that need access to the shared data. This is done in `ThemeProvider.js`. The `ThemeProvider` component holds the state (the theme) and makes it available to any component within the tree.

```javascript
// Context/ThemeProvider.js
import { useState } from 'react';
import ThemeContext from './ThemeContext';

export default function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light'); // default theme is 'light'

  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === 'dark' ? 'light' : 'dark'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

#### 3. **Consume the Context in Child Components**

Finally, any deeply nested component can access the `theme` and `toggleTheme` values using `useContext`. No need to pass props through intermediate components!

```javascript
// Components/Panel.js
import { useContext } from 'react';
import ThemeContext from '../Context/ThemeContext';

export default function Panel({ title, children }) {
  const { theme } = useContext(ThemeContext); // consuming the theme context
  const className = 'panel-' + theme;

  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}
```

In this example, `useContext(ThemeContext)` gives access to the theme without needing to pass it explicitly through intermediate components.

---

### Performance Considerations

While `useContext` is powerful, there are performance considerations to keep in mind. When the context value changes, **all** components that consume the context will re-render, even if only one component’s state changed. This can lead to performance issues if many components subscribe to the same context.

Here’s how you can optimize performance:

**Splitting Context**: If a context holds multiple values, and only one frequently changes, consider splitting the context into multiple smaller contexts so that components only re-render when the specific value they use changes.

**Example**: In a more complex app with both theme and user authentication, you might split the context into separate `ThemeContext` and `AuthContext` to prevent re-renders when the theme changes but authentication data doesn't.

---

### A More Complex Example: Authentication

Let’s extend the context example to handle user authentication. Here, we’ll create an `AuthContext` to provide the current user’s data across the app without passing it through intermediate components.

```javascript
// Context/AuthContext.js
import { createContext } from 'react';

const AuthContext = createContext(null); // No user by default
export default AuthContext;
```

```javascript
// Context/AuthProvider.js
import { useState } from 'react';
import AuthContext from './AuthContext';

export default function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => {
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}
```

Now, any component in the app can consume the `user` value or call the `login` or `logout` functions.

```javascript
// Components/UserProfile.js
import { useContext } from 'react';
import AuthContext from '../Context/AuthContext';

export default function UserProfile() {
  const { user } = useContext(AuthContext);

  if (!user) {
    return <div>Please log in</div>;
  }

  return <div>Hello, {user.name}!</div>;
}
```

This avoids the need to pass user authentication data through intermediate components, improving maintainability and reducing prop drilling.

---

### Error Handling with Context

What happens if a component tries to access context outside a provider? React will use the default value specified in `createContext`. However, this can lead to unexpected behavior if the component expects the context to be non-null.

To handle this gracefully, you can throw an error if a component is trying to access context outside the provider:

```javascript
// useAuth.js (Custom Hook for AuthContext)
import { useContext } from 'react';
import AuthContext from './AuthContext';

export default function useAuth() {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }

  return context;
}
```

This custom hook checks whether the component is within an `AuthProvider` and throws an error if not. It’s a good practice to avoid subtle bugs in larger applications.


---


### Context and the Publish-Subscribe Pattern

React’s Context API functions similarly to the **publish-subscribe pattern**. In this pattern:
- **Publisher**: The `Provider` component serves as the "publisher," holding state and making it available to subscribed components.
- **Subscribers**: Components that call `useContext` are "subscribers," meaning they access the published data and automatically re-render when the provider's state changes.

For example, in a theme switcher:
- **ThemeProvider** is the publisher, managing the current theme and exposing a function to toggle it.
- Any component that uses `useContext(ThemeContext)` subscribes to this data and updates when the theme changes, mimicking the pub-sub model.


---

### Context and CSS Property Inheritance

The way React Context provides values down the component tree mirrors **CSS property inheritance**. In CSS, properties such as `color` set on a parent element are automatically applied to its children unless explicitly overridden.

In React:
- A **Context Provider** passes values to its descendants. These components automatically inherit the context unless a nested provider overrides it, much like how a CSS property like `color` cascades down through child elements.

---

### Recap

- **Context vs. Prop Drilling**: `useContext` helps avoid prop drilling in deep component trees by providing global data without explicitly passing props.
- **Performance**: Beware of unnecessary re-renders when using context. Split contexts and use memoization to optimize performance.
- **Error Handling**: Ensure components are wrapped by their respective providers, and handle errors when accessing context improperly.
- **Complex Use Cases**: For features like authentication, splitting context and using custom hooks can help maintain clean, scalable code.

---
### Key Steps:

In the [React documentation](https://react.dev/learn/passing-data-deeply-with-context), the three steps listed are:

1. **Create a context** using `React.createContext()`.
2. **Use the context** with `useContext()` or `Context.Consumer` in the component that needs it.
3. **Provide the context** by using `Context.Provider`.

Note that the last: step "Provide the context" implies wrapping a parent component with the `Context.Provider`. The documentation groups this into the broader "Provide the context" step rather than splitting it into two separate ones. This simplifies the explanation and focuses more on the logical flow of using context rather than detailing every technical action as a separate step.

However, rember that "wrapping the parent with `Context.Provider`" is a critical part of using context, and it could arguably be considered its own distinct step. The documentation keeps it concise, but breaking it out might help some learners.

So, while React documentation condenses it into three steps for simplicity, you could consider it an additional "4th" step if that helps clarify the process for you.

---

### Links

- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
- [React `useContext` Reference](https://react.dev/reference/react/useContext)
- [Publish–subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)
- [Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context)
- [w3schools:usecontext](https://www.w3schools.com/react/react_usecontext.asp)
- [Integration Patterns – Publish/Subscribe (Part2)](https://techtrantor.com/integration-patterns-publishsubscribe_25/)


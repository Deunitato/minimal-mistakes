---
published: false
---
Todays topic: React Hooks. Part of the learning process of react
Ref:

# Introduction

- Context API: Clean and easy way to share state between components
- Hoocks: Tap into the inner workings of react in functional component

The use of react context and Hooks is Redux-like. 

Expectations:
- Grasp of javascript
- Moderate understanding of react
- Create class and using props

# Context API

## Context Provider

Gives us a way to share state up and down component tree without using props. This is good because as the number of components increased, it wont get so messy. We do not need to pass it down as props. 

This is also an alteranative to redux.
![reacthooks_1.PNG]({{site.baseurl}}/img/reacthooks_1.PNG)

Normally we pass each component as a prop. But theres a problem where some compoenent does not need the prop and only act as a carrier. This approach will be quite messy especially when the application become bigger. Context API tries to solve this.

What we do is to create a context. Provide it to our component tree so that the components can access it.

![reacthooks_2.PNG]({{site.baseurl}}/img/reacthooks_2.PNG)

Check this docs: [Context docs](https://reactjs.org/docs/context.html)

We would define a theme context component

```Typescript
import React, { Component, createContext } from 'react'

export const ThemeContext = createContext();

class ThemeContextProvider extends Component {
    state = {
        isLightTheme: true,
        light: { syntax: '#555', ui: '#ddd', bg: '#eee'},
        dark: { syntax: '#ddd', ui: '#333', bg: '#555'}
    }
    render() {
        return (
            <ThemeContext.Provider value={{...this.state}}>
                {this.props.children /**This refer to the children that this tag wraps around */}
            </ThemeContext.Provider>
        );
    }
}

export default ThemeContextProvider;
```

Using the `<ThemeContext>` as a parent tag would allow the children to access these variables under the state values.

Usage in the app.js

```Typescript
function App() {
  return (
    <div className="App">
      <ThemeContextProvider>
        <Navbar />
        <BookList/>
      </ThemeContextProvider>
    </div>
  );
}
```

Usage in the component:

```Typescript
class Navbar extends Component {
    static contextType = ThemeContext; //This would look up the component tree and find the nearest provider
    render () {
        console.log(this.context);
        const {isLightTheme, light, dark} = this.context;
        const theme = isLightTheme ? light : dark;
        return (
            <nav style = {{background: theme.ui, color:theme.syntax}}>
                <h1>Context App</h1>
                <ul>
                    <li>Home</li>
                    <li>About</li>
                    <li>Contact</li>
                </ul>
            </nav>
        )
    }
}


```

What this does is that it would look up the nearest contextprovider, which is in the app.js and locate the values from there. It would be able to have access to the values. `console.log(this.context)` would print out all the values taken from state in ThemeContext

## Context consumer

Usage in Component:

```Typescript
class Navbar extends Component {
    render () {
        return (
            <ThemeContext.Consumer>{(context)=> {
                const {isLightTheme, light, dark} = context;
                const theme = isLightTheme ? light : dark;
                return (
                    <nav style = {{background: theme.ui, color:theme}}>
                    <h1>Context App</h1>
                    <ul>
                        <li>Home</li>
                        <li>About</li>
                        <li>Contact</li>
                    </ul>
                </nav>
                );
            }}
            </ThemeContext.Consumer>
        )
    }
}
```

It works the same as a provider, only that we are returning a function that takes in a value `context` and a tag from `ThemeContext.consumer`.


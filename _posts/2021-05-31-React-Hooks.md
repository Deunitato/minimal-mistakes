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
Gives us a way to share state up and down component tree without using props. This is good because as the number of components increased, it wont get so messy. We do not need to pass it down as props. 

This is also an alteranative to redux.
![reacthooks_1.PNG]({{site.baseurl}}/img/reacthooks_1.PNG)

Normally we pass each component as a prop. But theres a problem where some compoenent does not need the prop and only act as a carrier. This approach will be quite messy especially when the application become bigger. Context API tries to solve this.

What we do is to create a context. Provide it to our component tree so that the components can access it.

![reacthooks_2.PNG]({{site.baseurl}}/img/reacthooks_2.PNG)


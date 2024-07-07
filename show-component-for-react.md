---
title: Show component for React
date: 2024-03-21T01:20:00.000+00:00
---

As a React developer, you know that conditional rendering is a task that comes up all the time. While you can always rely on basic inline conditional logic (using && and ternary operators), things can get messy when you have multiple elements that need to render or not render.  
  <br>
To make conditional UI rendering simpler and more declarative, I've adopted a custom Show component for my projects. Here's how it works:  
The Show component accepts React elements wrapped in a When and Else component for conditional logic.  
  <br>
Now, say goodbye to the verbosity of condition ? "When true" : "Else case". Embrace the elegance and simplicity of the Show component for a more streamlined and intuitive approach to conditional rendering in your React projects.  
No more parsing complex ternary statements or logical operators to figure out what will actually render!
<br><br>
```ts
import React, { Children, ReactElement } from 'react';

interface WhenProps {
  isTrue: boolean;
  children: React.ReactNode;
}

interface ElseProps {
  render?: React.ReactNode;
  children: React.ReactNode;
}

export const Show = (props: React.PropsWithChildren) => {
  let when: React.ReactNode | null = null;
  let otherwise: React.ReactNode | null = null;

  Children.forEach(props.children, child => {
    if ((child as ReactElement<WhenProps>).props.isTrue === undefined) {
      otherwise = child;
    } else if (!when && (child as ReactElement<WhenProps>).props.isTrue) {
      when = child;
    }
  });

  return when || otherwise;
};

Show.When = ({ isTrue, children }: WhenProps) => isTrue && children;

Show.Else = ({ render, children }: ElseProps) => render || children;
```
<br><br>
```ts
export default function App() {
  const condition = true; // Add your condition 
  return (
    <div className="p-6">
      <Show>
        <Show.When isTrue={condition}>Renders when condition is true</Show.When>
        <Show.Else>Render when condition is not true</Show.Else>
      </Show>
    </div>
  );
}
```

<p class="text-center">fin.</p>
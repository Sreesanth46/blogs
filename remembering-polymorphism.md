---
title: Remembering Polymorphism
date: 2026-01-18 22:34:18 +0530
tags:
  - javascript
  - polymorphism
---
Polymorphism isn’t new.  
It isn’t fancy.  
And it definitely isn’t something you’ll find trending on Twitter or LinkedIn.

Yet, while working on a recent **Vue 2 → Vue 3 migration**, it hit me how badly we had ignored it—and how much that neglect shaped the mess in our legacy codebase.

This post isn’t about discovering polymorphism.  
It’s about **remembering why we needed it in the first place**.
## The Legacy Code Reality

If you’ve worked on a long-lived frontend project, you know this story:
- Same UI component, different APIs
- Same logic, slightly different data shapes
- Same feature, but handled with `if`, `else if`, and `switch` everywhere

Our legacy Vue 2 codebase was littered with components that handled multiple types of data with massive conditional chains. Picture this:

```js
export default {
  props: {
    item: Object
  },
  computed: {
    displayValue() {
      if (this.item.type === 'user') {
        return `${this.item.firstName} ${this.item.lastName}`;
      } else if (this.item.type === 'company') {
        return this.item.companyName;
      } else if (this.item.type === 'bot') {
        return `Bot: ${this.item.botId}`;
      }
    },
    icon() {
      if (this.item.type === 'user') {
        return 'user-icon';
      } else if (this.item.type === 'company') {
        return 'building-icon';
      } else if (this.item.type === 'bot') {
        return 'robot-icon';
      }
    },
    actionRoute() {
      if (this.item.type === 'user') {
        return `/users/${this.item.id}`;
      } else if (this.item.type === 'company') {
        return `/companies/${this.item.id}`;
      } else if (this.item.type === 'bot') {
        return `/bots/${this.item.id}`;
      }
    }
  },
  methods: {
    handleAction() {
      if (this.item.type === 'user') {
        // user-specific logic
      } else if (this.item.type === 'company') {
        // company-specific logic
      } else if (this.item.type === 'bot') {
        // bot-specific logic
      }
    }
  }
}
```

Does this look familiar? Every time we added a new item type, we'd hunt through the component for every if-else chain and add another branch. Every time we needed to change user behavior, we'd hold our breath hoping we didn't break company or bot behavior. The component grew. The tests grew. The fear grew.

## What Polymorphism Actually Means Here

Polymorphism—the ability for objects of different types to be handled through the same interface—isn't just an OOP concept. It's a pattern for organizing code so that new types don't require you to modify existing logic. It's about writing code that's open to extension but closed to modification.

In Vue, this looks like letting each type handle itself.

## The Better Approach

Instead of one component with conditional logic, we can create a strategy for each item type:
```js
// strategies/itemStrategies.js
const strategies = {
  user: {
    getDisplayValue(item) {
      return `${item.firstName} ${item.lastName}`;
    },
    getIcon() {
      return 'user-icon';
    },
    getActionRoute(item) {
      return `/users/${item.id}`;
    },
    handleAction(item, context) {
      // user-specific action logic
    }
  },
  company: {
    getDisplayValue(item) {
      return item.companyName;
    },
    getIcon() {
      return 'building-icon';
    },
    getActionRoute(item) {
      return `/companies/${item.id}`;
    },
    handleAction(item, context) {
      // company-specific action logic
    }
  },
  bot: {
    getDisplayValue(item) {
      return `Bot: ${item.botId}`;
    },
    getIcon() {
      return 'robot-icon';
    },
    getActionRoute(item) {
      return `/bots/${item.id}`;
    },
    handleAction(item, context) {
      // bot-specific action logic
    }
  }
};

export const getStrategy = (type) => {
  return strategies[type] || strategies.user; // fallback
};
```

Now the component becomes straightforward:

```js
import { getStrategy } from '@/strategies/itemStrategies.js';

export default {
  props: {
    item: Object
  },
  computed: {
    strategy() {
      return getStrategy(this.item.type);
    },
    displayValue() {
      return this.strategy.getDisplayValue(this.item);
    },
    icon() {
      return this.strategy.getIcon();
    },
    actionRoute() {
      return this.strategy.getActionRoute(this.item);
    }
  },
  methods: {
    handleAction() {
      this.strategy.handleAction(this.item, this);
    }
  }
}
```

Need to add a new type called `partner`? You don't touch the component. You don't hunt for hidden if-else chains. You add a new strategy:

```js
strategies.partner = {
  getDisplayValue(item) {
    return item.partnerName;
  },
  getIcon() {
    return 'handshake-icon';
  },
  getActionRoute(item) {
    return `/partners/${item.id}`;
  },
  handleAction(item, context) {
    // partner-specific logic
  }
};
```

## An Alternative: Class-Based Polymorphism

If your codebase favors a more formal structure, you can achieve the same pattern using classes and inheritance:

```js
// strategies/ItemStrategy.js
class ItemStrategy {
  getDisplayValue(item) {
    throw new Error('getDisplayValue must be implemented');
  }
  getIcon() {
    throw new Error('getIcon must be implemented');
  }
  getActionRoute(item) {
    throw new Error('getActionRoute must be implemented');
  }
  handleAction(item, context) {
    throw new Error('handleAction must be implemented');
  }
}

class UserStrategy extends ItemStrategy {
  getDisplayValue(item) {
    return `${item.firstName} ${item.lastName}`;
  }
  getIcon() {
    return 'user-icon';
  }
  getActionRoute(item) {
    return `/users/${item.id}`;
  }
  handleAction(item, context) {
    // user-specific logic
  }
}

class CompanyStrategy extends ItemStrategy {
  getDisplayValue(item) {
    return item.companyName;
  }
  getIcon() {
    return 'building-icon';
  }
  getActionRoute(item) {
    return `/companies/${item.id}`;
  }
  handleAction(item, context) {
    // company-specific logic
  }
}

class BotStrategy extends ItemStrategy {
  getDisplayValue(item) {
    return `Bot: ${item.botId}`;
  }
  getIcon() {
    return 'robot-icon';
  }
  getActionRoute(item) {
    return `/bots/${item.id}`;
  }
  handleAction(item, context) {
    // bot-specific logic
  }
}

const strategyMap = {
  user: new UserStrategy(),
  company: new CompanyStrategy(),
  bot: new BotStrategy()
};

export const getStrategy = (type) => {
  return strategyMap[type] || strategyMap.user;
};
```
## Why We Forget This

Polymorphism isn't forgotten because it's hard. It's forgotten because we can avoid it. When you're building the first version, one if-else feels simpler than setting up strategies. The second type? Still feels fine. By the third or fourth type, you've already gone down the road and changing course feels like more work than just adding another branch.

I've done this. You've probably done this. It's not laziness—it's pragmatism in the moment colliding with reality later.

## The Real Cost

During our Vue 2 to Vue 3 migration, those tangled components were expensive. Every refactoring meant carefully testing multiple type-specific paths. Every bug fix risked breaking something in another type's logic. Components that should've been simple were hundreds of lines long because they were doing the job of five components.

That's when polymorphism stops being a nice-to-have principle and becomes a practical necessity.

## A Gentle Reminder

This isn't a call to over-engineer everything. Not every component needs polymorphism. But the next time you're writing the second conditional in a component, ask yourself: will there be a third? If the answer is yes—and honestly, it usually is—take five minutes to refactor toward polymorphism now instead of six months of refactoring later.

Your future self, dealing with whatever comes next, will thank you.

<p style="text-align: center;">fin.</p>

# h() function in Vue

When working with Vue.js render function (h()), you gain full power of javascript. 

The h() Function is designed to be very flexible, this is somewhere I found this function handy.

```js
import { h } from 'vue';

const Typography = (props, ctx) =>
    h(
        props.component,
        // component name ex: 'div | span ...'
        {
            class: 'text-gray-800',
            style: {
                'font-weight': '600'
            }
            // other attrs
        }[
        ctx.slots.default ? ctx.slots.default() : props.content
        // childrens
        ]
    );
    
export default Typography;
```

```vue
<script setup>
import Typography from './Typography';

const values = [
  { type: 'h1', content: 'H1 element using Typography' },
  { type: 'p', content: 'p tag using Typography' },
  { type: 'span', content: 'span element using Typography' }
];
</script>

<template>
  <Typography v-for="{ type, content } in values" :type="type" :key="content">
    {{ content }}
  </Typography>
</template>
```
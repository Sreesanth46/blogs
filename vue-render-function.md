# h() function in Vue

When working with Vue.js render function (h()), you gain full power of javascript. 

The h() Function is designed to be very flexible, this is somewhere I found this function handy.

```js
import { h } from 'vue'

const Typography = (props, ctx) => h(
props.component,
{
class: "text-gray-800"
}
)
```
---
title: Vue 3's Expose
date: 2024-09-23 03:58:30 +0530
---
##### TheCounter.vue
```vue
<template>
	<div>
		<p>Counter: {{ counter }}</p>
		<button @click="reset">Reset</button>
		<button @click="stop">Stop</button>
	</div>
</template>

<script>
import { ref } from 'vue'
export default { 
	setup () { 
		const counter = ref(0)
		
		const interval = setInterval(() => {
			counter.value++ 
		}, 1000) 
		
		const reset = () => {
			counter.value = 0 
		}
		
		const stop = () => {
			clearInterval(interval)
		} 
		
		return { 
			counter,
			reset,
			stop 
		}
	}
} 
</script>
```

##### App.vue
```vue
<template>
	<div>
		<TheCounter ref="counter" />
		<button @click="reset">Reset from parent</button>
		<button @click="stop">Stop from parent</button>
	</div>
</template>

<script>
import TheCounter from '@/components/TheCounter.vue'

export default {
	name: 'App',
	components: { TheCounter }, 
	methods: {
		reset () {
			this.$refs.counter.reset()
		},
		stop () {
			this.$refs.counter.stop()
		}
	}
}
</script>
```


Now lets update the `TheCounter.vue`
##### TheCounter.vue
```vue
<script>
import { ref } from 'vue'
export default { 
	setup (props, context) { 
		const counter = ref(0)
		
		const interval = setInterval(() => {
			counter.value++ 
		}, 1000) 
		
		const reset = () => {
			counter.value = 0 
		}
		
		const stop = () => {
			clearInterval(interval)
		}
		
		context.expose({ reset })
		
		return { 
			counter,
			reset,
			stop 
		}
	}
} 
</script>
```


Now if we run the example again, and "click stop from parent" button, we'll get a JavaScript error
> Uncaught TypeError: this.$refs.counter.terminate is not a function


Components using `<script setup>` are **closed by default** - i.e. the public instance of the component, which is retrieved via template refs or `$parent` chains, will **not** expose any of the bindings declared inside `<script setup>`.
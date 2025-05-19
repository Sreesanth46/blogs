---
title: Handling event listeners in React
date: 2025-04-09 18:31:36 +0530
---
```js
export default function App() {
	useEffect(() => {
		const onDrag = () => {};
		const onDragStart = () => {};
		const onDragEnd = () => {};

		window.addEventListener("drag", onDrag);
		window.addEventListener("dragstart", onDragStart);
		window.addEventListener("dragend", onDragEnd);

		return () => {
			window.removeEventListener("drag", onDrag);
			window.removeEventListener("dragstart", onDragStart);
			window.removeEventListener("dragend", onDragEnd);
		}
	}, [])
}
```


to 

```js
export default function App() {
	useEffect(() => {
		const onDrag = () => {};
		const onDragStart = () => {};
		const onDragEnd = () => {};
		const ctrl = new AbortController();
		const signal = ctrl.signal;

		window.addEventListener("drag", onDrag, { signal });
		window.addEventListener("dragstart", onDragStart, ctrl);
		window.addEventListener("dragend", onDragEnd, ctrl);

		return () => {
			ctrl.abort();
		}
	}, [])
}
```

Can even update the function in-line

```js
export default function App() {
	useEffect(() => {
		const ctrl = new AbortController();
		const signal = ctrl.signal;

		window.addEventListener("drag", () => {}, { signal });
		window.addEventListener("dragstart", () => {}, ctrl);
		window.addEventListener("dragend", () => {}, ctrl);

		return () => {
			ctrl.abort();
		}
	}, [])
}
```

[refer](https://youtube.com/shorts/Z09xJq5iA0c?si=hxPjTnL1Ui4ngz4x)
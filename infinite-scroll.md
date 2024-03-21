# How to Implement Infinite Scroll: A Step-by-Step Guide

Have you ever wondered how websites load data while scrolling? Infinite scrolling is the design approach used to continuously load data as the user scrolls, enhancing the user experience and improving performance by loading content on-demand.

In this guide, we'll explore how to implement infinite scroll in React using the Intersection Observer API. This approach can be applied to any front-end framework.

#### Prerequisites

Before we dive in, ensure you have a basic understanding of React, JavaScript, and working with APIs. We'll be implementing infinite scroll in a chat window, where older messages will be loaded automatically when the user scrolls to the top.
The Intersection Observer API

The Intersection Observer API provides a way to watch for changes in the intersection between a target element and its containing element or the viewport. This API is particularly useful for implementing features like infinite scroll, lazy loading, and more.

#### Step-by-Step Guide

1. Set up the React Component and State: 
  * Create a React component to represent the chat window.
  * Initialize the component state to store the loaded messages and any other necessary data.

2. Implement the Intersection Observer Logic: 
  * Create an Intersection Observer instance.
  * Define a reference to the container element where the messages will be rendered. (scrollTopRef)
  * Observe the container element using the Intersection Observer instance.

3. Handle Intersection Events: 
  * In the Intersection Observer callback, check if the observed element is intersecting with the viewport or container.
  * If the element is intersecting, trigger the data fetching logic.

4. Fetch Data and Update Component State: 
  * Implement a function to fetch older messages from the server or data source.
  * Update the component state with the newly fetched messages, appending them to the existing messages.

5. Render the Loaded Data: 
  * In the component's render method, map over the loaded messages and display them in the chat window.
  * Ensure the container element is present for the Intersection Observer to observe.

Example Usage

```ts
function ChatWindow() {
  const [messages, setMessages] = useState<IMessage[]>([]);
  const [page, setPage] = useState(0);
  const [hasMore, setHasMore] = useState(true);
  const chatContainerRef = useRef<HTMLDivElement>(null);
  const scrollTopRef = useRef<HTMLDivElement>(null);

  /** Fetch message initially */
  useEffect(() => {
    fetchMessage();
  }, []);

  /**
  *
  * Create intersection observer instance when message state updates.
  * root - observing container (Boundary)
  * rootMargin - enables one to enlarge (or restrict) the area of the root container
  * The rootMargin value is set to '300px', which means the intersection will be considered 
  * to occur when the observed element is 300 pixels away from the root (viewport or container element).
  *
  */
  useEffect(() => {
    const observer = new IntersectionObserver(onIntersection, {
      root: chatContainerRef.current,
      rootMargin: "300px",
    });

    if (observer && scrollTopRef.current) {
      observer.observe(scrollTopRef.current);
    }

    return () => {
      if (observer) observer.disconnect();
    };
  }, [messages]);

  const fetchMessage = (limit = 10) => {
    const skip = limit * page;

    fetch(`messages?skip=${skip}&limit=${limit}`)
      .then((res) => res.json())
      .then((data) => {
        setMessages((prev) => [...prev, ...data.comments]);
        setPage((prev) => prev + 1);

        if (data.comments.length < limit) {
          setHasMore(false);
        }
      })
      .catch((err) => {
        console.log(err);
      });
  };

  const onIntersection = (entries: IntersectionObserverEntry[]) => {
    const first = entries[0];

    if (first.isIntersecting && hasMore) {
      fetchMessage();
    }
  };

  return (
    <div className="h-screen w-screen bg-slate-50 flex justify-center items-center">
      <div
        className="h-[36rem] w-[20rem] bg-gray-800 rounded-md p-4 overflow-auto flex flex-col-reverse gap-4"
        ref={chatContainerRef}
      >
        {messages.map((message) => (
          <Message message={message.body} key={message.id} />
        ))}
        <div ref={scrollTopRef} />
      </div>
    </div>
  );
}

export default ChatWindow;
```

The chat container has the style `flex-direction: row-reverse;`, which implements a smooth infinite scroll experience for a chat window. This ensures that new messages are displayed at the bottom of the container, while older messages are appended at the top as they are loaded. By reversing the column order, the newest messages appear at the bottom, mirroring the expected behavior of a chat application.


#### Conclusion

The Intersection Observer API provides a powerful and efficient way to implement infinite scroll in web applications. By observing the intersection between elements and the viewport, you can seamlessly load data as the user scrolls, enhancing the user experience and improving performance. Experiment with the code examples provided, and explore further resources to deepen your understanding of this API.

[Source code](https://github.com/Sreesanth46/infinite-scroll)

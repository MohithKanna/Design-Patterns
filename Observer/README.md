# Observer Pattern

## Problem it solves

Sometimes one event needs to trigger multiple, independent reactions — e.g. when a document's content changes, the toolbar needs to update, the word count needs to refresh, and auto-save needs to trigger.

A naive approach hardcodes every reaction inside one function:

```ts
function onContentChange(newContent) {
    updateToolbar(newContent);
    updateWordCount(newContent);
    saveToIndexedDB(newContent);
}
```

This doesn't scale. Adding or removing a reaction means editing this function directly, risking merge conflicts and accidental breakage — especially once multiple features depend on the same event.

The **Observer Pattern** fixes this by decoupling "something happened" from "here's everyone who reacts to it." Reactions (observers) can subscribe or unsubscribe freely, without the event source ever changing.

## Real-world analogy

Think of a YouTube channel. The channel (subject) posts a video and notifies every subscriber (observer). The channel doesn't know or care who its subscribers are, or what they do when notified — it just notifies whoever is currently subscribed.

## The pieces

- **Subject** — holds the list of observers and calls `notify()` when something changes. Doesn't know or care what each observer does internally.
- **Observer** — a function (or object) that reacts when notified. Can subscribe/unsubscribe at any time, independently of other observers.
- **Key trait**: **one event → many reactions**. This is what distinguishes Observer from Strategy/Factory, which each pick exactly _one_ option.

## Code

```ts
type Observer<T> = (data: T) => void;

class Subject<T> {
    private observers: Observer<T>[] = [];

    subscribe(observer: Observer<T>) {
        this.observers.push(observer);
    }

    unsubscribe(observer: Observer<T>) {
        this.observers = this.observers.filter((obs) => obs !== observer);
    }

    notify(data: T) {
        this.observers.forEach((obs) => obs(data));
    }
}
```

**Usage:**

```ts
type ContentChangeData = {
    text: string;
    wordCount: number;
};

const contentSubject = new Subject<ContentChangeData>();

const updateWordCount = (data: ContentChangeData) => {
    console.log(`Word count is now: ${data.wordCount}`);
};

const updateToolbar = (data: ContentChangeData) => {
    console.log(data.text);
};

contentSubject.subscribe(updateWordCount);
contentSubject.subscribe(updateToolbar);

contentSubject.notify({ text: "hello world", wordCount: 2 });
// both updateWordCount and updateToolbar fire, each using only the fields it needs
```

To stop a reaction (e.g. disable auto-save temporarily), just unsubscribe — no changes needed to `Subject` itself:

```ts
contentSubject.unsubscribe(saveToIndexedDB);
```

## Where this shows up in the wild

React's `useEffect` + dependency array is Observer Pattern under the hood: the watched value (state) acts like the Subject, and the effect callback is an Observer that "subscribes" via its dependency array and reacts only when that value changes.

```ts
useEffect(() => {
    console.log("runs when count changes");
}, [count]);
```

Node's built-in `EventEmitter` is another well-known real-world implementation of this exact pattern.

## When to use it

- Multiple, independent parts of your app need to react to the same event (state change, data update, user action).
- You want to add/remove reactions without touching the code that triggers the event.
- You're building anything event-driven: pub/sub systems, real-time UI updates, editors with multiple dependent panels (toolbar, word count, autosave, live preview).

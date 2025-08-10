# 📘 Project Reactor Essentials – Summary

## 🎯 Purpose of the Playlist

This tutorial series covers **Project Reactor Essentials**, serving as a foundation before diving into **Spring WebFlux**. Without a solid understanding of Project Reactor, working with WebFlux can be challenging.

---

## 🚦 Why Reactive Programming?

**Reactive Programming** is a **programming paradigm** based on **asynchronous, event-driven** data streams.

### Core Ideas:
- Reactive is about **events** and **streaming data**.
- Events can include user clicks, HTTP requests, sensor data, etc.
- Everything is modeled as a **data stream**.
- Enables **asynchronous** and **non-blocking** operations.

---

## 🧱 Core Reactive Concepts

| Concept                  | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| **Asynchronous**         | Events happen independently; no blocking on I/O or waiting threads.        |
| **Non-blocking**         | Threads are not blocked waiting for data (e.g., from databases).           |
| **Backpressure**         | Mechanism for slowing down producers when consumers can't keep up.         |
| **Functional Programming** | Encourages immutability, pure functions, and testable code.               |

### 🎂 Backpressure Analogy:
- A cake factory makes 10 cakes/minute.
- The consumer can only eat 1 cake/minute.
- Backpressure ensures the producer slows down to the consumer's pace.

---

## 🔥 Hot vs ❄️ Cold (Lazy) Publishers

| Type        | Behavior                                                                 |
|-------------|--------------------------------------------------------------------------|
| **Cold**    | Lazy – start emitting data **only after** a subscriber subscribes.       |
| **Hot**     | Eager – **already producing data**, whether there's a subscriber or not. |

Example:
- Cold: Download starts **only** when requested.
- Hot: Live sensor streaming data continuously.

---

## 🔄 Reactive Streams Initiative

A **specification** that standardizes asynchronous, non-blocking data stream handling with backpressure.

### Key Interfaces:
| Interface      | Role                                                              |
|----------------|-------------------------------------------------------------------|
| `Publisher`    | Emits data (events).                                              |
| `Subscriber`   | Consumes data.                                                    |
| `Subscription` | Controls flow (e.g., request 1 item per second).                  |
| `Processor`    | Both `Publisher` and `Subscriber` – sits in the middle of stream. |

> Introduced officially in Java 9 under `java.util.concurrent.Flow`.

---

## ⚙️ Project Reactor

Project Reactor is a **Reactive Streams implementation** used by **Spring WebFlux**.

### Why Project Reactor?
- Default for Spring WebFlux.
- Fully **non-blocking**.
- Supports **backpressure**.
- Works with HTTP, TCP, UDP.
- Offers rich set of **operators** for stream processing.

### Core Types:
| Type   | Description                                         |
|--------|-----------------------------------------------------|
| `Mono` | A stream that emits **0 or 1 item** (like Optional).|
| `Flux` | A stream that emits **0 to many items** (like List).|

> Think: `Mono` = single item, `Flux` = list of items.

---

## 🧰 Stream Operators

Project Reactor provides powerful operators to:
- Transform (`map`, `flatMap`)
- Filter (`filter`)
- Combine (`merge`, `zip`)
- Handle errors (`onErrorResume`)
- Delay or time streams

These operators are essential for writing declarative and readable reactive code.

---

## 📚 Resources
- 📄 [Project Reactor Documentation](https://projectreactor.io/docs)
- 🌐 [Reactive Streams Initiative](https://www.reactive-streams.org/)
- 💡 Explore the `reactor-core` library for in-depth API usage.

---

## 🔄 Asynchronous vs Non-blocking in Reactive Streams

| Concept                      | **Asynchronous**                                                                                                    | **Non-blocking**                                                                                                            |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Definition**               | Work is done **later**, usually on another thread, and the result is delivered via a **callback** or **publisher**. | The thread **doesn’t wait** (block) for a task to complete; instead, it moves on immediately.                               |
| **Thread usage**             | Often uses **other threads** (e.g., schedulers, thread pools).                                                      | A single thread can **continue** working without getting stuck.                                                             |
| **Example**                  | A network call returns immediately, and the result is delivered via a `Mono` or `Flux` later.                       | A call to read data from a stream doesn’t block if data is not ready; it returns control, and data is delivered when ready. |
| **Does it imply the other?** | ❌ Not always non-blocking. Asynchronous code might **still block internally** (e.g., in another thread).            | ✅ Often asynchronous, but focused more on **not holding up threads**.                                                       |
| **Goal**                     | Schedule and defer work                                                                                             | Avoid wasting threads/resources                                                                                             |
| **Common use in Reactive**   | `Mono.fromCallable(() -> blockingOperation())` is asynchronous but still blocking.                                  | `Mono.fromSupplier(() -> nonBlockingOperation())` is non-blocking.                                                          |

## ✅ Example 1: Asynchronous but Blocking
```java
Mono.fromCallable(() -> {
    Thread.sleep(1000); // blocking
    return "Hello";
});

```
- The call is deferred and asynchronous (runs later).
- But still blocks a thread for 1 second.
## ✅ Example 2: Asynchronous and Non-blocking
```java
Mono.delay(Duration.ofSeconds(1))
    .map(tick -> "Hello");

```
- Delays for 1 second non-blockingly.
- No thread is held during the wait — event-driven!

# Reactive Streams Recap (Before Starting Project Reactor)

## 📘 What is Reactive Streams?
Reactive Streams is a **standard** for asynchronous stream processing with:

- ✅ **Asynchronous** behavior
- ✅ **Non-blocking** execution
- ✅ **Backpressure** support (to avoid overwhelming consumers)

It defines **four core interfaces**:
1. `Publisher`
2. `Subscriber`
3. `Subscription`
4. `Processor` 

---

## 🔄 Key Concepts

### 📤 Publisher
- Produces and emits data (events).
- It’s **lazy** — nothing happens until a `Subscriber` subscribes.
- Starts the data flow once subscribed.

### 📥 Subscriber
- Consumes the data.
- When subscribed, receives a `Subscription` object.
- Uses `subscription.request(n)` to control how many items it receives (**backpressure**).

### 🔁 Subscription
- Manages the connection between Publisher and Subscriber.
- Handles **backpressure** using the `request(n)` method.

---

## ⚙️ Flow Lifecycle

1. `Subscriber` subscribes to a `Publisher`.
2. `Publisher` calls `onSubscribe(subscription)` on the subscriber.
3. `Subscriber` calls `request(n)` to ask for `n` items. - handles `Back Pressure`
4. `Publisher` sends items using `onNext(item)` until:
    - All requested items are sent.
    - All available items are sent (`onComplete()` is called).
    - An error occurs (`onError()` is called).

---

## 🍰 Backpressure Example

If the Publisher can produce **10 cakes**, but the Subscriber can only handle **5**, it will request 5 — and only those 5 will be delivered.

---

## ✅ Completion Conditions

- `onComplete()`: All items delivered.
- `onError()`: An error occurred (e.g., out of ingredients).
- After either, the subscription is **cancelled** automatically.

---

## 🧠 Key Takeaways

- Reactive Streams = **async + non-blocking + backpressure**.
- Core flow:  
  **Publisher → Subscriber → Subscription → request(n) → onNext / onComplete / onError**
- Mastering this flow is essential before diving into **Mono**, **Flux**, and **Project Reactor**.


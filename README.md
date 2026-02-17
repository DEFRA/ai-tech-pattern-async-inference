# Async LLM Inference Technical Pattern

This repository contains a technical pattern and accompanying code samples / examples for implementing asynchronous LLM interactions that is compatible with GDS progressive enhancement and other accessibility principles.

## Who this pattern is for
- Developers building services that integrate with LLMs or other long-running AI processes
- Teams looking to implement accessible, reliable, and user-friendly AI services
- Anyone interested in learning how to design and implement async interactions with LLMs while adhering to GDS principles

## The problem
Large Language Models (LLMs) and other computational intensive AI services, when compared to traditional web services, typically have users waiting much longer for a response. This dependent on many factors which could include:
- The AI model being used and its size - For example, a model like Claude 3 Sonnet will use more computational resources and take longer to generate a response than a smaller model like Claude 3 Haiku.
- The complexity of the prompt / user request
- The load on the computational resources

Although users are used to near-instantaneous responses from traditional web services, we can manage their expectations and improve the UX by implementing techniques such as:
- Providing a loading indicator or progress bar to show that the service is working
- Providing an estimated time for the response, if possible
- Allowing users to cancel the request if they do not want to wait
- Partial results or step by step progress updates / events

### A naive approach
Imagine a scenario where you have an AI chatbot which:
- Uses a small LLM that can handle simple queries
- Your user research indicates users will be asking very simple questions
- Your testing indicates that the average response time for the LLM is around 5 seconds

In this scenario, you might be tempted to implement a synchronous HTTP request-response cycle, where:
1. The user submits a query through a form that sends a POST request to the server
2. The server sends the query to the LLM and waits for the response
3. Once the response is received, the server sends it back to the client, which then updates / redirects the user to a new page with the response
4. You accept that the user will only see the browser's default loading indicator during the wait time

Although this approach is simple to implement, it can quickly cause user experience issues. Many users interpret a default browser loading indicator as a sign that something has gone wrong or that the page has frozen. At best, this leads to confusion and frustration; at worst, users may conclude the service is slow or broken.

Another significant risk is timeouts. Whether from the client side (browser timeout) or via a gateway (e.g., load balancer timeout), users will encounter a generic, unbranded error page that provides no helpful guidance. Although, your testing might indicate in a controlled environment that the LLM responds within timeout limits, real-world conditions can vary widely. Unless you purchase expensive infrastructure to guarantee response times, you will be inevitably sharing compute with other services / tenants. 

Even if your LLM responds within timeout limits, various factors can cause delays: high server load, network issues, or slow user connections. When timeouts occur, your service crashes or becomes unreliable—because it isn't architected to handle these scenarios gracefully.

### A better but not accessible approach
Techniques like WebSockets, Server-Sent Events (SSE), and long polling have been used in tools like Claude, ChatGPT, and others to dispatch status updates and partial results directly to the client. These approaches solve the UX problem by giving users branded, in-page loading indicators, typing animations, and potentially streaming responses. This can help manage expectations and makes the service more resilient to timeouts.

But there's a critical flaw: these techniques on their own breach GDS progressive enhancement and accessibility principles. These techniques require client-side JavaScript to function, offering no fallback for users with JavaScript disabled or assistive technologies. This means you're trading one user experience problem for another, which is excluding users who rely on accessible web technologies or work in constrained environments.

As with mentioned in the [problem statement](#the-problem), users are accustomed to near-instantaneous responses from traditional web services. This is also reflected in the current field of GDS components and patterns, of which there are not many that are designed to handle long-running processes.

## The solution

The solution follows [GDS progressive enhancement principles](https://www.gov.uk/service-manual/technology/using-progressive-enhancement): build a baseline experience that works for everyone, then layer on enhancements for users whose browsers support them.

> [!NOTE]
> The core concept is standard asynchronous architecture - decouple the message submission from the response delivery. This applies to any long-running operation, not just LLMs.

Continuing with the chatbot example from the previous section, the a general flow would look like:
1. User submits a query through a standard HTML form
2. Process asynchronously - Queue the message for background processing (using job queues, message queues, or background tasks)
3. Return a response immediately - Respond to the initial request with an HTTP 202 (Accepted) status, indicating the message is being processed
    * If JavaScript is disabled - Redirect to the conversation page where a loading indicator (spinner, typing animation) is rendered server-side, along with a prompt to refresh the page for updates
    * If JavaScript is enabled - Prevent the default form submission, subscribe to real-time updates (via WebSockets or SSE), and update the page dynamically when the response is ready

> [!TIP]
> WebSockets are normally excessive for this type of interaction, as the communication is mostly one-way (server to client). SSE is a simpler and more efficient choice for real-time updates in this scenario. See https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events for more details.

This eliminates timeout risks, gives users control, and ensures the service remains accessible to all users regardless of their technology constraints.

## Implementation patterns

This repository contains several implementation patterns, each building on the core async concept with different levels of sophistication:

- [**Loading Spinner / Typing Animation**](patterns/loading-spinner.md) - The simplest pattern. Baseline: users manually refresh a status page. Enhancement: automatic polling updates the page when ready.

## Examples

The `example/` directory contains working implementations of these patterns:

# Pausing Between Reconnect Attempts

It doesn’t make much sense to try to connect to the same server over and over. To prevent this sort of thrashing, and wasted reconnect attempts, especially when using TLS, libraries provide a wait setting. Generally clients make sure that between two reconnect attempts to the **same** server at least a certain amount of time has passed. The concrete implementation depends on the library used.

This setting not only prevents wasting client resources, it also alleviates a [_thundering herd_](random.md) situation when additional servers are not available.

{% tabs %}
{% tab title="Go" %}
```go
// Set reconnect interval to 10 seconds
nc, err := nats.Connect("demo.nats.io", nats.ReconnectWait(10*time.Second))
if err != nil {
    log.Fatal(err)
}
defer nc.Close()

// Do something with the connection
```
{% endtab %}

{% tab title="Java" %}
```java
Options options = new Options.Builder().
                            server("nats://demo.nats.io:4222").
                            reconnectWait(Duration.ofSeconds(10)).  // Set Reconnect Wait
                            build();
Connection nc = Nats.connect(options);

// Do something with the connection

nc.close();
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
let nc = NATS.connect({
    servers: ["nats://demo.nats.io:4222"]
});
```
{% endtab %}

{% tab title="Python" %}
```python
nc = NATS()
await nc.connect(
   servers=["nats://demo.nats.io:4222"],
   reconnect_time_wait=10,
   )

# Do something with the connection

await nc.close()
```
{% endtab %}

{% tab title="Ruby" %}
```ruby
require 'nats/client'

NATS.start(servers: ["nats://127.0.0.1:1222", "nats://127.0.0.1:1223", "nats://127.0.0.1:1224"], reconnect_time_wait: 10) do |nc|
   # Do something with the connection

   # Close the connection
   nc.close
end
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
// will throw an exception if connection fails
let nc = await connect({
    servers: ["nats://demo.nats.io:4222"]
});
nc.close();
```
{% endtab %}
{% endtabs %}

Some libraries will allow you to specify some random jitter to add to the reconnect wait specified above.

{% tabs %}
{% tab title="Go" %}
```go
// Set some jitter up to 100 millisecond for non TLS connections and 1 second for TLS connections.
nc, err := nats.Connect("demo.nats.io", nats.ReconnectJitter(100*time.Millisecond, 1*time.Second))
if err != nil {
    log.Fatal(err)
}
defer nc.Close()

// Do something with the connection
```
{% endtab %}
{% endtabs %}

You can also instead specify a custom reconnect delay callback that will be invoked by the library when the whole list of servers has been tried unsuccesfully. The library will wait for the duration returned by this callback.

{% tabs %}
{% tab title="Go" %}
```go
// Set a custom callback that returns some backoff duration. The library passes the number of attempts
// of the whole list of server URLs, which can be useful to determine a specific delay.
nc, err := nats.Connect("demo.nats.io", nats.CustomReconnectDelay(func(attempts int) time.Duration {
    return someBackoffFunction(attempts)
}))
if err != nil {
    log.Fatal(err)
}
defer nc.Close()

// Do something with the connection
```
{% endtab %}
{% endtabs %}

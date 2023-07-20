---
title: "Rate Limiter in API Design"
date: 2023-07-11T19:19:18-04:00
draft: true
---

A rate limiter is a mechanism used to control the rate at which certain operations or requests are allowed to be processed by a system or an API. It helps prevent abuse, protect system resources, and ensure fair usage by limiting the number of requests or actions that can be performed within a specific timeframe. And Richard Schneeman has put it nicely in his [tweet](https://twitter.com/schneems/status/1138899094137651200?s=20) where he said:
> If you provide an API client that doesn't include rate limiting, you don't really have an API client. You've got an exception generator with a remote timer.

![image](/funnel.png#center)

Here are some reasons why rate limiters are important in system design:

1. **Protection against abuse and malicious activity**: Rate limiters help mitigate the impact of abusive behavior, such as denial-of-service (DoS) attacks or brute-force attacks. By limiting the rate at which requests can be made, they prevent an individual or a group from overwhelming the system.

2. **Ensure fair resource allocation**: Rate limiters ensure fair usage of system resources by preventing a single user or client from monopolizing them. By limiting the rate of requests, they promote equitable access for all users and prevent resource exhaustion.

3. **Avoidance of performance degradation**: When a system experiences a sudden surge in traffic or a burst of requests, it can lead to performance degradation or even system failure. Rate limiters help smoothen the traffic by capping the rate of incoming requests, preventing spikes and maintaining system stability.

4. **API management and monetization**: In API design, rate limiters are crucial for managing API usage and enforcing API rate limits. They allow API providers to control the number of requests that can be made by individual users or across different tiers of API subscriptions. This helps manage resources, maintain service quality, and enable monetization models based on usage tiers.

5. **Capacity planning and resource optimization**: Rate limiters provide insights into the usage patterns and demands on a system. By analyzing request rates and identifying peak loads, system administrators can make informed decisions about capacity planning, resource allocation, and infrastructure scaling.


## Algorithms

There are numerous rate-limiting algorithms are readily available, and multiple libraries have implemented different variations of these algorithms. Here are a few widely used rate-limiting algorithms:

1. **Fixed Window:** In this algorithm, a fixed time window is defined (e.g., 1 second), and within that window, a maximum number of requests or events are allowed. Once the limit is reached, subsequent requests are rejected until the next window starts.

2. **Sliding Window:** Similar to the fixed window algorithm, a sliding window also uses a time window to track requests. However, the window slides continuously, allowing a limited number of requests in a given interval. The maximum number of requests is typically calculated based on an average rate per unit of time.

3. **Token Bucket:** The token bucket algorithm works based on tokens. A bucket is filled with a certain number of tokens, and each request or event consumes a specific number of tokens. If there are enough tokens available, the request is processed, and tokens are deducted. If the bucket is empty, requests are either rejected or delayed until tokens are replenished over time.

4. **Leaky Bucket:** This algorithm is similar to the token bucket algorithm but works in the opposite way. Instead of tokens, the system assumes there is a "leaky bucket" that can hold a specific volume of requests. Requests are added to the bucket, and if it overflows, excess requests are discarded or delayed. The requests leave the bucket at a constant rate, ensuring a controlled flow.

5. **Adaptive Rate Limiting:** This approach dynamically adjusts the rate limit based on system conditions and historical patterns. It takes into account factors such as system load, response times, and the overall capacity of the system. By adapting the rate limit, it can handle sudden bursts of traffic while maintaining system stability.

These algorithms can be implemented at different levels, such as application-level rate limiting or network-level rate limiting, depending on the specific requirements and infrastructure of the system.

## Implementation

There are different approaches to implementing rate limiting in a Go application, but one commonly used method is the **token bucket** algorithm. This algorithm allows you to define a certain number of tokens (representing requests or actions) that can be consumed over a given period of time.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type RateLimiter struct {
	rate       float64
	capacity   float64
	tokens     float64
	lastUpdate time.Time
	mu         sync.Mutex
}

func NewRateLimiter(rate float64, capacity float64) *RateLimiter {
	return &RateLimiter{
		rate:     rate,
		capacity: capacity,
		tokens:   capacity,
	}
}

func (rl *RateLimiter) Allow() bool {
	rl.mu.Lock()
	defer rl.mu.Unlock()

	now := time.Now()
	elapsed := now.Sub(rl.lastUpdate).Seconds()
	rl.tokens += elapsed * rl.rate
	if rl.tokens > rl.capacity {
		rl.tokens = rl.capacity
	}
	rl.lastUpdate = now

	if rl.tokens >= 1 {
		rl.tokens -= 1
		return true
	}
	return false
}

func main() {
	limiter := NewRateLimiter(2, 5) // Allow 2 requests per second with a capacity of 5 tokens

	for i := 1; i <= 10; i++ {
		if limiter.Allow() {
			fmt.Println("Request", i, "allowed")
		} else {
			fmt.Println("Request", i, "blocked")
		}
		time.Sleep(500 * time.Millisecond) // Simulating requests every 500 milliseconds
	}
}

```
Here `Allow()` function implements the token bucket. It calculates the elapsed time since the last token update, adds new tokens based on the rate until capacity is reached. If a token is available then it decrements the token count & returns `true` else it returns `false` & request is blocked.

In the `main` function, we create a `RateLimiter` instance to allow 2 requests per second with a capacity of 5 tokens. Then, a loop simulates 10 requests, checking if each request is allowed or blocked based on the rate limiter's decision.


{{< details "Ruby Implementation" >}}

Here I will demonstrate how to incorporate a rate limiter into our Ruby on Rails API. We'll utilize the widely-used **[rack-attack](https://github.com/rack/rack-attack)** gem, which offers built-in features such as throttling, safelisting, blocklisting, and graphing capabilities.

**Step 1:** Add the `rack-attack` gem to your Gemfile and run `bundle install` to install it.

```ruby
gem 'rack-attack'
```

**Step 2:** Create a new initializer file, such as `config/initializers/rack_attack.rb`, and add the following configuration:

```ruby
# config/initializers/rack_attack.rb
class Rack::Attack
  Rack::Attack.cache.store = ActiveSupport::Cache::MemoryStore.new

  # Throttle requests from a single IP address
  throttle('req/ip', limit: 100, period: 1.minute) do |req|
    req.ip
  end
end
```

In this example, we're throttling requests from a single IP address to 100 requests per minute. You can adjust these values according to your needs.


**Step 3:** Optionally, you can customize the response when the rate limit is exceeded. You can add the following method to your `Rack::Attack` configuration block:

```ruby
# config/initializers/rack_attack.rb
class Rack::Attack
  # ...

  self.throttled_response = lambda do |env|
    # Customize the response as needed
    [429, {}, ['Rate limit exceeded.']]
  end
end
```

In this example, a response with a status code of 429 (Too Many Requests) and a message "Rate limit exceeded." will be returned when the rate limit is exceeded.

**Step 4:** Restart your Rails server for the changes to take effect.

With these steps, you've added a basic rate limiter to your Ruby on Rails API using the `rack-attack` gem.
{{< /details >}}

## Conclusion

Overall, rate limiters play a vital role in maintaining the stability, security, and fairness of systems and APIs. They help balance the trade-off between system performance and resource utilization, ensuring that the system operates within its capacity and delivers a consistent user experience.

## Resources
- [Rate-Limiter](https://en.wikipedia.org/wiki/Rate_limiting)
- **[rack-attack](https://github.com/rack/rack-attack)** gem
- [Go's ratelimit](https://github.com/uber-go/ratelimit)

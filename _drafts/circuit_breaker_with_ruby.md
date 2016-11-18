---
layout: post
categories: ruby circuit-breaker
---
At the previous post we see how use Dependency Injection. Today I will try to show how we can use CircuitBreaker with Ruby. I will use rails to show all examples. but be free to use everything that you want.

First, we need to think about circuit-breaker. Let me see... Oh Circuit-Breaker pattern works like a home circuit-breaker, Obvious. But try to understand the mechanism behind this behavior.

When a circuit-breaker receives heavy charge of energy, it's stop to pass energy to another place of circuit. Circuit-breakers pattern has a similar behavior, when something goes wrong, the circuit-breaker stops to call the falling call and pass to responds only with a fallback method. After a time period, the circuit-breaker

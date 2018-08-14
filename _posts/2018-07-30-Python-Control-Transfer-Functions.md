---
layout: blog
title: "Simulating real-time process outputs"
date: "2018-07-30"
author: ["Siang Lim"]
---

Given a transfer function representation of a system, how do we determine its output in response to external control actions?

If we know all the history of all control actions, $$U$$, from the initial state of the system at $$t=0$$ to the present $$t=t$$, then we can compute the output easily in python-control using `forced_response()`.

If this is our transfer function,

$$
G(s) = \frac{1}{0.5s^2+0.5s+1}
$$

Then we can do:
```python
T = np.linspace(0,20,100)
U = np.zeros(len(T))
U[10:50] = 1 # Our control action U
G_s = control.tf([1.0],[0.5,0.5,1])
t, yout, _ = control.forced_response(G_s, T, U)
```

To get this:
![transfer function]({{ "/assets/images/transfer_function.gif" | absolute_url }}){:class="center-image"}

### Unknown Actions, $$U$$
Consider the situation where a human operator is making control actions based on process behaviour. Every action taken by the operator affects the process output.

How can we simulate this? A naive and inefficient computation of the process output using `forced_response` would be to store all past control actions and recompute the output at each time step using the entire set of control actions. 

To do this, we could use a Python generator and the `yield` statement:

1. Compute the output of the system at time $$t$$ using all past control actions, $$U$$
2. Store the state of all local variables
3. Yield control and observations to the user
4. Take in new control actions at $$t+1$$, repeat.

Here's the implementation:
```python
def TFGenerator():
    # Initial values
    G_s = control.tf([1.0],[0.5,0.5,1])
    T = np.linspace(0,2,2+1)
    actions = np.zeros(len(T))
    while True:
        t, yout, _ = control.forced_response(G_s, T, actions)
        u          = yield t, yout
        T          = np.append(T, t[-1]+1)
        actions    = np.append(actions, u)
        print(t[-1],yout[-1])
```

We'll initialize the generator using:
```python
tf = TFGenerator()
t, yout = tf.send(None)
```

And now we can start sending control actions, step by step:
```python
t, yout = tf.send(0)
t, yout = tf.send(0)
t, yout = tf.send(1)
...
```

#### So what's the problem with this? 
The transfer function doesn't have a 'memory'. To compute the output of the system at time $$t$$, we need to provide all inputs starting from $$t=0$$. We can't use it to get the correct output going from $$t-1$$ to $$t$$.

With the current method, at time $$t$$, **we are recalculating all our previous outputs** from $$t=0$$ to $$t-1$$ using `forced_response()`. This seems wasteful and very computationally inefficient.

If we are working with Linear Time Invariant (LTI) systems, we can use the superposition principle and sum the outputs from each individual control action to get the final response. This would still involve storing, summing and tracking each and every control action and process output.

Is there a cleaner and more efficient way to do this? To be continued...

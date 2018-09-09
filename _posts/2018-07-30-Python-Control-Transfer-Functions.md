---
layout: blog
title: "Simulating real-time process outputs"
date: "2018-07-30"
author: ["Siang Lim"]
---

Given a transfer function representation of a system, $$ G(s) $$, how do we determine its output, $$ y(t) $$, in response to a control action, $$ u(t) $$, in real time?

### Case 1: Known control actions, $$U$$

If we know the history of all control actions, $$U = u(t), u(t-1), \dots, u(0)$$, from the initial state of the system at $$t=0$$ to the entire simulation time of interest $$t=t$$, then we can compute the output, $$y(t)$$ easily in [python-control](https://github.com/python-control/python-control) using `forced_response()`. This is equivalent to using `lsim` in MATLAB. Note that the forcing functions and input functions both refer to control actions.

To illustrate, given a transfer function, let's say:

$$
G(s) = \frac{1}{0.5s^2+0.5s+1}
$$

Using the python-control library to simulate a pulse input, we can do:
```python
T = np.linspace(0,20,100)
U = np.zeros(len(T))
U[10:50] = 1 # Our control action U
G_s = control.tf([1.0],[0.5,0.5,1])
t, yout, _ = control.forced_response(G_s, T, U)
```

To get this 20-second simulated output of $$ y(t) $$:

{% include figure.html url="/assets/images/transfer_function.gif" alt="Transfer Function" caption="Figure 1: 20-second simulated output of the transfer function" %}

### Case 2: Unknown $$U$$
What if we don't have the entire history of control actions?

Consider the situation where operators at a manufacturing facility are making control actions based on process behaviour. Every action taken by the operator affects the process output. 

{% include figure.html class="large-image" url="/assets/images/manufacture.jpg" alt="Manufacturing" caption="Figure 2: Control or operations room in a plant" %}

Let's say we want to mimic the behaviour of said manufacturing plant on a smaller scale. Given a process and its transfer function representation, we want users to send control actions to the process and simulate the output in real time. Let's further assume that we want a 20-second simulation again like in Case 1.

The difference here is that since this is a real-time simulation, we are simulating the output, $$ y(t) $$ step by step, let's say one second at a time. We don't know the entire 20-second set of control actions at each step since we can't predict the future.

A naive and inefficient computation of the process output using `forced_response` would be to store all past control actions and recompute the output at each time step using the entire set of control actions. 

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
The transfer function doesn't have a 'memory'. It only provides a mapping from inputs to outputs. To compute the output of the system at time $$t$$, we need to provide all inputs starting from $$t=0$$. We cannot use it to get the correct output going from $$y(t-1)$$ to $$y(t)$$ unless we know all previous control actions $$ u(t-1), u(t-2), \dots. u(0) $$.

With the current method, at time $$t$$, **we are recalculating all our previous outputs that we already know** from $$t=0$$ to $$t-1$$ by repeatedly calling `forced_response()`. This seems wasteful and very computationally inefficient.

If we are working with Linear Time Invariant (LTI) systems, we can use the superposition principle and sum the outputs from each individual control action to get the final response. This would still involve storing, summing and tracking each and every control action and process output.

Is there a cleaner and more efficient way to do this?
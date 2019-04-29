# A framework for building finite state machines in Rust

[![Build Status][build-badge]][build-link]

The `rust-fsm` crate provides a simple and universal framework for building
state machines in Rust with minimum effort. This is achieved by two
components:

* The `rust-fsm` crate, that provides data types for building state machines
  and convenience wrappers for these types.
* The `rust-fsm-dsl` crate, that contains the `state_machine` macro that
  parses a simple DSL and generates all boilerplate code for the described
  state machine.

The essential part of this crate is the [`StateMachine`] trait. This trait
allows a developer to provide a strict state machine definition, e.g.
specify its:

* An input alphabet - a set of entities that the state machine takes as
  inputs and performs state transitions based on them.
* Possible states - a set of states this machine could be in.
* An output alphabet - a set of entities that the state machine may output
  as results of its work.
* A transition function - a function that changes the state of the state
  machine based on its current state and the provided input.
* An output function - a function that outputs something from the output
  alphabet based on the current state and the provided inputs.
* The initial state of the machine.

Note that on the implementation level such abstraction allows build any type
of state machines:

* A classical state machine by providing only an input alphabet, a set of
  states and a transition function.
* A Mealy machine by providing all entities listed above.
* A Moore machine by providing an output function that do not depend on the
  provided inputs.

## Use

Initially this library was designed to build an easy to use DSL for defining
state machines on top of it. Using the DSL will require to connect an
additional crate `rust-fsm-dsl` (this is due to limitation of the procedural
macros system). 

### Using the DSL for defining state machines

The DSL is parsed by the `state_machine` macro. Here is a little example.

```rust
#[macro_use]
extern crate rust_fsm_dsl;

use use rust_fsm::*;

state_machine! {
    CircuitBreaker(Closed)

    Closed(Unsuccessful) => Open [SetupTimer],
    Open(TimerTriggered) => HalfOpen,
    HalfOpen(Successful) => Closed,
    HalfOpen(Unsuccessful) => Open [SetupTimer],
}
```

This code sample:

* Defines a state machine called `CircuitBreaker`;
* Sets the initial state of this state machine to `Closed`;
* Defines state transitions. For example: on receiving the `Successful`
  input when in the `HalfOpen` state, the machine must move to the `Closed`
  state;
* Defines outputs. For example: on receiving `Unsuccessful` in the
  `Closed` state, the machine must output `SetupTimer`.

This state machine can be used as follows:

```rust
// Initialize the state machine. The state is `Closed` now.
let mut machine: StateMachineWrapper<CircuitBreaker> = StateMachineWrapper::new();
// Consume the `Successful` input. No state transition is performed. Output
// is `None`.
machine.consume_anyway(&CircuitBreakerInput::Successful);
// Consume the `Unsuccesful` input. The machine is moved to the `Open`
// state. The output is `SetupTimer`.
let output = machine.consume_anyway(&CircuitBreakerInput::Unsuccesful);
// Check the output
if output == Some(CircuitBreakerOutput::SetupTimer) {
    // Set up the timer...
}
// Check the state
if machine.state() == &CircuitBreakerState::Open {
    // Do something...
}
```

As you can see, the following entities are generated:

* An empty structure `CircuitBreaker` that implements the `StateMachine`
  trait.
* Enums `CircuitBreakerState`, `CircuitBreakerInput` and
  `CircuitBreakerOutput` that represent the state, the input alphabet and
  the output alphabet respectively.

Note that if there is no outputs in the specification, the output alphabet
is set to `()`. The set of states and the input alphabet must be non-empty
sets.

### Without DSL

The `state_machine` macro has limited capabilities (for example, a state
cannot carry any additional data), so in certain complex cases a user might
want to write a more complex state machine by hand.

All you need to do to build a state machine is to implement the
`StateMachine` trait and use it in conjuctions with some of the provided
wrappers (for now there is only `StateMachineWrapper`).

You can see an example of the Circuit Breaker state machine in the
[project repository][repo].

[repo]: https://github.com/eugene-babichenko/rust-fsm/blob/master/examples/circuit_breaker.rs
[build-badge]: https://travis-ci.org/eugene-babichenko/rust-fsm.svg?branch=master
[build-link]: https://travis-ci.org/eugene-babichenko/rust-fsm

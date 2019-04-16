---
layout: post
title: Java Design Patterns|Command
---
# Builder

## Purpose
*   Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

## Structure
![Builder_Model1](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/design-patterns/Command_Model1.gif)

*   <span style="font-weight:bold;">Command : declares an interface for executing an operation.</span>
*   <span style="font-weight:bold;">ConcreteCommand</span> : defines a binding between a Receiver object and an action. implements Execute by invoking the corresponding operation(s) on Receiver.
*   <span style="font-weight:bold;">Client</span> : creates a ConcreteCommand object and sets its receiver.
*   <span style="font-weight:bold;">Invoker</span> : asks the command to carry out the request.
*   <span style="font-weight:bold;">Receiver</span> : knows how to perform the operations associated with carrying out a request. Any class may serve as a Receiver.

## Interaction
![Builder_Model1](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/design-patterns/Command_Seq1.gif)

*   The client creates a ConcreteCommand object and specifies its receiver.
*   An Invoker object stores the ConcreteCommand object.
*   The invoker issues a request by calling Execute on the command. When commands are undoable, ConcreteCommand stores state for undoing the command prior to invoking Execute.
*   The ConcreteCommand object invokes operations on its receiver to carry out the request.

## Applications
*   parameterize objects by an action to perform, as MenuItem objects did above. You can express such parameterization in a procedural language with a callback function, that is, a function that's registered somewhere to be called at a later point. Commands are an object-oriented replacement for callbacks.
*   specify, queue, and execute requests at different times. A Command object can have a lifetime independent of the original request. If the receiver of a request can be represented in an address space-independent way, then you can transfer a command object for the request to a different process and fulfill the request there.
*   support undo. The Command's Execute operation can store state for reversing its effects in the command itself. The Command interface must have an added Unexecute operation that reverses the effects of a previous call to Execute. Executed commands are stored in a history list. Unlimited-level undo and redo is achieved by traversing this list backwards and forwards calling Unexecute and Execute, respectively.
*   support logging changes so that they can be reapplied in case of a system crash. By augmenting the Command interface with load and store operations, you can keep a persistent log of changes. Recovering from a crash involves reloading logged commands from disk and reexecuting them with the Execute operation.

## Consequences
*   Command decouples the object that invokes the operation from the one that knows how to perform it.
*   Commands are first-class objects. They can be manipulated and extended like any other object.
*   You can assemble commands into a composite command. An example is the MacroCommand class described earlier. 
*   It's easy to add new Commands, because you don't have to change existing classes.
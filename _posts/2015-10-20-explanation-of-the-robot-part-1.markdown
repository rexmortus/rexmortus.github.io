---
published: true
title: Explanation of the Robot, Part I
layout: post
tags: [software design, tutorials, ruby]
categories: [tutorials]
---
Here are some notes I've collected about the famous *Robot Challenge* using Ruby as the reference language. For now, it's called "Part I" because there is still more to write. However so that I'd have reason to complete it, here is section 4: "Application Knowledge".

## 4 Application Knowledge

A useful starting point when designing an application is to ask the question, "What does this application need to know?"

In the robot example, our application needs to know:

- How to read input from the user, both dynamically and from a file
- How to interpret input as commands for the robot
- What are acceptable commands
- How to move the robot around a surface

In the following section, we will examine each aspect of application knowledge and discuss their particular design concerns.

### 4.1 Reading input

The very first thing our application must know is how to accept input from the user. According to the brief, there are two modes to accommodate - real-time input, and input from text files.

In particular, the challenge is to build a flexible and robust interface which handles both modes *while writing a minimum of code*. This is an interesting problem because our design here will have serious ramifications for the overall complexity of our solution.

*The Art of UNIX Programming* offers advice on this issue:

> Its hard to avoid programming overcomplicated monoliths if none of your programs can talk to each other.

By designing our application's interface to easily communicate with other programs, we save ourselves the trouble of writing and maintaining two separate interfaces (one for streams and one for files), and negate the need for a lot of complicated code. This is called "The Rule of Composition", which advises us to keep our program interfaces textual and stream-oriented. In this case, our goal is to design a singular interface that can handle both streams and files with one code path.

### 4.2 Interpreting and delegating input

The second thing our application must know is how to interpret this textual input.

In particular, the challenge is to parse the input into a format the `Robot` can eventually understand, without duplicating knowledge of "what are acceptable commands" and without resorting to needlessly complex structures or algorithms.

In order of operations, we must:

- Receive input from the program's main interface
- Handle both single and multiple lines
- Separate commands and arguments
- Return a representation of the input

Receiving the input, handling single/multiple lines, and separating commands and arguments is remarkably easy if we're dealing with text. In Ruby, [Strings](http://www.ruby-doc.org/core-2.1.5/String.html) already have everything we need to accomplish this *without* resorting to our own logic or, god forbid, regular expressions.

The *real* challenge lies in deciding how to represent commands in a way that works well with the rest of our program, not just the `Robot`. Soon, our commands will pass through some kind of intermediary object (a delegator), the countours of which will be largely determined by how we design this representation. The trick will be to ensure that the delegator doesn't require any knowledge of the `Robot`'s interface, since this would contradict the principle of "Single Source of Truth".

To illustrate, consider the most common approach of using a switch:

```

commands.each do |command, args|

    case command
    when 'place'
        robot.place(args)
    when 'move'
        robot.move
    ...
    end
end

```
This is a poor solution because it is a duplication of the `Robot`'s interface. Extend or alter `Robot` and you'll have to change this `case`, too. How we package parse2d input for subsequent delegation is central to avoiding strategies like the one above.

### 4.3 Defining Commands

One of the central problems is defining `Robot` so that it provides the Single Source of Truth for  "what are acceptable commands." As demonstrated above, spreading this knowledge around is an easy mistake to make.

The answer here is simple: the Single Source of Truth for should be the public interface of `Robot`. To answer "is this an acceptable command" we can use Ruby's dynamic messaging system, with calls such as: `method_defined?` for validating commands, `method_missing` for handling invalid commands, and `send` for sending commands.

Another way to think of this: whenever the application needs to know if a command is valid or not, it should *always* get the the answer from `Robot` directly.

The second main concern is in designing `Robot` to be extensible. Luckily, Ruby makes it easy to  compose `Robot`'s interface with modules and mixins, so the question *really* is what kinds of abstractions facilitate rapid extension of the simulation. For exaxmple, if you wanted to introduce a new class of object to the surface - "impassable obstacle" - it would share certain characteristics with  `Robot`, but not others.

> Design for the future, for it will be here sooner than you think.

Thus, modular inheritance makes the most sense here.

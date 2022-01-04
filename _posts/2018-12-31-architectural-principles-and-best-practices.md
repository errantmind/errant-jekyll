---
layout: post
title: "Architectural Principles and Best Practices"
categories: software
---

### Overview

Solution design starts by defining and prioritizing architectural principles. These principles are **values** which guide implementation decisions.

Best practices are needed to ensure architectural principles are both maximized and implemented consistently. They take the form of rules / guidelines / heuristics

### Architectural Principles

- Simplicity (YAGNI)

  - **Reasoning**: Developers often prematurely implement abstractions. Their intent varies:

    - They are genuinely trying to reduce future complexity. They may have run into issues on a previous project and were able to mitigate them by implementing a particular abstraction. They want to avoid the issue in the future so they abstract before the issue ever arises
    - They have an agenda. They may have read about a particular design pattern, methodology, or practice and become *true believers*.  They now strongly believe in *a priori* best practices.  In the process they sacrifice their objectivity as they have materialized their beliefs as a part of their identity.  They now have a vested interest in applying their beliefs, regardless of other options

    Both of these intentions often lead to cost overruns or outright failure due to a common folly: there is always funding for adding new features, but rarely funding for refactoring.  As abstractions are added and complexity compounds, the likelihood of them ever being refactored or removed falls to zero

  - **Action(s)**: Minimize the use of abstractions in both software development (design patterns, functions, classes, shared state, asynchronous code, parallel code, *etc.*) and cloud infrastructure (unnecessary / shiny AWS services) unless there would otherwise be an immediate, unavoidable increase in complexity or excessive cost

- Performance (Traces => Optimization)

  - **Reasoning**: Optimized processes reduce operational costs and increase developer velocity
    - Cloud resources are often charged by the minute.  The longer a process executes, the more it costs
    - Context switching is a drain on developer productivity.  The longer a developer has to wait for feedback when developing, the greater the chances of distraction and derailment from flow
  - **Action(s)**:
    - Decrease execution time by occasionally profiling application performance and refactoring *hot-spots*.  This cuts costs by decreasing total process execution time and, further, shortens developer feedback loops
    - Minimize context switching (mental overhead) by actively mitigating incidental complexity in developers' critical path: build times, test execution times, merge conflicts, interpersonal interruptions (*e.g.,* meetings, cross-chat), project management overhead

- Reliability (Fault Tolerance, Error Handling and Recovery)

  - **Reasoning**: Resilient systems reduce both operational cost and outages
  - **Action(s)**: Be intentional and mindful when developing the system.  Code should always simultaneously perform work *and* express intent.  Strive for / to :
    - Consistent Clarity: *e.g.,* expressive names for variables, classes, and functions, comments as a last resort
    - Consistent Style: *e.g.,* indentation, line wrapping
    - Avoid side effects: *e.g.,* avoid shared state, data mutations, early returns, unhandled exceptions
    - Implement In-Place, Functional Validations: *e.g.,* parameter, return, expected behavior checks

- Scalability (Divisible Unit of Work)

  - **Reasoning**: A scalable process can deliver time-sensitive value to the business
  - **Actions(s)**: Implement the process with a configurable, deterministic unit of work, which can be parallelized across many cloud resources on-demand.  Avoid infrastructure and service lock-in, as this limits the flexible adoption of superior technology as it arrives

### Best Practices

**Let Abstraction Evolve from Clear, Primitive Blocks of Code**

- Reasoning:
  - Code should be written in such a way that it can be maintained and optimized in the future.  Developers can maintain and optimize code only after they understand it. Understanding is easier when the code is well organized and clear
  - One way write clear code is to begin by creating small, conceptual blocks which look a lot like *inline* functions.  These blocks retain flexibility as long as possible by not prematurely abstracting to functions or other types.  Functions are useful but creating them when it is unnecessary is almost as bad as not creating them when they are necessary
  - This is the idea behind the *local scope* (scoped timer) pattern.  Each local scope has some set of inputs available in the outer scope but the output is clearly defined above the timer and the value is clearly stated in the timer.  These locally scoped code blocks are easy to reason about and also often have the property of being able to be easily moved to a new function if they need to be shared
- Action: Use local scoping (scoped timers) to organize code conceptually

**Write Pure Functions and Logically Separate Them From Impure Functions**

- Definition: A function is pure if it:

  - Doesn't access outside state
  - Doesn't call functions which access outside state (etc.)
  - Doesn't cause any side-effects
  - Doesn't call functions which cause side-effects (etc.)

  That means what is returned by the function is determined exclusively by the function's inputs -- its parameters. Functions with this property are called deterministic because the outcome can be determined ahead of time if given only the values passed to the function as parameters

- Reasoning:
  - Purity is useful because it allows a function to be clearly reasoned about. Pure functions can be selected and assembled in potentially non-obvious ways with confidence
    - Pure functions never have dependencies on one another
    - Pure function calls can be reordered if there are no data dependencies
    - Pure functions can always be called parallel
  - Pure functions, mainly because of their lack of dependencies and side effects, have a much lower incidence of bugs on average.  Separating them from impure functions makes it easier to track down technical debt

- Action:
  - Write pure functions in such a way as they are easily recognizable as being pure. Make them static as a first step
    - A static function only prevents itself from accessing instance state on the class, it doesn't stop code from being written which accesses outside state. If state must be accessed or modified (like fetching inputs or fetching configuration), make it obvious this is happening and isolate it to the highest layers of the solution
  - Configuration should be accessed as high up in the process as possible. From there they can be passed to relevant functions. This will allow the lower-level logic layer to be composed of mostly 'pure' functions.  Inputs should be fetched in the outermost layers of the program and passed inwards

**Use the Most Restrictive Data Structures Available**

* Reasoning: Restrictive types can communicate intent and also prevent bugs.  For example:
  * If a unique set of values is needed, they could be stored in a `List<T>`.  A List is not guaranteed to be unique though, so seeing a List doesn't communicate anything concrete about the values in the List.  A List also will not prevent the accidental addition of duplicate values.  However, there is a collection called a `HashSet<T> ` which does guarantee the contents will be unique and will prevent duplicates from being added
* Action: Be aware of what types are available to you and use the most specific, restrictive types possible

**Avoid Mutating / Modifying Collections and Data**

- Reasoning:
  - When debugging it is useful to know data won't ever be changed. This makes it much easier to understand the state of the application at any given position in the code because assignment can be seen as an absolute
  - Mutating data or collections may indicate poorly organized and structured code as non-obvious changes, drops, or additions of data are bug hot-spots
- Action:
  - All collection and data mutations should be avoided.  Create new collections or data models instead.  If absolutely necessary, mutations should be deliberate, obvious, and isolated from other code
  - Use const, read-only, and immutable types by default
  - Use the restrictive parameter modifier `in` by default on all function parameters to signify the parameter cannot be modified

**Minimize Cyclomatic Complexity**

- Definition: Cyclomatic complexity is defined as the number of linearly independent paths through a program.  Conditional logic most often creates these paths
- Reasoning:
  - Most `if` statements, regardless of whether or not they have an accompanying `else`, create at least two paths through the solution.  From a testing perspective, code with conditionals requires more testing to cover all the potential ways that code can execute.  This is called having *branch coverage*
  - The more cyclomatic complexity, the more potential error states there are in the program.  The more potential error states, the easier it is to introduce defects.  Conditionals can often be avoided or elevated to a higher level of abstraction by breaking functions apart along conditional lines or by making the conditional unnecessary through reformulation of the original problem.
    - A more advanced technique is to create and use very specific types to avoid unnecessary checks.  For example, a HashSet implies uniqueness by definition so no conditions or logic is needed to verify this.  The creation of types is a core concept in functional programming
- Action: Avoid cyclomatic complexity by minimizing branching whenever possible.  If branching cannot be avoided consider where the branching should occur: decisions should be forced to the outer-most layers of the solution.  If a conditional cannot be eliminated attempt to elevate it by breaking apart the function.  If it doesn't make sense to elevate the conditional then ensure it is as clear and isolated as possible

**Organize Code by Proximity to Caller**

- Reasoning: Code can be more easily understood when its position in the solution informs the context of its surroundings. This property is useful when useful when navigating through the solution because the location itself gives the developer information based purely on project and directory structure
- Action
  - Place classes and functions in logical layers closest to their use. If functions are used across projects put the logic in the closest appropriate shared location
  - Do not place anything other than constructors in data models. Other functionality should be organized by proximity, not by class affiliation

**Declare Types Explicitly**

- Reasoning: Types clearly signal significant information about a declaration or assignment. After taking a look at its signature, scanning a function for what types are used is a quick way to get an idea of what a function does and how complex it is. To get this information using `var`, the developer has to infer the type by mentally evaluating the assignment. This pushes the mental effort onto other developers in the future. Convenience and brevity do not trump clarity
- Action: Only use var when ‘newing’ up objects where the type is explicitly specified.  Types should be obvious at all times

**Write Descriptive Variable and Function Names**

- Reasoning: Developers see and read variable and function names when trying to understand what a block of code does. If the names describe what they are or what they do, the developer will not have to navigate to the definition of the type to get an idea of what it is or what it does. Good names eliminate a layer of indirection
- Action: Give variables, functions, and classes names which express their intent

**Avoid Comments Unless Documenting Unintuitive Code**

- Reasoning: Most code can be written clearly, eliminating the need for comments. However, sometimes a developer can't get convey their intent with simple code. This can happen if the business requirements are especially counter-intuitive or if the code is especially complex. In these cases comments may be useful to more precisely explain the context and intent of the code
- Action: Only write comments if the code can not be written in such a way that it is self-explanatory. Most logic can be written clearly if the developer is experienced at problem decomposition

**Write Parameter Validations for Functions**

- Reasoning: Parameter validations are assertions about the acceptable states of the inputs for the function to operate successfully. They are useful for catching misalignments in intent between the developer and the user before entering more complex sequences of logic.  This proactively addresses error states and helps narrow down where a bug is occurring by saying: an exception may have been thrown in this function but it is the caller's mistake
- Action: Validate assumptions about a function's inputs using a parameter validation region at the top

**Flatten the Call Stack**

- Reasoning: Deep, cross-cutting call stacks are difficult to debug and understand due to *indirection*.  This occurs when code is not deliberately organized, pruned, and layered.  It manifests in the form of excessive classes and functions and generally poor isolation
- Action: When adding a layer to the solution (depth to the call stack), the layer should be deliberate and well defined, otherwise, avoid layering where possible.  Move decisions as high up in the call stack as possible and avoiding creating unnecessary functions and classes.  Generally speaking, the deeper into the call stack, the more pure and direct the functions should be

**Use Consistent Styling**

- Reasoning: Consistently styled code is easier to read and understand than inconsistently styled code
- Action:
  - Use consistent indentation
  - Conditionals and loops always have braces, even if they are only one statement long
  - Conditionals explicitly cover cases.  For example, `if` statements should have an accompanying `else`, `switch` statements should have a `default` case.  Branching should never be non-obvious.  If the code appears cluttered with too many `else`, the code likely needs to be restructured.  Hiding the complexity hides the problem

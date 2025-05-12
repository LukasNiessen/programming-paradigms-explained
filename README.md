# Programming Paradigms: What We've Learned Not to Do

I want to present a rather untypical view of what programming paradigms are. So far, we’ve got three major paradigms:

1. Structured Programming,
2. Object-Oriented Programming (OOP),
3. Functional Programming.

And no, there probably won’t be a fourth. Here’s why.

Programming Paradigms are fundamental ways of structuring code. They tell you what structures to use and, more importantly, **what to avoid**. The paradigms do not create new power but actually limit our power. They impose rules on how to write code.

Let's walk through it.

## Structured Programming

In the early days of programming, Edsger Dijkstra recognized a fundamental problem: programming is hard, and programmers don't do it very well. Programs would grow in complexity in become a big mess, impossible to manage.

So he proposed applying the mathematical discipline of proof to programming. This basically means:

1. Start with small units that you can prove to be correct.
2. Use these units to glue together a bigger unit. Since the small units are proven correct, the bigger unit is correct too (if done right).

Similar to what we nowadays would say DRY (don't repeat yourself) would be in the sense of creating functions and moduralizing your code - but with proof.

Dijkstra discovered that certain uses of `goto` statements makes this decomposition very difficult. Other uses of `goto`, however, did not. And the latter mapped to simpler selection and iteration control structures like `if/then/else` and `do/while`.

So he proposed to remove `goto` entirely and allow only `if/then/else` and `do/while`. **This is structured programming.**

That's really all it is. And he was right, so his proposal "won" over time. The actual proofs never came of course, but his proposal of what we now call _structured programming_ succeeded.

### In Short

No `goto`, only `if/then/else` and `do/while` = Structured Programming

So yes, we did not give new power to the devs, but we removed power from the devs.

## Object-Oriented Programming (OOP)

OOP is basically just moving the function call stack frame to a heap.

By this, local variables declared by a function can exist long after the function returned. The function became a constructor for a class, the local variables became instance variables, and the nested functions became methods.

This is OOP.

Now, OOP is often associated with "modeling the real world" or the trio of encapsulation, inheritance, and polymorphism, but all of that was possible before. The biggest power of OOP is arguably polymorphism. It allows dependency version, plugin architecture and more. OOP did not invent it though, as we will see in a second, it just made it safe and easy by restricting the devs how to do it.

### Polymorphism in C

As promised, here an example of how polymorphism was achieved before OOP was a thing. C programmers used techniques like function pointers to achieve similar results. Here a simplified example.

Scenario: we want to process different kinds of data packets received over a network. Each packet type requires a specific processing function, but we want a generic way to handle any incoming packet.

```C
// Define the function pointer type for processing any packet
typedef void (_process_func_ptr)(void_ packet_data);
```

```C
// Generic header includes a pointer to the specific processor
typedef struct {
    int packet_type;
    int packet_length;
    process_func_ptr process; // Pointer to the specific function
    void* data; // Pointer to the actual packet data
} GenericPacket;
```

When we receive and identify a specific packet type, say an AuthPacket, we would create a GenericPacket instance and set its process pointer to the address of the process_auth function, and data to point to the actual AuthPacket data:

```C
// Specific packet data structure
typedef struct { ... authentication fields... } AuthPacketData;

// Specific processing function
void process_auth(void* packet_data) {
    AuthPacketData* auth_data = (AuthPacketData\*)packet_data;
    // ... process authentication data ...
    printf("Processing Auth Packet\n");
}

// ... elsewhere, when an auth packet arrives ...
AuthPacketData specific_auth_data; // Assume this is filled
GenericPacket incoming_packet;
incoming_packet.packet_type = AUTH_TYPE;
incoming_packet.packet_length = sizeof(AuthPacketData);
incoming_packet.process = process_auth; // Point to the correct function
incoming_packet.data = &specific_auth_data;
```

Now, a generic handling loop could simply call the function pointer stored within the GenericPacket:

```C
void handle_incoming(GenericPacket\* packet) {
    // Polymorphic call: executes the function pointed to by 'process'
    packet->process(packet->data);
}

// ... calling the generic handler ...
handle_incoming(&incoming_packet); // This will call process_auth
```

If the next packet would be a DataPacket, we'd initialize a GenericPacket with its process pointer set to process_data, and handle_incoming would execute process_data instead, despite the call looking identical (`packet->process(packet->data)`). The behavior changes based on the function pointer assigned, which depends on the type of packet being handled.

This way of achieving polymorphic behavior is also used for IO device independence and many other things.

### Why OO is still a Benefit?

While C for example can achieve polymorphism, it requires careful manual setup and you need to adherence to conventions. It's error-prone.

OOP languages like Java or C# didn't invent polymorphism, but they formalized and automated this pattern. Features like virtual functions, inheritance, and interfaces handle the underlying function pointer management (like vtables) automatically. So all the aforementioned negatives are gone. You even get type safety.

### In Short

OOP did not invent polymorphism (or inheritance or encapsulation). It just created an easy and safe way for us to do it and restricts devs to use that way. So again, devs did not gain new power by OOP. Their power was restricted by OOP.

## Functional Programming (FP)

FP is all about immutability immutability. You can not change the value of a variable. Ever. So state isn't modified; new state is created.

Think about it: What causes most concurrency bugs? Race conditions, deadlocks, concurrent update issues? They all stem from multiple threads trying to change the same piece of data at the same time.

If data never changes, those problems vanish. And this is what FP is about.

### Is Pure Immutability Practical?

There are some purely functional languages like Haskell and Lisp, but most languages now are not purely functional. They just incorporate FP ideas, for example:

- Java has final variables and immutable record types,
- TypeScript: readonly modifiers, strict null checks,
- Rust: Variables immutable by default (let), requires mut for mutability,
- Kotlin has val (immutable) vs. var (mutable) and immutable collections by default.

### Architectural Impact

Immutability makes state much easier for the reasons mentioned. Patterns like Event Sourcing, where you store a sequence of events (immutable facts) rather than mutable state, are directly inspired by FP principles.

### In Short

In FP, you cannot change the value of a variable. Again, the developer is being restricted.

## Summary

The pattern is clear. Programming paradigms restrict devs:

- **Structured**: Took away `goto`.
- **OOP**: Took away raw function pointers.
- **Functional**: Took away unrestricted assignment.

Paradigms tell us what not to do. Or differently put, we've learned over the last 50 years that programming freedom can be dangerous. Constraints make us build better systems.

So back to my original claim that there will be no fourth. What more do you want to take away? Also, all these paradigms were discovered between 1950 and 1970. So probably we will not see a fourth one.

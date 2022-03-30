# ECS-Based Networking Is the Next Step in Game Networking Modularization

For long, people have networked games in an explicit way. Inventing predefined messages (say `A`, `B` and `C`) and sending them between the client and server. In a similar way, the exchange of information on website used to be concentrated around the `REST` paradigm. That is, making endpoints for specific functionalities and working around that. After that, `GraphQL` came, which replaced the `API` paradigm is a custom language that would make the exchange of data more modular.

According to Wikipedia, ECS `follows the principle of composition over inheritance, meaning that every entity is defined not by a "type", but by the components that are associated with it. The design of how components relate to entities depend upon the Entity Component System being used`. Like the transition from `GraphQL` came from `REST`, `ECS` came from the `Unity-like GameObject-Component` systems, which them came from the `Actor systems` from the 90s and early 2000s.

Everything is moving towards modularization.

What if networking can be more modularized in the same way? `ECS` splits things into `data (entities and components)` and `behavior (systems)`. Clients and servers can communicate by interchanging components, which they can share merely by using the same `ECS` system.

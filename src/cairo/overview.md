> You should have a good understanding of Cairo before proceeding. If you're unfamiliar with Cairo, we recommend you read the [Cairo documentation](https://book.cairo-lang.org/title-page.html) first.

## A New Approach to Game Development

Dojo provides an advanced abstraction layer over Cairo, mirroring React's relationship with JavaScript. Its specialized architecture simplifies game design and development. By leveraging Dojo, developers can use succinct commands that transform into comprehensive queries at compile time. This chapter delves deeper into Dojo's unique architecture.

#### Delving into the Architecture

Dojo efficiently encapsulates boilerplate contracts within the compiler, letting developers concentrate on the distinct aspects of their game or app.

Consider this as the most basic Dojo world setup:

```rust,ignore
- src
  - main.cairo
  - lib.cairo
- Scarb.toml
```

While seemingly simple, behind the scenes Dojo generates foundational contracts, setting the stage for you to focus purely on data and logic. 

Lets take a look at the `main.cairo`:

```rust,ignore
use starknet::ContractAddress;

// dojo data models
#[derive(Model, Copy, Drop, Serde)]
struct Position {
    #[key] // primary key
    player: ContractAddress,
    vec: Vec2,
}

// regular cairo struct
#[derive(Copy, Drop, Serde, Introspect)]
struct Vec2 {
    x: u32,
    y: u32
}

// interface
#[starknet::interface]
trait IPlayerActions<TContractState> {
    fn spawn(self: @TContractState);
}

// contract
#[dojo::contract]
mod player_actions {
    use starknet::{ContractAddress, get_caller_address};
    use super::{Position, Vec2};
    use super::IPlayerActions;

    #[external(v0)]
    impl PlayerActionsImpl of IPlayerActions<ContractState> {
        // 
        // NOTICE: we pass the world dispatcher as an argument to every function. 
        // This is how we interact with the world contract.
        //
        fn spawn(self: @ContractState) {
            // Access the world dispatcher for reading.
            let world = self.world_dispatcher.read();

            // get player address
            let player = get_caller_address();

            // dojo command - get player position
            let position = get!(world, player, (Position));

            // dojo command - set player position
            set!(world, (Position { player, vec: Vec2 { x: 10, y: 10 } }));
        }
    }
}
```

### Breakdown

This just a regular Cairo contract, with some specifics.

#### `Position` struct

In a Dojo world, state is defined using models. These are structs marked with the `#[derive(Model)]` attribute, functioning similarly to a keypair store. The primary key for a model is indicated using the `#[key]` attribute; for instance, the `player` field serves as the primary key in this context.

Read more about models [here](./models.md).

#### `spawn` function - a dojo system

In the `spawn` function, take note of the second parameter: the `IWorldDispatcher` interface. This provides a gateway to the world contract. By integrating it into the function, it enables the `get!` and `set!` macros to interface directly with the world contract. 

Commands, a significant innovation in Dojo, are further explored [here](./commands.md).

### High level transaction flow of a world

To call a Dojo world you invoke a system, which then calls the [world](./world.md) and does the necessary state changes.

![Dojo World](../images/world_flow.png)

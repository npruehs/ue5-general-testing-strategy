# Towards a general testing strategy for Unreal Engine games

As I've been comparing the classical software development industry to the games industry, I've been finding that both have reached different levels of professionalization. To give a popular example, test automation does not seem to be quite as common among game developers as it is among other software developers. Especially young and small companies often seem to discard the idea of automatically testing games as "too difficult" or "not worth the investment". In this article, I propose a general testing strategy for game development. This strategy is targeted at [Unreal Engine](https://www.unrealengine.com/en-US/) games, but clearly, it can be applied to other games and game engines as well.

## Application Architecture

Most games, especially real-time games and games created using real-time engines, seem to be hard to test. The main reason for this is a lack of [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns), as recommended for most other modern business applications. Even if developers introduce layers of abstractions instead of just adding code directly to actors or components in engines like Unreal or Unity, they rarely keep the ability to test these layers in mind.

Consider the following structure, which has been common among modern web application backends:

TODO: Image

In this structure, HTTP REST requests are sent to the _endpoint_ layer, which usually just defines the endpoints themselves, probably with access control. The endpoint layer doesn't contain any other business logic, but delegates any invocations to the _service layer_ immediately.

The service layer is the heart of the application. It contains many services, usually at least one per feature, and implements the business logic. To further improve on separation of concerns (and the ability to test the application), some applications use dedicated _mapper_ classes for mapping between data transfer objects (DTOs) received from the client and other data representations used by the application (e.g. database entities).

Finally, reading from/writing to a database or a similar persistent storage is handled by the _data access object (DAO)_ layer. The DAO layer minimizes the amount of business logic again, focusing on the persistence side of things.

Having this clear structure makes it easier for developers to understand the application and its individual components, and allows for easier testing. All dependencies can easily be mocked away to test a specific component, assuming other components work as expected. Then, a small set of integration tests can make sure that the full stack of layers is actually working as well.

## Game Architecture

The idea of using a similar modular approach to game development isn't a new one. A popular take on a general architecture for games called _component-based entity systems_ has been proposed by people like Adam Martin [[1]](#references) or Scott Bilas [[2]](#references) before. Here, game entities are entirely made up of _components_, which contain data, only. All game logic is implemented in _systems_, which operate on the components, but in turn don't contain any data themselves. This approach also allows for easy serialization of the game state, e.g. to disk for savegames, or to a database in case of an online game.

## Unreal Game Architecture

The goal of this article is to come up with an architecture for Unreal Engine games that leverages the advantages of a clean, layered structure for easier testing, while preserving the many powerful features provided by the engine itself. More specifically, developers should still be able to work with blueprints or automatically replicate game state over the network, for instance. Traditional approaches to build well-structured games with Unreal Engines often discard many of these powerful features, which I want to avoid. Also, the goal is to be able to use Unreal Engine without having to modify its source code, as modifying the sources and compiling the engine is not feasible for every team.

### Actor Layer

Similar that how component-based entity systems work, we'll be using Unreal Engine [actor components](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Components/) attached to [actors](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/) to hold all of our game data. These components should not contain any game logic, but just `UPROPERTY`s that represent the current state of the game. Using actors and components allows us to expose that game data to [blueprints](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/Blueprints/), which still can act as templates for entities in the game (e.g. an actor blueprint for each enemy type). Moreover, we'll be able to use Unreal's built-in [replication](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Networking/Actors/) system for transferring game data over the network.

TODO: Image

### System Layer

As actors don't contain any game logic, they should leave changing the game state to the system layer. Similar to the endpoint layer mentioned in the [Application Architecture](#application-architecture]) section, actors can listen to replicated property changes using `ReplicatedUsing`, but should delegate all method invocations to the system layer immediately.

These systems don't have to be actors; they don't even need to be `UObject`s, which makes them really easy to construct and test. Clearly, systems can become quite complex, with the need to split them up further into multiple classes (e.g. navigation systems). 

Now, systems are allowed to modify the game state by changing the properties of actor components.

Ideally, systems should be as isolated as possible. However, that might not always be possible, so it is up the specific game implementation whether systems should directly invoke other system's methods (classical approach), rely on Unreal Engine delegates and events (more decoupled), or even use some kind of generic event bus for inter-system communication (lowest coupling).

## Future Work

This article is still in a very early state. Here's what I'm planning to do during the next months to further refine the ideas presented herein:

* Explain how all of this relates to existing engine functionality (e.g. game framework, CharacterMovementComponent)
* Provide a detailed example based on some popular Unreal Engine template (e.g. ShooterGame)
* Check for other important Unreal Engine aspects I need to consider as well
* Evaluate the possibility of having system blueprints to expose game functionality to designers, and how to test these
* Listen to your feedback!

## Contributing

There are several ways you can contribute to the idea of a general testing strategy for Unreal Engine games.

First, you can always [open an issue](https://github.com/npruehs/ue5-general-testing-strategy/issues) to share your thoughts, especially if you've already made some experience with a similar approach. As this repository doesn't contain (much) code to contribute to, consider the issue tracker more like a classical forum for discussing different aspects of this article.

If you feel like you'd want to contribute even more, and you've confirmed your ideas in a small discussion in the issue tracker, you can fork the repository, check out the source of this article, suggest your changes and finally [open a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request).

Note that this article is distributed under the MIT License. So will be your modifications.

## License

Towards a general testing strategy for Unreal Engine games is released under the [MIT License](https://github.com/npruehs/ue5-general-testing-strategy/blob/main/LICENSE).

## References

1. Martin, Adam. Entity Systems are the future of MMOG development. http://t-machine.org/index.php/2007/09/03/entity-systems-are-the-future-of-mmog-development-part-1/
1. Bilas, Scott. A Data-Driven Game Object System. https://www.gamedevs.org/uploads/data-driven-game-object-system.pdf

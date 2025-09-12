---
layout: post 
title: "Platformer Game in C++ Part 3: Implementing Collision Detection"
categories: tech 
tags: c++ platformer gamedev 
author: rishabh 
share: true 
image: "/assets/platformer_part_1.png"
comments: true

---

In the previous part, we implemented the `InputHandler` class, which allows us to handle keyboard and mouse inputs effectively. With this system in place, we are now ready to detect and resolve collisions in our game. 

### Overview of Collision Detection

Collision detection is a critical feature in any platformer game. It ensures that entities interact with the environment and each other in a realistic way. For example, the player should not pass through walls or fall through the ground. In our architecture, collision detection will be implemented in the `Entity` class. This approach allows us to enable or disable collision detection for any entity in the game.

### Design Philosophy

The collision detection system will be flexible and extensible:
- **Enable/Disable Collisions**: Certain entities, such as decorative objects, may not require collision detection. We will add a flag to enable or disable collisions for each entity.
- **Inheritance**: By implementing collision detection in the `Entity` class, all derived classes (e.g., `Player`, `Enemy`) will automatically inherit this functionality.

This design ensures that collision detection is both efficient and easy to manage across different types of entities.

So let's get started. The header file for the `Entity` Class looks like this with the functions listed:
```cpp 
#pragma once 
#include <iostream> 
#include <SFML/Graphics.hpp>


class Entity : public sf::Drawable {
    protected: 

        // base shape params
        sf::Vector2f position;
        sf::Vector2f velocity{0.0f,0.0f};
        sf::Vector2f size; 
        sf::Color color{sf::Color::Red};

        // the base shape 
        sf::RectangleShape entity;
        
        
        // physics constants 
        const float GRAVITY{30.0f};
        
        // the draw method from the sf::Drawable class 
        
        private: 
        // overiding to convert this entity into an drawable object on the sfml window
        void draw(sf::RenderTarget& target, sf::RenderStates states) const override {
            target.draw(entity, states);
        }
        
        public: 
        
        // flag to turn on physics on this entity
        bool physical{false};

        // Collision Constants 
        // this flag coulb be used for entities like sign boards etc to switch of collision detection
        bool CollisionActive{true};

        Entity();


        // setter and getter functions 
        void setPosition(sf::Vector2f pos);
        sf::Vector2f getPosition();
        void setSize(sf::Vector2f newSize);
        sf::Vector2f getSize();

        // update function to update params each frame 
        // here we will have all physics being updated if it is there. 
        virtual void update(float deltaTime);

        // physics function 
        // void applyPhysics();

        // sprite functions 
        // to be implemented after 
        // void bindSprite()


        // collision detection functions
        bool isColliding(Entity& other);
        void resolveCollision(Entity& other);
};

```

so here we have the `CollisionActive` flag in place and the functions `isColliding` and `resolveCollision` to handle the collision part and a few new setters and getters have also been added.

### Refining the Entity Class for Collision Detection

The `Entity` class has been updated to include collision detection functionality. This ensures that all derived classes, such as `Player` and `Enemy`, can inherit and utilize these features. Below is a detailed explanation of the new additions:

#### CollisionActive Flag

The `CollisionActive` flag determines whether collision detection is enabled for an entity. This is useful for entities like decorative objects (e.g., signboards) that do not require collision handling.

```cpp
bool CollisionActive{true};
```

- **Default State**: The flag is set to `true` by default, enabling collision detection for most entities.
- **Customizability**: Developers can toggle this flag to disable collision detection for specific entities.

#### Setter and Getter Functions

New setter and getter functions have been added to manage the position and size of the entity. These functions encapsulate the `position` and `size` variables, ensuring controlled access.

```cpp
void setPosition(sf::Vector2f pos);
sf::Vector2f getPosition();
void setSize(sf::Vector2f newSize);
sf::Vector2f getSize();
```

- **Position Management**: The `setPosition` and `getPosition` methods allow the entity’s position to be updated or retrieved.
- **Size Management**: The `setSize` and `getSize` methods provide similar functionality for the entity’s size.

#### Collision Detection Functions

The `isColliding` and `resolveCollision` functions handle collision detection and resolution between entities.

```cpp
bool isColliding(Entity& other);
void resolveCollision(Entity& other);
```

- **`isColliding`**: This function checks whether the current entity is colliding with another entity.
- **`resolveCollision`**: This function resolves the collision by adjusting the positions or velocities of the entities involved.

#### Integration with the Game

With these additions, the `Entity` class now serves as a robust base for all game objects. Derived classes can override or extend these functions to implement specific collision behaviors. For example, the `Player` class can use these functions to interact with the environment and other entities.

### Next Steps

In the next section, we will implement the `isColliding` and `resolveCollision` functions. These will use the `position` and `size` variables to detect and resolve collisions based on bounding box overlap.
---
layout: post 
title: "Platformer Game in C++ Part 3: Implementing Collision Detection and Adding Graphics To The Game"
categories: tech 
tags: c++ platformer gamedev 
author: rishabh 
share: true 
image: "/assets/platformer_part_3.png"
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
- **Customizability**: We can toggle this flag to disable collision detection for specific entities.

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

in this game all objects will be simple rectangular shapes, so we will be using bounding boxes based collision resolution method.
You can read more about it here https://garagefarm.net/blog/collision-detection-in-2d-and-3d-concepts-and-techniques.

after implementing both the functions our `Entity` Class code looks like this:

```cpp 


#include "entity.hpp"



Entity::Entity(){
    // Set default position and size
    position = sf::Vector2f(100.0f, 100.0f);
    size = sf::Vector2f(50.0f, 50.0f);
    
    // Configure the rectangle shape
    entity.setPosition(position);
    entity.setSize(size);
    entity.setFillColor(color);
}

// implement the setters and getter 

void Entity::setPosition(sf::Vector2f pos) {
    position = pos;
    // entity.setPosition(position);
}

sf::Vector2f Entity::getPosition() {
    return position;
}

void Entity::setSize(sf::Vector2f s) {
    size = s;
    entity.setSize(size);
}

sf::Vector2f Entity::getSize() {
    return size;
}


bool Entity::isColliding(Entity& other){

    if (!CollisionActive) return false;

    sf::FloatRect thisBounds = entity.getGlobalBounds();
    sf::FloatRect otherBounds = other.entity.getGlobalBounds();
    return thisBounds.intersects(otherBounds);
}

void Entity::resolveCollision(Entity& other){

    if (!CollisionActive) return;


    sf::FloatRect thisBounds = entity.getGlobalBounds();
    sf::FloatRect otherBounds = other.entity.getGlobalBounds();

    if (thisBounds.intersects(otherBounds)) {
        float dx = (thisBounds.left + thisBounds.width / 2) - (otherBounds.left + otherBounds.width / 2);
        float dy = (thisBounds.top + thisBounds.height / 2) - (otherBounds.top + otherBounds.height / 2);

        float overlapX = (thisBounds.width / 2 + otherBounds.width / 2) - std::abs(dx);
        float overlapY = (thisBounds.height / 2 + otherBounds.height / 2) - std::abs(dy);

        if (overlapX < overlapY) {
            if (dx > 0)
                position.x += overlapX;
            else
                position.x -= overlapX;
            velocity.x = 0;
        } else {
            if (dy > 0)
                position.y += overlapY;
            else
                position.y -= overlapY;
            velocity.y = 0;
        }
        entity.setPosition(position);
    }
}



void Entity::update(float deltaTime){
    // here update all the params that you want to update 
    // ex: physics params, the position of the entity 

    // keep updating the position 
    position.x += velocity.x;
    position.y += velocity.y;
    
    // apply gravity 
    if (physical){
        // apply the acceleration 
        velocity.y += GRAVITY * deltaTime;

    }

    // set the position for the entity
    entity.setPosition(position);
}

```




#### **Testing the Collision System**

To verify that the collision detection system works as expected, we can create a platform entity and a player entity. By placing them in the game world, we can observe how the player interacts with the platform. Below is an example setup:

```cpp
Entity platform;
platform.setPosition(sf::Vector2f(200.0f, 400.0f));
platform.setSize(sf::Vector2f(300.0f, 50.0f));
platform.physical = false; // The platform does not move
platform.CollisionActive = true; // Enable collision detection for the platform

Entity player;
player.setPosition(sf::Vector2f(250.0f, 300.0f));
player.setSize(sf::Vector2f(50.0f, 50.0f));
player.physical = true; // The player is affected by gravity
player.CollisionActive = true; // Enable collision detection for the player

// In the game loop
if (player.isColliding(platform)) {
    player.resolveCollision(platform);
}
```

- **Platform Setup**: The platform is stationary and has collision detection enabled.
- **Player Setup**: The player is affected by gravity and can collide with the platform.
- **Collision Handling**: When the player collides with the platform, the `resolveCollision` function ensures that the player stops moving and does not pass through the platform.

This setup ensures that the player will stop moving when it collides with the platform, and gravity will no longer pull the player through the platform.

#### **Next Steps**

In the upcoming posts, we will focus on adding graphics to our game to enhance its visual appeal. Specifically, we will:

- Implement a `SpriteManager` class to manage all the graphics in the game.
- Use sprites to replace the basic shapes currently used for entities.
- Add animations and textures to make the game more engaging.

Stay tuned as we continue to build and refine our platformer game!
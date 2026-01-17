---
layout: post
title: "Platformer Game in C++ Part 1: Implementing Entity and Player Class"
date: 2025-09-10T15:48:00.000+05:30
categories: "tech "
tags:
  - platformer
  - gamedev
  - c++
author: rishabh
share: true
---
## Introduction

In this series, I will document the creation of my first game: a 2D platformer written in C++ using  [SFML](https://www.sfml-dev.org/) .
I will assume here that you know how to setup the environment and compile the files to run.

## What We'll Build in Part 1

In this post, we'll implement:

* A base `Entity` class to encapsulate common properties (position, velocity, rendering, collision).
* A derived `Player` class to handle input and movement.

## Architecture Overview

Our game objects will inherit from the `Entity` base class, allowing us to enable or disable features (physics, collisions) via compile-time flags.

Here is the overview of the class hierarchy:

![Architecture Diagram](/assets/flow.png "Class Hierarchy")

## Implementing the `Entity` Class

So let's start implementing the `Entity` Class:
we will start by writing the header file and then implementing all those functions:

```cpp
 // /entities/entity.hpp

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
        const float GRAVITY{9.0f};
        const float VEL_DEL{10.0f};

        // the draw method from the sf::Drawable class 
    
    private: 
        // overiding to convert this entity into an drawable object on the sfml window
        void draw(sf::RenderTarget& target, sf::RenderStates states) const override {
            target.draw(entity, states);
        }

    public: 

        // flag to turn on physics on this entity
        bool physical{false};

        Entity();

        // update function to update params each frame 
        // here we will have all physics being updated if it is there. 
        virtual void update(float deltaTime);

};
```

### Understanding the `Entity` Class

Our `Entity` class serves as the base for all drawable, movable objects in the game. Here's a detailed breakdown:

* **Inheritance from `sf::Drawable`**
  We inherit from [sf::Drawable](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Drawable.php) so SFML can render our object via the overridden `draw()` method. This gets called automatically when we use `window.draw(entityInstance)` on a [sf::RenderWindow](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1RenderWindow.php).
* **Core Member Variables**  

  * `sf::Vector2f position` and `velocity` ([docs](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Vector2.php)): track where the entity is and how fast it's moving.  
  * `sf::Vector2f size`: defines the width and height of the rectangle.  
  * `sf::Color color` ([docs](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Color.php)): default fill color for our shape.  
  * `sf::RectangleShape entity` ([docs](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1RectangleShape.php)): the visual representation—initialized with `size`, `color`, and `position` in the constructor.
* **Physics Constants**  

  * `const float GRAVITY`: the acceleration applied downward each frame (pixels/s²).  
  * `const float VEL_DEL`: a modifier for velocity adjustments (e.g., jump impulse).
* **`physical` Flag**\
  A boolean that toggles physics updates. If `true`, the `update()` method applies gravity and movement; if `false`, the entity remains static.
* **Constructor (`Entity()`)**\
  In the implementation (`entity.cpp`), the constructor should:

  1. Set `entity.setSize(size);` and `entity.setFillColor(color);`  
  2. Initialize `position` (e.g., `(0,0)`) and call `entity.setPosition(position);`  
  3. Optionally configure origin or other shape properties.
* **`update(float deltaTime)`**\
  Called once per frame with `deltaTime` (time elapsed since last frame), ensuring frame-rate-independent behavior:

  1. If `physical` is enabled, apply gravity: `velocity.y += GRAVITY * deltaTime;`  
  2. Update position: `position += velocity * deltaTime;`  
  3. Sync the shape: `entity.setPosition(position);`
* **`draw(sf::RenderTarget&, sf::RenderStates)`**\
    This private override simply tells SMFL window how to draw our Entity on the window:   

  ```cpp
  void draw(sf::RenderTarget& target, sf::RenderStates states) const override {
      target.draw(entity, states);
  }
  ```

  By centralizing rendering, movement, and physics in one base class, derived classes (like `Player` or `Enemy`) can focus on their own behaviors—input handling, AI, animations—without rewriting common logic.

Next, we'll implement these methods in `entity.cpp`.

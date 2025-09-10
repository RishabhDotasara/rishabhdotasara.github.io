---
layout: post
title: "Platformer Game in C++ Part 1: Implementing Entity and Player Class"
date: 2025-09-10T15:48:00.000+05:30
categories: "tech "
tags:
  - platformer
author: rishabh
share: true
---
## Introduction





Here is the overview of the class hierarchy:
![Entity Class Diagram]({{ site.baseurl }}/assets/images/platformer_images/flow.png)



```cpp/entities/entity.hpp
#pragma once #include <iostream> #include <SFML/Graphics.hpp>
class Entity : public sf::Drawable {    protected: 
        // base shape params        sf::Vector2f position;        sf::Vector2f velocity{0.0f,0.0f};        sf::Vector2f size;         sf::Color color{sf::Color::Red};
        // the base shape         sf::RectangleShape entity;

};
```

**\## Understanding the Entity Class**
Our \



---
layout: post 
title: "Platformer Game in C++ Part 4: Implementing SpriteManager Class"
categories: tech 
tags: c++ platformer gamedev 
author: rishabh 
share: true 
image: "/assets/platformer_part_4.png"
comments: true
--- 

In the previous post, we successfully implemented collision detection and resolution. In this post, we will introduce the `SpriteManager` class, which will be responsible for managing and displaying graphics in the game.

The folder structure looks like:
```
platformer/
├── src/
│   ├── main.cpp
│   ├── entities/
│   │   ├── entity.hpp
│   │   ├── entity.cpp
│   │   └── player.hpp
│   │   └── player.cpp
│   ├── input/
│   │   ├── input.hpp
│   │   └── input.cpp
│   └── graphics/
│       ├── spriteManager.hpp
│       └── spriteManager.cpp

├── build/
└── .vscode/
    ├── tasks.json
``` 

### Overview

The `SpriteManager` class will handle the loading, storing, and retrieval of sprites. This ensures that graphics are efficiently managed and reused throughout the game. By centralizing sprite management, we can reduce memory usage and improve performance.

Additionally, we will implement several other classes that utilize the `SpriteManager` to create dynamic and visually appealing game scenes. In upcoming posts, we will also introduce a map generation class to procedurally generate levels.

### Architecture

The architecture we will follow is as follows:
- **`SpriteManager` Class**: This class will handle all sprite-related operations, including loading textures, storing them in memory, and providing access to entities using assigned names.
- **Entity Integration**: Each entity in the game will be able to request and use sprites from the `SpriteManager` by referencing a unique name.

This design ensures that graphics are decoupled from game logic, making the codebase more modular and maintainable.

let's get to implementing the `SpriteManager` Class, the header file code goes as follows:
```cpp 
#pragma once
#include "SFML/Graphics.hpp"
#include <iostream>
#include <unordered_map>

class SpriteManager
{
    // so sprites after creation are stored in memory for faster access
    // so we need a map to store out sprites and textures in the memory

private:
    std::unordered_map<std::string, sf::Sprite> spriteCollection;
    std::unordered_map<std::string, sf::Texture> textureCollection;

public:

    // constructor 
    SpriteManager();


    // so we will have functions here to load textures, create sprites
    void loadTexture(std::string path, std::string name, bool repeated = false);

    // this should create and return the sprite so that it can then be used by the entity to get uniqueSprite for each
    // we will also take the textureRect as input to create a sprite from a part of the texture
    sf::Sprite createSprite(std::string textureName, std::string spriteName, sf::IntRect textureRect = sf::IntRect());

    // helper functions 
    bool hasSprite(std::string spriteName);
    sf::Sprite getSprite(std::string spriteName);

    // function to load all the textures 
    void loadAllTextures();

    // we will have the sprite binding functionality implemented in the entity class itself,
    // so that each entity takes care of its sprite and updates its position accordingly.
};

```

and implementing all these functions the code looks like: 
```cpp

#include "spriteManager.hpp"
#include <iostream>



SpriteManager::SpriteManager(){
    // load all the textures here 
    loadAllTextures();
    std::cout << "SpriteManager initialized and all textures loaded into memory." << std::endl;
}



void SpriteManager::loadTexture(std::string path, std::string name, bool repeated)
{
    sf::Texture texture;
    if (!texture.loadFromFile(path))
    {
        std::cout << "Error loading texture from the file! Check if the file " << path << " exists.";
        return;
    }
    texture.setRepeated(repeated);
    textureCollection[name] = texture;
    std::cout << "Texture " << name << " loaded successfully from " << path << std::endl;
}

sf::Sprite SpriteManager::createSprite(std::string textureName, std::string spriteName, sf::IntRect textureRect)
{
    // check if the texture exists in the memory
    if (textureCollection.find(textureName) == textureCollection.end())
    {
        std::cout << "Texture " << textureName << " is not loaded in memory, please load it before creating a sprite with it!";
        return sf::Sprite();
    }

    // check if the sprite with this name already exists, if yes return error 
    if (spriteCollection.find(spriteName) != spriteCollection.end()){
        std::cout << "Sprite with this name already exists, choose another name: NameError: "<< spriteName << "\n" << "Returning the already existing sprite." << std::endl;
        return spriteCollection[spriteName];
    }

    sf::Sprite sprite(textureCollection[textureName]);
    sprite.setTextureRect(textureRect);
    
    spriteCollection[spriteName] = sprite;
    std::cout << "Sprite " << spriteName << " created successfully with texture " << textureName << std::endl;
    return spriteCollection[spriteName];
}


void SpriteManager::loadAllTextures(){
    // load all textures here and create sprites for them which entities can directly refer to 
    
    // load the spritesheet for now to check if it works 
    // load all the character frames first 
    // loadTexture("assets/characters.png", "walk1");
    // createSprite("walk1", "walk1", sf::IntRect(32*0, 32*1, 32, 32));
    // spriteCollection["walk1"].setScale(2.0f,2.0f); // scale the player sprite to make it 64x64
    // loadTexture("assets/characters.png", "walk2");
    // createSprite("walk2", "walk2", sf::IntRect(32*0, 32*1, 32, 32));
    // spriteCollection["walk2"].setScale(2.0f,2.0f); // scale the player sprite to make it 64x64
    // loadTexture("assets/characters.png", "walk3");
    // createSprite("walk3", "walk3", sf::IntRect(32*0, 32*1, 32, 32));
    // spriteCollection["walk3"].setScale(2.0f,2.0f); // scale the player sprite to make it 64x64
    // loadTexture("assets/characters.png", "walk4");
    // createSprite("walk4", "walk4", sf::IntRect(32*0, 32*1, 32, 32));
    // spriteCollection["walk4"].setScale(2.0f,2.0f); // scale the player sprite to make it 64x64


    // loadTexture("assets/dirt.png","dirt");
    // createSprite("dirt", "dirtTile", sf::IntRect(0, 0, textureCollection["dirt"].getSize().x, textureCollection["dirt"].getSize().y));
    // loadTexture("assets/grass.png","grass");
    // createSprite("grass","grassTile",sf::IntRect(0,0,16,16));
    // spriteCollection["grassTile"].setScale(2.0f,2.0f); // scale the grass tile to make it 32x32
    // spriteCollection["dirtTile"].setScale(2.0f,2.0f); // scale the dirt tile to make it 32x32
    // spriteCollection["test"].setScale(2.0f,2.0f); // scale the player sprite to make it 64x64
}

sf::Sprite SpriteManager::getSprite(std::string spriteName){
    if (spriteCollection.find(spriteName) == spriteCollection.end()){
        std::cout << "Sprite with this name does not exist, please check the name: NameError: "<< spriteName << "\n" << "Returning an empty sprite." << std::endl;
        return sf::Sprite();
    }
    return spriteCollection[spriteName];
}
```

### Refining the SpriteManager Class

The `SpriteManager` class is designed to handle all sprite-related operations, including loading textures, creating sprites, and managing them in memory for efficient access. Below is a detailed explanation of each function in the class:

#### **Private Members**

```cpp
std::unordered_map<std::string, sf::Sprite> spriteCollection;
std::unordered_map<std::string, sf::Texture> textureCollection;
```
- **`spriteCollection`**: Stores all created sprites, mapped by their unique names.
- **`textureCollection`**: Stores all loaded textures, mapped by their unique names.

#### **Constructor**

```cpp
SpriteManager::SpriteManager() {
    loadAllTextures();
    std::cout << "SpriteManager initialized and all textures loaded into memory." << std::endl;
}
```
- Automatically loads all textures into memory when the `SpriteManager` is initialized.

#### **`loadTexture` Function**

```cpp
void SpriteManager::loadTexture(std::string path, std::string name, bool repeated) {
    sf::Texture texture;
    if (!texture.loadFromFile(path)) {
        std::cout << "Error loading texture from the file! Check if the file " << path << " exists.";
        return;
    }
    texture.setRepeated(repeated);
    textureCollection[name] = texture;
    std::cout << "Texture " << name << " loaded successfully from " << path << std::endl;
}
```
- **Purpose**: Loads a texture from a file and stores it in `textureCollection`.
- **Parameters**:
  - `path`: Path to the texture file.
  - `name`: Unique name for the texture.
  - `repeated`: Whether the texture should repeat when tiled.
- **Error Handling**: Prints an error message if the texture cannot be loaded.

#### **`createSprite` Function**
Integration with Entities
The SpriteManager class is designed to work seamlessly with the Entity class. Each entity can request a sprite by name and bind it to its position and size. This ensures that graphics are decoupled from game logic, making the code more modular and maintainable.
```cpp
sf::Sprite SpriteManager::createSprite(std::string textureName, std::string spriteName, sf::IntRect textureRect) {
    if (textureCollection.find(textureName) == textureCollection.end()) {
        std::cout << "Texture " << textureName << " is not loaded in memory.";
        return sf::Sprite();
    }
    if (spriteCollection.find(spriteName) != spriteCollection.end()) {
        std::cout << "Sprite with this name already exists: " << spriteName << std::endl;
        return spriteCollection[spriteName];
    }
    sf::Sprite sprite(textureCollection[textureName]);
    sprite.setTextureRect(textureRect);
    spriteCollection[spriteName] = sprite;
    std::cout << "Sprite " << spriteName << " created successfully." << std::endl;
    return spriteCollection[spriteName];
}
```
- **Purpose**: Creates a sprite using a loaded texture and stores it in `spriteCollection`.
- **Parameters**:
  - `textureName`: Name of the texture to use.
  - `spriteName`: Unique name for the sprite.
  - `textureRect`: Rectangle defining the portion of the texture to use.
- **Error Handling**: Checks if the texture exists and if the sprite name is already in use.

#### **`hasSprite` Function**

```cpp
bool SpriteManager::hasSprite(std::string spriteName) {
    return spriteCollection.find(spriteName) != spriteCollection.end();
}
```
- **Purpose**: Checks if a sprite with the given name exists in `spriteCollection`.

#### **`getSprite` Function**

```cpp
sf::Sprite SpriteManager::getSprite(std::string spriteName) {
    if (spriteCollection.find(spriteName) == spriteCollection.end()) {
        std::cout << "Sprite with this name does not exist: " << spriteName << std::endl;
        return sf::Sprite();
    }
    return spriteCollection[spriteName];
}
```
- **Purpose**: Retrieves a sprite by its name.
- **Error Handling**: Prints an error message if the sprite does not exist.

#### **`loadAllTextures` Function**

```cpp
void SpriteManager::loadAllTextures() {
    // Example: Load textures and create sprites
    // loadTexture("assets/characters.png", "character");
    // createSprite("character", "player", sf::IntRect(0, 0, 32, 32));
}
```
- **Purpose**: Preloads all textures and creates default sprites for the game.
- **Example Usage**: Uncomment and customize the code to load specific textures and sprites.

you can have a look at the SFML Docs for sprite [here](https://www.sfml-dev.org/tutorials/3.0/graphics/sprite/) to have a further good understanding of how the code works.

now, the next step is that we will have a function in the entity class which binds the sprite to the entity, what bind here basically means is that the sprite will be following the entity after we bind it to it, sprite's position will be continously set to where the entity is, which will give the illusion that the entity has some skin on it.

so the modified entity class looks like:
```cpp 
#pragma once
#include <iostream>
#include <SFML/Graphics.hpp>
#include "graphics/spriteManager.hpp"

class Entity : public sf::Drawable
{
protected:
    // base shape params
    sf::Vector2f position;
    sf::Vector2f velocity{0.0f, 0.0f};
    sf::Vector2f size;
    sf::Color color{sf::Color::Transparent};

    // the base shape
    sf::RectangleShape entity;

    // physics constants
    const float GRAVITY{980.0f};

    // sprite management for the entity
    sf::Sprite sprite;

private:
    // overiding to convert this entity into an drawable object on the sfml window
    void draw(sf::RenderTarget &target, sf::RenderStates states) const override
    {

        target.draw(entity, states);

        // dont forget to add this else the sprite wont show up
        target.draw(sprite, states);
    }

public:
    // flag to turn on physics on this entity
    bool isOnGround{false};
    bool physical{false};

    // Collision Constants
    // this flag coulb be used for entities like sign boards etc to switch of collision detection
    bool CollisionActive{true};

    Entity();
    Entity(std::string spriteName, SpriteManager &spriteManager);

    // setter and getter functions
    void setPosition(sf::Vector2f pos);
    sf::Vector2f getPosition();
    void setSize(sf::Vector2f newSize);
    sf::Vector2f getSize();

    // update function to update params each frame
    // here we will have all physics being updated if it is there.
    virtual void update(float deltaTime);

    // sprite functions
    void bindSprite(std::string spriteName, SpriteManager &spriteManager);

    // collision detection functions
    bool isColliding(Entity &other);
    void resolveCollision(Entity &other);
};

```

if you look carefully we also made a change in the `draw` function as when we draw this entity we also want to draw thw sprite as well.

now implementing the new function the code goes like: 
```cpp 
#include "entity.hpp"



Entity::Entity(){
    // Set default position and size
    position = sf::Vector2f(100.0f, 100.0f);
    size = sf::Vector2f(32.0f, 32.0f);
    
    // Configure the rectangle shape
    entity.setPosition(position);
    entity.setSize(size);
    
    entity.setFillColor(color);
}

Entity::Entity(std::string spriteName, SpriteManager& spriteManager){
    // Set default position and size
    position = sf::Vector2f(100.0f, 100.0f);
    size = sf::Vector2f(64.0f, 64.0f);

    // Configure the rectangle shape
    entity.setPosition(position);
    entity.setSize(size);
    entity.setFillColor(color);

    // create the sprite and bind it to the texture 
    // scale the sprite to fit the entity size 
    bindSprite(spriteName, spriteManager);
    // sprite.setScale(2.0f, 2.0f);
    

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

void Entity::resolveCollision(Entity& other) {
    if (!CollisionActive) return;

    sf::FloatRect thisBounds = entity.getGlobalBounds();
    sf::FloatRect otherBounds = other.entity.getGlobalBounds();

    if (thisBounds.intersects(otherBounds)) {
        float dx = (thisBounds.left + thisBounds.width / 2) - (otherBounds.left + otherBounds.width / 2);
        float dy = (thisBounds.top + thisBounds.height / 2) - (otherBounds.top + otherBounds.height / 2);

        float overlapX = (thisBounds.width / 2 + otherBounds.width / 2) - std::abs(dx);
        float overlapY = (thisBounds.height / 2 + otherBounds.height / 2) - std::abs(dy);

        // Check if collision is more vertical than horizontal
        if (overlapY < overlapX) {
            if (dy > 0) {
                // Collision from below (hitting ceiling)
                position.y += overlapY;
                velocity.y = 0;
            } else {
                // Collision from above (standing on platform)
                position.y -= overlapY;
                velocity.y = 0;
                isOnGround = true; // Set to true when standing on platform
            }
        } else {
            // Horizontal collision
            if (dx > 0) {
                position.x += overlapX;
            } else {
                position.x -= overlapX;
            }
            velocity.x = 0;
        }

        // Update sprite position after collision resolution
        entity.setPosition(position);
    }
}


void Entity::update(float deltaTime){


    // here update all the params that you want to update 
    // ex: physics params, the position of the entity 

    // keep updating the position 
    position.x += velocity.x * deltaTime;
    position.y += velocity.y * deltaTime;
    
    // apply gravity 
    if (physical && !isOnGround){
        // apply the acceleration 
        velocity.y += GRAVITY * deltaTime;

    }

    // set the position for the entity
    entity.setPosition(position);
    sprite.setPosition(position);
    // so now we can instead sync the position of the sprite and update position
    

}

void Entity::bindSprite(std::string spriteName, SpriteManager& spriteManager){
    // create the sprite and return an instance to it to be used by other functions in the class 
    sprite = spriteManager.getSprite(spriteName);
    sprite.setPosition(position);
}

```

here in the `bindSprite` function, we just take the sprite from the spriteManager and assign to the sprite object we initialised in the entity, and then in the update function we sync the sprite's position on each frame with the entity's position and that's how it perfectly works.

the resources that i have used in here can be found [here]() and [here]().

so now we will try to setup a basic scene and will see how it looks on the screen with sprites up on the entities.

just uncomment all those textures loaded in the `loadTexture` function.

and the setup in the `main.cpp` looks like:

```cpp 

#include <SFML/Graphics.hpp>
#include <iostream>
#include "entities/player.hpp"
#include "input/input.hpp"
#include "graphics/spriteManager.hpp"



int main()
{
    // Create the main window
    sf::RenderWindow window(sf::VideoMode(800, 600), "Platformer Game");
    window.setFramerateLimit(60);
    
    // get an instance of the spriteManager 
    SpriteManager spriteManager;


    // create an input handler instance 
    InputHandler inputHandler;

    // Create a player entity
    Player player{sf::Vector2f(20.0f,0.0f), sf::Vector2f(5.0f,-10.0f), &inputHandler, "walk1", spriteManager};
    player.physical = true;


    // all this is going to be abstracted into another platform class later 
    // Create multiple adjacent platforms
    std::vector<Entity> platforms;
    float startX = 0.0f;
    float platformWidth = 400.0f;
    float platformHeight = 16.0f;
    int numPlatforms = platformWidth/16;

    for (int i = 0; i < numPlatforms; ++i) {
        Entity plat;
        plat.setPosition(sf::Vector2f{startX + i * 16, 450});
        // plat.setSize(sf::Vector2f{platformWidth, platformHeight});
        plat.physical = false; // platforms are static, so no physics
        plat.bindSprite("grassTile", spriteManager);
        platforms.push_back(plat);
    }

    // init the clock 
    sf::Clock clock;

    // Main game loop
    while (window.isOpen())
    {

        // get the deltaTime using clocok 
        float deltaTime = clock.restart().asSeconds();

        // Process events
        sf::Event event;
        while (window.pollEvent(event))
        {
            // Close window: exit
            if (event.type == sf::Event::Closed)
                window.close();
                
            // Escape key: exit
            if (event.type == sf::Event::KeyPressed)
            {
                if (event.key.code == sf::Keyboard::Escape)
                    window.close();
            }
        }

        // input Handler update each frame
        inputHandler.update(deltaTime);

        // Update the entity
        player.isOnGround = false; // Reset isOnGround at the start of each update
        player.update(deltaTime);
        

        for (auto& platform : platforms) {
            platform.update(deltaTime);
        }

        // resolve collision for each platform
        for (auto& platform : platforms) {
            if (player.isColliding(platform)) {
            player.resolveCollision(platform);
            }
        }

        // change the sprite for player each frame 
        int i = 1 ;
        while(true){
            std::string spriteName = "walk" + std::to_string(i);
            
            player.bindSprite(spriteName, spriteManager);
            i++;
            break;


            if (i == 4) i = 1;
        }

        // Clear screen
        window.clear(sf::Color::Black);
        // Draw the entity
        window.draw(player);
        for (const auto& platform : platforms) {
            window.draw(platform);
        }
        // Update the window
        window.display();
    }

    return 0;
}


```

building and running this we get a frame like this:

![Platformer Game Screenshot](/assets/platformer_demo_2.png)


so ig everything looks good till here, one problem you might face you are not able to compile program because of undefined reference, do make sure you include the new graphics folder in the build task.

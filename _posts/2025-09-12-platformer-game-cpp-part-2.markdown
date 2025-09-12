---
layout: post 
title: "Platformer Game in C++ Part 2: Implementing Input Handler and Player Movement Mechanics"
categories: tech 
tags: c++ platformer gamedev 
author: rishabh 
share: true 
image: "/assets/platformer_part_2.png"
comments: true
---

So, in the last post we implemented the Base `Entity` Class and a simple `Player` Class.
In this blog we are going to implement the `Input Handler` Class and integrate movement in the `Player` Class using the `Input Handler`.

Below is the header file for the `InputHandler` class, which is a crucial component for managing player input in our platformer game. This class is designed to handle keyboard inputs in a decoupled way and be a whole different system where different entities from the game can bind the keys to specific functions.

```cpp

#pragma once
#include <iostream> 
#include <unordered_map>
#include <SFML/Window.hpp>
#include <functional>




class InputHandler {
    private: 
        // map to store the state of the key at any timeframe 
        std::unordered_map<sf::Keyboard::Key, bool> currentKeys;
        std::unordered_map<sf::Keyboard::Key, std::function<void()>> keyBindings;


    public:

        // constructor 
        InputHandler();

        // have some state methods, getters and setters 
        bool keyDown(sf::Keyboard::Key key);
        bool keyUp(sf::Keyboard::Key key);

        // to check if a key is already bound to some other function
        bool isKeyBound(sf::Keyboard::Key key);
        void bindKey(sf::Keyboard::Key key, std::function<void()> func);
        void releaseKey(sf::Keyboard::Key);


        // a public update function to update the states each frame 
        void update(float deltaTime);

};

```

### Understanding the InputHandler Header File

The `InputHandler` class is a crucial component of our platformer game. It is responsible for managing player input, binding keys to specific actions, and updating the state of keys during the game loop. This ensures that the game can respond dynamically to user input.

#### Private Members

The `InputHandler` class has two private members:

1. **`currentKeys`**:
   - This is an `std::unordered_map` that stores the state of each key (pressed or not) at any given time.
   - It allows the game to track which keys are currently being pressed, enabling smooth and responsive controls.

2. **`keyBindings`**:
   - This is another `std::unordered_map` that maps keys to specific actions using `std::function<void()>`.
   - It allows us to bind keys to custom functions, making the input system flexible and extensible.

#### Public Methods

The `InputHandler` class provides several public methods to interact with its private members:

1. **Constructor**:
   - Initializes the `InputHandler` object.

2. **State Methods**:
   - `keyDown(sf::Keyboard::Key key)`: Checks if a specific key is currently pressed.
   - `keyUp(sf::Keyboard::Key key)`: Checks if a specific key is currently released.

3. **Key Binding Methods**:
   - `isKeyBound(sf::Keyboard::Key key)`: Checks if a key is already bound to an action.
   - `bindKey(sf::Keyboard::Key key, std::function<void()> func)`: Binds a key to a specific function.
   - `releaseKey(sf::Keyboard::Key key)`: Unbinds a key from its action.

4. **Update Method**:
   - `update(float deltaTime)`: Updates the state of keys each frame. This method will be called in the game loop to ensure the input system remains up-to-date.


#### Next Steps

In the next section, we will implement the `InputHandler` class and integrate it with the `Player` class. This will enable the player to move and interact with the game world based on user input.

So implementing the `Input Handler` Class the code goes like:
```cpp
#include "input.hpp"


// initialise the constructor 
InputHandler::InputHandler(){
    // initialise all keys as false 
    // all keys (enumerations) are internally represented using integers
    for (int key = sf::Keyboard::A; key <= sf::Keyboard::KeyCount; key++){
        currentKeys[static_cast<sf::Keyboard::Key> (key)] = false;
    }
}

bool InputHandler::keyDown(sf::Keyboard::Key key){
    return currentKeys[key] == true;
}

bool InputHandler::keyUp(sf::Keyboard::Key key){
    return currentKeys[key] == false;
}

bool InputHandler::isKeyBound(sf::Keyboard::Key key){
    return keyBindings.find(key) != keyBindings.end();
}

void InputHandler::bindKey(sf::Keyboard::Key key, std::function<void()> func){
    // if the key is bound 
    if(isKeyBound(key)) {
        std::cout << "Key " << key << "is already bound to a function Release the key before binding to a new function!";
        return;
    }

    keyBindings[key] = func;
}

void InputHandler::releaseKey(sf::Keyboard::Key key){
    keyBindings.erase(key);
}


// the update each frame function 
void InputHandler::update(float deltaTime){
    // ig we dont need this deltaTime here though 
    // so here we are going to check the tasks that need to be check each frame like:
    // - check the state of the keys again
    // - call all the functions that meant for the keys if the particular key is down

    // update the key states
    for (int key = sf::Keyboard::A; key <= sf::Keyboard::KeyCount; key++){
        sf::Keyboard::Key sfKey = static_cast<sf::Keyboard::Key>(key);
        if (sf::Keyboard::isKeyPressed(sfKey)){
            currentKeys[sfKey] = true;
            if (isKeyBound(sfKey)){
                keyBindings[sfKey]();
            }
        }
        else {
            currentKeys[sfKey] = false;
        }
    }
}


```

### Implementation of the InputHandler Class

The implementation of the `InputHandler` class brings the header file to life, enabling the game to handle player input dynamically. Below is a detailed breakdown of the implementation:

#### Constructor

The constructor initializes the `InputHandler` object by setting all keys to `false` in the `currentKeys` map. This ensures that no keys are considered pressed at the start of the game.

```cpp
InputHandler::InputHandler(){
    for (int key = sf::Keyboard::A; key <= sf::Keyboard::KeyCount; key++){
        currentKeys[static_cast<sf::Keyboard::Key> (key)] = false;
    }
}
```

- **Key Initialization**: The loop iterates through all keys in the `sf::Keyboard` enumeration, setting their state to `false`.
- **SFML Reference**: Learn more about `sf::Keyboard` [here](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Keyboard.php).

#### State Methods

The `keyDown` and `keyUp` methods check whether a specific key is pressed or released, respectively.

```cpp
bool InputHandler::keyDown(sf::Keyboard::Key key){
    return currentKeys[key] == true;
}

bool InputHandler::keyUp(sf::Keyboard::Key key){
    return currentKeys[key] == false;
}
```

- **Key State Check**: These methods access the `currentKeys` map to determine the state of a key.
- **C++ Reference**: Learn more about `std::unordered_map` [here](https://en.cppreference.com/w/cpp/container/unordered_map).

#### Key Binding Methods

The `isKeyBound`, `bindKey`, and `releaseKey` methods manage key bindings, allowing keys to be associated with specific actions.

```cpp
bool InputHandler::isKeyBound(sf::Keyboard::Key key){
    return keyBindings.find(key) != keyBindings.end();
}

void InputHandler::bindKey(sf::Keyboard::Key key, std::function<void()> func){
    if(isKeyBound(key)) {
        std::cout << "Key " << key << " is already bound to a function. Release the key before binding to a new function!";
        return;
    }
    keyBindings[key] = func;
}

void InputHandler::releaseKey(sf::Keyboard::Key key){
    keyBindings.erase(key);
}
```

- **Key Binding**: The `bindKey` method associates a key with a function, while `releaseKey` removes the binding.
- **Function Wrapping**: The `std::function<void()>` type allows any callable object to be bound to a key. Learn more about `std::function` [here](https://en.cppreference.com/w/cpp/utility/functional/function).

#### Update Method

The `update` method is called each frame to update key states and trigger bound actions.

```cpp
void InputHandler::update(float deltaTime){
    for (int key = sf::Keyboard::A; key <= sf::Keyboard::KeyCount; key++){
        sf::Keyboard::Key sfKey = static_cast<sf::Keyboard::Key>(key);
        if (sf::Keyboard::isKeyPressed(sfKey)){
            currentKeys[sfKey] = true;
            if (isKeyBound(sfKey)){
                keyBindings[sfKey]();
            }
        }
        else {
            currentKeys[sfKey] = false;
        }
    }
}
```

- **Key State Update**: The method checks the state of each key and updates the `currentKeys` map.
- **Action Triggering**: If a key is pressed and bound to a function, the function is called.
- **Delta Time**: Although not used here, `deltaTime` can be utilized for time-dependent actions.

#### Integration with the Game

The `InputHandler` class is integrated into the game loop, where the `update` method is called each frame. This ensures that key states and actions remain synchronized with the game’s logic. The `Player` class can use the `InputHandler` to bind movement keys and respond to user input dynamically.

```cpp 
//main.cpp

...rest code 

// in main game loop 
inputHandler.update()

```

#### Resources for Further Reading

- SFML Documentation: [Keyboard Input](https://www.sfml-dev.org/tutorials/2.5/window-inputs.php)
- C++ Reference: [std::unordered_map](https://en.cppreference.com/w/cpp/container/unordered_map)
- C++ Reference: [std::function](https://en.cppreference.com/w/cpp/utility/functional/function)

With the `InputHandler` class implemented, the next step is to integrate it with the `Player` class to enable movement and interaction. Let's get started with it.

We will setup the input handler first in the `main.cpp` file and use the same input handler everywhere to maintain consistent key bindings everywhere, so the main file looks like:

```cpp 
#include <SFML/Graphics.hpp>
#include <iostream>
#include "entities/player.hpp"
#include "input/input.hpp"




int main()
{
    // Create the main window
    sf::RenderWindow window(sf::VideoMode(800, 600), "Platformer Game");
    window.setFramerateLimit(60);
    
    // create an input handler instance 
    InputHandler inputHandler;

    // Create an entity
    Player player{sf::Vector2f(20.0f,0.0f), sf::Vector2f(5.0f,-10.0f), &inputHandler};
    player.physical = false;


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
        player.update(deltaTime);

        // Clear screen
        window.clear(sf::Color::Black);

        // Draw the entity
        window.draw(player);

        // Update the window
        window.display();
    }

    return 0;
}

```

The changes you can see is the initialisation of the `InputHandler` instance `inputHandler` and the calling of the `inputHandler.update()` function in the main loop to keep updating all the key states every frame.

Now we can move on to implementing movement mechanics in the player class.

The new header file looks like:
```cpp 
#include <SFML/Graphics.hpp>
#include "entity.hpp"
#include "input/input.hpp"


class Player : public Entity{

    private: 
        // we will implement the player movement physics here, will take the inputHandler as input
        // and will use here to get all the active keystrokes and keyBindings

        void moveLeft();
        void moveRight();
        void moveUp();
        void moveDown();

        // speed params 
        sf::Vector2f speed{10.0f, 10.0f};

        
        public: 

        // constructor with all initial params 
        Player(sf::Vector2f pos, sf::Vector2f vel, InputHandler* inputHandler);

        // here we don't need to implement the draw function again, as we did it in the 
        // base entity class, which will be inherited here

        // write some getter and setter function here for the player 
        void setPosition(sf::Vector2f pos);
        sf::Vector2f getPosition();
    
};


```

You can see the four new functions that will bound to the respective keys in the constructor, and also the `Player` Constructor now takes in `inputHandler` pointer as an argument to get access to key bindings and states.

the implementation finally looks like:

```cpp
#include "player.hpp"


// we are here getting the same Inputhandler and not creating a new one, so as to maintain single key bindings across whole game
// which prevents us from assigning same key to multiple functions

Player::Player(sf::Vector2f pos, sf::Vector2f vel, InputHandler* inputHandler){
    // init the player with these parameters 
    position = pos; 
    vel = vel;

    // sync the shape with the variables 
    entity.setPosition(pos);


    // set the important flags here in the contructor
    // like we want the player to always have some physics 
    // so we will set physical to true.
    physical = true;


    // setup the key bindings for the player 
    inputHandler->bindKey(sf::Keyboard::Up, [this]() { this->moveUp(); });
    inputHandler->bindKey(sf::Keyboard::Down, [this]() { this->moveDown(); });
    inputHandler->bindKey(sf::Keyboard::Left, [this]() { this->moveLeft(); });
    inputHandler->bindKey(sf::Keyboard::Right, [this]() { this->moveRight(); });
}


// implement the four functions to move the players 
// we will not use any acceleration here in updating the positions, 
// simple position manipulation without acceleration.
void Player::moveUp(){
    position.y -= speed.y;
}
void Player::moveDown(){
    position.y += speed.y;
}

void Player::moveLeft(){
    position.x -= speed.x;
}

void Player::moveRight(){
    position.x += speed.x;
}

// implement the setter and getter functions 

void Player::setPosition(sf::Vector2f pos){
    position = pos;
}

sf::Vector2f Player::getPosition(){
    return position;
}

```

### Refining the Player Class Implementation

The `Player` class is now integrated with the `InputHandler` to enable movement based on user input. Below is a refined explanation of the new implementation:

#### Constructor

The `Player` constructor initializes the player’s position, velocity, and key bindings. By passing the `InputHandler` instance as a parameter, the `Player` class ensures consistent key bindings across the game.

```cpp
Player::Player(sf::Vector2f pos, sf::Vector2f vel, InputHandler* inputHandler){
    position = pos; 
    vel = vel;

    // Sync the shape with the position
    entity.setPosition(pos);

    // Set the player to be affected by physics
    physical = true;

    // Bind movement keys to respective functions
    inputHandler->bindKey(sf::Keyboard::Up, [this]() { this->moveUp(); });
    inputHandler->bindKey(sf::Keyboard::Down, [this]() { this->moveDown(); });
    inputHandler->bindKey(sf::Keyboard::Left, [this]() { this->moveLeft(); });
    inputHandler->bindKey(sf::Keyboard::Right, [this]() { this->moveRight(); });
}
```

- **Key Bindings**: The `bindKey` method associates movement keys with the `moveUp`, `moveDown`, `moveLeft`, and `moveRight` methods.
- **Physics Flag**: The `physical` flag ensures that the player is affected by game physics.

#### Movement Methods

The movement methods update the player’s position based on the `speed` vector. These methods are called when the respective keys are pressed.

```cpp
void Player::moveUp(){
    position.y -= speed.y;
}

void Player::moveDown(){
    position.y += speed.y;
}

void Player::moveLeft(){
    position.x -= speed.x;
}

void Player::moveRight(){
    position.x += speed.x;
}
```

- **Simple Movement**: The position is updated directly without acceleration for simplicity.
- **Speed Vector**: The `speed` vector determines the movement step size.

#### Getter and Setter Methods

The `setPosition` and `getPosition` methods allow the player’s position to be updated or retrieved.

```cpp
void Player::setPosition(sf::Vector2f pos){
    position = pos;
}

sf::Vector2f Player::getPosition(){
    return position;
}
```

- **Encapsulation**: These methods encapsulate the `position` variable, ensuring controlled access.

#### Integration with the Game Loop

The `Player` class is updated in the game loop, responding to user input dynamically. The `InputHandler` ensures that key states are updated each frame, and the bound functions are triggered accordingly.

```cpp
// Main game loop
while (window.isOpen()){
    float deltaTime = clock.restart().asSeconds();

    // Update input handler
    inputHandler.update(deltaTime);

    // Update player
    player.update(deltaTime);

    // Render player
    window.clear();
    window.draw(player);
    window.display();
}
```

- **Delta Time**: The `deltaTime` variable can be used for time-dependent updates in the future.
- **Rendering**: The player’s position is updated and rendered each frame.

#### Next Steps

With the `Player` class integrated with the `InputHandler`, the next step is to implement collision detection and other advanced mechanics. We will be doing it in the upcoming blog posts.

That's it for today, do come back to read the other parts and do comment if you find any better approaches anywhere, always open to suggestions.

Bye Bye.

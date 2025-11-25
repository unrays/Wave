# Wave
High-performance C++ ECS with cache-friendly design.

I understand that this might seem intimidating and very laborious to read, but I assure you it's interesting. It's a small part of my life, and I wanted to share it because things like this are what drive me every day. Here is the account and the result of a month of learning and exploring things that still fascinate me so much today :)

I'm probably gonna add a description here very soon :)
It's not perfect and I don't claim to be an expert in the field of ECS, but I'm 100% satisfied with my progress since I discovered the subject a month ago. In my two years of programming, ECS is the thing that has impacted me the most; I'm telling you, it's like a revelation for me :)
Coded on November 14, 2025

Hello everyone, good to see you! If you've stumbled upon this repo, it's because you're potentially interested in the concept of ECS systems. That's perfect because that's exactly my case too. Okay, where to begin... First, I want to say that this project is potentially the most technically demanding I can do right now in terms of optimizing and understanding ECS. I'm definitely not an expert, but I'm intrigued enough by the subject to say I know a little bit about what I'm talking about :)

This wonderful story began a month and a half ago (as I write this, November 25, 2025). I was discussing many questions and concepts with ChatGPT that fascinated me, and I was mainly trying to solve a very recurring problem in software architecture: single responsibility and extensibility. To be honest, I have a bit of a problem with general programming concepts, not that I dislike them or don't understand the semantics and structure, but rather because I can't stand the fact that single responsibility is being sold to us as a universal doctrine while these same people are the first to create Player classes with members like Health, Position, Velocity, etc., which, in my opinion, completely breaks the SRP and the extensibility sought by applying these principles. However, I'm not criticizing anyone; I don't consider myself to have absolute knowledge, and like everyone else, I often make mistakes and say incorrect things. It's part of education, what can you do... Okay, where were we, yes, ECS.

Well, after much deliberation, I've come to the conclusion that the only way to properly apply design principles (subjective, I know) is to separate the logic from the "dynamic" elements (entities) of the system. First, I looked into SOA (Service-Oriented Architecture, I think) and coded a mini-game engine in C# based on the concept, which I then posted on GitHub under the name Quark. :) This experience (the first week after I started thinking about it) was extremely beneficial to my skills in visualizing and designing systems where a player shouldn't know their health and vice versa; the two should be independent. In this way, I coded the engine with services here and there, and I have no complaints about the benefits of such an architecture. It's almost magical how the systems fit together seamlessly without any one depending on the other. Okay, enough about the advantages of Quark, now let's talk about the problem: performance. And yes, who would have thought that running 12,800 different services, each containing a component entity dictionary, is very inefficient and becomes exponentially more expensive as the number of entities increases?

So what do you think the next step was? I've been talking about components and entities for ages :) Good answer, ECS! Actually, it was while researching service-oriented architectures that I noticed a topic that came up very often and seemed quite complex. But hey, complexity doesn't bother me, and I dove into the world of entity component systems. It's now been two weeks since the "start" of my obsession. For the next week and a half, I'll be spending all my time, 3-4 hours each evening and 8-12 hours each weekend day, just doing and thinking about ECS. This led me to start coding an ECS in C# on November 7th, named Apex, which will become a personal benchmark for the evolution of my ECS skills in just one week. During this project, which involved a total of 11 iterations, I weighed the pros and cons, advantages and disadvantages, and above all, what ECS offers in general. Furthermore, I also experienced, for the first time in my life, optimization with a real timer that measured the time taken to generate and access 100K, 1M, etc... entities!

This allowed me to keep an eye on my objectives and see in real time the optimizations I was making and testing. Sometimes certain optimizations were less efficient than previous iterations, and other times, much more efficient. I also experimented with new concepts like jagged arrays, contiguous memory, data memory cost and bandwidth, structs and polymorphism, as well as concepts like boxing and unsafe references in C#. Furthermore, I think I learned to assess and make better choices when time comes to write efficient code, thus improving my perspective as a professional optimizer. :) In addition, I also encountered some rather difficult algorithmic problems that bothered me for short periods, such as trying to maintain O(1) accesses as opposed to O(n) in my first iteration. More seriously, I worked for a very hard week to achieve a result that is, listen carefully, about 20 to 100 times faster than my first iteration.

Finally, a week later, I had the result I wanted, but I wasn't 100% satisfied. So I decided to port it to C++, a much more efficient language than C#, to see the maximum my design could offer. And here's the result: a C++ ECS that represents the culmination of my learning over the last two weeks. I'm very proud of my design. I know it's not perfect and there's still room for optimization, but keep in mind that no program is perfect. Finally, to conclude—yes, finally—I want to thank you if you've read this. I put a lot of effort into it, and I felt I had to reflect that in my writing. In the future, I hope to expand my knowledge to other topics related to video games and optimization. It seems to be in my blood; I love designing and thinking architecturally, while keeping performance as the essential element of any good architecture. This ECS is probably the basis of a future personal game engine that I want to create, so keep an eye out for new updates :) Thank you very much for reading my repo and I wish you a very nice day.

If you're wondering why I'm doing this, it's primarily to create a portfolio and archive of my evolution as a programmer. I'm a very nostalgic person who greatly values ​​learning, both for myself and for others.

test run on a configuration with a Ryzen 5 9600X overclocked to 5.6GHz and 32GB of RAM at 7400MHz CL34
```console
Setup 10000 entities with 2 components each: 0.0397 ms
Accessed and modified Position and Velocity components: 0.0082 ms

Setup 100000 entities with 2 components each: 0.2458 ms
Accessed and modified Position and Velocity components: 0.0773 ms

Setup 1000000 entities with 2 components each: 4.8043 ms
Accessed and modified Position and Velocity components: 0.8158 ms

Setup 10000000 entities with 2 components each: 56.771 ms
Accessed and modified Position and Velocity components: 8.6409 ms

Setup 100000000 entities with 2 components each: 617.791 ms
Accessed and modified Position and Velocity components: 86.2993 ms
```

## Code
```cpp
// Copyright (c) November 2025 Félix-Olivier Dumas. All rights reserved.
// Licensed under the terms described in the LICENSE file

#pragma once
#include <iostream>
#include <vector>
#include <chrono>
#include <cstdint>
#include <stdexcept>

class EntityId {
private:
    static std::uint32_t _nextId;

public:
    static std::uint32_t Next() { return _nextId++; }
};

std::uint32_t EntityId::_nextId = 0;

struct Entity {
    std::uint32_t Value;

    Entity() : Value(EntityId::Next()) {}
};

enum class ComponentMask : std::uint8_t {
    None = 0,
    Position = 1 << 0,
    Velocity = 1 << 1,
    Rotation = 1 << 2,
    Scale = 1 << 3,
    Color = 1 << 4,
};

struct Position { std::uint32_t X, Y; };
struct Velocity { std::uint32_t VX, VY; };
struct Rotation { std::uint8_t Angle; };
struct Scale { std::uint8_t X, Y; };
struct Color { std::uint8_t R, G, B, A; };

class Registry {
private:
    static constexpr std::uint32_t InitialEntityCapacity = 131072;
    static constexpr std::uint32_t InitialPoolCapacity = 262143;

    std::vector<ComponentMask> _cmask;
    std::vector<ComponentMask> _ctype;

    std::vector<Position> _positions;
    std::vector<int> _entityToPosIndex;

    std::vector<Velocity> _velocities;
    std::vector<int> _entityToVelIndex;

    std::vector<Rotation> _rotations;
    std::vector<int> _entityToRotIndex;

    std::vector<Scale> _scales;
    std::vector<int> _entityToScaleIndex;

    std::vector<Color> _colors;
    std::vector<int> _entityToColorIndex;

private:
    template <typename T> static ComponentMask MaskOf() {
        static_assert(std::is_trivially_copyable_v<T>);
        if constexpr (std::is_same_v<T, Position>) return ComponentMask::Position;
        if constexpr (std::is_same_v<T, Velocity>) return ComponentMask::Velocity;
        if constexpr (std::is_same_v<T, Rotation>) return ComponentMask::Rotation;
        if constexpr (std::is_same_v<T, Scale>)    return ComponentMask::Scale;
        if constexpr (std::is_same_v<T, Color>)    return ComponentMask::Color;
        throw std::runtime_error("Component type not supported");
    }

public:
    Registry(std::size_t maxEntities, std::size_t maxPool) {
        _cmask.resize(maxEntities, ComponentMask::None);
        _entityToPosIndex.resize(maxEntities, -1);
        _entityToVelIndex.resize(maxEntities, -1);
        _entityToRotIndex.resize(maxEntities, -1);
        _entityToScaleIndex.resize(maxEntities, -1);
        _entityToColorIndex.resize(maxEntities, -1);

        _positions.reserve(maxPool);
        _velocities.reserve(maxPool);
        _rotations.reserve(maxPool);
        _scales.reserve(maxPool);
        _colors.reserve(maxPool);
    }

    template <typename T> void Add(std::uint32_t eidx) {
        if constexpr (std::is_same_v<T, Position>) {
            if (_entityToPosIndex[eidx] != -1) {
                printf("[ERROR] Entity %u already has Position component\n", eidx);
                return;
            }
            _entityToPosIndex[eidx] = _positions.size();
            _positions.push_back(Position{});
        }
        else if constexpr (std::is_same_v<T, Velocity>) {
            if (_entityToVelIndex[eidx] != -1) {
                printf("[ERROR] Entity %u already has Velocity component\n", eidx);
                return;
            }
            _entityToVelIndex[eidx] = _velocities.size();
            _velocities.push_back(Velocity{});
        }
        else if constexpr (std::is_same_v<T, Rotation>) {
            if (_entityToRotIndex[eidx] != -1) {
                printf("[ERROR] Entity %u already has Rotation component\n", eidx);
                return;
            }
            _entityToRotIndex[eidx] = _rotations.size();
            _rotations.push_back(Rotation{});
        }
        else if constexpr (std::is_same_v<T, Scale>) {
            if (_entityToScaleIndex[eidx] != -1) {
                printf("[ERROR] Entity %u already has Scale component\n", eidx);
                return;
            }
            _entityToScaleIndex[eidx] = _scales.size();
            _scales.push_back(Scale{});
        }
        else if constexpr (std::is_same_v<T, Color>) {
            if (_entityToColorIndex[eidx] != -1) {
                printf("[ERROR] Entity %u already has Color component\n", eidx);
                return;
            }
            _entityToColorIndex[eidx] = _colors.size();
            _colors.push_back(Color{});
        }
    }

    template<typename T> T& Get(std::uint32_t eidx) {
        if constexpr (std::is_same_v<T, Position>) {
            if (_entityToPosIndex[eidx] == -1)
                throw std::runtime_error(
                    "Entity " + std::to_string(eidx) + " does NOT have Position component"
                );
            return _positions[_entityToPosIndex[eidx]];
        }
        else if constexpr (std::is_same_v<T, Velocity>) {
            if (_entityToVelIndex[eidx] == -1)
                throw std::runtime_error(
                    "Entity " + std::to_string(eidx) + " does NOT have Velocity component"
                );
            return _velocities[_entityToVelIndex[eidx]];
        }
        else if constexpr (std::is_same_v<T, Rotation>) {
            if (_entityToRotIndex[eidx] == -1)
                throw std::runtime_error(
                    "Entity " + std::to_string(eidx) + " does NOT have Rotation component"
                );
            return _rotations[_entityToRotIndex[eidx]];
        }
        else if constexpr (std::is_same_v<T, Scale>) {
            if (_entityToScaleIndex[eidx] == -1)
                throw std::runtime_error(
                    "Entity " + std::to_string(eidx) + " does NOT have Scale component"
                );
            return _scales[_entityToScaleIndex[eidx]];
        }
        else if constexpr (std::is_same_v<T, Color>) {
            if (_entityToColorIndex[eidx] == -1)
                throw std::runtime_error(
                    "Entity " + std::to_string(eidx) + " does NOT have Color component"
                );
            return _colors[_entityToColorIndex[eidx]];
        }
        throw std::runtime_error("Component type not supported");
    }
};

int main() {
    constexpr size_t entityCount = 100000000;
    constexpr size_t maxPool = 200000;

    Registry registry(entityCount, maxPool);
    std::vector<Entity> entities(entityCount);

    auto start = std::chrono::high_resolution_clock::now();

    for (uint32_t i = 0; i < entityCount; ++i) {
        entities[i].Value = i;
        registry.Add<Position>(i);
        registry.Add<Velocity>(i);
    }
    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double, std::milli> elapsed = end - start;
    std::cout << "Setup " << entityCount << " entities with 2 components each: "
        << elapsed.count() << " ms\n";

    start = std::chrono::high_resolution_clock::now();
    for (uint32_t i = 0; i < entityCount; ++i) {
        auto& pos = registry.Get<Position>(entities[i].Value);
        auto& vel = registry.Get<Velocity>(entities[i].Value);
        pos.X = i; pos.Y = i * 2;
        vel.VX = i; vel.VY = i * 2;
    }
    end = std::chrono::high_resolution_clock::now();
    elapsed = end - start;
    std::cout << "Accessed and modified Position and Velocity components: "
        << elapsed.count() << " ms\n";
}

```

# Wave
High-performance C++ ECS with cache-friendly design.

I'm probably gonna add a description here very soon :)
It's not perfect and I don't claim to be an expert in the field of ECS, but I'm 100% satisfied with my progress since I discovered the subject a month ago. In my two years of programming, ECS is the thing that has impacted me the most; I'm telling you, it's like a revelation for me :)
Coded on November 14, 2025

```console
test run on a configuration with a Ryzen 5 9600X overclocked to 5.6GHz, 32GB of RAM at 7400MHz CL34

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

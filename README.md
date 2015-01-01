# EntityX - A fast, idiomatic C++11 Entity-Component System

This version of EntityX is header-only and utilizes a bunch of template
metaprogramming to provide compile-time component metadata. It also provides
support for alternative storage engines for component data.

For example, the number of components and their size is known at compile time.

Example code:


```c++
#include "entityx.hh"

using namespace entityx;

struct Position {
  Position(float x, float y) : x(x), y(y) {}
  float x, y;
};

struct Health {
  Health(float health) : health(health) {}
  float health;
};

struct Direction {
  Direction(float x, float y): x(x), y(y), z(0) {}
  float x, y, z;
};


// Convenience types for our entity system.
typedef Components<Position, Health, Direction> GameComponents;
typedef EntityX<GameComponents, ColumnStorage<GameComponents>> Entities;
template <typename C> using Component = Entities::Component<C>;
using Entity = Entities::Entity;

int main() {
  Entities entities;
  Entity a = entities.create();
  a.assign<Position>(1, 2);
  Entity b = entities.create();
  Component<Health> health = b.assign<Health>(10);
  Component<Direction> direction = b.assign<Direction>(3, 4);

  // Retrieve component.
  Component<Position> position = a.component<Position>();

  printf("position x = %f, y = %f\n", position->x, position->y);
  printf("has position = %d\n", bool(a.component<Position>()));

  // Remove component from entity.
  position.remove();
  printf("has position = %d\n", bool(a.component<Position>()));

}
```

## Performance

This version of EntityX is (generally) 2-5x faster than the previous version, depending on the operations performed:

| Benchmark | Old version | New version | Improvement |
|-----------|-------------|-------------|-------------|
| Creating 10M entities | 0.456865s | 0.256681s | **1.8x** |
| Creating 10M entities (create_many()) | 0.456865s | 0.134333s | **3.4x** |
| Destroying 10M entities | 0.308511s | 0.074304s | **4.2x** |
| Creating 10M entities with observer | 0.693537s | 0.465937s | **1.5x** |
| Creating 10M entities with create_many() with observer | 0.693537s | 0.144255s | **4.8x** |
| Destroying 10M entities with observer | 0.344164s | 0.093211s | **3.7x** |
| Iterating over 10M entities, unpacking one component | 0.059354s | 0.043844s | **1.4x** |
| Iterating over 10M entities, unpacking two components | 0.093078s | 0.07617s | **1.2x** |

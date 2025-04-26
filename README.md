https://docs.github.com/ko/enterprise-cloud@latest/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams


```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
```mermaid
classDiagram
    class Creature {
        +String name
        +int hp
        +move()
        +attack()
    }

    class Character {
        +int level
        +equipWeapon()
    }

    class Enemy {
        +String type
        +taunt()
    }

    Creature <|-- Character
    Creature <|-- Enemy
```

```stl
solid cube_corner
  facet normal 0.0 -1.0 0.0
    outer loop
      vertex 0.0 0.0 0.0
      vertex 1.0 0.0 0.0
      vertex 0.0 0.0 1.0
    endloop
  endfacet
  facet normal 0.0 0.0 -1.0
    outer loop
      vertex 0.0 0.0 0.0
      vertex 0.0 1.0 0.0
      vertex 1.0 0.0 0.0
    endloop
  endfacet
  facet normal -1.0 0.0 0.0
    outer loop
      vertex 0.0 0.0 0.0
      vertex 0.0 0.0 1.0
      vertex 0.0 1.0 0.0
    endloop
  endfacet
  facet normal 0.577 0.577 0.577
    outer loop
      vertex 1.0 0.0 0.0
      vertex 0.0 1.0 0.0
      vertex 0.0 0.0 1.0
    endloop
  endfacet
endsolid
```

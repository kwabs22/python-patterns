# Entity-Component-System (ECS) Architecture

## Overview
ECS is a architectural pattern commonly used in game development that separates data (Components) from behavior (Systems) and uses Entities as simple identifiers.

## Related Patterns

### Composite Pattern
**Reference:** [composite](../patterns/structural/composite.py)

The Composite pattern is foundational to scene graphs and entity hierarchies in games.

**Game Implementation:**
```python
class GameObject:
    """Base entity that can have components and children"""
    def __init__(self, name):
        self.name = name
        self.components = []
        self.children = []
        self.parent = None

    def add_child(self, child):
        child.parent = self
        self.children.append(child)

    def add_component(self, component):
        component.owner = self
        self.components.append(component)

    def update(self, dt):
        # Update all components
        for component in self.components:
            component.update(dt)

        # Update all children
        for child in self.children:
            child.update(dt)

    def get_component(self, component_type):
        for comp in self.components:
            if isinstance(comp, component_type):
                return comp
        return None
```

### Registry Pattern
**Reference:** [registry](../patterns/behavioral/registry.py)

Track all entities and components in the game.

**Game Implementation:**
```python
class EntityRegistry:
    """Central registry for all game entities"""
    _entities = {}
    _next_id = 0

    @classmethod
    def create_entity(cls, name="Entity"):
        entity_id = cls._next_id
        cls._next_id += 1
        entity = GameObject(name)
        entity.id = entity_id
        cls._entities[entity_id] = entity
        return entity

    @classmethod
    def get_entity(cls, entity_id):
        return cls._entities.get(entity_id)

    @classmethod
    def destroy_entity(cls, entity_id):
        if entity_id in cls._entities:
            del cls._entities[entity_id]

    @classmethod
    def get_entities_with_component(cls, component_type):
        """Find all entities that have a specific component"""
        return [
            entity for entity in cls._entities.values()
            if entity.get_component(component_type) is not None
        ]
```

## Components

Components are pure data containers with no logic.

```python
class Component:
    """Base component class"""
    def __init__(self):
        self.owner = None
        self.enabled = True

    def update(self, dt):
        pass

class TransformComponent(Component):
    """Position, rotation, scale"""
    def __init__(self, x=0, y=0, rotation=0):
        super().__init__()
        self.x = x
        self.y = y
        self.rotation = rotation
        self.scale = 1.0

class SpriteComponent(Component):
    """Visual representation"""
    def __init__(self, sprite_name):
        super().__init__()
        self.sprite_name = sprite_name
        self.visible = True

class PhysicsComponent(Component):
    """Physics properties"""
    def __init__(self):
        super().__init__()
        self.velocity_x = 0
        self.velocity_y = 0
        self.mass = 1.0
        self.use_gravity = True

class HealthComponent(Component):
    """Health/damage tracking"""
    def __init__(self, max_health=100):
        super().__init__()
        self.max_health = max_health
        self.current_health = max_health

    def take_damage(self, amount):
        self.current_health = max(0, self.current_health - amount)
        return self.current_health <= 0  # Returns True if dead

    def heal(self, amount):
        self.current_health = min(self.max_health, self.current_health + amount)
```

## Systems

Systems contain the game logic and operate on entities with specific components.

```python
class System:
    """Base system class"""
    def update(self, entities, dt):
        raise NotImplementedError

class PhysicsSystem(System):
    """Updates physics for all entities with physics components"""
    def __init__(self):
        self.gravity = 9.8

    def update(self, entities, dt):
        for entity in entities:
            physics = entity.get_component(PhysicsComponent)
            transform = entity.get_component(TransformComponent)

            if physics and transform and physics.enabled:
                # Apply gravity
                if physics.use_gravity:
                    physics.velocity_y += self.gravity * dt

                # Update position
                transform.x += physics.velocity_x * dt
                transform.y += physics.velocity_y * dt

class RenderSystem(System):
    """Renders all entities with sprite components"""
    def update(self, entities, dt):
        for entity in entities:
            sprite = entity.get_component(SpriteComponent)
            transform = entity.get_component(TransformComponent)

            if sprite and transform and sprite.enabled and sprite.visible:
                self.render_sprite(sprite.sprite_name, transform.x, transform.y,
                                 transform.rotation, transform.scale)

    def render_sprite(self, sprite_name, x, y, rotation, scale):
        # Actual rendering code here
        pass

class CombatSystem(System):
    """Handles combat interactions"""
    def update(self, entities, dt):
        # Find all entities with health components
        combatants = [e for e in entities if e.get_component(HealthComponent)]

        # Process combat logic
        # (collision detection, damage application, etc.)
        pass
```

## Example: Creating a Player Entity

```python
# Create player entity
player = EntityRegistry.create_entity("Player")

# Add components
player.add_component(TransformComponent(x=100, y=100))
player.add_component(SpriteComponent("player_sprite"))
player.add_component(PhysicsComponent())
player.add_component(HealthComponent(max_health=100))

# Player can also have child entities (like equipped weapon)
weapon = EntityRegistry.create_entity("Sword")
weapon.add_component(TransformComponent(x=10, y=0))  # Offset from player
weapon.add_component(SpriteComponent("sword_sprite"))
player.add_child(weapon)
```

## Game Loop Integration

```python
class Game:
    def __init__(self):
        self.systems = [
            PhysicsSystem(),
            CombatSystem(),
            RenderSystem()
        ]

    def update(self, dt):
        # Get all entities
        entities = list(EntityRegistry._entities.values())

        # Run all systems
        for system in self.systems:
            system.update(entities, dt)
```

## Advantages

1. **Composition over Inheritance**: Build entities by combining components
2. **Data-Oriented**: Components are data, systems are logic - cache friendly
3. **Flexible**: Easy to add/remove capabilities at runtime
4. **Reusable**: Components and systems are highly reusable
5. **Testable**: Systems can be tested independently

## Disadvantages

1. **Complexity**: More initial setup than simple inheritance
2. **Communication**: Components need ways to communicate (events, shared data)
3. **Performance**: May need optimization for large numbers of entities

## When to Use

- Large games with many entity types
- Games requiring runtime composition
- When you need data-driven entity creation
- Performance-critical games (with careful implementation)

## Alternatives

- **Simple Inheritance**: Good for small games with few entity types
- **Hybrid Approach**: Combine some inheritance with components for common behavior

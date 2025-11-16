# Object Pooling for Games

## Overview
Object pooling is critical in games to avoid frequent memory allocation/deallocation during gameplay. Instead of creating and destroying objects constantly, we reuse them from a pool.

## Related Pattern
**Reference:** [pool](../patterns/creational/pool.py)

## Why Object Pooling Matters in Games

### Performance Issues with Creating/Destroying
```python
# BAD: Creates garbage every frame
class BadBulletSystem:
    def fire_bullet(self, x, y, direction):
        bullet = Bullet(x, y, direction)  # New allocation
        self.bullets.append(bullet)

    def update(self, dt):
        for bullet in self.bullets:
            if bullet.is_off_screen():
                self.bullets.remove(bullet)  # Garbage created
```

**Problems:**
- Memory allocation is slow
- Garbage collection can cause frame hitches
- Unpredictable performance spikes
- Memory fragmentation

## Basic Pool Implementation

```python
class Bullet:
    def __init__(self):
        self.x = 0
        self.y = 0
        self.velocity_x = 0
        self.velocity_y = 0
        self.active = False
        self.damage = 10

    def activate(self, x, y, velocity_x, velocity_y):
        """Reset and activate a pooled bullet"""
        self.x = x
        self.y = y
        self.velocity_x = velocity_x
        self.velocity_y = velocity_y
        self.active = True

    def deactivate(self):
        """Return bullet to pool"""
        self.active = False

    def update(self, dt):
        if self.active:
            self.x += self.velocity_x * dt
            self.y += self.velocity_y * dt

class BulletPool:
    def __init__(self, size=100):
        # Pre-allocate all bullets
        self.pool = [Bullet() for _ in range(size)]
        self.active_bullets = []

    def get_bullet(self):
        """Get an inactive bullet from the pool"""
        for bullet in self.pool:
            if not bullet.active:
                self.active_bullets.append(bullet)
                return bullet

        # Pool exhausted - could expand or return None
        print("Warning: Bullet pool exhausted!")
        return None

    def spawn_bullet(self, x, y, velocity_x, velocity_y):
        """Convenience method to spawn a bullet"""
        bullet = self.get_bullet()
        if bullet:
            bullet.activate(x, y, velocity_x, velocity_y)
        return bullet

    def update(self, dt):
        """Update all active bullets"""
        for bullet in self.active_bullets[:]:  # Copy list to allow modification
            bullet.update(dt)

            # Check if bullet should be deactivated
            if self.is_off_screen(bullet) or bullet.has_hit_target():
                bullet.deactivate()
                self.active_bullets.remove(bullet)

    def is_off_screen(self, bullet):
        # Check bounds
        return (bullet.x < -100 or bullet.x > 1920 or
                bullet.y < -100 or bullet.y > 1080)
```

## Advanced: Generic Pool

```python
class ObjectPool:
    """Generic object pool for any type"""
    def __init__(self, object_type, initial_size=50, max_size=None):
        self.object_type = object_type
        self.max_size = max_size
        self.available = []
        self.in_use = set()

        # Pre-allocate
        for _ in range(initial_size):
            obj = object_type()
            self.available.append(obj)

    def acquire(self):
        """Get an object from the pool"""
        if self.available:
            obj = self.available.pop()
        elif self.max_size is None or len(self.in_use) < self.max_size:
            # Create new object if pool can grow
            obj = self.object_type()
        else:
            # Pool exhausted and at max size
            return None

        self.in_use.add(obj)
        return obj

    def release(self, obj):
        """Return an object to the pool"""
        if obj in self.in_use:
            self.in_use.remove(obj)

            # Reset object state if it has a reset method
            if hasattr(obj, 'reset'):
                obj.reset()

            self.available.append(obj)

    def release_all(self):
        """Return all objects to the pool"""
        for obj in list(self.in_use):
            self.release(obj)

# Usage
bullet_pool = ObjectPool(Bullet, initial_size=100, max_size=200)
particle_pool = ObjectPool(Particle, initial_size=500, max_size=1000)

# Get object
bullet = bullet_pool.acquire()
if bullet:
    bullet.activate(x, y, vx, vy)

# Return object
bullet_pool.release(bullet)
```

## Common Game Uses

### 1. Projectiles (Bullets, Arrows, Missiles)

```python
class ProjectileManager:
    def __init__(self):
        self.bullet_pool = ObjectPool(Bullet, initial_size=100)
        self.rocket_pool = ObjectPool(Rocket, initial_size=20)
        self.arrow_pool = ObjectPool(Arrow, initial_size=50)

    def fire_weapon(self, weapon_type, x, y, target_x, target_y):
        if weapon_type == "gun":
            projectile = self.bullet_pool.acquire()
        elif weapon_type == "launcher":
            projectile = self.rocket_pool.acquire()
        elif weapon_type == "bow":
            projectile = self.arrow_pool.acquire()

        if projectile:
            projectile.fire(x, y, target_x, target_y)
            return projectile
        return None
```

### 2. Particle Systems

```python
class Particle:
    def __init__(self):
        self.x = 0
        self.y = 0
        self.velocity_x = 0
        self.velocity_y = 0
        self.lifetime = 0
        self.max_lifetime = 1.0
        self.color = (255, 255, 255)
        self.size = 1
        self.active = False

    def reset(self):
        self.active = False
        self.lifetime = 0

    def update(self, dt):
        if self.active:
            self.x += self.velocity_x * dt
            self.y += self.velocity_y * dt
            self.lifetime += dt

            if self.lifetime >= self.max_lifetime:
                self.active = False

class ParticleEmitter:
    def __init__(self, max_particles=1000):
        self.pool = ObjectPool(Particle, initial_size=max_particles)
        self.particles = []

    def emit(self, x, y, count=10):
        """Emit particles from a position"""
        import random
        for _ in range(count):
            particle = self.pool.acquire()
            if particle:
                particle.x = x
                particle.y = y
                particle.velocity_x = random.uniform(-100, 100)
                particle.velocity_y = random.uniform(-100, 100)
                particle.lifetime = 0
                particle.max_lifetime = random.uniform(0.5, 2.0)
                particle.active = True
                self.particles.append(particle)

    def update(self, dt):
        for particle in self.particles[:]:
            particle.update(dt)
            if not particle.active:
                self.pool.release(particle)
                self.particles.remove(particle)
```

### 3. Enemy Spawning

```python
class Enemy:
    def __init__(self):
        self.x = 0
        self.y = 0
        self.health = 100
        self.active = False
        self.enemy_type = "basic"

    def spawn(self, x, y, enemy_type="basic"):
        self.x = x
        self.y = y
        self.enemy_type = enemy_type
        self.health = 100
        self.active = True

    def reset(self):
        self.active = False
        self.health = 100

class WaveManager:
    def __init__(self):
        self.enemy_pool = ObjectPool(Enemy, initial_size=50, max_size=100)
        self.active_enemies = []

    def spawn_wave(self, wave_number):
        enemy_count = wave_number * 5
        for i in range(enemy_count):
            enemy = self.enemy_pool.acquire()
            if enemy:
                spawn_x = i * 100
                spawn_y = 0
                enemy.spawn(spawn_x, spawn_y, "basic")
                self.active_enemies.append(enemy)

    def update(self, dt):
        for enemy in self.active_enemies[:]:
            if enemy.health <= 0:
                self.enemy_pool.release(enemy)
                self.active_enemies.remove(enemy)
```

### 4. Audio Sources

```python
class AudioSource:
    def __init__(self):
        self.sound = None
        self.playing = False
        self.volume = 1.0
        self.position = (0, 0, 0)

    def play(self, sound_name, position=(0, 0, 0), volume=1.0):
        self.sound = sound_name
        self.position = position
        self.volume = volume
        self.playing = True
        # Actual audio playback code...

    def reset(self):
        self.playing = False
        self.sound = None

class AudioManager:
    def __init__(self, max_sources=32):
        self.source_pool = ObjectPool(AudioSource, initial_size=max_sources)

    def play_sound(self, sound_name, position=(0, 0, 0), volume=1.0):
        source = self.source_pool.acquire()
        if source:
            source.play(sound_name, position, volume)
            return source
        else:
            # All sources busy - could prioritize and steal one
            print("No audio sources available")
            return None
```

### 5. VFX Effects

```python
class Effect:
    def __init__(self):
        self.x = 0
        self.y = 0
        self.effect_type = None
        self.animation_time = 0
        self.duration = 1.0
        self.active = False

    def play(self, effect_type, x, y, duration=1.0):
        self.effect_type = effect_type
        self.x = x
        self.y = y
        self.duration = duration
        self.animation_time = 0
        self.active = True

    def update(self, dt):
        if self.active:
            self.animation_time += dt
            if self.animation_time >= self.duration:
                self.active = False

    def reset(self):
        self.active = False

class EffectsManager:
    def __init__(self):
        self.pool = ObjectPool(Effect, initial_size=50)
        self.active_effects = []

    def play_effect(self, effect_type, x, y, duration=1.0):
        effect = self.pool.acquire()
        if effect:
            effect.play(effect_type, x, y, duration)
            self.active_effects.append(effect)

    def update(self, dt):
        for effect in self.active_effects[:]:
            effect.update(dt)
            if not effect.active:
                self.pool.release(effect)
                self.active_effects.remove(effect)
```

## Best Practices

1. **Pre-allocate at Load Time**: Create pool during level load, not gameplay
2. **Monitor Pool Usage**: Log warnings when pool is exhausted
3. **Reset State Properly**: Always reset objects when returning to pool
4. **Choose Appropriate Sizes**: Profile to find optimal pool sizes
5. **Consider Growing Pools**: Allow expansion for rare spikes
6. **Pool by Type**: Separate pools for different object types

## Performance Comparison

```python
import time

# Without pooling
def benchmark_no_pool(iterations=10000):
    start = time.time()
    bullets = []
    for i in range(iterations):
        bullet = Bullet()
        bullets.append(bullet)
        if len(bullets) > 100:
            bullets.pop(0)  # Remove old bullets
    end = time.time()
    return end - start

# With pooling
def benchmark_with_pool(iterations=10000):
    pool = BulletPool(size=100)
    start = time.time()
    for i in range(iterations):
        bullet = pool.get_bullet()
        if bullet:
            bullet.activate(0, 0, 1, 1)
        # Simulate some bullets deactivating
        if i % 10 == 0 and pool.active_bullets:
            pool.active_bullets[0].deactivate()
            pool.active_bullets.pop(0)
    end = time.time()
    return end - start

# Pooling is typically 3-10x faster and produces no garbage
```

## When to Use Object Pooling

**Good for:**
- Projectiles/bullets
- Particles
- Audio sources
- VFX effects
- Enemies in wave-based games
- UI elements that appear/disappear frequently

**Not necessary for:**
- Objects created once (player, managers)
- Level geometry
- Infrequently created objects
- Objects with complex setup/teardown

## Integration with ECS

```python
# Pool can work alongside ECS
class EntityPool:
    def __init__(self, registry, initial_size=100):
        self.registry = registry
        self.available_entities = []

        # Pre-create entities
        for _ in range(initial_size):
            entity = registry.create_entity()
            entity.deactivate()
            self.available_entities.append(entity)

    def spawn_entity(self, components):
        if self.available_entities:
            entity = self.available_entities.pop()
            entity.activate()
            for component in components:
                entity.add_component(component)
            return entity
        return None

    def despawn_entity(self, entity):
        entity.remove_all_components()
        entity.deactivate()
        self.available_entities.append(entity)
```

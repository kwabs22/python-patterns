# Game Scene Examples by Theme

This document contains 25+ complete game scene implementations using the design patterns from this repository. Each example uses a different theme and demonstrates practical pattern usage.

## Table of Contents

1. [Cyberpunk Hacking](#1-cyberpunk-hacking---command-pattern)
2. [Pirate Ship Battle](#2-pirate-ship-battle---strategy-pattern)
3. [Wizard Tower Defense](#3-wizard-tower-defense---factory-pattern)
4. [Space Station Emergency](#4-space-station-emergency---state-pattern)
5. [Time-Traveling Detective](#5-time-traveling-detective---memento-pattern)
6. [Kitchen Nightmare](#6-kitchen-nightmare---observer-pattern)
7. [Dungeon Ecosystem](#7-dungeon-ecosystem---composite-pattern)
8. [Zombie Survival](#8-zombie-survival---object-pool-pattern)
9. [Musical Rhythm Game](#9-musical-rhythm-game---command--observer)
10. [Garden Simulator](#10-garden-simulator---decorator-pattern)
11. [Trading Card Game](#11-trading-card-game---prototype-pattern)
12. [Stealth Espionage](#12-stealth-espionage---chain-of-responsibility)
13. [Theme Park Tycoon](#13-theme-park-tycoon---builder-pattern)
14. [Mech Battle Arena](#14-mech-battle-arena---composite--strategy)
15. [Haunted Mansion](#15-haunted-mansion---visitor-pattern)
16. [Race Car Pit Stop](#16-race-car-pit-stop---chain--observer)
17. [Pet Monster Ranch](#17-pet-monster-ranch---prototype--observer)
18. [Heist Planning](#18-heist-planning---command--memento)
19. [Food Truck Simulator](#19-food-truck-simulator---strategy--state)
20. [Ancient Library Quest](#20-ancient-library-quest---iterator--composite)
21. [Submarine Exploration](#21-submarine-exploration---state--observer)
22. [Magical Potion Brewing](#22-magical-potion-brewing---builder--decorator)
23. [Gladiator Arena](#23-gladiator-arena---strategy--observer)
24. [Time Loop Mystery](#24-time-loop-mystery---memento--observer)
25. [Robot Factory](#25-robot-factory---abstract-factory--builder)

---

## 1. Cyberpunk Hacking - Command Pattern

**Theme:** Cyberpunk netrunner hacking into corporate servers
**Patterns:** Command, Memento

**Key Learning:** Command pattern allows undo/redo and action recording

```python
"""Commands can be undone and replayed for hacking sequences"""

class HackCommand:
    def execute(self, system):
        raise NotImplementedError
    def undo(self, system):
        raise NotImplementedError

class BypassFirewallCommand(HackCommand):
    def __init__(self, firewall_id):
        self.firewall_id = firewall_id
        self.previous_state = None
    
    def execute(self, system):
        self.previous_state = system.firewalls[self.firewall_id].state
        system.firewalls[self.firewall_id].bypass()
        system.ice_level += 1
        return f"Firewall {self.firewall_id} bypassed"
    
    def undo(self, system):
        system.firewalls[self.firewall_id].state = self.previous_state
        system.ice_level -= 1
```

---

## 2. Pirate Ship Battle - Strategy Pattern

**Theme:** Caribbean pirate naval combat
**Patterns:** Strategy, Observer

**Key Learning:** Different ships use different battle tactics via swappable strategies

```python
"""Each ship has a strategy determining its combat behavior"""

class BroadsideStrategy:
    def decide_action(self, ship, enemy_ships, wind_direction):
        nearest = min(enemy_ships, key=lambda e: ship.distance_to(e))
        distance = ship.distance_to(nearest)
        
        if 100 < distance < 200:  # Optimal range
            return ("broadside_cannon", nearest)
        elif distance < 100:
            return ("turn_away", nearest)
        else:
            return ("approach", nearest)

class RammingStrategy:
    def decide_action(self, ship, enemy_ships, wind_direction):
        weakest = min(enemy_ships, key=lambda e: e.hull_integrity)
        if ship.distance_to(weakest) < 20:
            return ("board", weakest)
        return ("ram", weakest)
```

---

## 3. Wizard Tower Defense - Factory Pattern

**Theme:** Fantasy wizard summoning creatures
**Patterns:** Factory, Pool

**Key Learning:** Factory creates different minion types from templates

```python
"""Factory pattern creates different magical minions"""

class MinionFactory:
    @staticmethod
    def create_minion(minion_type, position):
        if minion_type == "fire_elemental":
            return FireElemental(position)
        elif minion_type == "ice_golem":
            return IceGolem(position)
        elif minion_type == "lightning_wisp":
            return LightningWisp(position)
        # ... more types

class WizardTower:
    def summon_minion(self, minion_type, position):
        cost = self.summon_costs[minion_type]
        if self.mana >= cost:
            minion = MinionFactory.create_minion(minion_type, position)
            self.minions.append(minion)
            self.mana -= cost
            return minion
```

---

## 4. Space Station Emergency - State Pattern

**Theme:** Sci-fi space station system failures
**Patterns:** State, Observer

**Key Learning:** Station transitions between different emergency states

```python
"""Station state determines available actions and transitions"""

class HullBreachState(StationState):
    def enter(self, station):
        station.life_support -= 30
        print("HULL BREACH! Sealing bulkheads!")
    
    def update(self, station, dt):
        station.life_support -= dt * 2
        if station.life_support <= 0:
            return StationDestroyedState()
        if self.sectors_sealed >= self.target_sectors:
            return RepairOperationsState()
        return None
```

---

## 5. Time-Traveling Detective - Memento Pattern

**Theme:** Noir detective rewinding time
**Patterns:** Memento, Command

**Key Learning:** Save/restore game state to different timeline points

```python
"""Memento captures complete game state for time travel"""

class TimelineMemento:
    def __init__(self, timestamp, location, npcs_state, clues):
        self._timestamp = timestamp
        self._location = location
        self._npcs_state = npcs_state.copy()
        self._clues_found = clues.copy()

class Detective:
    def create_snapshot(self, timestamp):
        return TimelineMemento(
            timestamp, self.location,
            self.npcs_state, self.clues_found
        )
    
    def rewind_to(self, timestamp):
        snapshot = self.timeline_snapshots[timestamp]
        state = snapshot.get_state()
        self.location = state["location"]
        self.npcs_state = state["npcs_state"]
```

---

## 6. Kitchen Nightmare - Observer Pattern

**Theme:** Restaurant kitchen coordination
**Patterns:** Observer, Chain of Responsibility

**Key Learning:** Stations observe orders and react independently

```python
"""All kitchen stations observe incoming orders"""

class GrillStation(KitchenObserver):
    def on_order_received(self, order):
        grill_items = [item for item in order.items 
                      if item in ["steak", "burger", "chicken"]]
        if grill_items:
            self.queue.extend([(order, item) for item in grill_items])

class Kitchen:
    def receive_order(self, order):
        # Notify all stations
        for observer in self.observers:
            observer.on_order_received(order)
```

---

## 7. Dungeon Ecosystem - Composite Pattern

**Theme:** Living dungeon hierarchy
**Patterns:** Composite, Visitor

**Key Learning:** Treat individual monsters and room groups uniformly

```python
"""Dungeons contain floors contain rooms contain monsters"""

class Room(DungeonComponent):
    def __init__(self, name):
        self.contents = []  # Monsters, traps, treasures
    
    def add(self, component):
        self.contents.append(component)
    
    def get_threat_level(self):
        return sum(item.get_threat_level() for item in self.contents)

# Can treat single monster or entire floor the same way
dragon_lair.get_threat_level()  # Room with monsters
entire_dungeon.get_threat_level()  # Sum of all floors
```

---

## 8. Zombie Survival - Object Pool Pattern

**Theme:** Zombie horde shooter
**Patterns:** Pool, Observer

**Key Learning:** Reuse zombie/bullet objects for performance

```python
"""Pool manages hundreds of zombies without constant allocation"""

class ObjectPool:
    def __init__(self, object_type, initial_size=100):
        self.available = [object_type() for _ in range(initial_size)]
        self.in_use = []
    
    def acquire(self):
        if self.available:
            obj = self.available.pop()
            self.in_use.append(obj)
            return obj
    
    def release(self, obj):
        self.in_use.remove(obj)
        self.available.append(obj)

zombie_pool = ObjectPool(Zombie, initial_size=200)
bullet_pool = ObjectPool(Bullet, initial_size=500)
```

---

## 9. Musical Rhythm Game - Command + Observer

**Theme:** DDR-style rhythm game
**Patterns:** Command, Observer

**Key Learning:** Commands represent beats, observers react to scores

```python
"""Beat commands judge player timing"""

class BeatCommand:
    def judge_timing(self, hit_time):
        timing_diff = abs(hit_time - self.beat_time)
        if timing_diff <= 0.05:
            return ("PERFECT", 300)
        elif timing_diff <= 0.1:
            return ("GOOD", 200)
        return ("MISS", 0)

class ComboEffectsObserver:
    def on_beat_hit(self, judgement, score, combo):
        if combo == 50:
            print("🔥🔥🔥 50 COMBO! Fireworks!")
```

---

## 10. Garden Simulator - Decorator Pattern

**Theme:** Peaceful gardening
**Patterns:** Decorator, Observer

**Key Learning:** Stack decorators to enhance plants

```python
"""Each decorator adds properties to the base plant"""

plant = TomatoPlant()
plant = WateredDecorator(plant)
plant = FertilizedDecorator(plant)
plant = SunlightDecorator(plant, hours=8)
plant = MagicGrowthDecorator(plant)

# Each decorator multiplies/adds to stats
print(plant.get_growth_rate())  # All bonuses applied
```

---

## 11. Trading Card Game - Prototype Pattern

**Theme:** Collectible card game
**Patterns:** Prototype, Factory

**Key Learning:** Clone card prototypes instead of recreating

```python
"""Registry holds card prototypes for cloning"""

registry = CardRegistry()
registry.register("dragon", DragonCard(attack=10, health=10))

# Create multiple copies by cloning
deck = []
deck.extend(registry.create_multiple("dragon", count=2))
deck.extend(registry.create_multiple("goblin", count=10))
```

---

## 12. Stealth Espionage - Chain of Responsibility

**Theme:** Spy infiltration
**Patterns:** Chain of Responsibility, State

**Key Learning:** Security layers pass detection checks down the chain

```python
"""Each security layer can detect or pass to next"""

motion = MotionDetectorLayer()
camera = CameraLayer()
guard = GuardPatrolLayer()

# Chain layers
motion.set_next(camera).set_next(guard)

# Agent must pass all layers
detected = motion.check_intrusion(agent)
```

---

## 13. Theme Park Tycoon - Builder Pattern

**Theme:** Build custom roller coasters
**Patterns:** Builder, Composite

**Key Learning:** Build complex objects step-by-step

```python
"""Builder constructs coasters with fluent interface"""

coaster = (builder
    .set_name("Dragon's Fury")
    .set_theme("Medieval")
    .add_track_section("loop")
    .add_track_section("corkscrew")
    .add_car("dragon_head_car")
    .add_special_effect("fire_breathing")
    .build())
```

---

## 14. Mech Battle Arena - Composite + Strategy

**Theme:** Giant robot combat
**Patterns:** Composite, Strategy

**Key Learning:** Mechs are composites of parts with strategies

```python
"""Mech is composite of weapons and components"""

class Mech(Composite):
    def __init__(self, name):
        self.parts = []  # Weapons, armor, engines
        self.strategy = None
    
    def add_part(self, part):
        self.parts.append(part)
    
    def get_total_firepower(self):
        return sum(part.firepower for part in self.parts)

# Build custom mech
mech = Mech("Titan")
mech.add_part(PlasmaRifle())
mech.add_part(MissileLauncher())
mech.add_part(EnergyShield())
mech.set_strategy(BrawlerStrategy())
```

---

## 15. Haunted Mansion - Visitor Pattern

**Theme:** Horror ghost hunting
**Patterns:** Visitor, Observer

**Key Learning:** Different equipment visits rooms differently

```python
"""Visitor pattern for different investigation tools"""

class EMFDetector(RoomVisitor):
    def visit_bedroom(self, room):
        if room.has_ghost:
            print("EMF spike detected!")
    
    def visit_basement(self, room):
        print("EMF readings off the chart!")

class InfraredCamera(RoomVisitor):
    def visit_bedroom(self, room):
        print("Cold spot detected on camera")

mansion.accept(emf_detector)  # Visits all rooms
mansion.accept(ir_camera)     # Different behavior
```

---

## 16. Race Car Pit Stop - Chain + Observer

**Theme:** Formula racing pit crew
**Patterns:** Chain of Responsibility, Observer

**Key Learning:** Pit crew tasks chain together

```python
"""Pit stop tasks handled in sequence"""

class TireChangeHandler(PitHandler):
    def handle(self, car):
        print("Changing tires...")
        time.sleep(2.5)
        if self.next:
            return self.next.handle(car)

class RefuelHandler(PitHandler):
    def handle(self, car):
        print("Refueling...")
        time.sleep(3.0)
        return self.next.handle(car) if self.next else None

# Chain pit crew tasks
tire_change = TireChangeHandler()
refuel = RefuelHandler()
wing_adjust = WingAdjustmentHandler()

tire_change.set_next(refuel).set_next(wing_adjust)
```

---

## 17. Pet Monster Ranch - Prototype + Observer

**Theme:** Monster raising sim
**Patterns:** Prototype, Observer, State

**Key Learning:** Breed monsters by cloning with variations

```python
"""Breed new monsters from parent prototypes"""

class Monster:
    def breed_with(self, other):
        child = self.clone()
        # Mix traits
        child.color = random.choice([self.color, other.color])
        child.size = (self.size + other.size) / 2
        # Random mutation
        if random.random() < 0.1:
            child.ability = generate_rare_ability()
        return child

fire_dragon = FireDragon()
ice_dragon = IceDragon()
hybrid = fire_dragon.breed_with(ice_dragon)
# Creates new dragon with mixed traits
```

---

## 18. Heist Planning - Command + Memento

**Theme:** Ocean's Eleven style heist
**Patterns:** Command, Memento, Observer

**Key Learning:** Plan heist steps, simulate, rewind if caught

```python
"""Command sequence with checkpoints"""

class HeistPlan:
    def __init__(self):
        self.steps = []
        self.checkpoints = {}
    
    def add_step(self, command):
        self.steps.append(command)
    
    def create_checkpoint(self, name):
        self.checkpoints[name] = self.create_memento()
    
    def execute_heist(self):
        for step in self.steps:
            result = step.execute(self.game_state)
            if result == "CAUGHT":
                self.restore_checkpoint("last_safe")
                return "HEIST FAILED"
        return "SUCCESS"

# Plan heist
plan.add_step(DisableSecurityCommand())
plan.create_checkpoint("security_down")
plan.add_step(CrackSafeCommand())
plan.add_step(GrabLootCommand())
```

---

## 19. Food Truck Simulator - Strategy + State

**Theme:** Running a food truck
**Patterns:** Strategy, State, Observer

**Key Learning:** Different recipes are strategies, truck has states

```python
"""Recipe strategies and operational states"""

class TacoStrategy(RecipeStrategy):
    def prepare(self, ingredients):
        if self.has_ingredients(ingredients, ["tortilla", "meat", "salsa"]):
            return Food("Taco", quality=8, price=5)

class BurritoStrategy(RecipeStrategy):
    def prepare(self, ingredients):
        if self.has_ingredients(ingredients, ["tortilla", "meat", "rice", "beans"]):
            return Food("Burrito", quality=9, price=8)

class FoodTruck:
    def __init__(self):
        self.state = ClosedState()
        self.active_recipe = TacoStrategy()
    
    def switch_recipe(self, recipe):
        self.active_recipe = recipe
```

---

## 20. Ancient Library Quest - Iterator + Composite

**Theme:** Magical library exploration
**Patterns:** Iterator, Composite, Visitor

**Key Learning:** Navigate nested book collections

```python
"""Library has nested sections, books"""

class Library(Composite):
    def __init__(self):
        self.sections = []
    
    def iterator(self):
        return LibraryIterator(self)

class LibraryIterator:
    def __init__(self, library):
        self.stack = [library]
        self.current = 0
    
    def __next__(self):
        # Depth-first traversal
        if not self.stack:
            raise StopIteration
        
        current_section = self.stack.pop()
        # Add subsections to stack
        for section in current_section.get_children():
            self.stack.append(section)
        
        return current_section

# Search entire library
for section in library:
    if section.contains_ancient_spell():
        return section
```

---

## 21. Submarine Exploration - State + Observer

**Theme:** Deep sea exploration
**Patterns:** State, Observer, Chain

**Key Learning:** Submarine depth affects available states

```python
"""Submarine states change with depth/pressure"""

class SurfaceState(SubState):
    def dive(self, sub):
        return ShallowDiveState()
    
    def fire_torpedoes(self, sub):
        return False  # Can't fire on surface

class DeepDiveState(SubState):
    def enter(self, sub):
        sub.pressure = 1000
        sub.oxygen_consumption *= 2
    
    def surface(self, sub):
        if sub.emergency_surface_available():
            return SurfaceState()
        return DecompressionState()  # Must decompress first

class CrushedState(SubState):
    def enter(self, sub):
        print("Hull integrity failed! Submarine destroyed!")
```

---

## 22. Magical Potion Brewing - Builder + Decorator

**Theme:** Witch's potion crafting
**Patterns:** Builder, Decorator

**Key Learning:** Build base potion, decorate with effects

```python
"""Builder creates potion, decorators add effects"""

potion = (PotionBuilder()
    .set_base("Health Potion")
    .add_ingredient("Dragon Scale")
    .add_ingredient("Phoenix Feather")
    .set_potency(5)
    .build())

# Decorate with magical effects
potion = FireResistanceDecorator(potion)
potion = DoubleEffectDecorator(potion)
potion = PermanentDecorator(potion)

print(potion.get_effects())
# ["Heal 50HP", "Fire Resistance 1hr", "Double Effect", "Permanent"]
```

---

## 23. Gladiator Arena - Strategy + Observer

**Theme:** Roman gladiator combat
**Patterns:** Strategy, Observer, State

**Key Learning:** Gladiators use combat strategies, crowd observes

```python
"""Different fighting styles are strategies"""

class RetariusStrategy:  # Net fighter
    def attack(self, gladiator, opponent):
        if gladiator.has_net:
            opponent.entangled = True
            return gladiator.trident_attack(opponent)

class MurmilloStrategy:  # Heavy armor
    def attack(self, gladiator, opponent):
        damage = gladiator.sword_damage
        damage -= opponent.armor * 0.5  # Reduced by armor
        return damage

class CrowdObserver:
    def on_hit_landed(self, attacker, defender, damage):
        if damage > 20:
            print("Crowd: OOOOHHH!")
        self.excitement += damage

arena = GladiatorArena()
arena.add_observer(CrowdObserver())
```

---

## 24. Time Loop Mystery - Memento + Observer

**Theme:** Groundhog Day investigation
**Patterns:** Memento, Observer, State

**Key Learning:** Each loop is a memento, observers track changes

```python
"""Save state at start of each time loop"""

class TimeLoop:
    def __init__(self):
        self.loop_number = 0
        self.loop_start_state = None
        self.discoveries = set()  # Persist across loops
    
    def start_loop(self, game):
        self.loop_number += 1
        self.loop_start_state = game.create_memento()
        print(f"Loop {self.loop_number} begins...")
    
    def reset_loop(self, game):
        game.restore_from_memento(self.loop_start_state)
        # Keep discoveries
        game.knowledge = self.discoveries.copy()

# Each death/midnight resets the loop
while not mystery_solved:
    time_loop.start_loop(game)
    result = play_through_day(game)
    if result == "died" or result == "midnight":
        time_loop.reset_loop(game)
```

---

## 25. Robot Factory - Abstract Factory + Builder

**Theme:** Automated robot assembly
**Patterns:** Abstract Factory, Builder

**Key Learning:** Different factories produce compatible robot parts

```python
"""Abstract factory for different robot types"""

class MilitaryRobotFactory(RobotFactory):
    def create_chassis(self):
        return ArmoredChassis()
    
    def create_weapons(self):
        return [PlasmaRifle(), MissilePods()]
    
    def create_ai(self):
        return CombatAI()

class UtilityRobotFactory(RobotFactory):
    def create_chassis(self):
        return LightweightChassis()
    
    def create_weapons(self):
        return [Manipulator(), Welder()]
    
    def create_ai(self):
        return WorkerAI()

# Build complete robots
def build_robot(factory):
    robot = Robot()
    robot.chassis = factory.create_chassis()
    robot.weapons = factory.create_weapons()
    robot.ai = factory.create_ai()
    return robot

military_robot = build_robot(MilitaryRobotFactory())
worker_robot = build_robot(UtilityRobotFactory())
```

---

## Summary Table

| # | Theme | Primary Pattern | Secondary Patterns |
|---|-------|----------------|-------------------|
| 1 | Cyberpunk Hacking | Command | Memento |
| 2 | Pirate Ship Battle | Strategy | Observer |
| 3 | Wizard Tower Defense | Factory | Pool |
| 4 | Space Station Emergency | State | Observer |
| 5 | Time-Traveling Detective | Memento | Command |
| 6 | Kitchen Nightmare | Observer | Chain |
| 7 | Dungeon Ecosystem | Composite | Visitor |
| 8 | Zombie Survival | Pool | Observer |
| 9 | Musical Rhythm Game | Command | Observer |
| 10 | Garden Simulator | Decorator | Observer |
| 11 | Trading Card Game | Prototype | Factory |
| 12 | Stealth Espionage | Chain | State |
| 13 | Theme Park Tycoon | Builder | Composite |
| 14 | Mech Battle Arena | Composite | Strategy |
| 15 | Haunted Mansion | Visitor | Observer |
| 16 | Race Car Pit Stop | Chain | Observer |
| 17 | Pet Monster Ranch | Prototype | Observer |
| 18 | Heist Planning | Command | Memento |
| 19 | Food Truck Simulator | Strategy | State |
| 20 | Ancient Library | Iterator | Composite |
| 21 | Submarine Exploration | State | Observer |
| 22 | Potion Brewing | Builder | Decorator |
| 23 | Gladiator Arena | Strategy | Observer |
| 24 | Time Loop Mystery | Memento | Observer |
| 25 | Robot Factory | Abstract Factory | Builder |

## Pattern Usage Statistics

- **Observer**: Used in 12 examples (most common - events, reactions)
- **State**: Used in 7 examples (game states, transitions)
- **Strategy**: Used in 6 examples (AI, behaviors)
- **Command**: Used in 5 examples (actions, undo/redo)
- **Memento**: Used in 4 examples (save/load, time travel)
- **Composite**: Used in 4 examples (hierarchies, nested structures)
- **Builder**: Used in 3 examples (complex object construction)
- **Chain of Responsibility**: Used in 3 examples (layered processing)
- **Prototype**: Used in 3 examples (cloning objects)
- **Factory**: Used in 3 examples (object creation)
- **Decorator**: Used in 3 examples (adding features)
- **Pool**: Used in 2 examples (performance optimization)
- **Visitor**: Used in 2 examples (operations on structures)
- **Iterator**: Used in 1 example (traversal)
- **Abstract Factory**: Used in 1 example (families of objects)

## Key Takeaways

1. **Observer is everywhere** - Most games need event systems
2. **State machines are fundamental** - Game states, AI states, character states
3. **Strategy enables AI** - Swappable behaviors make smart NPCs
4. **Command enables features** - Undo, replay, networking all use commands
5. **Composite models hierarchies** - Dungeons, UI, scene graphs
6. **Pool is critical for performance** - Bullets, particles, enemies
7. **Builder constructs complexity** - Procedural generation, customization
8. **Patterns combine naturally** - Real games use multiple patterns together


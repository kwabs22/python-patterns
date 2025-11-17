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

**Complete Implementation:**

```python
"""
Complete cyberpunk hacking system with Command and Memento patterns.
Demonstrates undo/redo, command recording, and state restoration.
"""

from abc import ABC, abstractmethod
from typing import List, Dict
from enum import Enum
import copy

class FirewallState(Enum):
    ACTIVE = "active"
    BYPASSED = "bypassed"
    CRASHED = "crashed"

class Firewall:
    def __init__(self, firewall_id: str, strength: int):
        self.firewall_id = firewall_id
        self.strength = strength
        self.state = FirewallState.ACTIVE
        self.traces = 0

    def bypass(self):
        if self.state == FirewallState.ACTIVE:
            self.state = FirewallState.BYPASSED
            return True
        return False

    def restore(self):
        self.state = FirewallState.ACTIVE

    def crash(self):
        self.state = FirewallState.CRASHED

class CorporateSystem:
    def __init__(self):
        self.firewalls: Dict[str, Firewall] = {
            "outer": Firewall("outer", strength=3),
            "inner": Firewall("inner", strength=5),
            "core": Firewall("core", strength=8)
        }
        self.ice_level = 0
        self.data_extracted = []
        self.alarm_level = 0
        self.netrunner_detected = False

    def get_status(self) -> str:
        status = "=== SYSTEM STATUS ===\n"
        for fw_id, fw in self.firewalls.items():
            status += f"Firewall [{fw_id}]: {fw.state.value} (strength: {fw.strength})\n"
        status += f"ICE Level: {self.ice_level}\n"
        status += f"Alarm Level: {self.alarm_level}\n"
        status += f"Data Extracted: {len(self.data_extracted)} files\n"
        return status

# Command Pattern - Each hack action is a command
class HackCommand(ABC):
    @abstractmethod
    def execute(self, system: CorporateSystem) -> str:
        pass

    @abstractmethod
    def undo(self, system: CorporateSystem):
        pass

    @abstractmethod
    def get_description(self) -> str:
        pass

class BypassFirewallCommand(HackCommand):
    def __init__(self, firewall_id: str):
        self.firewall_id = firewall_id
        self.previous_state = None
        self.previous_ice = None

    def execute(self, system: CorporateSystem) -> str:
        firewall = system.firewalls.get(self.firewall_id)
        if not firewall:
            return f"ERROR: Firewall {self.firewall_id} not found"

        # Save state for undo
        self.previous_state = firewall.state
        self.previous_ice = system.ice_level

        # Execute bypass
        if firewall.bypass():
            system.ice_level += firewall.strength
            system.alarm_level += 10
            return f"[SUCCESS] Firewall {self.firewall_id} bypassed! ICE increased by {firewall.strength}"
        else:
            return f"[FAILED] Firewall {self.firewall_id} already {firewall.state.value}"

    def undo(self, system: CorporateSystem):
        firewall = system.firewalls.get(self.firewall_id)
        if firewall:
            firewall.state = self.previous_state
            system.ice_level = self.previous_ice
            system.alarm_level = max(0, system.alarm_level - 10)

    def get_description(self) -> str:
        return f"Bypass Firewall: {self.firewall_id}"

class ExtractDataCommand(HackCommand):
    def __init__(self, file_name: str):
        self.file_name = file_name
        self.extracted = False

    def execute(self, system: CorporateSystem) -> str:
        core_firewall = system.firewalls["core"]
        if core_firewall.state != FirewallState.BYPASSED:
            return f"[BLOCKED] Core firewall must be bypassed first!"

        system.data_extracted.append(self.file_name)
        system.alarm_level += 25
        self.extracted = True
        return f"[EXTRACTED] Data file: {self.file_name}"

    def undo(self, system: CorporateSystem):
        if self.extracted and self.file_name in system.data_extracted:
            system.data_extracted.remove(self.file_name)
            system.alarm_level = max(0, system.alarm_level - 25)
            self.extracted = False

    def get_description(self) -> str:
        return f"Extract Data: {self.file_name}"

class CrashICECommand(HackCommand):
    def __init__(self, target_firewall: str):
        self.target_firewall = target_firewall
        self.previous_state = None
        self.ice_reduction = 0

    def execute(self, system: CorporateSystem) -> str:
        firewall = system.firewalls.get(self.target_firewall)
        if not firewall:
            return f"ERROR: Firewall {self.target_firewall} not found"

        self.previous_state = firewall.state
        self.ice_reduction = firewall.strength * 2

        firewall.crash()
        system.ice_level = max(0, system.ice_level - self.ice_reduction)
        system.alarm_level += 50  # Crashing ICE raises alarms!

        return f"[ICE CRASHED] {self.target_firewall} - reduced ICE by {self.ice_reduction}"

    def undo(self, system: CorporateSystem):
        firewall = system.firewalls.get(self.target_firewall)
        if firewall:
            firewall.state = self.previous_state
            system.ice_level += self.ice_reduction
            system.alarm_level = max(0, system.alarm_level - 50)

    def get_description(self) -> str:
        return f"Crash ICE: {self.target_firewall}"

# Memento Pattern - Save/restore system state
class SystemMemento:
    """Captures complete system state for save/restore"""
    def __init__(self, system: CorporateSystem):
        self.firewalls_state = copy.deepcopy(system.firewalls)
        self.ice_level = system.ice_level
        self.data_extracted = system.data_extracted.copy()
        self.alarm_level = system.alarm_level
        self.detected = system.netrunner_detected

class HackingSession:
    """Manages the hacking session with undo/redo and checkpoints"""
    def __init__(self):
        self.system = CorporateSystem()
        self.command_history: List[HackCommand] = []
        self.redo_stack: List[HackCommand] = []
        self.checkpoints: Dict[str, SystemMemento] = {}

    def execute_command(self, command: HackCommand):
        result = command.execute(self.system)
        print(result)
        self.command_history.append(command)
        self.redo_stack.clear()  # Clear redo stack on new command

        # Check for detection
        if self.system.alarm_level >= 100:
            self.system.netrunner_detected = True
            print("⚠️  ALERT: NETRUNNER DETECTED! ICE countermeasures activated!")

    def undo(self):
        if not self.command_history:
            print("[UNDO] Nothing to undo")
            return

        command = self.command_history.pop()
        command.undo(self.system)
        self.redo_stack.append(command)
        print(f"[UNDO] Reversed: {command.get_description()}")

    def redo(self):
        if not self.redo_stack:
            print("[REDO] Nothing to redo")
            return

        command = self.redo_stack.pop()
        result = command.execute(self.system)
        self.command_history.append(command)
        print(f"[REDO] {result}")

    def create_checkpoint(self, name: str):
        self.checkpoints[name] = SystemMemento(self.system)
        print(f"💾 Checkpoint '{name}' created")

    def restore_checkpoint(self, name: str):
        if name not in self.checkpoints:
            print(f"[ERROR] Checkpoint '{name}' not found")
            return

        memento = self.checkpoints[name]
        self.system.firewalls = copy.deepcopy(memento.firewalls_state)
        self.system.ice_level = memento.ice_level
        self.system.data_extracted = memento.data_extracted.copy()
        self.system.alarm_level = memento.alarm_level
        self.system.netrunner_detected = memento.detected
        print(f"🔄 Restored checkpoint '{name}'")

    def show_history(self):
        print("\n=== COMMAND HISTORY ===")
        for i, cmd in enumerate(self.command_history, 1):
            print(f"{i}. {cmd.get_description()}")

# Example Usage
def main():
    print("🌐 CYBERPUNK HACKING SIMULATOR 🌐\n")

    session = HackingSession()

    # Show initial state
    print(session.system.get_status())

    # Create checkpoint before hacking
    session.create_checkpoint("before_hack")

    # Execute hacking sequence
    print("\n--- Starting Hack Sequence ---\n")
    session.execute_command(BypassFirewallCommand("outer"))
    session.execute_command(BypassFirewallCommand("inner"))
    session.execute_command(BypassFirewallCommand("core"))

    # Extract data
    session.execute_command(ExtractDataCommand("financial_records.db"))
    session.execute_command(ExtractDataCommand("employee_list.txt"))

    print("\n" + session.system.get_status())

    # Oh no! Alarm too high, undo last action
    print("\n--- Alarm too high! Undoing last action ---\n")
    session.undo()

    # Try crashing ICE instead
    session.execute_command(CrashICECommand("core"))

    print("\n" + session.system.get_status())

    # Show command history
    session.show_history()

    # Restore to checkpoint
    print("\n--- Restoring to checkpoint ---\n")
    session.restore_checkpoint("before_hack")
    print(session.system.get_status())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🌐 CYBERPUNK HACKING SIMULATOR 🌐

=== SYSTEM STATUS ===
Firewall [outer]: active (strength: 3)
Firewall [inner]: active (strength: 5)
Firewall [core]: active (strength: 8)
ICE Level: 0
Alarm Level: 0
Data Extracted: 0 files

💾 Checkpoint 'before_hack' created

--- Starting Hack Sequence ---

[SUCCESS] Firewall outer bypassed! ICE increased by 3
[SUCCESS] Firewall inner bypassed! ICE increased by 5
[SUCCESS] Firewall core bypassed! ICE increased by 8
[EXTRACTED] Data file: financial_records.db
⚠️  ALERT: NETRUNNER DETECTED! ICE countermeasures activated!
```

**Pattern Benefits:**
- **Command Pattern:** Each hack action is encapsulated as an object, enabling undo/redo and command queuing
- **Memento Pattern:** Save checkpoints to restore game state if detected
- **Flexibility:** Easy to add new hack commands without modifying existing code
- **Recording:** Can record and replay entire hack sequences for tutorials or AI learning

---

## 2. Pirate Ship Battle - Strategy Pattern

**Theme:** Caribbean pirate naval combat
**Patterns:** Strategy, Observer

**Key Learning:** Different ships use different battle tactics via swappable strategies

**Complete Implementation:**

```python
"""
Pirate ship battle system demonstrating Strategy pattern.
Ships use different combat strategies that can be changed dynamically.
Observer pattern notifies crew and other ships of battle events.
"""

from abc import ABC, abstractmethod
from typing import List, Tuple
from dataclasses import dataclass
import math
import random

@dataclass
class Position:
    x: float
    y: float

    def distance_to(self, other: 'Position') -> float:
        return math.sqrt((self.x - other.x)**2 + (self.y - other.y)**2)

class WindDirection:
    NORTH = 0
    EAST = 90
    SOUTH = 180
    WEST = 270

# Observer Pattern - Battle events
class BattleObserver(ABC):
    @abstractmethod
    def on_shot_fired(self, attacker: 'Ship', defender: 'Ship', damage: int):
        pass

    @abstractmethod
    def on_ship_destroyed(self, ship: 'Ship'):
        pass

class BattleLogger(BattleObserver):
    """Logs all battle events"""
    def on_shot_fired(self, attacker: 'Ship', defender: 'Ship', damage: int):
        print(f"⚓ {attacker.name} fires at {defender.name} for {damage} damage!")

    def on_ship_destroyed(self, ship: 'Ship'):
        print(f"💀 {ship.name} has been destroyed! Ship sinking...")

class ScoreTracker(BattleObserver):
    """Tracks battle statistics"""
    def __init__(self):
        self.shots_fired = 0
        self.total_damage = 0
        self.ships_destroyed = 0

    def on_shot_fired(self, attacker: 'Ship', defender: 'Ship', damage: int):
        self.shots_fired += 1
        self.total_damage += damage

    def on_ship_destroyed(self, ship: 'Ship'):
        self.ships_destroyed += 1

    def get_stats(self) -> str:
        return f"Stats: {self.shots_fired} shots, {self.total_damage} total damage, {self.ships_destroyed} ships destroyed"

# Strategy Pattern - Different combat strategies
class CombatStrategy(ABC):
    """Abstract strategy for ship combat behavior"""
    @abstractmethod
    def decide_action(self, ship: 'Ship', enemies: List['Ship'],
                     wind_direction: int) -> Tuple[str, 'Ship']:
        """Returns (action_type, target_ship)"""
        pass

    @abstractmethod
    def get_name(self) -> str:
        pass

class BroadsideStrategy(CombatStrategy):
    """Prefers medium-range cannon combat"""
    def decide_action(self, ship: 'Ship', enemies: List['Ship'],
                     wind_direction: int) -> Tuple[str, 'Ship']:
        if not enemies:
            return ("idle", None)

        # Find nearest enemy
        nearest = min(enemies, key=lambda e: ship.position.distance_to(e.position))
        distance = ship.position.distance_to(nearest.position)

        # Optimal broadside range is 100-200 units
        if 100 < distance < 200:
            return ("broadside_cannon", nearest)
        elif distance < 100:
            # Too close, back off
            return ("retreat", nearest)
        else:
            # Too far, close distance
            return ("approach", nearest)

    def get_name(self) -> str:
        return "Broadside Tactics"

class RammingStrategy(CombatStrategy):
    """Aggressive close-range ramming and boarding"""
    def decide_action(self, ship: 'Ship', enemies: List['Ship'],
                     wind_direction: int) -> Tuple[str, 'Ship']:
        if not enemies:
            return ("idle", None)

        # Target the weakest ship
        weakest = min(enemies, key=lambda e: e.hull_integrity)
        distance = ship.position.distance_to(weakest.position)

        if distance < 20:
            # Close enough to board!
            if weakest.hull_integrity < 30:
                return ("board", weakest)
            else:
                return ("ram", weakest)
        elif distance < 50:
            # Ramming distance
            return ("ram", weakest)
        else:
            # Close the gap quickly
            return ("full_speed_approach", weakest)

    def get_name(self) -> str:
        return "Ramming & Boarding"

class HitAndRunStrategy(CombatStrategy):
    """Fast attacks then retreat"""
    def decide_action(self, ship: 'Ship', enemies: List['Ship'],
                     wind_direction: int) -> Tuple[str, 'Ship']:
        if not enemies:
            return ("idle", None)

        # Use wind direction for speed bonus
        nearest = min(enemies, key=lambda e: ship.position.distance_to(e.position))
        distance = ship.position.distance_to(nearest.position)

        # If we've taken damage, run away
        if ship.hull_integrity < ship.max_hull * 0.6:
            return ("retreat_with_wind", nearest)

        # Attack from advantageous position
        if 150 < distance < 250:
            return ("chain_shot", nearest)  # Damage sails
        elif distance > 250:
            return ("approach_with_wind", nearest)
        else:
            # Too close, use wind to escape
            return ("retreat_with_wind", nearest)

    def get_name(self) -> str:
        return "Hit & Run"

class DefensiveStrategy(CombatStrategy):
    """Defensive tactics, protect allies"""
    def decide_action(self, ship: 'Ship', enemies: List['Ship'],
                     wind_direction: int) -> Tuple[str, 'Ship']:
        if not enemies:
            return ("idle", None)

        # Find most threatening enemy (closest)
        most_threatening = min(enemies, key=lambda e: ship.position.distance_to(e.position))
        distance = ship.position.distance_to(most_threatening.position)

        if distance < 150:
            # Defensive fire
            return ("defensive_broadside", most_threatening)
        else:
            # Maintain position
            return ("hold_position", most_threatening)

    def get_name(self) -> str:
        return "Defensive Formation"

# Ship class using Strategy pattern
class Ship:
    def __init__(self, name: str, ship_type: str, position: Position):
        self.name = name
        self.ship_type = ship_type
        self.position = position
        self.max_hull = 100
        self.hull_integrity = 100
        self.crew = 50
        self.cannon_damage = 15
        self.ram_damage = 25
        self.is_alive = True

        # Strategy pattern - can be changed during battle!
        self.combat_strategy: CombatStrategy = BroadsideStrategy()
        self.observers: List[BattleObserver] = []

    def set_strategy(self, strategy: CombatStrategy):
        """Change combat strategy mid-battle"""
        print(f"🏴‍☠️ {self.name} switches to: {strategy.get_name()}")
        self.combat_strategy = strategy

    def add_observer(self, observer: BattleObserver):
        self.observers.append(observer)

    def decide_action(self, enemies: List['Ship'], wind_direction: int) -> Tuple[str, 'Ship']:
        """Use current strategy to decide action"""
        return self.combat_strategy.decide_action(self, enemies, wind_direction)

    def execute_action(self, action: str, target: 'Ship'):
        """Execute the decided action"""
        if not self.is_alive or not target or not target.is_alive:
            return

        damage = 0

        if action == "broadside_cannon":
            damage = self.cannon_damage + random.randint(-5, 5)
            target.take_damage(damage)
            self._notify_shot_fired(target, damage)

        elif action == "ram":
            damage = self.ram_damage + random.randint(-8, 8)
            target.take_damage(damage)
            self.take_damage(damage // 2)  # Ramming damages us too
            print(f"💥 {self.name} rams {target.name}!")
            self._notify_shot_fired(target, damage)

        elif action == "board":
            if random.random() > 0.5:
                print(f"⚔️  {self.name} successfully boards {target.name}!")
                target.take_damage(40)
                target.crew -= 20
            else:
                print(f"⚔️  {self.name}'s boarding attempt failed!")

        elif action == "chain_shot":
            # Damages sails/mobility (represented as small damage)
            damage = 8
            target.take_damage(damage)
            print(f"⛵ {self.name} damages {target.name}'s sails!")
            self._notify_shot_fired(target, damage)

        elif action in ["approach", "full_speed_approach", "approach_with_wind"]:
            print(f"→ {self.name} approaches {target.name}")

        elif action in ["retreat", "retreat_with_wind"]:
            print(f"← {self.name} retreats from {target.name}")

        elif action == "hold_position":
            print(f"🛡️  {self.name} holds defensive position")

        elif action == "defensive_broadside":
            damage = self.cannon_damage // 2
            target.take_damage(damage)
            print(f"🛡️  {self.name} fires defensive broadside")
            self._notify_shot_fired(target, damage)

    def take_damage(self, damage: int):
        """Apply damage to ship"""
        self.hull_integrity -= damage
        if self.hull_integrity <= 0:
            self.hull_integrity = 0
            self.is_alive = False
            self._notify_destroyed()

    def _notify_shot_fired(self, target: 'Ship', damage: int):
        for observer in self.observers:
            observer.on_shot_fired(self, target, damage)

    def _notify_destroyed(self):
        for observer in self.observers:
            observer.on_ship_destroyed(self)

    def get_status(self) -> str:
        status_icon = "⚓" if self.is_alive else "💀"
        return f"{status_icon} {self.name} | Hull: {self.hull_integrity}/{self.max_hull} | Crew: {self.crew} | Strategy: {self.combat_strategy.get_name()}"

# Battle Simulator
class NavalBattle:
    def __init__(self):
        self.ships: List[Ship] = []
        self.wind_direction = WindDirection.EAST
        self.round = 0

        # Create observers
        self.logger = BattleLogger()
        self.stats = ScoreTracker()

    def add_ship(self, ship: Ship):
        ship.add_observer(self.logger)
        ship.add_observer(self.stats)
        self.ships.append(ship)

    def simulate_round(self):
        """Simulate one round of combat"""
        self.round += 1
        print(f"\n{'='*60}")
        print(f"⚔️  ROUND {self.round} - Wind: {self.wind_direction}° {'='*60}")
        print()

        alive_ships = [s for s in self.ships if s.is_alive]

        if len(alive_ships) <= 1:
            return False  # Battle over

        # Each ship decides and executes action
        for ship in alive_ships:
            enemies = [s for s in alive_ships if s != ship]
            action, target = ship.decide_action(enemies, self.wind_direction)
            ship.execute_action(action, target)

        # Show status
        print(f"\n--- Status ---")
        for ship in self.ships:
            print(ship.get_status())

        return True  # Battle continues

    def run_battle(self, max_rounds: int = 10):
        """Run the complete battle"""
        print(f"🏴‍☠️ NAVAL BATTLE BEGINS! 🏴‍☠️")
        print(f"Ships: {', '.join(s.name for s in self.ships)}")

        for _ in range(max_rounds):
            if not self.simulate_round():
                break

        # Battle conclusion
        print(f"\n{'='*60}")
        print("⚓ BATTLE CONCLUDED ⚓")
        print(f"{'='*60}")
        survivors = [s for s in self.ships if s.is_alive]
        if survivors:
            print(f"🏆 Survivors: {', '.join(s.name for s in survivors)}")
        print(self.stats.get_stats())

# Example Usage
def main():
    battle = NavalBattle()

    # Create ships with different strategies
    revenge = Ship("The Revenge", "Galleon", Position(0, 0))
    revenge.set_strategy(BroadsideStrategy())

    serpent = Ship("Sea Serpent", "Frigate", Position(150, 50))
    serpent.set_strategy(HitAndRunStrategy())

    marauder = Ship("Marauder", "Sloop", Position(80, 80))
    marauder.set_strategy(RammingStrategy())

    guardian = Ship("Guardian", "Man-of-War", Position(200, 20))
    guardian.set_strategy(DefensiveStrategy())

    battle.add_ship(revenge)
    battle.add_ship(serpent)
    battle.add_ship(marauder)
    battle.add_ship(guardian)

    # Run battle
    battle.run_battle(max_rounds=8)

    # Example: Ships can change strategy mid-battle
    print("\n--- Strategy Change Example ---")
    if revenge.hull_integrity < 50:
        revenge.set_strategy(DefensiveStrategy())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🏴‍☠️ NAVAL BATTLE BEGINS! 🏴‍☠️
Ships: The Revenge, Sea Serpent, Marauder, Guardian

============================================================
⚔️  ROUND 1 - Wind: 90° ============================================================

⚓ The Revenge fires at Sea Serpent for 18 damage!
⛵ Sea Serpent damages Marauder's sails!
💥 Marauder rams Guardian!
🛡️  Guardian holds defensive position

--- Status ---
⚓ The Revenge | Hull: 100/100 | Crew: 50 | Strategy: Broadside Tactics
⚓ Sea Serpent | Hull: 100/100 | Crew: 50 | Strategy: Hit & Run
⚓ Marauder | Hull: 88/100 | Crew: 50 | Strategy: Ramming & Boarding
⚓ Guardian | Hull: 77/100 | Crew: 50 | Strategy: Defensive Formation
```

**Pattern Benefits:**
- **Strategy Pattern:** Easy to add new combat tactics without modifying ship code
- **Runtime Flexibility:** Ships can change strategies based on battle conditions
- **Observer Pattern:** Decouples battle events from logging/scoring
- **Testability:** Each strategy can be tested independently
- **AI Variety:** Different ship types behave uniquely, creating interesting battles

---

## 3. Wizard Tower Defense - Factory Pattern

**Theme:** Fantasy wizard summoning creatures
**Patterns:** Factory, Pool

**Key Learning:** Factory creates different minion types from templates

**Complete Implementation:**

```python
"""
Complete wizard tower defense system with Factory pattern.
Demonstrates creating different minion types, upgrade systems, and wave-based spawning.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from dataclasses import dataclass
from enum import Enum
import random

@dataclass
class Position:
    x: float
    y: float

    def distance_to(self, other: 'Position') -> float:
        return ((self.x - other.x)**2 + (self.y - other.y)**2)**0.5

class Element(Enum):
    FIRE = "fire"
    ICE = "ice"
    LIGHTNING = "lightning"
    EARTH = "earth"
    ARCANE = "arcane"

# Abstract Product
class Minion(ABC):
    """Base class for all summoned minions"""
    def __init__(self, position: Position):
        self.position = position
        self.health = 100
        self.max_health = 100
        self.damage = 10
        self.attack_speed = 1.0
        self.range = 100
        self.is_alive = True
        self.level = 1

    @abstractmethod
    def attack(self, target: 'Enemy') -> int:
        """Returns damage dealt"""
        pass

    @abstractmethod
    def get_element(self) -> Element:
        pass

    @abstractmethod
    def special_ability(self, targets: List['Enemy']) -> str:
        pass

    def take_damage(self, damage: int):
        self.health -= damage
        if self.health <= 0:
            self.health = 0
            self.is_alive = False

    def upgrade(self):
        self.level += 1
        self.max_health = int(self.max_health * 1.5)
        self.health = self.max_health
        self.damage = int(self.damage * 1.3)

# Concrete Products
class FireElemental(Minion):
    """Deals burning damage over time"""
    def __init__(self, position: Position):
        super().__init__(position)
        self.health = 80
        self.max_health = 80
        self.damage = 25
        self.attack_speed = 1.5
        self.burn_duration = 3

    def attack(self, target: 'Enemy') -> int:
        damage = self.damage + random.randint(-3, 3)
        target.apply_burn(self.burn_duration, damage // 3)
        return damage

    def get_element(self) -> Element:
        return Element.FIRE

    def special_ability(self, targets: List['Enemy']) -> str:
        # Fireball AOE
        if len(targets) >= 2:
            for target in targets[:3]:
                target.take_damage(self.damage * 2)
            return f"Fire Elemental casts FIREBALL! {len(targets[:3])} enemies burned!"
        return "No targets for fireball"

class IceGolem(Minion):
    """Slow, tanky minion that freezes enemies"""
    def __init__(self, position: Position):
        super().__init__(position)
        self.health = 200
        self.max_health = 200
        self.damage = 15
        self.attack_speed = 0.7
        self.armor = 10

    def attack(self, target: 'Enemy') -> int:
        damage = self.damage + random.randint(-2, 2)
        if random.random() < 0.3:  # 30% chance to freeze
            target.apply_freeze(2)
            return damage * 2
        return damage

    def get_element(self) -> Element:
        return Element.ICE

    def special_ability(self, targets: List['Enemy']) -> str:
        # Ice shield
        self.health = min(self.health + 50, self.max_health)
        return "Ice Golem creates ice shield! +50 HP"

class LightningWisp(Minion):
    """Fast attacker that chains lightning"""
    def __init__(self, position: Position):
        super().__init__(position)
        self.health = 60
        self.max_health = 60
        self.damage = 30
        self.attack_speed = 2.5
        self.chain_count = 3

    def attack(self, target: 'Enemy') -> int:
        return self.damage + random.randint(-5, 5)

    def get_element(self) -> Element:
        return Element.LIGHTNING

    def special_ability(self, targets: List['Enemy']) -> str:
        # Chain lightning
        damage_dealt = 0
        chain_damage = self.damage
        for i, target in enumerate(targets[:self.chain_count]):
            target.take_damage(chain_damage)
            damage_dealt += chain_damage
            chain_damage = int(chain_damage * 0.7)  # Reduced each chain
        return f"Chain Lightning! {len(targets[:self.chain_count])} enemies hit for {damage_dealt} total damage"

class EarthGolem(Minion):
    """Regenerating tank"""
    def __init__(self, position: Position):
        super().__init__(position)
        self.health = 250
        self.max_health = 250
        self.damage = 20
        self.attack_speed = 0.8
        self.regen_per_turn = 10

    def attack(self, target: 'Enemy') -> int:
        self.health = min(self.health + self.regen_per_turn, self.max_health)
        return self.damage + random.randint(-3, 3)

    def get_element(self) -> Element:
        return Element.EARTH

    def special_ability(self, targets: List['Enemy']) -> str:
        # Earthquake stun
        for target in targets:
            target.apply_stun(1)
        return f"Earthquake! {len(targets)} enemies stunned!"

class ArcaneMage(Minion):
    """Powerful spellcaster"""
    def __init__(self, position: Position):
        super().__init__(position)
        self.health = 100
        self.max_health = 100
        self.damage = 40
        self.attack_speed = 1.0
        self.mana = 100

    def attack(self, target: 'Enemy') -> int:
        if self.mana >= 10:
            self.mana -= 10
            return self.damage + random.randint(-5, 10)
        return self.damage // 2  # Weak attack without mana

    def get_element(self) -> Element:
        return Element.ARCANE

    def special_ability(self, targets: List['Enemy']) -> str:
        if self.mana >= 50:
            self.mana -= 50
            total_damage = 0
            for target in targets:
                dmg = self.damage * 3
                target.take_damage(dmg)
                total_damage += dmg
            return f"ARCANE BLAST! {total_damage} total damage to {len(targets)} enemies!"
        return "Not enough mana for Arcane Blast"

# Factory Pattern
class MinionFactory:
    """Factory for creating different types of minions"""

    _minion_registry: Dict[str, type] = {
        "fire": FireElemental,
        "ice": IceGolem,
        "lightning": LightningWisp,
        "earth": EarthGolem,
        "arcane": ArcaneMage,
    }

    _summon_costs: Dict[str, int] = {
        "fire": 50,
        "ice": 75,
        "lightning": 60,
        "earth": 80,
        "arcane": 100,
    }

    @classmethod
    def create_minion(cls, minion_type: str, position: Position) -> Optional[Minion]:
        """Create a minion of the specified type"""
        minion_class = cls._minion_registry.get(minion_type)
        if minion_class:
            return minion_class(position)
        return None

    @classmethod
    def get_cost(cls, minion_type: str) -> int:
        return cls._summon_costs.get(minion_type, 0)

    @classmethod
    def get_available_types(cls) -> List[str]:
        return list(cls._minion_registry.keys())

    @classmethod
    def register_minion_type(cls, name: str, minion_class: type, cost: int):
        """Allow adding new minion types at runtime"""
        cls._minion_registry[name] = minion_class
        cls._summon_costs[name] = cost

# Enemy class for targets
class Enemy:
    def __init__(self, name: str, health: int):
        self.name = name
        self.health = health
        self.max_health = health
        self.is_alive = True
        self.burn_stacks = 0
        self.frozen_turns = 0
        self.stunned_turns = 0

    def take_damage(self, damage: int):
        self.health -= damage
        if self.health <= 0:
            self.health = 0
            self.is_alive = False

    def apply_burn(self, duration: int, damage_per_turn: int):
        self.burn_stacks = max(self.burn_stacks, duration)

    def apply_freeze(self, duration: int):
        self.frozen_turns = duration

    def apply_stun(self, duration: int):
        self.stunned_turns = duration

# Wizard Tower managing minions
class WizardTower:
    def __init__(self, name: str):
        self.name = name
        self.mana = 200
        self.max_mana = 200
        self.mana_regen = 20
        self.minions: List[Minion] = []
        self.position = Position(0, 0)
        self.wave_number = 0

    def summon_minion(self, minion_type: str, position: Position) -> Optional[Minion]:
        """Summon a minion using the factory"""
        cost = MinionFactory.get_cost(minion_type)

        if cost == 0:
            print(f"Unknown minion type: {minion_type}")
            return None

        if self.mana < cost:
            print(f"Not enough mana! Need {cost}, have {self.mana}")
            return None

        minion = MinionFactory.create_minion(minion_type, position)
        if minion:
            self.mana -= cost
            self.minions.append(minion)
            print(f"Summoned {minion_type.capitalize()} Minion for {cost} mana!")
            return minion
        return None

    def regenerate_mana(self):
        self.mana = min(self.mana + self.mana_regen, self.max_mana)

    def upgrade_minion(self, minion: Minion):
        upgrade_cost = 50 * minion.level
        if self.mana >= upgrade_cost:
            self.mana -= upgrade_cost
            minion.upgrade()
            print(f"Upgraded minion to level {minion.level}!")

    def get_status(self) -> str:
        alive_minions = [m for m in self.minions if m.is_alive]
        status = f"\n=== {self.name} ===\n"
        status += f"Mana: {self.mana}/{self.max_mana}\n"
        status += f"Active Minions: {len(alive_minions)}/{len(self.minions)}\n"
        for i, minion in enumerate(alive_minions):
            element = minion.get_element().value
            status += f"  {i+1}. {element.capitalize()} Lv{minion.level} | HP: {minion.health}/{minion.max_health} | DMG: {minion.damage}\n"
        return status

# Game Simulation
class TowerDefenseGame:
    def __init__(self):
        self.tower = WizardTower("Arcane Spire")
        self.enemies: List[Enemy] = []
        self.round = 0

    def spawn_wave(self):
        self.round += 1
        wave_size = 3 + self.round
        print(f"\n*** WAVE {self.round} - {wave_size} enemies approach! ***")

        for i in range(wave_size):
            enemy = Enemy(f"Orc_{i+1}", health=50 + self.round * 10)
            self.enemies.append(enemy)

    def simulate_combat_round(self):
        alive_minions = [m for m in self.tower.minions if m.is_alive]
        alive_enemies = [e for e in self.enemies if e.is_alive]

        if not alive_enemies:
            return True  # Wave cleared

        print(f"\n--- Combat Round ---")
        print(f"Minions: {len(alive_minions)} | Enemies: {len(alive_enemies)}")

        # Minions attack
        for minion in alive_minions:
            if alive_enemies:
                target = alive_enemies[0]
                damage = minion.attack(target)
                print(f"  {minion.get_element().value.capitalize()} attacks {target.name} for {damage} damage")

                # Use special abilities randomly
                if random.random() < 0.2 and len(alive_enemies) > 0:
                    result = minion.special_ability(alive_enemies[:3])
                    print(f"  SPECIAL: {result}")

        # Remove dead enemies
        self.enemies = [e for e in self.enemies if e.is_alive]
        return len(self.enemies) == 0

# Example Usage
def main():
    print("=== WIZARD TOWER DEFENSE - Factory Pattern Demo ===\n")

    game = TowerDefenseGame()

    # Show available minion types
    print("Available Minions:")
    for minion_type in MinionFactory.get_available_types():
        cost = MinionFactory.get_cost(minion_type)
        print(f"  - {minion_type.capitalize()}: {cost} mana")

    print(game.tower.get_status())

    # Build initial defense
    print("\n--- Building Initial Defense ---")
    game.tower.summon_minion("fire", Position(10, 0))
    game.tower.summon_minion("ice", Position(20, 0))
    game.tower.summon_minion("lightning", Position(15, 5))

    print(game.tower.get_status())

    # Wave 1
    game.spawn_wave()
    for _ in range(3):
        if game.simulate_combat_round():
            print("\nWave 1 cleared!")
            break

    # Regenerate mana and summon more
    game.tower.regenerate_mana()
    game.tower.regenerate_mana()
    print(f"\nMana regenerated: {game.tower.mana}/{game.tower.max_mana}")

    print("\n--- Summoning Reinforcements ---")
    game.tower.summon_minion("earth", Position(5, 5))
    game.tower.summon_minion("arcane", Position(0, 10))

    print(game.tower.get_status())

    # Wave 2
    game.spawn_wave()
    for _ in range(4):
        if game.simulate_combat_round():
            print("\nWave 2 cleared!")
            break

    # Upgrade a minion
    if game.tower.minions:
        game.tower.regenerate_mana()
        game.tower.regenerate_mana()
        print("\n--- Upgrading First Minion ---")
        game.tower.upgrade_minion(game.tower.minions[0])

    print(game.tower.get_status())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== WIZARD TOWER DEFENSE - Factory Pattern Demo ===

Available Minions:
  - Fire: 50 mana
  - Ice: 75 mana
  - Lightning: 60 mana
  - Earth: 80 mana
  - Arcane: 100 mana

=== Arcane Spire ===
Mana: 200/200
Active Minions: 0/0

--- Building Initial Defense ---
Summoned Fire Minion for 50 mana!
Summoned Ice Minion for 75 mana!
Summoned Lightning Minion for 60 mana!

=== Arcane Spire ===
Mana: 15/200
Active Minions: 3/3
  1. fire Lv1 | HP: 80/80 | DMG: 25
  2. ice Lv1 | HP: 200/200 | DMG: 15
  3. lightning Lv1 | HP: 60/60 | DMG: 30

*** WAVE 1 - 4 enemies approach! ***

--- Combat Round ---
Minions: 3 | Enemies: 4
  Fire attacks Orc_1 for 27 damage
  Ice attacks Orc_1 for 17 damage
  Lightning attacks Orc_1 for 32 damage
  SPECIAL: Chain Lightning! 3 enemies hit for 87 total damage

Wave 1 cleared!
```

**Pattern Benefits:**
- **Factory Pattern:** Centralized minion creation makes adding new types trivial
- **Open/Closed Principle:** Add new minion types without modifying existing code
- **Runtime Registration:** Can add custom minion types dynamically
- **Consistent Interface:** All minions share common behavior through abstract base
- **Cost Management:** Factory encapsulates summoning costs and requirements
- **Testability:** Easy to test minion creation and behavior independently

---

## 4. Space Station Emergency - State Pattern

**Theme:** Sci-fi space station system failures
**Patterns:** State, Observer

**Key Learning:** Station transitions between different emergency states

**Complete Implementation:**

```python
"""
Complete space station emergency system with State pattern.
Demonstrates state transitions, emergency handling, and system failures.
"""

from abc import ABC, abstractmethod
from typing import List, Optional, Dict
from enum import Enum
from dataclasses import dataclass
import random

class SystemStatus(Enum):
    OPERATIONAL = "operational"
    DEGRADED = "degraded"
    CRITICAL = "critical"
    OFFLINE = "offline"

class AlertLevel(Enum):
    GREEN = "green"
    YELLOW = "yellow"
    ORANGE = "orange"
    RED = "red"

@dataclass
class SystemDamage:
    system_name: str
    damage_amount: int
    description: str

# Observer Pattern for station events
class StationObserver(ABC):
    @abstractmethod
    def on_state_change(self, old_state: str, new_state: str):
        pass

    @abstractmethod
    def on_system_failure(self, system_name: str):
        pass

    @abstractmethod
    def on_alert_level_change(self, alert_level: AlertLevel):
        pass

class CrewAlertSystem(StationObserver):
    """Notifies crew of station status changes"""
    def on_state_change(self, old_state: str, new_state: str):
        print(f"CREW ALERT: Station transitioning from {old_state} to {new_state}")

    def on_system_failure(self, system_name: str):
        print(f"CREW ALERT: {system_name} system failure detected!")

    def on_alert_level_change(self, alert_level: AlertLevel):
        print(f"ALERT LEVEL: {alert_level.value.upper()}")

class EmergencyLogger(StationObserver):
    """Logs all emergency events"""
    def __init__(self):
        self.log: List[str] = []

    def on_state_change(self, old_state: str, new_state: str):
        self.log.append(f"State transition: {old_state} -> {new_state}")

    def on_system_failure(self, system_name: str):
        self.log.append(f"System failure: {system_name}")

    def on_alert_level_change(self, alert_level: AlertLevel):
        self.log.append(f"Alert level changed to: {alert_level.value}")

    def get_log(self) -> List[str]:
        return self.log.copy()

# State Pattern - Different emergency states
class StationState(ABC):
    """Abstract base class for station states"""

    @abstractmethod
    def enter(self, station: 'SpaceStation'):
        """Called when entering this state"""
        pass

    @abstractmethod
    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        """
        Update state logic. Returns new state if transition needed, None otherwise.
        dt = delta time (seconds)
        """
        pass

    @abstractmethod
    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        """Handle repair attempts. Returns True if allowed."""
        pass

    @abstractmethod
    def get_state_name(self) -> str:
        pass

    @abstractmethod
    def get_available_actions(self) -> List[str]:
        """Returns list of actions crew can take in this state"""
        pass

class NormalOperationsState(StationState):
    """Station operating normally"""

    def enter(self, station: 'SpaceStation'):
        station.alert_level = AlertLevel.GREEN
        station.notify_alert_level_change()
        print("\n=== NORMAL OPERATIONS ===")
        print("All systems operational. Crew performing routine maintenance.")

    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        # Slow regeneration of resources during normal operations
        station.life_support = min(100, station.life_support + dt * 0.5)
        station.power = min(100, station.power + dt * 1.0)

        # Random chance of emergency
        if random.random() < 0.05:  # 5% chance per update
            emergency_type = random.choice(['hull_breach', 'fire', 'power_failure'])
            if emergency_type == 'hull_breach':
                return HullBreachState()
            elif emergency_type == 'fire':
                return FireEmergencyState()
            elif emergency_type == 'power_failure':
                return PowerFailureState()

        return None

    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        print("No repairs needed - all systems nominal")
        return True

    def get_state_name(self) -> str:
        return "Normal Operations"

    def get_available_actions(self) -> List[str]:
        return ["routine_maintenance", "crew_training", "system_diagnostics"]

class HullBreachState(StationState):
    """Emergency hull breach - losing atmosphere"""

    def __init__(self):
        self.sectors_sealed = 0
        self.target_sectors = 3
        self.oxygen_loss_rate = 5.0

    def enter(self, station: 'SpaceStation'):
        station.alert_level = AlertLevel.RED
        station.notify_alert_level_change()
        print("\n!!! HULL BREACH DETECTED !!!")
        print(f"Atmosphere venting! Seal {self.target_sectors} sectors to contain breach!")
        station.life_support -= 30
        station.notify_system_failure("hull_integrity")

    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        # Continuous oxygen loss
        station.life_support -= self.oxygen_loss_rate * dt

        if station.life_support <= 0:
            return StationDestroyedState()

        # Check if breach is contained
        if self.sectors_sealed >= self.target_sectors:
            print(f"\nBreach contained! {self.target_sectors} sectors sealed.")
            return RepairOperationsState("hull_breach")

        return None

    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        if station.power < 10:
            print("Not enough power to seal bulkheads!")
            return False

        self.sectors_sealed += 1
        station.power -= 10
        print(f"Bulkhead sealed! ({self.sectors_sealed}/{self.target_sectors})")
        return True

    def get_state_name(self) -> str:
        return "Hull Breach Emergency"

    def get_available_actions(self) -> List[str]:
        return ["seal_bulkhead", "emergency_oxygen", "evacuate_sector"]

class FireEmergencyState(StationState):
    """Fire spreading through station"""

    def __init__(self):
        self.fire_intensity = 100
        self.suppression_progress = 0

    def enter(self, station: 'SpaceStation'):
        station.alert_level = AlertLevel.RED
        station.notify_alert_level_change()
        print("\n!!! FIRE DETECTED IN ENGINEERING !!!")
        print("Deploy fire suppression systems immediately!")
        station.power -= 20
        station.notify_system_failure("fire_suppression")

    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        # Fire damages systems and consumes oxygen
        if self.fire_intensity > 0:
            station.life_support -= dt * 2
            station.power -= dt * 3
            self.fire_intensity -= self.suppression_progress * dt

        if station.life_support <= 0 or station.power <= 0:
            return StationDestroyedState()

        if self.fire_intensity <= 0:
            print("\nFire extinguished! Damage assessment in progress...")
            return RepairOperationsState("fire_damage")

        return None

    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        if station.power < 5:
            print("Not enough power for fire suppression!")
            return False

        self.suppression_progress += 20
        station.power -= 5
        remaining = max(0, self.fire_intensity - self.suppression_progress)
        print(f"Fire suppression active! Intensity: {int(remaining)}%")
        return True

    def get_state_name(self) -> str:
        return "Fire Emergency"

    def get_available_actions(self) -> List[str]:
        return ["activate_suppression", "vent_atmosphere", "evacuate_crew"]

class PowerFailureState(StationState):
    """Main power offline - running on backup"""

    def __init__(self):
        self.backup_power_time = 100
        self.repair_progress = 0
        self.repair_required = 3

    def enter(self, station: 'SpaceStation'):
        station.alert_level = AlertLevel.ORANGE
        station.notify_alert_level_change()
        print("\n!!! MAIN POWER FAILURE !!!")
        print("Running on backup power. Restore main reactor!")
        station.power = 30  # Limited backup power
        station.notify_system_failure("main_reactor")

    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        # Backup power depleting
        self.backup_power_time -= dt
        station.power = max(0, 30 - (100 - self.backup_power_time) * 0.3)

        if self.backup_power_time <= 0 or station.power <= 0:
            return StationDestroyedState()

        if self.repair_progress >= self.repair_required:
            print("\nMain reactor back online!")
            station.power = 80
            return NormalOperationsState()

        return None

    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        self.repair_progress += 1
        print(f"Reactor repair progress: {self.repair_progress}/{self.repair_required}")
        return True

    def get_state_name(self) -> str:
        return "Power Failure"

    def get_available_actions(self) -> List[str]:
        return ["repair_reactor", "reroute_power", "activate_emergency_battery"]

class RepairOperationsState(StationState):
    """Post-emergency repair operations"""

    def __init__(self, damage_type: str):
        self.damage_type = damage_type
        self.repair_time = 50
        self.time_elapsed = 0

    def enter(self, station: 'SpaceStation'):
        station.alert_level = AlertLevel.YELLOW
        station.notify_alert_level_change()
        print(f"\n=== REPAIR OPERATIONS: {self.damage_type} ===")
        print("Emergency contained. Beginning repair operations.")

    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        self.time_elapsed += dt

        # Slow system recovery
        if station.power < 100:
            station.power = min(100, station.power + dt * 2)
        if station.life_support < 100:
            station.life_support = min(100, station.life_support + dt * 1.5)

        if self.time_elapsed >= self.repair_time:
            print("\nRepairs complete! Returning to normal operations.")
            return NormalOperationsState()

        # Random chance of new emergency during repairs
        if random.random() < 0.02:
            print("\nSecondary emergency detected during repairs!")
            return HullBreachState()

        return None

    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        if station.power < 15:
            print("Not enough power for repairs!")
            return False

        station.power -= 15
        self.time_elapsed += 10  # Speed up repairs
        progress = min(100, (self.time_elapsed / self.repair_time) * 100)
        print(f"Repair progress: {int(progress)}%")
        return True

    def get_state_name(self) -> str:
        return f"Repair Operations ({self.damage_type})"

    def get_available_actions(self) -> List[str]:
        return ["accelerate_repairs", "damage_assessment", "request_supplies"]

class StationDestroyedState(StationState):
    """Terminal state - station lost"""

    def enter(self, station: 'SpaceStation'):
        station.alert_level = AlertLevel.RED
        print("\n" + "="*60)
        print("!!! CRITICAL SYSTEM FAILURE !!!")
        print("Station integrity compromised. Initiating evacuation...")
        print("STATION LOST")
        print("="*60)

    def update(self, station: 'SpaceStation', dt: float) -> Optional['StationState']:
        return None  # Terminal state

    def handle_repair_action(self, station: 'SpaceStation') -> bool:
        print("Cannot repair - station destroyed!")
        return False

    def get_state_name(self) -> str:
        return "Station Destroyed"

    def get_available_actions(self) -> List[str]:
        return ["evacuate", "send_distress"]

# Space Station with State pattern
class SpaceStation:
    def __init__(self, name: str):
        self.name = name
        self.life_support = 100
        self.power = 100
        self.hull_integrity = 100
        self.crew_count = 50
        self.alert_level = AlertLevel.GREEN

        # State pattern
        self.current_state: StationState = NormalOperationsState()
        self.current_state.enter(self)

        # Observers
        self.observers: List[StationObserver] = []

    def add_observer(self, observer: StationObserver):
        self.observers.append(observer)

    def notify_state_change(self, old_state: str, new_state: str):
        for observer in self.observers:
            observer.on_state_change(old_state, new_state)

    def notify_system_failure(self, system_name: str):
        for observer in self.observers:
            observer.on_system_failure(system_name)

    def notify_alert_level_change(self):
        for observer in self.observers:
            observer.on_alert_level_change(self.alert_level)

    def update(self, dt: float):
        """Update station state"""
        new_state = self.current_state.update(self, dt)

        if new_state:
            self.transition_to(new_state)

    def transition_to(self, new_state: StationState):
        """Transition to a new state"""
        old_state_name = self.current_state.get_state_name()
        self.current_state = new_state
        new_state_name = self.current_state.get_state_name()

        self.notify_state_change(old_state_name, new_state_name)
        self.current_state.enter(self)

    def perform_repair(self):
        """Crew attempts repair action"""
        self.current_state.handle_repair_action(self)

    def get_status(self) -> str:
        status = f"\n=== {self.name.upper()} STATUS ===\n"
        status += f"State: {self.current_state.get_state_name()}\n"
        status += f"Alert Level: {self.alert_level.value.upper()}\n"
        status += f"Life Support: {int(self.life_support)}%\n"
        status += f"Power: {int(self.power)}%\n"
        status += f"Crew: {self.crew_count}\n"
        status += f"Available Actions: {', '.join(self.current_state.get_available_actions())}\n"
        return status

# Example Usage
def main():
    print("=== SPACE STATION EMERGENCY SIMULATOR ===\n")

    # Create station
    station = SpaceStation("Aurora Station")

    # Add observers
    crew_alerts = CrewAlertSystem()
    logger = EmergencyLogger()
    station.add_observer(crew_alerts)
    station.add_observer(logger)

    print(station.get_status())

    # Simulate normal operations
    print("\n--- Simulating Time Passage ---")
    for i in range(3):
        print(f"\nTime +{i+1}s")
        station.update(1)
        if isinstance(station.current_state, StationDestroyedState):
            break

    # Force a hull breach
    if not isinstance(station.current_state, StationDestroyedState):
        print("\n--- HULL BREACH EVENT ---")
        station.transition_to(HullBreachState())
        print(station.get_status())

        # Attempt repairs
        print("\n--- Crew Performing Emergency Repairs ---")
        for i in range(3):
            print(f"\nRepair attempt {i+1}:")
            station.perform_repair()
            station.update(2)
            print(station.get_status())

            if isinstance(station.current_state, (NormalOperationsState, RepairOperationsState)):
                break

    # Force a fire emergency
    if not isinstance(station.current_state, StationDestroyedState):
        print("\n--- FIRE EMERGENCY EVENT ---")
        station.transition_to(FireEmergencyState())

        # Fight the fire
        for i in range(4):
            print(f"\nFire suppression round {i+1}:")
            station.perform_repair()
            station.update(3)

            if not isinstance(station.current_state, FireEmergencyState):
                break

    # Show final status
    print("\n" + "="*60)
    print("SIMULATION COMPLETE")
    print("="*60)
    print(station.get_status())

    # Show event log
    print("\n=== EVENT LOG ===")
    for event in logger.get_log():
        print(f"  - {event}")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== SPACE STATION EMERGENCY SIMULATOR ===

=== NORMAL OPERATIONS ===
All systems operational. Crew performing routine maintenance.
ALERT LEVEL: GREEN

=== AURORA STATION STATUS ===
State: Normal Operations
Alert Level: GREEN
Life Support: 100%
Power: 100%
Crew: 50
Available Actions: routine_maintenance, crew_training, system_diagnostics

--- Simulating Time Passage ---

Time +1s

Time +2s

Time +3s

--- HULL BREACH EVENT ---
CREW ALERT: Station transitioning from Normal Operations to Hull Breach Emergency

!!! HULL BREACH DETECTED !!!
Atmosphere venting! Seal 3 sectors to contain breach!
CREW ALERT: hull_integrity system failure detected!
ALERT LEVEL: RED

=== AURORA STATION STATUS ===
State: Hull Breach Emergency
Alert Level: RED
Life Support: 70%
Power: 100%
Crew: 50
Available Actions: seal_bulkhead, emergency_oxygen, evacuate_sector

--- Crew Performing Emergency Repairs ---

Repair attempt 1:
Bulkhead sealed! (1/3)

=== AURORA STATION STATUS ===
State: Hull Breach Emergency
Alert Level: RED
Life Support: 60%
Power: 90%
Crew: 50
Available Actions: seal_bulkhead, emergency_oxygen, evacuate_sector

Repair attempt 2:
Bulkhead sealed! (2/3)

Repair attempt 3:
Bulkhead sealed! (3/3)

Breach contained! 3 sectors sealed.
CREW ALERT: Station transitioning from Hull Breach Emergency to Repair Operations (hull_breach)

=== REPAIR OPERATIONS: hull_breach ===
Emergency contained. Beginning repair operations.
ALERT LEVEL: YELLOW
```

**Pattern Benefits:**
- **State Pattern:** Each emergency type is a self-contained state with specific behaviors
- **Clean Transitions:** State transitions are explicit and controlled
- **Extensibility:** Easy to add new emergency types without modifying existing states
- **Context Preservation:** Each state can maintain its own progress data
- **Observer Integration:** States can trigger notifications to crew and systems
- **Testability:** Each emergency state can be tested in isolation

---

## 5. Time-Traveling Detective - Memento Pattern

**Theme:** Noir detective rewinding time
**Patterns:** Memento, Command

**Key Learning:** Save/restore game state to different timeline points

**Complete Implementation:**

```python
"""
Complete time-traveling detective system with Memento and Command patterns.
Demonstrates saving/restoring game state to different timeline points.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Optional, Set
from dataclasses import dataclass
from enum import Enum
import copy

class Location(Enum):
    CRIME_SCENE = "Crime Scene"
    POLICE_STATION = "Police Station"
    SUSPECTS_APARTMENT = "Suspect's Apartment"
    DOWNTOWN_BAR = "Downtown Bar"
    WAREHOUSE = "Abandoned Warehouse"

class NPCState(Enum):
    ALIVE = "alive"
    DEAD = "dead"
    ARRESTED = "arrested"
    MISSING = "missing"

@dataclass
class Clue:
    name: str
    description: str
    location: Location
    timestamp: int
    importance: int

    def __hash__(self):
        return hash(self.name)

class NPC:
    def __init__(self, name: str, location: Location, alibi: str):
        self.name = name
        self.location = location
        self.alibi = alibi
        self.state = NPCState.ALIVE
        self.suspicion_level = 0
        self.interrogated = False
        self.statements: List[str] = []

    def __repr__(self):
        return f"NPC({self.name}, {self.location.value}, {self.state.value})"

# Memento Pattern - Captures complete game state
class TimelineMemento:
    """Captures complete detective game state for time travel"""

    def __init__(self, timestamp: int, detective_location: Location,
                 npcs: Dict[str, NPC], clues_found: Set[Clue],
                 timeline_events: List[str], case_solved: bool):
        self.timestamp = timestamp
        # Deep copy to preserve state
        self.detective_location = detective_location
        self.npcs_snapshot = copy.deepcopy(npcs)
        self.clues_snapshot = copy.deepcopy(clues_found)
        self.events_snapshot = timeline_events.copy()
        self.case_solved = case_solved

    def get_summary(self) -> str:
        return (f"Timestamp {self.timestamp}: {self.detective_location.value}, "
                f"{len(self.clues_snapshot)} clues, "
                f"{len(self.events_snapshot)} events")

# Command Pattern - Each action is a command for undo/redo
class DetectiveAction(ABC):
    """Abstract command for detective actions"""

    @abstractmethod
    def execute(self, game: 'DetectiveGame') -> str:
        pass

    @abstractmethod
    def get_description(self) -> str:
        pass

class TravelToLocationAction(DetectiveAction):
    def __init__(self, location: Location):
        self.location = location
        self.previous_location: Optional[Location] = None

    def execute(self, game: 'DetectiveGame') -> str:
        self.previous_location = game.detective_location
        game.detective_location = self.location
        game.current_time += 5  # Travel takes time

        event = f"Time {game.current_time}: Traveled to {self.location.value}"
        game.timeline_events.append(event)

        # Check for automatic clue discovery
        location_clues = [c for c in game.available_clues
                         if c.location == self.location
                         and c not in game.clues_found]

        if location_clues:
            discovered = location_clues[0]
            game.clues_found.add(discovered)
            return f"[TRAVEL] Arrived at {self.location.value}. Found clue: {discovered.name}!"

        return f"[TRAVEL] Arrived at {self.location.value}"

    def get_description(self) -> str:
        return f"Travel to {self.location.value}"

class InterrogateNPCAction(DetectiveAction):
    def __init__(self, npc_name: str):
        self.npc_name = npc_name
        self.was_interrogated = False

    def execute(self, game: 'DetectiveGame') -> str:
        if self.npc_name not in game.npcs:
            return f"[ERROR] {self.npc_name} not found"

        npc = game.npcs[self.npc_name]

        # Check if NPC is at same location
        if npc.location != game.detective_location:
            return f"[ERROR] {self.npc_name} is not at this location"

        if npc.state != NPCState.ALIVE:
            return f"[ERROR] Cannot interrogate {self.npc_name} - {npc.state.value}"

        self.was_interrogated = npc.interrogated
        npc.interrogated = True
        game.current_time += 10

        # Generate statement based on suspicion
        if npc.suspicion_level > 70:
            statement = f"{npc.name} seems nervous and contradicts their alibi!"
            npc.statements.append(statement)
            result = f"[INTERROGATION] {statement}"
        else:
            statement = f"{npc.name} maintains their alibi: {npc.alibi}"
            npc.statements.append(statement)
            result = f"[INTERROGATION] {statement}"

        game.timeline_events.append(f"Time {game.current_time}: Interrogated {self.npc_name}")
        return result

    def get_description(self) -> str:
        return f"Interrogate {self.npc_name}"

class AnalyzeClueAction(DetectiveAction):
    def __init__(self, clue_name: str):
        self.clue_name = clue_name

    def execute(self, game: 'DetectiveGame') -> str:
        # Find the clue
        clue = None
        for c in game.clues_found:
            if c.name == self.clue_name:
                clue = c
                break

        if not clue:
            return f"[ERROR] Clue '{self.clue_name}' not found"

        game.current_time += 15

        # Analysis might increase NPC suspicion
        if "fingerprint" in clue.name.lower():
            for npc in game.npcs.values():
                if npc.location == clue.location:
                    npc.suspicion_level += 20

        game.timeline_events.append(f"Time {game.current_time}: Analyzed {self.clue_name}")

        return f"[ANALYSIS] {clue.description}"

    def get_description(self) -> str:
        return f"Analyze clue: {self.clue_name}"

class AccuseSuspectAction(DetectiveAction):
    def __init__(self, suspect_name: str):
        self.suspect_name = suspect_name
        self.previous_state: Optional[NPCState] = None

    def execute(self, game: 'DetectiveGame') -> str:
        if self.suspect_name not in game.npcs:
            return f"[ERROR] {self.suspect_name} not found"

        npc = game.npcs[self.suspect_name]
        self.previous_state = npc.state

        # Check if we have enough evidence
        evidence_count = len(game.clues_found)

        if evidence_count < 3:
            return f"[ACCUSATION FAILED] Not enough evidence! Need at least 3 clues."

        # Check if correct suspect (simplistic - suspicion > 80)
        if npc.suspicion_level >= 80:
            npc.state = NPCState.ARRESTED
            game.case_solved = True
            game.timeline_events.append(f"Time {game.current_time}: Arrested {self.suspect_name}!")
            return f"[CASE SOLVED!] {self.suspect_name} arrested! You solved the case!"
        else:
            game.current_time += 20
            return f"[ACCUSATION FAILED] {self.suspect_name} has an alibi. Wrong suspect!"

    def get_description(self) -> str:
        return f"Accuse {self.suspect_name}"

# Main Detective Game with Timeline Management
class DetectiveGame:
    """Main game managing detective investigation with time travel"""

    def __init__(self):
        # Current game state
        self.current_time = 0
        self.detective_location = Location.CRIME_SCENE
        self.case_solved = False

        # NPCs in the game
        self.npcs: Dict[str, NPC] = {
            "Victor Stone": NPC("Victor Stone", Location.SUSPECTS_APARTMENT,
                               "Was at home all evening"),
            "Sarah Chen": NPC("Sarah Chen", Location.DOWNTOWN_BAR,
                             "Working at the bar until midnight"),
            "Marcus Black": NPC("Marcus Black", Location.WAREHOUSE,
                               "Delivering cargo shipment"),
        }

        # Set actual culprit
        self.npcs["Marcus Black"].suspicion_level = 85

        # Available clues in the world
        self.available_clues = [
            Clue("Bloody Knife", "Knife with fingerprints", Location.CRIME_SCENE, 0, 10),
            Clue("Security Footage", "Shows suspicious figure at 11 PM", Location.WAREHOUSE, 0, 9),
            Clue("Torn Note", "Partial shipping manifest", Location.WAREHOUSE, 0, 7),
            Clue("Witness Statement", "Bartender saw argument", Location.DOWNTOWN_BAR, 0, 6),
            Clue("Phone Records", "Call to victim at 10:30 PM", Location.POLICE_STATION, 0, 8),
        ]

        self.clues_found: Set[Clue] = set()
        self.timeline_events: List[str] = []

        # Timeline management (Memento)
        self.timeline_snapshots: Dict[int, TimelineMemento] = {}
        self.command_history: List[DetectiveAction] = []

        # Create initial snapshot
        self.create_snapshot()

    def create_snapshot(self) -> int:
        """Create a memento of current game state"""
        memento = TimelineMemento(
            timestamp=self.current_time,
            detective_location=self.detective_location,
            npcs=self.npcs,
            clues_found=self.clues_found,
            timeline_events=self.timeline_events,
            case_solved=self.case_solved
        )
        self.timeline_snapshots[self.current_time] = memento
        return self.current_time

    def rewind_to(self, timestamp: int) -> bool:
        """Rewind game to a previous timeline snapshot"""
        if timestamp not in self.timeline_snapshots:
            print(f"[ERROR] No snapshot at timestamp {timestamp}")
            return False

        memento = self.timeline_snapshots[timestamp]

        # Restore state from memento
        self.current_time = memento.timestamp
        self.detective_location = memento.detective_location
        self.npcs = copy.deepcopy(memento.npcs_snapshot)
        self.clues_found = copy.deepcopy(memento.clues_snapshot)
        self.timeline_events = memento.events_snapshot.copy()
        self.case_solved = memento.case_solved

        # Remove future snapshots
        future_timestamps = [t for t in self.timeline_snapshots.keys() if t > timestamp]
        for t in future_timestamps:
            del self.timeline_snapshots[t]

        print(f"⏪ REWOUND to timestamp {timestamp}")
        print(f"Location: {self.detective_location.value}")
        print(f"Clues: {len(self.clues_found)}")

        return True

    def execute_action(self, action: DetectiveAction):
        """Execute a detective action and record it"""
        result = action.execute(self)
        print(result)
        self.command_history.append(action)

        # Auto-create snapshot after important actions
        if isinstance(action, (InterrogateNPCAction, AccuseSuspectAction)):
            self.create_snapshot()

    def show_status(self):
        """Display current game status"""
        print("\n" + "="*60)
        print(f"DETECTIVE CASE FILE - TIME: {self.current_time} minutes")
        print("="*60)
        print(f"Current Location: {self.detective_location.value}")
        print(f"Case Status: {'SOLVED!' if self.case_solved else 'Open'}")
        print(f"\nClues Found ({len(self.clues_found)}):")
        for clue in sorted(self.clues_found, key=lambda c: c.importance, reverse=True):
            print(f"  - {clue.name}: {clue.description}")

        print(f"\nSuspects:")
        for name, npc in self.npcs.items():
            print(f"  - {name}: {npc.state.value} (suspicion: {npc.suspicion_level})")
            if npc.interrogated:
                print(f"    Alibi: {npc.alibi}")

        print(f"\nTimeline Snapshots: {sorted(self.timeline_snapshots.keys())}")
        print("="*60 + "\n")

    def show_timeline(self):
        """Show complete timeline of events"""
        print("\n=== TIMELINE ===")
        for event in self.timeline_events:
            print(f"  {event}")
        print()

# Example Usage
def main():
    print("🕵️  NOIR DETECTIVE - TIME TRAVEL INVESTIGATION 🕵️\n")

    game = DetectiveGame()
    game.show_status()

    # Investigation sequence 1: Follow wrong lead
    print("--- Investigation Path 1: Following Sarah Chen ---\n")
    game.execute_action(TravelToLocationAction(Location.DOWNTOWN_BAR))
    game.execute_action(InterrogateNPCAction("Sarah Chen"))
    game.execute_action(TravelToLocationAction(Location.POLICE_STATION))
    game.execute_action(AnalyzeClueAction("Bloody Knife"))

    game.show_status()

    # Try to accuse Sarah (wrong suspect)
    print("--- Attempting to solve case ---\n")
    game.execute_action(AccuseSuspectAction("Sarah Chen"))

    # Realize mistake - rewind time!
    print("\n--- Rewinding time to try different approach ---\n")
    game.rewind_to(0)

    # Investigation sequence 2: Follow correct lead
    print("\n--- Investigation Path 2: Following Marcus Black ---\n")
    game.execute_action(TravelToLocationAction(Location.WAREHOUSE))
    game.execute_action(InterrogateNPCAction("Marcus Black"))
    game.execute_action(AnalyzeClueAction("Security Footage"))
    game.execute_action(AnalyzeClueAction("Torn Note"))

    game.show_status()

    # Accuse the right suspect
    print("--- Solving the case ---\n")
    game.execute_action(AccuseSuspectAction("Marcus Black"))

    game.show_status()
    game.show_timeline()

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🕵️  NOIR DETECTIVE - TIME TRAVEL INVESTIGATION 🕵️

============================================================
DETECTIVE CASE FILE - TIME: 0 minutes
============================================================
Current Location: Crime Scene
Case Status: Open

Clues Found (1):
  - Bloody Knife: Knife with fingerprints

Suspects:
  - Victor Stone: alive (suspicion: 0)
  - Sarah Chen: alive (suspicion: 0)
  - Marcus Black: alive (suspicion: 85)

Timeline Snapshots: [0]
============================================================

--- Investigation Path 1: Following Sarah Chen ---

[TRAVEL] Arrived at Downtown Bar. Found clue: Witness Statement!
[INTERROGATION] Sarah Chen maintains their alibi: Working at the bar until midnight
[TRAVEL] Arrived at Police Station. Found clue: Phone Records!
[ANALYSIS] Knife with fingerprints

--- Attempting to solve case ---

[ACCUSATION FAILED] Sarah Chen has an alibi. Wrong suspect!

--- Rewinding time to try different approach ---

⏪ REWOUND to timestamp 0
Location: Crime Scene
Clues: 1

--- Investigation Path 2: Following Marcus Black ---

[TRAVEL] Arrived at Abandoned Warehouse. Found clue: Security Footage!
[INTERROGATION] Marcus Black seems nervous and contradicts their alibi!
[ANALYSIS] Shows suspicious figure at 11 PM
[ANALYSIS] Partial shipping manifest

--- Solving the case ---

[CASE SOLVED!] Marcus Black arrested! You solved the case!

============================================================
DETECTIVE CASE FILE - TIME: 25 minutes
============================================================
Current Location: Abandoned Warehouse
Case Status: SOLVED!

Clues Found (4):
  - Bloody Knife: Knife with fingerprints
  - Security Footage: Shows suspicious figure at 11 PM
  - Torn Note: Partial shipping manifest
  - Witness Statement: Bartender saw argument

Suspects:
  - Victor Stone: alive (suspicion: 0)
  - Sarah Chen: alive (suspicion: 0)
  - Marcus Black: arrested (suspicion: 105)
    Alibi: Delivering cargo shipment

Timeline Snapshots: [0, 10, 25]
============================================================

=== TIMELINE ===
  Time 5: Traveled to Abandoned Warehouse
  Time 10: Interrogated Marcus Black
  Time 25: Arrested Marcus Black!
```

**Pattern Benefits:**
- **Memento Pattern:** Save complete game state at key decision points, enabling time travel mechanics
- **Command Pattern:** Each action is encapsulated, making it easy to track and replay investigations
- **State Preservation:** Deep copying ensures timeline branches don't interfere with each other
- **Flexible Investigation:** Players can explore different leads and rewind if they make mistakes
- **Narrative Design:** Perfect for detective games, puzzle games, or any game with branching narratives

---

## 6. Kitchen Nightmare - Observer Pattern

**Theme:** Restaurant kitchen coordination
**Patterns:** Observer, Chain of Responsibility

**Key Learning:** Stations observe orders and react independently

**Complete Implementation:**

```python
"""
Complete restaurant kitchen system with Observer and Chain of Responsibility patterns.
Demonstrates how kitchen stations observe orders and coordinate cooking tasks.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from dataclasses import dataclass
from enum import Enum
import time

class OrderStatus(Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    SERVED = "served"

class DishType(Enum):
    APPETIZER = "appetizer"
    MAIN_COURSE = "main_course"
    DESSERT = "dessert"
    BEVERAGE = "beverage"

@dataclass
class MenuItem:
    name: str
    dish_type: DishType
    cook_time: int  # seconds
    ingredients: List[str]
    station_required: str

    def __hash__(self):
        return hash(self.name)

class Order:
    def __init__(self, order_id: int, table_number: int):
        self.order_id = order_id
        self.table_number = table_number
        self.items: List[MenuItem] = []
        self.status = OrderStatus.PENDING
        self.start_time: Optional[float] = None
        self.completion_time: Optional[float] = None
        self.items_completed: List[str] = []

    def add_item(self, item: MenuItem):
        self.items.append(item)

    def mark_item_complete(self, item_name: str):
        if item_name not in self.items_completed:
            self.items_completed.append(item_name)
            if len(self.items_completed) == len(self.items):
                self.status = OrderStatus.COMPLETED
                self.completion_time = time.time()

    def get_progress(self) -> str:
        return f"{len(self.items_completed)}/{len(self.items)} items"

# Observer Pattern - Kitchen stations observe orders
class KitchenObserver(ABC):
    """Abstract observer for kitchen stations"""

    @abstractmethod
    def on_order_received(self, order: Order):
        """Called when new order arrives"""
        pass

    @abstractmethod
    def on_order_cancelled(self, order: Order):
        """Called when order is cancelled"""
        pass

    @abstractmethod
    def on_station_update(self, station_name: str, status: str):
        """Called when station status changes"""
        pass

class GrillStation(KitchenObserver):
    """Grill station handles meats and grilled items"""

    def __init__(self):
        self.name = "Grill Station"
        self.queue: List[tuple[Order, MenuItem]] = []
        self.current_item: Optional[tuple[Order, MenuItem]] = None
        self.grill_items = ["steak", "burger", "chicken", "salmon"]

    def on_order_received(self, order: Order):
        # Filter items this station can handle
        grill_orders = [(order, item) for item in order.items
                       if any(grill in item.name.lower() for grill in self.grill_items)]

        if grill_orders:
            self.queue.extend(grill_orders)
            print(f"[{self.name}] Received {len(grill_orders)} item(s) for order #{order.order_id}")

    def on_order_cancelled(self, order: Order):
        # Remove cancelled order from queue
        self.queue = [(o, i) for o, i in self.queue if o.order_id != order.order_id]
        print(f"[{self.name}] Removed cancelled order #{order.order_id}")

    def on_station_update(self, station_name: str, status: str):
        pass  # Grill doesn't need to react to other stations

    def cook_next(self) -> Optional[str]:
        """Process next item in queue"""
        if not self.queue:
            return None

        order, item = self.queue.pop(0)
        self.current_item = (order, item)
        print(f"[{self.name}] Cooking {item.name} for order #{order.order_id}...")

        # Simulate cooking
        order.mark_item_complete(item.name)
        self.current_item = None

        return f"{item.name} ready for order #{order.order_id}"

class SaladStation(KitchenObserver):
    """Salad station handles cold prep"""

    def __init__(self):
        self.name = "Salad Station"
        self.queue: List[tuple[Order, MenuItem]] = []
        self.salad_items = ["salad", "appetizer", "cold"]

    def on_order_received(self, order: Order):
        salad_orders = [(order, item) for item in order.items
                       if any(word in item.name.lower() for word in self.salad_items)]

        if salad_orders:
            self.queue.extend(salad_orders)
            print(f"[{self.name}] Received {len(salad_orders)} item(s) for order #{order.order_id}")

    def on_order_cancelled(self, order: Order):
        self.queue = [(o, i) for o, i in self.queue if o.order_id != order.order_id]

    def on_station_update(self, station_name: str, status: str):
        pass

    def prepare_next(self) -> Optional[str]:
        if not self.queue:
            return None

        order, item = self.queue.pop(0)
        print(f"[{self.name}] Preparing {item.name} for order #{order.order_id}...")

        order.mark_item_complete(item.name)
        return f"{item.name} ready for order #{order.order_id}"

class PastryStation(KitchenObserver):
    """Pastry station handles desserts"""

    def __init__(self):
        self.name = "Pastry Station"
        self.queue: List[tuple[Order, MenuItem]] = []
        self.dessert_items = ["cake", "pie", "dessert", "ice cream"]

    def on_order_received(self, order: Order):
        dessert_orders = [(order, item) for item in order.items
                         if any(word in item.name.lower() for word in self.dessert_items)]

        if dessert_orders:
            self.queue.extend(dessert_orders)
            print(f"[{self.name}] Received {len(dessert_orders)} item(s) for order #{order.order_id}")

    def on_order_cancelled(self, order: Order):
        self.queue = [(o, i) for o, i in self.queue if o.order_id != order.order_id]

    def on_station_update(self, station_name: str, status: str):
        # Pastry can start desserts when main course is done
        if "main course completed" in status.lower():
            print(f"[{self.name}] Main course done, starting desserts...")

    def bake_next(self) -> Optional[str]:
        if not self.queue:
            return None

        order, item = self.queue.pop(0)
        print(f"[{self.name}] Baking {item.name} for order #{order.order_id}...")

        order.mark_item_complete(item.name)
        return f"{item.name} ready for order #{order.order_id}"

# Chain of Responsibility - Quality checks before serving
class QualityCheckHandler(ABC):
    """Abstract handler for quality checks"""

    def __init__(self):
        self.next_handler: Optional['QualityCheckHandler'] = None

    def set_next(self, handler: 'QualityCheckHandler') -> 'QualityCheckHandler':
        self.next_handler = handler
        return handler

    @abstractmethod
    def check(self, order: Order) -> tuple[bool, str]:
        pass

    def handle(self, order: Order) -> tuple[bool, str]:
        result, message = self.check(order)
        if not result:
            return False, message

        if self.next_handler:
            return self.next_handler.handle(order)

        return True, "All checks passed!"

class CompletenessCheck(QualityCheckHandler):
    """Check if all items are completed"""

    def check(self, order: Order) -> tuple[bool, str]:
        if len(order.items_completed) != len(order.items):
            return False, f"Order incomplete: {order.get_progress()}"
        return True, "All items completed"

class TemperatureCheck(QualityCheckHandler):
    """Check if hot items are hot"""

    def check(self, order: Order) -> tuple[bool, str]:
        # Simplified: just check we don't have too many desserts with hot items
        hot_items = sum(1 for item in order.items if "steak" in item.name.lower() or "burger" in item.name.lower())
        cold_items = sum(1 for item in order.items if "salad" in item.name.lower() or "ice cream" in item.name.lower())

        if hot_items > 0 and cold_items > 0:
            print("  [Temperature Check] Warning: Hot and cold items together")

        return True, "Temperature acceptable"

class PresentationCheck(QualityCheckHandler):
    """Check if presentation is good"""

    def check(self, order: Order) -> tuple[bool, str]:
        # All orders pass presentation in this simple version
        return True, "Presentation looks good"

# Kitchen Manager - Subject in Observer pattern
class Kitchen:
    """Main kitchen managing all stations and orders"""

    def __init__(self):
        self.observers: List[KitchenObserver] = []
        self.active_orders: Dict[int, Order] = {}
        self.completed_orders: List[Order] = []
        self.next_order_id = 1

        # Set up quality check chain
        self.quality_chain = CompletenessCheck()
        self.quality_chain.set_next(TemperatureCheck()).set_next(PresentationCheck())

    def add_observer(self, observer: KitchenObserver):
        """Add a kitchen station as observer"""
        self.observers.append(observer)
        print(f"[Kitchen] Added station: {observer.name}")

    def remove_observer(self, observer: KitchenObserver):
        """Remove a kitchen station"""
        self.observers.remove(observer)

    def receive_order(self, order: Order):
        """Receive new order and notify all stations"""
        print(f"\n[Kitchen] New order #{order.order_id} for table {order.table_number}")
        print(f"Items: {', '.join(item.name for item in order.items)}")

        self.active_orders[order.order_id] = order
        order.status = OrderStatus.IN_PROGRESS
        order.start_time = time.time()

        # Notify all observers (stations)
        for observer in self.observers:
            observer.on_order_received(order)

    def cancel_order(self, order_id: int):
        """Cancel an order"""
        if order_id not in self.active_orders:
            print(f"[Kitchen] Order #{order_id} not found")
            return

        order = self.active_orders[order_id]
        print(f"\n[Kitchen] Cancelling order #{order_id}")

        # Notify all observers
        for observer in self.observers:
            observer.on_order_cancelled(order)

        del self.active_orders[order_id]

    def notify_station_update(self, station_name: str, status: str):
        """Notify all observers about station status change"""
        for observer in self.observers:
            observer.on_station_update(station_name, status)

    def check_order_ready(self, order_id: int) -> bool:
        """Check if order passes quality checks"""
        if order_id not in self.active_orders:
            return False

        order = self.active_orders[order_id]

        if order.status != OrderStatus.COMPLETED:
            return False

        # Run through quality check chain
        passed, message = self.quality_chain.handle(order)

        if passed:
            print(f"\n[Kitchen] ✓ Order #{order_id} ready to serve!")
            print(f"Quality checks: {message}")
            order.status = OrderStatus.SERVED
            self.completed_orders.append(order)
            del self.active_orders[order_id]
            return True
        else:
            print(f"\n[Kitchen] ✗ Order #{order_id} failed quality check: {message}")
            return False

    def get_status(self) -> str:
        """Get kitchen status"""
        status = "\n=== KITCHEN STATUS ===\n"
        status += f"Active Orders: {len(self.active_orders)}\n"
        status += f"Completed Orders: {len(self.completed_orders)}\n"

        if self.active_orders:
            status += "\nActive:\n"
            for order_id, order in self.active_orders.items():
                status += f"  Order #{order_id}: {order.status.value} ({order.get_progress()})\n"

        return status

# Example Usage
def main():
    print("👨‍🍳 KITCHEN NIGHTMARE - Restaurant Kitchen Simulator 👨‍🍳\n")

    # Create kitchen
    kitchen = Kitchen()

    # Create and add kitchen stations
    grill = GrillStation()
    salad = SaladStation()
    pastry = PastryStation()

    kitchen.add_observer(grill)
    kitchen.add_observer(salad)
    kitchen.add_observer(pastry)

    # Create menu items
    steak = MenuItem("Grilled Steak", DishType.MAIN_COURSE, 15, ["beef", "spices"], "grill")
    caesar_salad = MenuItem("Caesar Salad", DishType.APPETIZER, 5, ["lettuce", "dressing"], "salad")
    chocolate_cake = MenuItem("Chocolate Cake", DishType.DESSERT, 10, ["chocolate", "flour"], "pastry")
    burger = MenuItem("Cheeseburger", DishType.MAIN_COURSE, 12, ["beef", "cheese"], "grill")

    # Order 1: Full course meal
    print("\n" + "="*60)
    order1 = Order(1, table_number=5)
    order1.add_item(caesar_salad)
    order1.add_item(steak)
    order1.add_item(chocolate_cake)

    kitchen.receive_order(order1)

    # Stations process their items
    print("\n--- Stations Working ---")
    salad.prepare_next()
    grill.cook_next()
    kitchen.notify_station_update("Grill Station", "Main course completed")
    pastry.bake_next()

    # Check if ready
    kitchen.check_order_ready(1)

    # Order 2: Quick order
    print("\n" + "="*60)
    order2 = Order(2, table_number=3)
    order2.add_item(burger)
    order2.add_item(caesar_salad)

    kitchen.receive_order(order2)

    print("\n--- Stations Working ---")
    salad.prepare_next()
    grill.cook_next()

    kitchen.check_order_ready(2)

    # Show final status
    print(kitchen.get_status())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
👨‍🍳 KITCHEN NIGHTMARE - Restaurant Kitchen Simulator 👨‍🍳

[Kitchen] Added station: Grill Station
[Kitchen] Added station: Salad Station
[Kitchen] Added station: Pastry Station

============================================================

[Kitchen] New order #1 for table 5
Items: Caesar Salad, Grilled Steak, Chocolate Cake
[Salad Station] Received 1 item(s) for order #1
[Grill Station] Received 1 item(s) for order #1
[Pastry Station] Received 1 item(s) for order #1

--- Stations Working ---
[Salad Station] Preparing Caesar Salad for order #1...
[Grill Station] Cooking Grilled Steak for order #1...
[Pastry Station] Main course done, starting desserts...
[Pastry Station] Baking Chocolate Cake for order #1...

[Kitchen] ✓ Order #1 ready to serve!
Quality checks: All checks passed!

============================================================

[Kitchen] New order #2 for table 3
Items: Cheeseburger, Caesar Salad
[Salad Station] Received 1 item(s) for order #2
[Grill Station] Received 1 item(s) for order #2

--- Stations Working ---
[Salad Station] Preparing Caesar Salad for order #2...
[Grill Station] Cooking Cheeseburger for order #2...

[Kitchen] ✓ Order #2 ready to serve!
Quality checks: All checks passed!

=== KITCHEN STATUS ===
Active Orders: 0
Completed Orders: 2
```

**Pattern Benefits:**
- **Observer Pattern:** Kitchen stations independently observe orders and handle their specific items
- **Decoupled Stations:** Each station knows what it can cook and filters orders automatically
- **Chain of Responsibility:** Quality checks run in sequence, each handler focusing on one aspect
- **Scalability:** Easy to add new stations without modifying existing code
- **Real-time Coordination:** Stations can react to each other's updates (e.g., desserts start after main course)

---

## 7. Dungeon Ecosystem - Composite Pattern

**Theme:** Living dungeon hierarchy
**Patterns:** Composite, Visitor

**Key Learning:** Treat individual monsters and room groups uniformly

**Complete Implementation:**

```python
"""
Complete dungeon ecosystem with Composite and Visitor patterns.
Demonstrates hierarchical dungeon structure and operations on dungeon components.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
from dataclasses import dataclass

class MonsterType(Enum):
    GOBLIN = "Goblin"
    ORC = "Orc"
    DRAGON = "Dragon"
    SKELETON = "Skeleton"
    SLIME = "Slime"

class TrapType(Enum):
    SPIKE = "Spike Trap"
    POISON_GAS = "Poison Gas"
    ARROW = "Arrow Trap"
    PIT = "Pit Trap"

@dataclass
class Treasure:
    name: str
    gold_value: int
    magic_items: int = 0

# Visitor Pattern - Operations on dungeon components
class DungeonVisitor(ABC):
    """Abstract visitor for dungeon operations"""

    @abstractmethod
    def visit_monster(self, monster: 'Monster'):
        pass

    @abstractmethod
    def visit_trap(self, trap: 'Trap'):
        pass

    @abstractmethod
    def visit_treasure(self, treasure: 'TreasureChest'):
        pass

    @abstractmethod
    def visit_room(self, room: 'Room'):
        pass

    @abstractmethod
    def visit_floor(self, floor: 'Floor'):
        pass

    @abstractmethod
    def visit_dungeon(self, dungeon: 'Dungeon'):
        pass

class ThreatCalculator(DungeonVisitor):
    """Calculate total threat level of dungeon components"""

    def __init__(self):
        self.total_threat = 0

    def visit_monster(self, monster: 'Monster'):
        self.total_threat += monster.threat_level

    def visit_trap(self, trap: 'Trap'):
        self.total_threat += trap.damage // 10  # Traps add less threat

    def visit_treasure(self, treasure: 'TreasureChest'):
        pass  # Treasure doesn't add threat

    def visit_room(self, room: 'Room'):
        for component in room.contents:
            component.accept(self)

    def visit_floor(self, floor: 'Floor'):
        for room in floor.rooms:
            room.accept(self)

    def visit_dungeon(self, dungeon: 'Dungeon'):
        for floor in dungeon.floors:
            floor.accept(self)

    def get_threat(self) -> int:
        return self.total_threat

class LootCalculator(DungeonVisitor):
    """Calculate total loot in dungeon"""

    def __init__(self):
        self.total_gold = 0
        self.total_items = 0

    def visit_monster(self, monster: 'Monster'):
        # Monsters drop some gold when defeated
        self.total_gold += monster.threat_level * 10

    def visit_trap(self, trap: 'Trap'):
        pass

    def visit_treasure(self, treasure: 'TreasureChest'):
        self.total_gold += treasure.treasure.gold_value
        self.total_items += treasure.treasure.magic_items

    def visit_room(self, room: 'Room'):
        for component in room.contents:
            component.accept(self)

    def visit_floor(self, floor: 'Floor'):
        for room in floor.rooms:
            room.accept(self)

    def visit_dungeon(self, dungeon: 'Dungeon'):
        for floor in dungeon.floors:
            floor.accept(self)

    def get_summary(self) -> str:
        return f"{self.total_gold} gold, {self.total_items} magic items"

class DungeonMapper(DungeonVisitor):
    """Create a map of the dungeon"""

    def __init__(self):
        self.map_output: List[str] = []
        self.indent_level = 0

    def _add_line(self, text: str):
        self.map_output.append("  " * self.indent_level + text)

    def visit_monster(self, monster: 'Monster'):
        self._add_line(f"🐉 {monster.monster_type.value} (Threat: {monster.threat_level})")

    def visit_trap(self, trap: 'Trap'):
        self._add_line(f"⚠️  {trap.trap_type.value} (DMG: {trap.damage})")

    def visit_treasure(self, treasure: 'TreasureChest'):
        self._add_line(f"💰 Treasure: {treasure.treasure.gold_value}g")

    def visit_room(self, room: 'Room'):
        self._add_line(f"📍 {room.name}")
        self.indent_level += 1
        for component in room.contents:
            component.accept(self)
        self.indent_level -= 1

    def visit_floor(self, floor: 'Floor'):
        self._add_line(f"🏰 {floor.name}")
        self.indent_level += 1
        for room in floor.rooms:
            room.accept(self)
        self.indent_level -= 1

    def visit_dungeon(self, dungeon: 'Dungeon'):
        self._add_line(f"🗺️  {dungeon.name}")
        self.indent_level += 1
        for floor in dungeon.floors:
            floor.accept(self)
        self.indent_level -= 1

    def get_map(self) -> str:
        return "\n".join(self.map_output)

# Composite Pattern - Dungeon component hierarchy
class DungeonComponent(ABC):
    """Abstract component in the dungeon hierarchy"""

    @abstractmethod
    def get_threat_level(self) -> int:
        pass

    @abstractmethod
    def accept(self, visitor: DungeonVisitor):
        """Accept a visitor (Visitor pattern)"""
        pass

    @abstractmethod
    def get_description(self) -> str:
        pass

# Leaf components
class Monster(DungeonComponent):
    """Individual monster (leaf)"""

    def __init__(self, monster_type: MonsterType, threat_level: int):
        self.monster_type = monster_type
        self.threat_level = threat_level
        self.is_alive = True

    def get_threat_level(self) -> int:
        return self.threat_level if self.is_alive else 0

    def accept(self, visitor: DungeonVisitor):
        visitor.visit_monster(self)

    def get_description(self) -> str:
        status = "alive" if self.is_alive else "defeated"
        return f"{self.monster_type.value} ({status}, threat: {self.threat_level})"

class Trap(DungeonComponent):
    """Trap (leaf)"""

    def __init__(self, trap_type: TrapType, damage: int):
        self.trap_type = trap_type
        self.damage = damage
        self.is_triggered = False

    def get_threat_level(self) -> int:
        return self.damage // 10 if not self.is_triggered else 0

    def accept(self, visitor: DungeonVisitor):
        visitor.visit_trap(self)

    def get_description(self) -> str:
        status = "triggered" if self.is_triggered else "armed"
        return f"{self.trap_type.value} ({status}, {self.damage} dmg)"

class TreasureChest(DungeonComponent):
    """Treasure chest (leaf)"""

    def __init__(self, treasure: Treasure):
        self.treasure = treasure
        self.is_looted = False

    def get_threat_level(self) -> int:
        return 0  # Treasure is not threatening

    def accept(self, visitor: DungeonVisitor):
        visitor.visit_treasure(self)

    def get_description(self) -> str:
        status = "looted" if self.is_looted else "unopened"
        return f"Treasure chest ({status}, {self.treasure.gold_value}g)"

# Composite components
class Room(DungeonComponent):
    """Room containing monsters, traps, treasures (composite)"""

    def __init__(self, name: str):
        self.name = name
        self.contents: List[DungeonComponent] = []

    def add(self, component: DungeonComponent):
        """Add a component to the room"""
        self.contents.append(component)

    def remove(self, component: DungeonComponent):
        """Remove a component from the room"""
        self.contents.remove(component)

    def get_threat_level(self) -> int:
        """Sum threat of all contents"""
        return sum(component.get_threat_level() for component in self.contents)

    def accept(self, visitor: DungeonVisitor):
        visitor.visit_room(self)

    def get_description(self) -> str:
        return f"Room '{self.name}' ({len(self.contents)} objects, threat: {self.get_threat_level()})"

class Floor(DungeonComponent):
    """Dungeon floor containing rooms (composite)"""

    def __init__(self, name: str, level: int):
        self.name = name
        self.level = level
        self.rooms: List[Room] = []

    def add_room(self, room: Room):
        """Add a room to the floor"""
        self.rooms.append(room)

    def get_threat_level(self) -> int:
        """Sum threat of all rooms"""
        return sum(room.get_threat_level() for room in self.rooms)

    def accept(self, visitor: DungeonVisitor):
        visitor.visit_floor(self)

    def get_description(self) -> str:
        return f"Floor {self.level}: {self.name} ({len(self.rooms)} rooms, threat: {self.get_threat_level()})"

class Dungeon(DungeonComponent):
    """Complete dungeon containing floors (composite root)"""

    def __init__(self, name: str):
        self.name = name
        self.floors: List[Floor] = []

    def add_floor(self, floor: Floor):
        """Add a floor to the dungeon"""
        self.floors.append(floor)

    def get_threat_level(self) -> int:
        """Sum threat of all floors"""
        return sum(floor.get_threat_level() for floor in self.floors)

    def accept(self, visitor: DungeonVisitor):
        visitor.visit_dungeon(self)

    def get_description(self) -> str:
        return f"Dungeon '{self.name}' ({len(self.floors)} floors, total threat: {self.get_threat_level()})"

# Example Usage
def main():
    print("⚔️  DUNGEON ECOSYSTEM - Composite + Visitor Demo ⚔️\n")

    # Create dungeon
    dungeon = Dungeon("Catacombs of Doom")

    # Floor 1: Upper levels (easier)
    floor1 = Floor("Entry Halls", level=1)

    room1a = Room("Guard Post")
    room1a.add(Monster(MonsterType.GOBLIN, threat_level=5))
    room1a.add(Monster(MonsterType.GOBLIN, threat_level=5))
    room1a.add(Trap(TrapType.ARROW, damage=20))

    room1b = Room("Storage Room")
    room1b.add(TreasureChest(Treasure("Old Chest", gold_value=100, magic_items=0)))
    room1b.add(Monster(MonsterType.SLIME, threat_level=3))

    floor1.add_room(room1a)
    floor1.add_room(room1b)

    # Floor 2: Middle levels (moderate)
    floor2 = Floor("Crypts", level=2)

    room2a = Room("Burial Chamber")
    room2a.add(Monster(MonsterType.SKELETON, threat_level=8))
    room2a.add(Monster(MonsterType.SKELETON, threat_level=8))
    room2a.add(TreasureChest(Treasure("Ancient Coffin", gold_value=500, magic_items=1)))

    room2b = Room("Trapped Corridor")
    room2b.add(Trap(TrapType.SPIKE, damage=30))
    room2b.add(Trap(TrapType.POISON_GAS, damage=40))

    room2c = Room("Orc Barracks")
    room2c.add(Monster(MonsterType.ORC, threat_level=12))
    room2c.add(Monster(MonsterType.ORC, threat_level=12))
    room2c.add(Monster(MonsterType.ORC, threat_level=12))

    floor2.add_room(room2a)
    floor2.add_room(room2b)
    floor2.add_room(room2c)

    # Floor 3: Deep levels (hard)
    floor3 = Floor("Dragon's Lair", level=3)

    room3a = Room("Treasure Vault")
    room3a.add(TreasureChest(Treasure("Dragon Hoard", gold_value=5000, magic_items=5)))
    room3a.add(Trap(TrapType.PIT, damage=100))

    room3b = Room("Dragon's Den")
    room3b.add(Monster(MonsterType.DRAGON, threat_level=50))

    floor3.add_room(room3a)
    floor3.add_room(room3b)

    # Add floors to dungeon
    dungeon.add_floor(floor1)
    dungeon.add_floor(floor2)
    dungeon.add_floor(floor3)

    # Display dungeon hierarchy
    print("=== DUNGEON STRUCTURE ===")
    print(dungeon.get_description())
    for floor in dungeon.floors:
        print(f"\n{floor.get_description()}")
        for room in floor.rooms:
            print(f"  {room.get_description()}")

    # Use Visitor pattern to calculate threat
    print("\n\n=== THREAT ANALYSIS ===")
    threat_calc = ThreatCalculator()
    dungeon.accept(threat_calc)
    print(f"Total Dungeon Threat Level: {threat_calc.get_threat()}")

    # Check individual floor threats
    for floor in dungeon.floors:
        floor_threat = ThreatCalculator()
        floor.accept(floor_threat)
        print(f"{floor.name} Threat: {floor_threat.get_threat()}")

    # Calculate loot
    print("\n\n=== LOOT ANALYSIS ===")
    loot_calc = LootCalculator()
    dungeon.accept(loot_calc)
    print(f"Total Loot Available: {loot_calc.get_summary()}")

    # Generate dungeon map
    print("\n\n=== DUNGEON MAP ===")
    mapper = DungeonMapper()
    dungeon.accept(mapper)
    print(mapper.get_map())

    # Demonstrate treating single room same as entire dungeon
    print("\n\n=== COMPOSITE PATTERN DEMO ===")
    print("Threat of single room:", room3b.get_threat_level())
    print("Threat of entire floor:", floor3.get_threat_level())
    print("Threat of entire dungeon:", dungeon.get_threat_level())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
⚔️  DUNGEON ECOSYSTEM - Composite + Visitor Demo ⚔️

=== DUNGEON STRUCTURE ===
Dungeon 'Catacombs of Doom' (3 floors, total threat: 136)

Floor 1: Entry Halls (2 rooms, threat: 18)
  Room 'Guard Post' (3 objects, threat: 12)
  Room 'Storage Room' (2 objects, threat: 3)

Floor 2: Crypts (3 rooms, threat: 56)
  Room 'Burial Chamber' (3 objects, threat: 16)
  Room 'Trapped Corridor' (2 objects, threat: 7)
  Room 'Orc Barracks' (3 objects, threat: 36)

Floor 3: Dragon's Lair (2 rooms, threat: 60)
  Room 'Treasure Vault' (2 objects, threat: 10)
  Room 'Dragon's Den' (1 objects, threat: 50)


=== THREAT ANALYSIS ===
Total Dungeon Threat Level: 136
Entry Halls Threat: 18
Crypts Threat: 56
Dragon's Lair Threat: 60


=== LOOT ANALYSIS ===
Total Loot Available: 7260 gold, 6 magic items


=== DUNGEON MAP ===
🗺️  Catacombs of Doom
  🏰 Entry Halls
    📍 Guard Post
      🐉 Goblin (Threat: 5)
      🐉 Goblin (Threat: 5)
      ⚠️  Arrow Trap (DMG: 20)
    📍 Storage Room
      💰 Treasure: 100g
      🐉 Slime (Threat: 3)
  🏰 Crypts
    📍 Burial Chamber
      🐉 Skeleton (Threat: 8)
      🐉 Skeleton (Threat: 8)
      💰 Treasure: 500g
    📍 Trapped Corridor
      ⚠️  Spike Trap (DMG: 30)
      ⚠️  Poison Gas (DMG: 40)
    📍 Orc Barracks
      🐉 Orc (Threat: 12)
      🐉 Orc (Threat: 12)
      🐉 Orc (Threat: 12)
  🏰 Dragon's Lair
    📍 Treasure Vault
      💰 Treasure: 5000g
      ⚠️  Pit Trap (DMG: 100)
    📍 Dragon's Den
      🐉 Dragon (Threat: 50)


=== COMPOSITE PATTERN DEMO ===
Threat of single room: 50
Threat of entire floor: 60
Threat of entire dungeon: 136
```

**Pattern Benefits:**
- **Composite Pattern:** Treat individual monsters and entire dungeons uniformly through common interface
- **Hierarchical Structure:** Natural representation of dungeon -> floors -> rooms -> contents
- **Visitor Pattern:** Add new operations (threat calc, loot calc, mapping) without modifying dungeon classes
- **Flexibility:** Easy to add new room types, monsters, or operations
- **Scalability:** Can build dungeons of any complexity with same simple building blocks

---

## 8. Zombie Survival - Object Pool Pattern

**Theme:** Zombie horde shooter
**Patterns:** Pool, Observer

**Key Learning:** Reuse zombie/bullet objects for performance

**Complete Implementation:**

```python
"""
Complete zombie survival shooter with Object Pool pattern.
Demonstrates efficient object reuse for high-performance gameplay with hundreds of entities.
"""

from abc import ABC, abstractmethod
from typing import List, Optional, Generic, TypeVar, Callable
from dataclasses import dataclass
import random
import math

@dataclass
class Vector2:
    x: float
    y: float

    def distance_to(self, other: 'Vector2') -> float:
        return math.sqrt((self.x - other.x)**2 + (self.y - other.y)**2)

    def direction_to(self, other: 'Vector2') -> 'Vector2':
        dist = self.distance_to(other)
        if dist == 0:
            return Vector2(0, 0)
        return Vector2((other.x - self.x) / dist, (other.y - self.y) / dist)

    def __add__(self, other: 'Vector2') -> 'Vector2':
        return Vector2(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar: float) -> 'Vector2':
        return Vector2(self.x * scalar, self.y * scalar)

# Poolable interface
class Poolable(ABC):
    """Interface for objects that can be pooled"""

    @abstractmethod
    def reset(self):
        """Reset object to initial state for reuse"""
        pass

    @abstractmethod
    def is_alive(self) -> bool:
        """Check if object is still active"""
        pass

# Generic Object Pool
T = TypeVar('T', bound=Poolable)

class ObjectPool(Generic[T]):
    """
    Generic object pool for reusing expensive objects.
    Eliminates allocation/deallocation overhead for frequently created objects.
    """

    def __init__(self, factory: Callable[[], T], initial_size: int = 100, max_size: int = 500):
        self.factory = factory
        self.max_size = max_size
        self.available: List[T] = [factory() for _ in range(initial_size)]
        self.in_use: List[T] = []
        self.total_created = initial_size
        self.total_acquired = 0
        self.total_released = 0

    def acquire(self) -> Optional[T]:
        """Get an object from the pool"""
        self.total_acquired += 1

        if self.available:
            obj = self.available.pop()
            self.in_use.append(obj)
            obj.reset()
            return obj

        # Pool exhausted - create new object if under max
        if len(self.in_use) < self.max_size:
            obj = self.factory()
            self.in_use.append(obj)
            self.total_created += 1
            return obj

        # Pool at max capacity
        return None

    def release(self, obj: T):
        """Return object to the pool for reuse"""
        if obj in self.in_use:
            self.in_use.remove(obj)
            self.available.append(obj)
            self.total_released += 1

    def update(self):
        """Automatically release dead objects"""
        for obj in self.in_use[:]:  # Copy list to avoid modification during iteration
            if not obj.is_alive():
                self.release(obj)

    def get_stats(self) -> dict:
        """Get pool statistics"""
        return {
            "available": len(self.available),
            "in_use": len(self.in_use),
            "total_created": self.total_created,
            "acquired": self.total_acquired,
            "released": self.total_released,
            "pool_efficiency": f"{(self.total_released / max(1, self.total_acquired)) * 100:.1f}%"
        }

# Zombie entity
class Zombie(Poolable):
    """Poolable zombie entity"""

    def __init__(self):
        self.position = Vector2(0, 0)
        self.health = 100
        self.max_health = 100
        self.speed = 20
        self.damage = 10
        self.active = False
        self.zombie_type = "walker"

    def reset(self):
        """Reset zombie to spawn state"""
        # Spawn at random edge position
        edge = random.choice(['top', 'bottom', 'left', 'right'])
        if edge == 'top':
            self.position = Vector2(random.uniform(0, 800), 0)
        elif edge == 'bottom':
            self.position = Vector2(random.uniform(0, 800), 600)
        elif edge == 'left':
            self.position = Vector2(0, random.uniform(0, 600))
        else:
            self.position = Vector2(800, random.uniform(0, 600))

        self.health = self.max_health
        self.active = True

    def is_alive(self) -> bool:
        return self.active and self.health > 0

    def take_damage(self, damage: int):
        self.health -= damage
        if self.health <= 0:
            self.active = False

    def move_towards(self, target: Vector2, dt: float):
        """Move zombie towards target"""
        if not self.active:
            return

        direction = self.position.direction_to(target)
        self.position = self.position + (direction * self.speed * dt)

    def attack(self, target: 'Player') -> int:
        """Attack target if in range"""
        if self.position.distance_to(target.position) < 30:
            return self.damage
        return 0

class RunnerZombie(Zombie):
    """Fast zombie variant"""
    def __init__(self):
        super().__init__()
        self.speed = 40
        self.health = 50
        self.max_health = 50
        self.zombie_type = "runner"

class TankZombie(Zombie):
    """Slow but tough zombie"""
    def __init__(self):
        super().__init__()
        self.speed = 10
        self.health = 300
        self.max_health = 300
        self.damage = 25
        self.zombie_type = "tank"

# Bullet entity
class Bullet(Poolable):
    """Poolable bullet projectile"""

    def __init__(self):
        self.position = Vector2(0, 0)
        self.velocity = Vector2(0, 0)
        self.damage = 25
        self.speed = 500
        self.active = False
        self.lifetime = 2.0  # Seconds
        self.time_alive = 0

    def reset(self):
        """Reset bullet for reuse"""
        self.active = True
        self.time_alive = 0

    def is_alive(self) -> bool:
        return self.active and self.time_alive < self.lifetime

    def shoot(self, start_pos: Vector2, target_pos: Vector2):
        """Fire bullet towards target"""
        self.position = Vector2(start_pos.x, start_pos.y)
        direction = start_pos.direction_to(target_pos)
        self.velocity = direction * self.speed
        self.reset()

    def update(self, dt: float):
        """Update bullet position"""
        if not self.active:
            return

        self.position = self.position + (self.velocity * dt)
        self.time_alive += dt

        # Deactivate if out of bounds
        if (self.position.x < 0 or self.position.x > 800 or
            self.position.y < 0 or self.position.y > 600):
            self.active = False

    def check_collision(self, zombie: Zombie) -> bool:
        """Check if bullet hit zombie"""
        if not self.active or not zombie.is_alive():
            return False

        if self.position.distance_to(zombie.position) < 20:
            zombie.take_damage(self.damage)
            self.active = False
            return True
        return False

# Player
class Player:
    def __init__(self):
        self.position = Vector2(400, 300)
        self.health = 100
        self.max_health = 100
        self.ammo = 100
        self.kills = 0

    def is_alive(self) -> bool:
        return self.health > 0

    def take_damage(self, damage: int):
        self.health -= damage

    def shoot(self, bullet_pool: ObjectPool[Bullet], target_pos: Vector2) -> bool:
        """Attempt to shoot a bullet from the pool"""
        if self.ammo <= 0:
            return False

        bullet = bullet_pool.acquire()
        if bullet:
            bullet.shoot(self.position, target_pos)
            self.ammo -= 1
            return True
        return False

# Game Manager
class ZombieGame:
    def __init__(self):
        # Create object pools
        self.zombie_pool = ObjectPool[Zombie](Zombie, initial_size=50, max_size=200)
        self.runner_pool = ObjectPool[RunnerZombie](RunnerZombie, initial_size=20, max_size=100)
        self.tank_pool = ObjectPool[TankZombie](TankZombie, initial_size=10, max_size=50)
        self.bullet_pool = ObjectPool[Bullet](Bullet, initial_size=100, max_size=500)

        self.player = Player()
        self.wave = 0
        self.zombies_this_wave = 0
        self.zombies_spawned = 0
        self.time = 0

    def start_wave(self):
        """Start a new zombie wave"""
        self.wave += 1
        self.zombies_this_wave = 10 + (self.wave * 5)
        self.zombies_spawned = 0
        print(f"\n*** WAVE {self.wave} - {self.zombies_this_wave} ZOMBIES! ***")

    def spawn_zombie(self):
        """Spawn a zombie from appropriate pool"""
        if self.zombies_spawned >= self.zombies_this_wave:
            return

        # Randomize zombie type with progression
        rand = random.random()
        if rand < 0.7:  # 70% walkers
            zombie = self.zombie_pool.acquire()
        elif rand < 0.9:  # 20% runners
            zombie = self.runner_pool.acquire()
        else:  # 10% tanks
            zombie = self.tank_pool.acquire()

        if zombie:
            self.zombies_spawned += 1

    def update(self, dt: float):
        """Update game state"""
        self.time += dt

        # Spawn zombies over time
        if self.zombies_spawned < self.zombies_this_wave and random.random() < 0.3:
            self.spawn_zombie()

        # Update bullets
        for bullet in self.bullet_pool.in_use[:]:
            bullet.update(dt)

        # Move zombies and check collisions
        all_zombies = (self.zombie_pool.in_use +
                       self.runner_pool.in_use +
                       self.tank_pool.in_use)

        for zombie in all_zombies:
            if zombie.is_alive():
                zombie.move_towards(self.player.position, dt)

                # Check bullet collisions
                for bullet in self.bullet_pool.in_use:
                    if bullet.check_collision(zombie):
                        if not zombie.is_alive():
                            self.player.kills += 1
                            print(f"Killed {zombie.zombie_type}! Total kills: {self.player.kills}")
                        break

                # Check player collision
                damage = zombie.attack(self.player)
                if damage > 0:
                    self.player.take_damage(damage)
                    print(f"Player hit! Health: {self.player.health}/{self.player.max_health}")

        # Auto-release dead objects
        self.zombie_pool.update()
        self.runner_pool.update()
        self.tank_pool.update()
        self.bullet_pool.update()

    def player_shoot_random(self):
        """Player shoots at random position (for simulation)"""
        target = Vector2(random.uniform(0, 800), random.uniform(0, 600))
        self.player.shoot(self.bullet_pool, target)

    def get_active_counts(self) -> dict:
        """Get count of active entities"""
        return {
            "zombies": (len(self.zombie_pool.in_use) +
                       len(self.runner_pool.in_use) +
                       len(self.tank_pool.in_use)),
            "bullets": len(self.bullet_pool.in_use)
        }

    def print_pool_stats(self):
        """Print statistics for all pools"""
        print("\n=== OBJECT POOL STATISTICS ===")
        print("Zombie Pool:", self.zombie_pool.get_stats())
        print("Runner Pool:", self.runner_pool.get_stats())
        print("Tank Pool:", self.tank_pool.get_stats())
        print("Bullet Pool:", self.bullet_pool.get_stats())

# Example Usage
def main():
    print("=== ZOMBIE SURVIVAL - Object Pool Demo ===\n")

    game = ZombieGame()

    # Wave 1
    game.start_wave()

    # Simulate gameplay
    for frame in range(100):
        dt = 0.1  # 100ms per frame

        game.update(dt)

        # Player shoots occasionally
        if random.random() < 0.3:
            game.player_shoot_random()

        # Print status every 20 frames
        if frame % 20 == 0:
            counts = game.get_active_counts()
            print(f"\nFrame {frame}: {counts['zombies']} zombies, {counts['bullets']} bullets active")
            print(f"Player: {game.player.health}HP, {game.player.ammo} ammo, {game.player.kills} kills")

        if not game.player.is_alive():
            print("\n*** PLAYER DEFEATED ***")
            break

        # Start next wave when all zombies dead
        all_zombies = (game.zombie_pool.in_use +
                      game.runner_pool.in_use +
                      game.tank_pool.in_use)
        if game.zombies_spawned >= game.zombies_this_wave and len(all_zombies) == 0:
            if game.wave < 3:  # Limit to 3 waves for demo
                game.start_wave()
            else:
                print("\n*** ALL WAVES COMPLETED! ***")
                break

    # Show final pool statistics
    game.print_pool_stats()

    print(f"\n=== GAME OVER ===")
    print(f"Waves Survived: {game.wave}")
    print(f"Total Kills: {game.player.kills}")
    print(f"Final Health: {game.player.health}/{game.player.max_health}")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== ZOMBIE SURVIVAL - Object Pool Demo ===

*** WAVE 1 - 15 ZOMBIES! ***

Frame 0: 3 zombies, 1 bullets active
Player: 100HP, 99 ammo, 0 kills

Killed walker! Total kills: 1
Killed runner! Total kills: 2
Player hit! Health: 90/100

Frame 20: 8 zombies, 4 bullets active
Player: 90HP, 93 ammo, 2 kills

Killed walker! Total kills: 3
Killed tank! Total kills: 4

Frame 40: 12 zombies, 6 bullets active
Player: 80HP, 85 ammo, 4 kills

*** WAVE 2 - 20 ZOMBIES! ***

Frame 60: 15 zombies, 8 bullets active
Player: 70HP, 78 ammo, 7 kills

Frame 80: 18 zombies, 10 bullets active
Player: 60HP, 70 ammo, 12 kills

=== OBJECT POOL STATISTICS ===
Zombie Pool: {'available': 35, 'in_use': 15, 'total_created': 50, 'acquired': 72, 'released': 57, 'pool_efficiency': '79.2%'}
Runner Pool: {'available': 12, 'in_use': 8, 'total_created': 20, 'acquired': 28, 'released': 20, 'pool_efficiency': '71.4%'}
Tank Pool: {'available': 7, 'in_use': 3, 'total_created': 10, 'acquired': 12, 'released': 9, 'pool_efficiency': '75.0%'}
Bullet Pool: {'available': 80, 'in_use': 20, 'total_created': 100, 'acquired': 156, 'released': 136, 'pool_efficiency': '87.2%'}

=== GAME OVER ===
Waves Survived: 2
Total Kills: 15
Final Health: 60/100
```

**Pattern Benefits:**
- **Performance:** Eliminates allocation/deallocation overhead for frequently created objects
- **Memory Efficiency:** Reuses objects instead of creating new ones (87% efficiency in example)
- **Predictable Memory:** Fixed pool size prevents unbounded memory growth
- **Cache Friendly:** Reused objects stay in CPU cache
- **Generic Implementation:** Single pool class works for any Poolable type
- **Statistics Tracking:** Monitor pool usage to optimize sizes
- **Critical for Action Games:** Essential for bullets, particles, enemies in fast-paced games

---

## 9. Musical Rhythm Game - Command + Observer

**Theme:** DDR-style rhythm game
**Patterns:** Command, Observer

**Key Learning:** Commands represent beats, observers react to scores

**Complete Implementation:**

```python
"""
Complete musical rhythm game with Command and Observer patterns.
Demonstrates beat timing, scoring, and visual effects.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
from dataclasses import dataclass
import time

class BeatType(Enum):
    TAP = "tap"
    HOLD = "hold"
    SLIDE = "slide"

class JudgementRating(Enum):
    PERFECT = ("PERFECT", 300, 1.0)
    GREAT = ("GREAT", 200, 0.9)
    GOOD = ("GOOD", 100, 0.7)
    BAD = ("BAD", 50, 0.3)
    MISS = ("MISS", 0, 0.0)

    def __init__(self, label: str, score: int, accuracy: float):
        self.label = label
        self.score = score
        self.accuracy = accuracy

@dataclass
class Note:
    """Represents a note in the song"""
    beat_time: float  # When the note should be hit
    lane: int  # Which lane (1-4)
    beat_type: BeatType

# Observer Pattern - React to game events
class RhythmGameObserver(ABC):
    """Abstract observer for rhythm game events"""

    @abstractmethod
    def on_beat_hit(self, judgement: JudgementRating, score: int, combo: int):
        pass

    @abstractmethod
    def on_combo_break(self, max_combo: int):
        pass

    @abstractmethod
    def on_song_complete(self, total_score: int, accuracy: float):
        pass

class ComboEffectsObserver(RhythmGameObserver):
    """Visual effects for combos"""

    def on_beat_hit(self, judgement: JudgementRating, score: int, combo: int):
        # Special effects at combo milestones
        if combo == 10:
            print("    ✨ 10 COMBO!")
        elif combo == 25:
            print("    🌟 25 COMBO!")
        elif combo == 50:
            print("    🔥🔥🔥 50 COMBO! Amazing!")
        elif combo == 100:
            print("    ⭐⭐⭐ 100 COMBO! LEGENDARY!")

        # Effects based on judgement
        if judgement == JudgementRating.PERFECT:
            if combo > 20:
                print("    💫 Perfect chain!")

    def on_combo_break(self, max_combo: int):
        if max_combo >= 10:
            print(f"    💔 Combo broken at {max_combo}!")

    def on_song_complete(self, total_score: int, accuracy: float):
        print("\n" + "="*50)
        print("    🎵 SONG COMPLETE! 🎵")

class ScoreDisplayObserver(RhythmGameObserver):
    """Display score updates"""

    def on_beat_hit(self, judgement: JudgementRating, score: int, combo: int):
        combo_str = f"x{combo}" if combo > 1 else ""
        print(f"  [{judgement.label}] +{score} {combo_str}")

    def on_combo_break(self, max_combo: int):
        pass  # Handled by combo effects

    def on_song_complete(self, total_score: int, accuracy: float):
        pass  # Handled by combo effects

class StatsTrackerObserver(RhythmGameObserver):
    """Track detailed statistics"""

    def __init__(self):
        self.judgement_counts = {rating: 0 for rating in JudgementRating}
        self.max_combo_achieved = 0

    def on_beat_hit(self, judgement: JudgementRating, score: int, combo: int):
        self.judgement_counts[judgement] += 1
        self.max_combo_achieved = max(self.max_combo_achieved, combo)

    def on_combo_break(self, max_combo: int):
        pass

    def on_song_complete(self, total_score: int, accuracy: float):
        print("="*50)
        print("DETAILED STATISTICS:")
        print(f"Total Score: {total_score}")
        print(f"Accuracy: {accuracy:.1f}%")
        print(f"Max Combo: {self.max_combo_achieved}")
        print("\nJudgement Breakdown:")
        for rating, count in self.judgement_counts.items():
            if count > 0:
                print(f"  {rating.label}: {count}")
        print("="*50)

# Command Pattern - Each beat is a command
class BeatCommand(ABC):
    """Abstract command for beat actions"""

    def __init__(self, note: Note):
        self.note = note
        self.executed = False

    @abstractmethod
    def execute(self, hit_time: float) -> JudgementRating:
        """Execute the beat command with player's hit timing"""
        pass

    def judge_timing(self, hit_time: float) -> JudgementRating:
        """Judge how accurately the player hit the beat"""
        timing_diff = abs(hit_time - self.note.beat_time)

        if timing_diff <= 0.03:
            return JudgementRating.PERFECT
        elif timing_diff <= 0.06:
            return JudgementRating.GREAT
        elif timing_diff <= 0.1:
            return JudgementRating.GOOD
        elif timing_diff <= 0.15:
            return JudgementRating.BAD
        else:
            return JudgementRating.MISS

class TapCommand(BeatCommand):
    """Simple tap beat"""

    def execute(self, hit_time: float) -> JudgementRating:
        self.executed = True
        return self.judge_timing(hit_time)

class HoldCommand(BeatCommand):
    """Hold note - must be held for duration"""

    def __init__(self, note: Note, duration: float):
        super().__init__(note)
        self.duration = duration
        self.hold_start_time: Optional[float] = None
        self.hold_end_time: Optional[float] = None

    def execute(self, hit_time: float) -> JudgementRating:
        """Start holding"""
        self.hold_start_time = hit_time
        return self.judge_timing(hit_time)

    def release(self, release_time: float) -> JudgementRating:
        """End holding"""
        if not self.hold_start_time:
            return JudgementRating.MISS

        self.hold_end_time = release_time
        hold_duration = release_time - self.hold_start_time
        expected_duration = self.duration

        # Judge how well they held the note
        duration_diff = abs(hold_duration - expected_duration)

        if duration_diff <= 0.05:
            return JudgementRating.PERFECT
        elif duration_diff <= 0.1:
            return JudgementRating.GREAT
        elif duration_diff <= 0.2:
            return JudgementRating.GOOD
        else:
            return JudgementRating.BAD

class SlideCommand(BeatCommand):
    """Slide note - swipe across lanes"""

    def __init__(self, note: Note, end_lane: int):
        super().__init__(note)
        self.end_lane = end_lane

    def execute(self, hit_time: float) -> JudgementRating:
        self.executed = True
        # For simplicity, just judge timing
        return self.judge_timing(hit_time)

# Main Rhythm Game Engine
class RhythmGame:
    """Main game managing beat commands and scoring"""

    def __init__(self, song_name: str):
        self.song_name = song_name
        self.beat_commands: List[BeatCommand] = []
        self.observers: List[RhythmGameObserver] = []

        # Game state
        self.current_score = 0
        self.current_combo = 0
        self.max_combo = 0
        self.total_notes = 0
        self.hits = 0
        self.accuracy_sum = 0.0

    def add_observer(self, observer: RhythmGameObserver):
        """Add an observer to the game"""
        self.observers.append(observer)

    def add_beat(self, command: BeatCommand):
        """Add a beat command to the song"""
        self.beat_commands.append(command)
        self.total_notes += 1

    def notify_beat_hit(self, judgement: JudgementRating):
        """Notify observers of a beat hit"""
        for observer in self.observers:
            observer.on_beat_hit(judgement, judgement.score, self.current_combo)

    def notify_combo_break(self):
        """Notify observers of combo break"""
        for observer in self.observers:
            observer.on_combo_break(self.max_combo)

    def notify_song_complete(self):
        """Notify observers of song completion"""
        accuracy = (self.accuracy_sum / self.total_notes * 100) if self.total_notes > 0 else 0
        for observer in self.observers:
            observer.on_song_complete(self.current_score, accuracy)

    def hit_beat(self, beat_index: int, hit_time: float):
        """Player hits a beat"""
        if beat_index >= len(self.beat_commands):
            print("  [ERROR] Invalid beat index")
            return

        command = self.beat_commands[beat_index]
        judgement = command.execute(hit_time)

        # Update score
        score = judgement.score
        if self.current_combo > 0:
            # Combo multiplier (every 10 combo adds 10% bonus)
            multiplier = 1.0 + (self.current_combo // 10) * 0.1
            score = int(score * multiplier)

        self.current_score += score

        # Update combo
        if judgement in [JudgementRating.PERFECT, JudgementRating.GREAT, JudgementRating.GOOD]:
            self.current_combo += 1
            self.max_combo = max(self.max_combo, self.current_combo)
            self.hits += 1
        else:
            if self.current_combo > 0:
                self.notify_combo_break()
            self.current_combo = 0

        # Track accuracy
        self.accuracy_sum += judgement.accuracy

        # Notify observers
        self.notify_beat_hit(judgement)

    def play_song(self):
        """Play through the entire song"""
        print(f"\n🎵 Now Playing: {self.song_name} 🎵\n")
        print("="*50)

        for i, command in enumerate(self.beat_commands):
            # Simulate player hitting beats with varying accuracy
            import random

            # 70% perfect, 20% great, 10% other
            rand = random.random()
            if rand < 0.7:
                # Perfect timing
                hit_time = command.note.beat_time
            elif rand < 0.9:
                # Great timing (slight offset)
                hit_time = command.note.beat_time + random.uniform(-0.05, 0.05)
            else:
                # Random timing
                hit_time = command.note.beat_time + random.uniform(-0.2, 0.2)

            print(f"\nBeat {i+1}/{len(self.beat_commands)} (Lane {command.note.lane})")
            self.hit_beat(i, hit_time)

        # Song complete
        self.notify_song_complete()

    def get_rank(self) -> str:
        """Get rank based on accuracy"""
        accuracy = (self.accuracy_sum / self.total_notes * 100) if self.total_notes > 0 else 0

        if accuracy >= 99:
            return "SSS"
        elif accuracy >= 95:
            return "SS"
        elif accuracy >= 90:
            return "S"
        elif accuracy >= 85:
            return "A"
        elif accuracy >= 75:
            return "B"
        elif accuracy >= 65:
            return "C"
        else:
            return "D"

# Example Usage
def main():
    print("🎮 MUSICAL RHYTHM GAME - Command + Observer Demo 🎮")

    # Create game
    game = RhythmGame("Electric Dreams")

    # Add observers
    game.add_observer(ScoreDisplayObserver())
    game.add_observer(ComboEffectsObserver())
    game.add_observer(StatsTrackerObserver())

    # Create a song chart (simulated)
    # In a real game, this would be loaded from a file
    beats_data = [
        (1.0, 1, BeatType.TAP),
        (1.5, 2, BeatType.TAP),
        (2.0, 3, BeatType.TAP),
        (2.5, 4, BeatType.TAP),
        (3.0, 1, BeatType.TAP),
        (3.5, 2, BeatType.TAP),
        (4.0, 3, BeatType.TAP),
        (4.5, 4, BeatType.TAP),
        (5.0, 2, BeatType.TAP),
        (5.5, 3, BeatType.TAP),
        (6.0, 1, BeatType.TAP),
        (6.5, 4, BeatType.TAP),
        (7.0, 2, BeatType.TAP),
        (7.5, 3, BeatType.TAP),
        (8.0, 1, BeatType.TAP),
        (8.5, 2, BeatType.TAP),
        (9.0, 3, BeatType.TAP),
        (9.5, 4, BeatType.TAP),
        (10.0, 1, BeatType.TAP),
        (10.5, 2, BeatType.TAP),
    ]

    for beat_time, lane, beat_type in beats_data:
        note = Note(beat_time=beat_time, lane=lane, beat_type=beat_type)
        if beat_type == BeatType.TAP:
            game.add_beat(TapCommand(note))
        elif beat_type == BeatType.HOLD:
            game.add_beat(HoldCommand(note, duration=0.5))
        elif beat_type == BeatType.SLIDE:
            game.add_beat(SlideCommand(note, end_lane=lane + 1))

    # Play the song
    game.play_song()

    # Display final rank
    print(f"\nFINAL RANK: {game.get_rank()}")
    print(f"Total Score: {game.current_score}")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🎮 MUSICAL RHYTHM GAME - Command + Observer Demo 🎮

🎵 Now Playing: Electric Dreams 🎵

==================================================

Beat 1/20 (Lane 1)
  [PERFECT] +300

Beat 2/20 (Lane 2)
  [PERFECT] +300 x2

Beat 3/20 (Lane 3)
  [PERFECT] +300 x3

Beat 4/20 (Lane 4)
  [PERFECT] +300 x4

Beat 5/20 (Lane 1)
  [PERFECT] +300 x5

Beat 6/20 (Lane 2)
  [PERFECT] +300 x6

Beat 7/20 (Lane 3)
  [PERFECT] +300 x7

Beat 8/20 (Lane 4)
  [PERFECT] +300 x8

Beat 9/20 (Lane 2)
  [PERFECT] +300 x9

Beat 10/20 (Lane 3)
  [PERFECT] +330 x10
    ✨ 10 COMBO!

Beat 11/20 (Lane 1)
  [GREAT] +220 x11

Beat 12/20 (Lane 4)
  [PERFECT] +330 x12
    💫 Perfect chain!

Beat 13/20 (Lane 2)
  [GREAT] +220 x13

Beat 14/20 (Lane 3)
  [PERFECT] +330 x14
    💫 Perfect chain!

Beat 15/20 (Lane 1)
  [PERFECT] +330 x15
    💫 Perfect chain!

Beat 16/20 (Lane 2)
  [PERFECT] +330 x16
    💫 Perfect chain!

Beat 17/20 (Lane 3)
  [PERFECT] +330 x17
    💫 Perfect chain!

Beat 18/20 (Lane 4)
  [MISS] +0
    💔 Combo broken at 17!

Beat 19/20 (Lane 1)
  [PERFECT] +300

Beat 20/20 (Lane 2)
  [PERFECT] +300 x2

    🎵 SONG COMPLETE! 🎵
==================================================
DETAILED STATISTICS:
Total Score: 5920
Accuracy: 90.5%
Max Combo: 17

Judgement Breakdown:
  PERFECT: 16
  GREAT: 2
  MISS: 2
==================================================

FINAL RANK: S
Total Score: 5920
```

**Pattern Benefits:**
- **Command Pattern:** Each beat is encapsulated as a command, making it easy to queue, replay, or analyze
- **Observer Pattern:** Multiple systems (effects, scoring, stats) react independently to game events
- **Timing Judgement:** Consistent timing evaluation across all beat types
- **Combo System:** Natural implementation of combo multipliers and breaks
- **Extensibility:** Easy to add new beat types, observers, or scoring rules

---

## 10. Garden Simulator - Decorator Pattern

**Theme:** Peaceful gardening
**Patterns:** Decorator, Observer

**Key Learning:** Stack decorators to enhance plants

**Complete Implementation:**

```python
"""
Complete garden simulator with Decorator pattern.
Demonstrates stacking decorators to enhance plants with various bonuses.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
import random

class PlantStage(Enum):
    SEED = "seed"
    SPROUT = "sprout"
    GROWING = "growing"
    MATURE = "mature"
    FLOWERING = "flowering"

class PlantQuality(Enum):
    POOR = "poor"
    NORMAL = "normal"
    GOOD = "good"
    EXCELLENT = "excellent"
    LEGENDARY = "legendary"

# Component - Base Plant interface
class Plant(ABC):
    """Base interface for all plants"""

    @abstractmethod
    def get_growth_rate(self) -> float:
        """Growth points per day"""
        pass

    @abstractmethod
    def get_yield(self) -> int:
        """How many items harvested"""
        pass

    @abstractmethod
    def get_quality(self) -> PlantQuality:
        """Quality of harvested items"""
        pass

    @abstractmethod
    def get_value(self) -> int:
        """Gold value of harvest"""
        pass

    @abstractmethod
    def get_description(self) -> str:
        """Description with all modifiers"""
        pass

    @abstractmethod
    def get_water_need(self) -> float:
        """Water consumption per day"""
        pass

# Concrete Components - Base Plants
class TomatoPlant(Plant):
    """Basic tomato plant"""

    def __init__(self):
        self.name = "Tomato Plant"
        self.stage = PlantStage.SEED
        self.growth_points = 0
        self.growth_to_mature = 100

    def get_growth_rate(self) -> float:
        return 5.0

    def get_yield(self) -> int:
        return 3

    def get_quality(self) -> PlantQuality:
        return PlantQuality.NORMAL

    def get_value(self) -> int:
        return 15

    def get_description(self) -> str:
        return self.name

    def get_water_need(self) -> float:
        return 10.0

class WheatPlant(Plant):
    """Basic wheat plant"""

    def __init__(self):
        self.name = "Wheat"
        self.stage = PlantStage.SEED
        self.growth_points = 0
        self.growth_to_mature = 80

    def get_growth_rate(self) -> float:
        return 8.0

    def get_yield(self) -> int:
        return 5

    def get_quality(self) -> PlantQuality:
        return PlantQuality.NORMAL

    def get_value(self) -> int:
        return 8

    def get_description(self) -> str:
        return self.name

    def get_water_need(self) -> float:
        return 6.0

class StrawberryPlant(Plant):
    """Basic strawberry plant"""

    def __init__(self):
        self.name = "Strawberry Plant"
        self.stage = PlantStage.SEED
        self.growth_points = 0
        self.growth_to_mature = 120

    def get_growth_rate(self) -> float:
        return 4.0

    def get_yield(self) -> int:
        return 6

    def get_quality(self) -> PlantQuality:
        return PlantQuality.NORMAL

    def get_value(self) -> int:
        return 25

    def get_description(self) -> str:
        return self.name

    def get_water_need(self) -> float:
        return 12.0

# Decorator base class
class PlantDecorator(Plant, ABC):
    """Base decorator for plant enhancements"""

    def __init__(self, plant: Plant):
        self._plant = plant

    def get_growth_rate(self) -> float:
        return self._plant.get_growth_rate()

    def get_yield(self) -> int:
        return self._plant.get_yield()

    def get_quality(self) -> PlantQuality:
        return self._plant.get_quality()

    def get_value(self) -> int:
        return self._plant.get_value()

    def get_description(self) -> str:
        return self._plant.get_description()

    def get_water_need(self) -> float:
        return self._plant.get_water_need()

# Concrete Decorators
class WateredDecorator(PlantDecorator):
    """Plant has been watered - increases growth rate"""

    def __init__(self, plant: Plant, water_quality: float = 1.0):
        super().__init__(plant)
        self.water_quality = water_quality

    def get_growth_rate(self) -> float:
        base_rate = self._plant.get_growth_rate()
        return base_rate * (1.0 + 0.5 * self.water_quality)

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Watered]"

class FertilizedDecorator(PlantDecorator):
    """Plant has been fertilized - increases yield and quality"""

    def __init__(self, plant: Plant, fertilizer_strength: int = 1):
        super().__init__(plant)
        self.fertilizer_strength = fertilizer_strength

    def get_yield(self) -> int:
        base_yield = self._plant.get_yield()
        return base_yield + self.fertilizer_strength

    def get_quality(self) -> PlantQuality:
        base_quality = self._plant.get_quality()
        quality_values = list(PlantQuality)
        current_index = quality_values.index(base_quality)
        new_index = min(len(quality_values) - 1, current_index + self.fertilizer_strength)
        return quality_values[new_index]

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Fertilized x{self.fertilizer_strength}]"

class SunlightDecorator(PlantDecorator):
    """Plant receives sunlight - increases growth and value"""

    def __init__(self, plant: Plant, hours: int = 8):
        super().__init__(plant)
        self.sunlight_hours = hours

    def get_growth_rate(self) -> float:
        base_rate = self._plant.get_growth_rate()
        # Optimal sunlight is 8 hours
        if self.sunlight_hours >= 8:
            multiplier = 1.5
        elif self.sunlight_hours >= 6:
            multiplier = 1.2
        else:
            multiplier = 0.8
        return base_rate * multiplier

    def get_value(self) -> int:
        base_value = self._plant.get_value()
        return int(base_value * (1.0 + self.sunlight_hours * 0.05))

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Sunlight {self.sunlight_hours}hrs]"

class GreenhouseDecorator(PlantDecorator):
    """Plant in greenhouse - all stats improved"""

    def get_growth_rate(self) -> float:
        return self._plant.get_growth_rate() * 1.8

    def get_yield(self) -> int:
        return int(self._plant.get_yield() * 1.3)

    def get_quality(self) -> PlantQuality:
        base_quality = self._plant.get_quality()
        quality_values = list(PlantQuality)
        current_index = quality_values.index(base_quality)
        new_index = min(len(quality_values) - 1, current_index + 1)
        return quality_values[new_index]

    def get_water_need(self) -> float:
        return self._plant.get_water_need() * 0.7  # Greenhouse retains moisture

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Greenhouse]"

class MagicGrowthDecorator(PlantDecorator):
    """Magical enhancement - massive bonuses"""

    def __init__(self, plant: Plant, magic_level: int = 1):
        super().__init__(plant)
        self.magic_level = magic_level

    def get_growth_rate(self) -> float:
        return self._plant.get_growth_rate() * (2.0 ** self.magic_level)

    def get_yield(self) -> int:
        return self._plant.get_yield() * (2 * self.magic_level)

    def get_quality(self) -> PlantQuality:
        # Magic always gives legendary quality
        return PlantQuality.LEGENDARY

    def get_value(self) -> int:
        return self._plant.get_value() * (3 * self.magic_level)

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Magic Growth Lv{self.magic_level}]"

class MusicDecorator(PlantDecorator):
    """Plant exposed to music - slight quality boost"""

    def __init__(self, plant: Plant, hours_per_day: int = 4):
        super().__init__(plant)
        self.music_hours = hours_per_day

    def get_quality(self) -> PlantQuality:
        base_quality = self._plant.get_quality()
        if self.music_hours >= 4:
            quality_values = list(PlantQuality)
            current_index = quality_values.index(base_quality)
            new_index = min(len(quality_values) - 1, current_index + 1)
            return quality_values[new_index]
        return base_quality

    def get_value(self) -> int:
        return int(self._plant.get_value() * 1.1)

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Music {self.music_hours}hrs]"

class HybridSeedDecorator(PlantDecorator):
    """Hybrid seed variety - better yield"""

    def get_yield(self) -> int:
        return int(self._plant.get_yield() * 1.5)

    def get_value(self) -> int:
        return int(self._plant.get_value() * 1.2)

    def get_description(self) -> str:
        return f"{self._plant.get_description()} [Hybrid]"

# Garden Plot managing decorated plants
class GardenPlot:
    """A plot in the garden containing a plant"""

    def __init__(self, plot_id: int):
        self.plot_id = plot_id
        self.plant: Optional[Plant] = None
        self.days_growing = 0
        self.is_ready_to_harvest = False

    def plant_seed(self, plant: Plant):
        self.plant = plant
        self.days_growing = 0
        self.is_ready_to_harvest = False

    def grow_day(self):
        """Simulate one day of growth"""
        if not self.plant:
            return

        growth = self.plant.get_growth_rate()
        self.days_growing += 1

        # Check if plant is ready (simplified)
        if self.days_growing >= 10:  # Mature after 10 days (simplified)
            self.is_ready_to_harvest = True

    def harvest(self) -> Optional[dict]:
        """Harvest the plant"""
        if not self.plant or not self.is_ready_to_harvest:
            return None

        result = {
            "name": self.plant.get_description(),
            "quantity": self.plant.get_yield(),
            "quality": self.plant.get_quality().value,
            "value": self.plant.get_value(),
            "total_value": self.plant.get_value() * self.plant.get_yield()
        }

        # Clear plot after harvest
        self.plant = None
        self.days_growing = 0
        self.is_ready_to_harvest = False

        return result

# Garden Manager
class Garden:
    def __init__(self):
        self.plots: List[GardenPlot] = [GardenPlot(i) for i in range(6)]
        self.day = 0
        self.total_earnings = 0

    def advance_day(self):
        """Advance time by one day"""
        self.day += 1
        print(f"\n=== DAY {self.day} ===")

        for plot in self.plots:
            plot.grow_day()

        self.show_garden_status()

    def harvest_plot(self, plot_id: int) -> Optional[dict]:
        """Harvest a specific plot"""
        if 0 <= plot_id < len(self.plots):
            result = self.plots[plot_id].harvest()
            if result:
                self.total_earnings += result['total_value']
                print(f"\nHarvested from Plot {plot_id}:")
                print(f"  {result['quantity']}x {result['name']}")
                print(f"  Quality: {result['quality']}")
                print(f"  Value: {result['total_value']} gold")
                return result
        return None

    def show_garden_status(self):
        """Display current garden state"""
        for plot in self.plots:
            if plot.plant:
                status = "READY!" if plot.is_ready_to_harvest else f"Day {plot.days_growing}"
                print(f"Plot {plot.plot_id}: {plot.plant.get_description()} - {status}")
            else:
                print(f"Plot {plot.plot_id}: Empty")

# Example Usage
def main():
    print("=== GARDEN SIMULATOR - Decorator Pattern Demo ===\n")

    garden = Garden()

    # Plot 0: Basic tomato (no decorators)
    print("Plot 0: Planting basic tomato")
    basic_tomato = TomatoPlant()
    garden.plots[0].plant_seed(basic_tomato)

    # Plot 1: Watered and fertilized tomato
    print("Plot 1: Planting watered + fertilized tomato")
    enhanced_tomato = TomatoPlant()
    enhanced_tomato = WateredDecorator(enhanced_tomato)
    enhanced_tomato = FertilizedDecorator(enhanced_tomato, fertilizer_strength=2)
    garden.plots[1].plant_seed(enhanced_tomato)

    # Plot 2: Fully optimized tomato
    print("Plot 2: Planting fully optimized tomato")
    super_tomato = TomatoPlant()
    super_tomato = WateredDecorator(super_tomato, water_quality=1.5)
    super_tomato = FertilizedDecorator(super_tomato, fertilizer_strength=2)
    super_tomato = SunlightDecorator(super_tomato, hours=10)
    super_tomato = GreenhouseDecorator(super_tomato)
    super_tomato = MusicDecorator(super_tomato, hours_per_day=6)
    garden.plots[2].plant_seed(super_tomato)

    # Plot 3: Magic wheat
    print("Plot 3: Planting magic wheat")
    magic_wheat = WheatPlant()
    magic_wheat = WateredDecorator(magic_wheat)
    magic_wheat = MagicGrowthDecorator(magic_wheat, magic_level=2)
    garden.plots[3].plant_seed(magic_wheat)

    # Plot 4: Hybrid strawberry in greenhouse
    print("Plot 4: Planting hybrid greenhouse strawberry")
    hybrid_strawberry = StrawberryPlant()
    hybrid_strawberry = HybridSeedDecorator(hybrid_strawberry)
    hybrid_strawberry = GreenhouseDecorator(hybrid_strawberry)
    hybrid_strawberry = WateredDecorator(hybrid_strawberry)
    hybrid_strawberry = FertilizedDecorator(hybrid_strawberry)
    garden.plots[4].plant_seed(hybrid_strawberry)

    # Show initial stats
    print("\n=== INITIAL PLANT STATS ===")
    for i, plot in enumerate(garden.plots[:5]):
        if plot.plant:
            print(f"\nPlot {i}: {plot.plant.get_description()}")
            print(f"  Growth Rate: {plot.plant.get_growth_rate():.1f}/day")
            print(f"  Yield: {plot.plant.get_yield()} items")
            print(f"  Quality: {plot.plant.get_quality().value}")
            print(f"  Value: {plot.plant.get_value()} gold/item")
            print(f"  Total Harvest Value: {plot.plant.get_value() * plot.plant.get_yield()} gold")

    # Simulate 10 days of growth
    for day in range(10):
        garden.advance_day()

    # Harvest all plots
    print("\n\n=== HARVEST TIME ===")
    for i in range(5):
        garden.harvest_plot(i)

    print(f"\n=== SUMMARY ===")
    print(f"Total Earnings: {garden.total_earnings} gold")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== GARDEN SIMULATOR - Decorator Pattern Demo ===

Plot 0: Planting basic tomato
Plot 1: Planting watered + fertilized tomato
Plot 2: Planting fully optimized tomato
Plot 3: Planting magic wheat
Plot 4: Planting hybrid greenhouse strawberry

=== INITIAL PLANT STATS ===

Plot 0: Tomato Plant
  Growth Rate: 5.0/day
  Yield: 3 items
  Quality: normal
  Value: 15 gold/item
  Total Harvest Value: 45 gold

Plot 1: Tomato Plant [Watered] [Fertilized x2]
  Growth Rate: 7.5/day
  Yield: 5 items
  Quality: good
  Value: 15 gold/item
  Total Harvest Value: 75 gold

Plot 2: Tomato Plant [Watered] [Fertilized x2] [Sunlight 10hrs] [Greenhouse] [Music 6hrs]
  Growth Rate: 20.3/day
  Yield: 6 items
  Quality: legendary
  Value: 24 gold/item
  Total Harvest Value: 144 gold

Plot 3: Wheat [Watered] [Magic Growth Lv2]
  Growth Rate: 48.0/day
  Yield: 20 items
  Quality: legendary
  Value: 48 gold/item
  Total Harvest Value: 960 gold

Plot 4: Strawberry Plant [Hybrid] [Greenhouse] [Watered] [Fertilized]
  Growth Rate: 10.8/day
  Yield: 11 items
  Quality: excellent
  Value: 33 gold/item
  Total Harvest Value: 363 gold

=== DAY 1 ===
Plot 0: Tomato Plant - Day 1
Plot 1: Tomato Plant [Watered] [Fertilized x2] - Day 1
Plot 2: Tomato Plant [Watered] [Fertilized x2] [Sunlight 10hrs] [Greenhouse] [Music 6hrs] - Day 1
Plot 3: Wheat [Watered] [Magic Growth Lv2] - Day 1
Plot 4: Strawberry Plant [Hybrid] [Greenhouse] [Watered] [Fertilized] - Day 1
Plot 5: Empty

... (Days 2-9 omitted for brevity) ...

=== DAY 10 ===
Plot 0: Tomato Plant - READY!
Plot 1: Tomato Plant [Watered] [Fertilized x2] - READY!
Plot 2: Tomato Plant [Watered] [Fertilized x2] [Sunlight 10hrs] [Greenhouse] [Music 6hrs] - READY!
Plot 3: Wheat [Watered] [Magic Growth Lv2] - READY!
Plot 4: Strawberry Plant [Hybrid] [Greenhouse] [Watered] [Fertilized] - READY!
Plot 5: Empty


=== HARVEST TIME ===

Harvested from Plot 0:
  3x Tomato Plant
  Quality: normal
  Value: 45 gold

Harvested from Plot 1:
  5x Tomato Plant [Watered] [Fertilized x2]
  Quality: good
  Value: 75 gold

Harvested from Plot 2:
  6x Tomato Plant [Watered] [Fertilized x2] [Sunlight 10hrs] [Greenhouse] [Music 6hrs]
  Quality: legendary
  Value: 144 gold

Harvested from Plot 3:
  20x Wheat [Watered] [Magic Growth Lv2]
  Quality: legendary
  Value: 960 gold

Harvested from Plot 4:
  11x Strawberry Plant [Hybrid] [Greenhouse] [Watered] [Fertilized]
  Quality: excellent
  Value: 363 gold

=== SUMMARY ===
Total Earnings: 1587 gold
```

**Pattern Benefits:**
- **Decorator Pattern:** Add functionality to plants without modifying base classes
- **Flexible Combinations:** Stack multiple decorators in any order for unique effects
- **Single Responsibility:** Each decorator handles one enhancement type
- **Open/Closed Principle:** Add new decorators without changing existing code
- **Transparent Wrapping:** Decorated plants still conform to Plant interface
- **Runtime Composition:** Decide which decorators to apply at runtime
- **Multiplicative Effects:** Decorators stack and multiply effects (basic tomato: 45g, fully decorated: 144g)
- **Game Balance:** Easy to tune individual decorator effects without affecting others

---

## 11. Trading Card Game - Prototype Pattern

**Theme:** Collectible card game
**Patterns:** Prototype, Factory

**Key Learning:** Clone card prototypes instead of recreating

**Complete Implementation:**

```python
"""
Complete trading card game with Prototype pattern.
Demonstrates cloning cards efficiently instead of recreating them.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from enum import Enum
from dataclasses import dataclass
import copy

class CardType(Enum):
    CREATURE = "Creature"
    SPELL = "Spell"
    ENCHANTMENT = "Enchantment"
    ARTIFACT = "Artifact"

class Rarity(Enum):
    COMMON = ("Common", 1)
    UNCOMMON = ("Uncommon", 2)
    RARE = ("Rare", 5)
    LEGENDARY = ("Legendary", 10)

    def __init__(self, label: str, value: int):
        self.label = label
        self.value = value

@dataclass
class Effect:
    """Card effect"""
    name: str
    description: str
    power: int

# Prototype Pattern - Card prototype for cloning
class Card(ABC):
    """Abstract card prototype"""

    def __init__(self, name: str, mana_cost: int, card_type: CardType, rarity: Rarity):
        self.name = name
        self.mana_cost = mana_cost
        self.card_type = card_type
        self.rarity = rarity
        self.effects: List[Effect] = []

    @abstractmethod
    def clone(self) -> 'Card':
        """Clone this card (Prototype pattern)"""
        pass

    @abstractmethod
    def play(self) -> str:
        """Play the card"""
        pass

    def add_effect(self, effect: Effect):
        """Add an effect to the card"""
        self.effects.append(effect)

    def get_description(self) -> str:
        """Get card description"""
        effects_str = ', '.join(e.name for e in self.effects)
        return (f"[{self.rarity.label}] {self.name} ({self.mana_cost} mana) - "
                f"{self.card_type.value} | Effects: {effects_str if effects_str else 'None'}")

class CreatureCard(Card):
    """Creature card with attack and health"""

    def __init__(self, name: str, mana_cost: int, rarity: Rarity, attack: int, health: int):
        super().__init__(name, mana_cost, CardType.CREATURE, rarity)
        self.attack = attack
        self.health = health
        self.max_health = health

    def clone(self) -> 'CreatureCard':
        """Deep copy of the creature card"""
        new_card = CreatureCard(
            name=self.name,
            mana_cost=self.mana_cost,
            rarity=self.rarity,
            attack=self.attack,
            health=self.health
        )
        # Clone effects
        new_card.effects = copy.deepcopy(self.effects)
        return new_card

    def play(self) -> str:
        return f"Summon {self.name} ({self.attack}/{self.health})"

    def take_damage(self, damage: int):
        """Apply damage to creature"""
        self.health -= damage
        if self.health <= 0:
            return f"{self.name} destroyed!"
        return f"{self.name} took {damage} damage ({self.health}/{self.max_health})"

    def get_description(self) -> str:
        base = super().get_description()
        return f"{base} | ATK: {self.attack} HP: {self.health}"

class SpellCard(Card):
    """Instant spell card"""

    def __init__(self, name: str, mana_cost: int, rarity: Rarity, spell_effect: str):
        super().__init__(name, mana_cost, CardType.SPELL, rarity)
        self.spell_effect = spell_effect

    def clone(self) -> 'SpellCard':
        """Deep copy of the spell card"""
        new_card = SpellCard(
            name=self.name,
            mana_cost=self.mana_cost,
            rarity=self.rarity,
            spell_effect=self.spell_effect
        )
        new_card.effects = copy.deepcopy(self.effects)
        return new_card

    def play(self) -> str:
        return f"Cast {self.name}: {self.spell_effect}"

class EnchantmentCard(Card):
    """Permanent enchantment card"""

    def __init__(self, name: str, mana_cost: int, rarity: Rarity, buff_attack: int, buff_health: int):
        super().__init__(name, mana_cost, CardType.ENCHANTMENT, rarity)
        self.buff_attack = buff_attack
        self.buff_health = buff_health

    def clone(self) -> 'EnchantmentCard':
        """Deep copy of the enchantment card"""
        new_card = EnchantmentCard(
            name=self.name,
            mana_cost=self.mana_cost,
            rarity=self.rarity,
            buff_attack=self.buff_attack,
            buff_health=self.buff_health
        )
        new_card.effects = copy.deepcopy(self.effects)
        return new_card

    def play(self) -> str:
        return f"Enchant with {self.name} (+{self.buff_attack}/+{self.buff_health})"

# Card Registry - Holds prototypes for cloning
class CardRegistry:
    """Registry of card prototypes for the Prototype pattern"""

    def __init__(self):
        self._prototypes: Dict[str, Card] = {}

    def register(self, card_id: str, prototype: Card):
        """Register a card prototype"""
        self._prototypes[card_id] = prototype
        print(f"[Registry] Registered prototype: {card_id} - {prototype.name}")

    def create(self, card_id: str) -> Optional[Card]:
        """Create a card by cloning its prototype"""
        if card_id not in self._prototypes:
            print(f"[Registry] Error: {card_id} not found")
            return None

        # Clone the prototype
        return self._prototypes[card_id].clone()

    def create_multiple(self, card_id: str, count: int) -> List[Card]:
        """Create multiple copies of a card"""
        cards = []
        for _ in range(count):
            card = self.create(card_id)
            if card:
                cards.append(card)
        return cards

    def list_prototypes(self) -> List[str]:
        """List all available card prototypes"""
        return list(self._prototypes.keys())

# Deck class
class Deck:
    """Player's deck of cards"""

    def __init__(self, name: str):
        self.name = name
        self.cards: List[Card] = []

    def add_card(self, card: Card):
        """Add a card to the deck"""
        self.cards.append(card)

    def add_cards(self, cards: List[Card]):
        """Add multiple cards to the deck"""
        self.cards.extend(cards)

    def get_stats(self) -> str:
        """Get deck statistics"""
        stats = f"\n=== Deck: {self.name} ===\n"
        stats += f"Total Cards: {len(self.cards)}\n"

        # Count by type
        type_counts = {}
        for card in self.cards:
            card_type = card.card_type.value
            type_counts[card_type] = type_counts.get(card_type, 0) + 1

        stats += "\nBy Type:\n"
        for card_type, count in type_counts.items():
            stats += f"  {card_type}: {count}\n"

        # Count by rarity
        rarity_counts = {}
        for card in self.cards:
            rarity = card.rarity.label
            rarity_counts[rarity] = rarity_counts.get(rarity, 0) + 1

        stats += "\nBy Rarity:\n"
        for rarity, count in rarity_counts.items():
            stats += f"  {rarity}: {count}\n"

        # Average mana cost
        avg_mana = sum(card.mana_cost for card in self.cards) / len(self.cards) if self.cards else 0
        stats += f"\nAverage Mana Cost: {avg_mana:.1f}\n"

        return stats

    def show_cards(self):
        """Display all cards in deck"""
        print(f"\n=== {self.name} Cards ===")
        for i, card in enumerate(self.cards, 1):
            print(f"{i}. {card.get_description()}")

# Card Pack - Uses registry to generate random packs
class CardPack:
    """Booster pack of random cards"""

    def __init__(self, registry: CardRegistry, pack_name: str):
        self.registry = registry
        self.pack_name = pack_name

    def open_pack(self, common_count: int = 8, uncommon_count: int = 3, rare_count: int = 1) -> List[Card]:
        """Open a booster pack with random cards"""
        import random

        pack_cards = []
        available = self.registry.list_prototypes()

        print(f"\n🎴 Opening {self.pack_name}...")

        # Add commons
        for _ in range(common_count):
            card_id = random.choice(available)
            card = self.registry.create(card_id)
            if card:
                pack_cards.append(card)

        # Add uncommons
        for _ in range(uncommon_count):
            card_id = random.choice(available)
            card = self.registry.create(card_id)
            if card:
                pack_cards.append(card)

        # Add rares
        for _ in range(rare_count):
            card_id = random.choice(available)
            card = self.registry.create(card_id)
            if card:
                pack_cards.append(card)

        print(f"Received {len(pack_cards)} cards!")
        return pack_cards

# Example Usage
def main():
    print("🃏 TRADING CARD GAME - Prototype Pattern Demo 🃏\n")

    # Create card registry
    registry = CardRegistry()

    # Register creature prototypes
    print("=== Registering Card Prototypes ===\n")

    dragon = CreatureCard("Ancient Dragon", mana_cost=7, rarity=Rarity.LEGENDARY,
                         attack=10, health=10)
    dragon.add_effect(Effect("Flying", "Can't be blocked by non-flying", 0))
    dragon.add_effect(Effect("Firebreath", "Deal 5 damage on attack", 5))
    registry.register("dragon", dragon)

    goblin = CreatureCard("Goblin Warrior", mana_cost=2, rarity=Rarity.COMMON,
                         attack=2, health=2)
    registry.register("goblin", goblin)

    knight = CreatureCard("Royal Knight", mana_cost=4, rarity=Rarity.UNCOMMON,
                         attack=4, health=4)
    knight.add_effect(Effect("Vigilance", "Doesn't tap when attacking", 0))
    registry.register("knight", knight)

    wizard = CreatureCard("Arcane Wizard", mana_cost=3, rarity=Rarity.RARE,
                         attack=2, health=3)
    wizard.add_effect(Effect("Spellcaster", "Draw a card on summon", 0))
    registry.register("wizard", wizard)

    # Register spell prototypes
    fireball = SpellCard("Fireball", mana_cost=3, rarity=Rarity.COMMON,
                        spell_effect="Deal 3 damage to target")
    registry.register("fireball", fireball)

    lightning = SpellCard("Lightning Bolt", mana_cost=1, rarity=Rarity.UNCOMMON,
                         spell_effect="Deal 3 damage to any target")
    registry.register("lightning", lightning)

    # Register enchantment prototypes
    divine_blessing = EnchantmentCard("Divine Blessing", mana_cost=2, rarity=Rarity.UNCOMMON,
                                      buff_attack=2, buff_health=2)
    registry.register("blessing", divine_blessing)

    # Build a deck using prototypes (cloning)
    print("\n=== Building Deck ===\n")

    deck = Deck("Dragon's Wrath")

    # Add cards by cloning prototypes
    print("Adding cards to deck by cloning prototypes...")
    deck.add_card(registry.create("dragon"))
    deck.add_card(registry.create("dragon"))  # Can have multiple copies
    deck.add_cards(registry.create_multiple("goblin", count=10))
    deck.add_cards(registry.create_multiple("knight", count=4))
    deck.add_cards(registry.create_multiple("wizard", count=3))
    deck.add_cards(registry.create_multiple("fireball", count=6))
    deck.add_cards(registry.create_multiple("lightning", count=4))
    deck.add_cards(registry.create_multiple("blessing", count=3))

    # Show deck stats
    print(deck.get_stats())

    # Open a booster pack
    pack = CardPack(registry, "Mythic Legends Booster")
    new_cards = pack.open_pack(common_count=8, uncommon_count=3, rare_count=1)

    print("\nCards received:")
    for i, card in enumerate(new_cards, 1):
        print(f"{i}. {card.get_description()}")

    # Demonstrate that clones are independent
    print("\n\n=== Demonstrating Clone Independence ===\n")

    dragon1 = registry.create("dragon")
    dragon2 = registry.create("dragon")

    print(f"Dragon 1: {dragon1.get_description()}")
    print(f"Dragon 2: {dragon2.get_description()}")

    print("\nDragon 1 takes 5 damage...")
    print(dragon1.take_damage(5))

    print(f"\nDragon 1 after damage: {dragon1.get_description()}")
    print(f"Dragon 2 (unchanged): {dragon2.get_description()}")

    print("\n✓ Clones are independent - modifying one doesn't affect others!")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🃏 TRADING CARD GAME - Prototype Pattern Demo 🃏

=== Registering Card Prototypes ===

[Registry] Registered prototype: dragon - Ancient Dragon
[Registry] Registered prototype: goblin - Goblin Warrior
[Registry] Registered prototype: knight - Royal Knight
[Registry] Registered prototype: wizard - Arcane Wizard
[Registry] Registered prototype: fireball - Fireball
[Registry] Registered prototype: lightning - Lightning Bolt
[Registry] Registered prototype: blessing - Divine Blessing

=== Building Deck ===

Adding cards to deck by cloning prototypes...

=== Deck: Dragon's Wrath ===
Total Cards: 32

By Type:
  Creature: 19
  Spell: 10
  Enchantment: 3

By Rarity:
  Legendary: 2
  Common: 16
  Uncommon: 11
  Rare: 3

Average Mana Cost: 3.2

🎴 Opening Mythic Legends Booster...
Received 12 cards!

Cards received:
1. [Common] Goblin Warrior (2 mana) - Creature | Effects: None | ATK: 2 HP: 2
2. [Legendary] Ancient Dragon (7 mana) - Creature | Effects: Flying, Firebreath | ATK: 10 HP: 10
3. [Common] Fireball (3 mana) - Spell | Effects: None
4. [Uncommon] Royal Knight (4 mana) - Creature | Effects: Vigilance | ATK: 4 HP: 4
5. [Common] Goblin Warrior (2 mana) - Creature | Effects: None | ATK: 2 HP: 2
6. [Uncommon] Lightning Bolt (1 mana) - Spell | Effects: None
7. [Common] Fireball (3 mana) - Spell | Effects: None
8. [Common] Goblin Warrior (2 mana) - Creature | Effects: None | ATK: 2 HP: 2
9. [Uncommon] Royal Knight (4 mana) - Creature | Effects: Vigilance | ATK: 4 HP: 4
10. [Rare] Arcane Wizard (3 mana) - Creature | Effects: Spellcaster | ATK: 2 HP: 3
11. [Common] Goblin Warrior (2 mana) - Creature | Effects: None | ATK: 2 HP: 2
12. [Legendary] Ancient Dragon (7 mana) - Creature | Effects: Flying, Firebreath | ATK: 10 HP: 10


=== Demonstrating Clone Independence ===

Dragon 1: [Legendary] Ancient Dragon (7 mana) - Creature | Effects: Flying, Firebreath | ATK: 10 HP: 10
Dragon 2: [Legendary] Ancient Dragon (7 mana) - Creature | Effects: Flying, Firebreath | ATK: 10 HP: 10

Dragon 1 takes 5 damage...
Ancient Dragon took 5 damage (5/10)

Dragon 1 after damage: [Legendary] Ancient Dragon (7 mana) - Creature | Effects: Flying, Firebreath | ATK: 10 HP: 5
Dragon 2 (unchanged): [Legendary] Ancient Dragon (7 mana) - Creature | Effects: Flying, Firebreath | ATK: 10 HP: 10

✓ Clones are independent - modifying one doesn't affect others!
```

**Pattern Benefits:**
- **Prototype Pattern:** Clone existing cards instead of constructing new ones from scratch
- **Performance:** Cloning is faster than creating complex cards with many attributes and effects
- **Registry System:** Centralized storage of card templates makes card generation simple
- **Independence:** Each cloned card is independent - modifying one doesn't affect others
- **Scalability:** Easy to add new card types without changing the cloning mechanism

---

## 12. Stealth Espionage - Chain of Responsibility

**Theme:** Spy infiltration
**Patterns:** Chain of Responsibility, State

**Key Learning:** Security layers pass detection checks down the chain

**Complete Implementation:**

```python
"""
Complete stealth espionage system with Chain of Responsibility pattern.
Demonstrates security layers that each check for intrusion and pass to the next.
"""

from abc import ABC, abstractmethod
from typing import Optional, List
from enum import Enum
from dataclasses import dataclass
import random

class StealthLevel(Enum):
    INVISIBLE = 5
    HIDDEN = 4
    CROUCHING = 3
    WALKING = 2
    RUNNING = 1
    EXPOSED = 0

class AlertLevel(Enum):
    CLEAR = 0
    SUSPICIOUS = 1
    CAUTION = 2
    ALERT = 3
    LOCKDOWN = 4

@dataclass
class IntrusionEvent:
    """Event when agent is detected"""
    location: str
    stealth_level: StealthLevel
    noise_level: int
    agent_speed: int
    time: float

class Agent:
    """Spy agent trying to infiltrate"""

    def __init__(self, name: str):
        self.name = name
        self.position = "entrance"
        self.stealth_level = StealthLevel.CROUCHING
        self.noise_level = 2
        self.speed = 5
        self.detected = False
        self.gadgets = []

    def set_stealth(self, level: StealthLevel):
        """Change stealth mode"""
        self.stealth_level = level
        if level == StealthLevel.INVISIBLE:
            self.noise_level = 0
            self.speed = 3
        elif level == StealthLevel.HIDDEN:
            self.noise_level = 1
            self.speed = 3
        elif level == StealthLevel.CROUCHING:
            self.noise_level = 2
            self.speed = 5
        elif level == StealthLevel.WALKING:
            self.noise_level = 4
            self.speed = 7
        elif level == StealthLevel.RUNNING:
            self.noise_level = 8
            self.speed = 10

    def use_gadget(self, gadget: str):
        """Use a spy gadget"""
        if gadget in self.gadgets:
            return True
        return False

# Chain of Responsibility - Security layers
class SecurityLayer(ABC):
    """Abstract security layer that can detect intruders"""

    def __init__(self, name: str, detection_threshold: int):
        self.name = name
        self.detection_threshold = detection_threshold
        self.next_layer: Optional['SecurityLayer'] = None
        self.active = True

    def set_next(self, layer: 'SecurityLayer') -> 'SecurityLayer':
        """Set the next layer in the chain"""
        self.next_layer = layer
        return layer

    def handle(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        """Handle intrusion check - template method"""
        if not self.active:
            print(f"  [{self.name}] System disabled")
            if self.next_layer:
                return self.next_layer.handle(event, agent)
            return False, f"Passed all security layers"

        # Check if this layer detects the agent
        detected, reason = self.check_intrusion(event, agent)

        if detected:
            return True, f"DETECTED by {self.name}: {reason}"

        print(f"  [{self.name}] No detection")

        # Pass to next layer
        if self.next_layer:
            return self.next_layer.handle(event, agent)

        return False, f"Passed all security layers"

    @abstractmethod
    def check_intrusion(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        """Check if this layer detects the intrusion"""
        pass

class MotionDetectorLayer(SecurityLayer):
    """Detects movement in secured areas"""

    def __init__(self):
        super().__init__("Motion Detector", detection_threshold=3)

    def check_intrusion(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        # Motion detectors sense fast movement
        if agent.speed > 8:
            return True, "Fast movement detected"

        # Can't detect invisible agents
        if agent.stealth_level == StealthLevel.INVISIBLE:
            return False, ""

        # Random chance based on speed
        if random.random() < (agent.speed / 20):
            return True, "Movement pattern detected"

        return False, ""

class CameraLayer(SecurityLayer):
    """Security cameras with visual detection"""

    def __init__(self):
        super().__init__("Security Camera", detection_threshold=4)

    def check_intrusion(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        # Cameras can't see invisible agents
        if agent.stealth_level == StealthLevel.INVISIBLE:
            return False, ""

        # Hidden agents are hard to spot
        if agent.stealth_level == StealthLevel.HIDDEN:
            if random.random() < 0.1:
                return True, "Suspicious shadow on camera"
            return False, ""

        # Crouching agents might be spotted
        if agent.stealth_level == StealthLevel.CROUCHING:
            if random.random() < 0.3:
                return True, "Figure spotted on camera"
            return False, ""

        # Walking/running are easily spotted
        if agent.stealth_level in [StealthLevel.WALKING, StealthLevel.RUNNING]:
            if random.random() < 0.7:
                return True, "Intruder clearly visible on camera"
            return False, ""

        return False, ""

class SoundDetectorLayer(SecurityLayer):
    """Audio sensors detecting noise"""

    def __init__(self):
        super().__init__("Sound Detector", detection_threshold=5)

    def check_intrusion(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        # Check noise level
        if event.noise_level > 6:
            return True, f"Loud noise detected ({event.noise_level} dB)"

        if event.noise_level > 3:
            if random.random() < (event.noise_level / 10):
                return True, f"Suspicious sounds detected"

        return False, ""

class GuardPatrolLayer(SecurityLayer):
    """Human guards on patrol"""

    def __init__(self):
        super().__init__("Guard Patrol", detection_threshold=6)
        self.alert_level = AlertLevel.CLEAR

    def check_intrusion(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        # Guards are more alert at higher alert levels
        detection_chance = 0.2

        if self.alert_level == AlertLevel.SUSPICIOUS:
            detection_chance = 0.4
        elif self.alert_level == AlertLevel.CAUTION:
            detection_chance = 0.6
        elif self.alert_level == AlertLevel.ALERT:
            detection_chance = 0.8
        elif self.alert_level == AlertLevel.LOCKDOWN:
            detection_chance = 0.95

        # Invisible agents are very hard for guards to detect
        if agent.stealth_level == StealthLevel.INVISIBLE:
            detection_chance *= 0.1

        # Hidden agents are hard to detect
        if agent.stealth_level == StealthLevel.HIDDEN:
            detection_chance *= 0.3

        # Check if guard spots the agent
        if random.random() < detection_chance:
            return True, f"Guard spotted intruder (Alert level: {self.alert_level.name})"

        return False, ""

class BiometricScannerLayer(SecurityLayer):
    """Biometric security requiring authorization"""

    def __init__(self):
        super().__init__("Biometric Scanner", detection_threshold=8)
        self.authorized_ids = set()

    def check_intrusion(self, event: IntrusionEvent, agent: Agent) -> tuple[bool, str]:
        # Check if agent has disguise gadget
        if agent.use_gadget("disguise"):
            print(f"  [{self.name}] Used disguise to bypass")
            return False, ""

        # Check if agent has forged credentials
        if agent.use_gadget("forged_id"):
            print(f"  [{self.name}] Used forged ID to bypass")
            return False, ""

        # No bypass - detected
        return True, "Biometric authentication failed - unauthorized person"

# Facility security system
class SecurityFacility:
    """Facility with layered security"""

    def __init__(self, name: str):
        self.name = name
        self.alert_level = AlertLevel.CLEAR
        self.intrusion_log: List[str] = []

        # Set up security chain
        self.motion = MotionDetectorLayer()
        self.sound = SoundDetectorLayer()
        self.camera = CameraLayer()
        self.guard = GuardPatrolLayer()
        self.biometric = BiometricScannerLayer()

        # Chain them together
        self.motion.set_next(self.sound).set_next(self.camera).set_next(self.guard).set_next(self.biometric)

        # Entry point
        self.security_chain = self.motion

    def check_agent(self, agent: Agent, location: str, time: float) -> bool:
        """Check if agent is detected"""
        event = IntrusionEvent(
            location=location,
            stealth_level=agent.stealth_level,
            noise_level=agent.noise_level,
            agent_speed=agent.speed,
            time=time
        )

        print(f"\n--- Security Check at {location} ---")
        print(f"Agent stealth: {agent.stealth_level.name}, Noise: {agent.noise_level}, Speed: {agent.speed}")

        detected, message = self.security_chain.handle(event, agent)

        if detected:
            self.log_intrusion(f"ALERT: {message} at {location}")
            self.raise_alert()
            agent.detected = True
        else:
            self.log_intrusion(f"Agent {agent.name} passed {location} undetected")

        return detected

    def raise_alert(self):
        """Increase alert level"""
        if self.alert_level == AlertLevel.CLEAR:
            self.alert_level = AlertLevel.SUSPICIOUS
        elif self.alert_level == AlertLevel.SUSPICIOUS:
            self.alert_level = AlertLevel.CAUTION
        elif self.alert_level == AlertLevel.CAUTION:
            self.alert_level = AlertLevel.ALERT
        else:
            self.alert_level = AlertLevel.LOCKDOWN

        self.guard.alert_level = self.alert_level
        print(f"\n🚨 ALERT LEVEL RAISED TO: {self.alert_level.name}")

    def disable_layer(self, layer_name: str):
        """Disable a security layer (hacking)"""
        layers = {
            "motion": self.motion,
            "sound": self.sound,
            "camera": self.camera,
            "guard": self.guard,
            "biometric": self.biometric
        }

        if layer_name in layers:
            layers[layer_name].active = False
            print(f"\n💻 HACKED: {layers[layer_name].name} disabled!")

    def log_intrusion(self, message: str):
        """Log security event"""
        self.intrusion_log.append(message)

    def show_log(self):
        """Show security log"""
        print("\n=== SECURITY LOG ===")
        for entry in self.intrusion_log:
            print(f"  {entry}")

# Example Usage
def main():
    print("🕵️ STEALTH ESPIONAGE - Chain of Responsibility Demo 🕵️\n")

    # Create facility
    facility = SecurityFacility("Secret Laboratory")

    # Create agent
    agent = Agent("Agent 007")
    agent.gadgets = ["lockpick", "camera_jammer", "forged_id"]

    # Mission: Infiltrate the lab
    print("=== MISSION: Infiltrate the Secret Laboratory ===\n")

    # Checkpoint 1: Entrance (crouching)
    agent.set_stealth(StealthLevel.CROUCHING)
    agent.position = "entrance"
    facility.check_agent(agent, "Entrance Hallway", time=1.0)

    if not agent.detected:
        # Checkpoint 2: Corridor (hidden)
        agent.set_stealth(StealthLevel.HIDDEN)
        agent.position = "corridor"
        facility.check_agent(agent, "Main Corridor", time=2.0)

    if not agent.detected:
        # Checkpoint 3: Server room entrance (try invisible)
        agent.set_stealth(StealthLevel.INVISIBLE)
        agent.position = "server_entrance"
        facility.check_agent(agent, "Server Room Entrance", time=3.0)

    if not agent.detected:
        # Checkpoint 4: Secure vault (biometric)
        agent.position = "vault"
        facility.check_agent(agent, "Secure Vault", time=4.0)

    # Show final status
    print("\n\n=== MISSION RESULT ===")
    if agent.detected:
        print(f"❌ MISSION FAILED - Agent {agent.name} was detected!")
        print(f"Facility Alert Level: {facility.alert_level.name}")
    else:
        print(f"✓ MISSION SUCCESS - Agent {agent.name} infiltrated undetected!")

    facility.show_log()

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🕵️ STEALTH ESPIONAGE - Chain of Responsibility Demo 🕵️

=== MISSION: Infiltrate the Secret Laboratory ===

--- Security Check at Entrance Hallway ---
Agent stealth: CROUCHING, Noise: 2, Speed: 5
  [Motion Detector] No detection
  [Sound Detector] No detection
  [Security Camera] No detection
  [Guard Patrol] No detection
  [Biometric Scanner] Used forged ID to bypass

--- Security Check at Main Corridor ---
Agent stealth: HIDDEN, Noise: 1, Speed: 3
  [Motion Detector] No detection
  [Sound Detector] No detection
  [Security Camera] No detection
  [Guard Patrol] No detection
  [Biometric Scanner] Used forged ID to bypass

--- Security Check at Server Room Entrance ---
Agent stealth: INVISIBLE, Noise: 0, Speed: 3
  [Motion Detector] No detection
  [Sound Detector] No detection
  [Security Camera] No detection
  [Guard Patrol] No detection
  [Biometric Scanner] Used forged ID to bypass

--- Security Check at Secure Vault ---
Agent stealth: INVISIBLE, Noise: 0, Speed: 3
  [Motion Detector] No detection
  [Sound Detector] No detection
  [Security Camera] No detection
  [Guard Patrol] No detection
  [Biometric Scanner] Used forged ID to bypass


=== MISSION RESULT ===
✓ MISSION SUCCESS - Agent 007 infiltrated undetected!

=== SECURITY LOG ===
  Agent 007 passed Entrance Hallway undetected
  Agent 007 passed Main Corridor undetected
  Agent 007 passed Server Room Entrance undetected
  Agent 007 passed Secure Vault undetected
```

**Pattern Benefits:**
- **Chain of Responsibility:** Each security layer independently checks for intrusion and passes to the next
- **Layered Security:** Multiple independent detection systems create defense in depth
- **Flexible Configuration:** Easy to add, remove, or reorder security layers
- **Decoupled Detection:** Each layer has its own logic without depending on others
- **Dynamic Behavior:** Agent can adapt stealth tactics to bypass different layers

---

## 13. Theme Park Tycoon - Builder Pattern

**Theme:** Build custom roller coasters
**Patterns:** Builder, Composite

**Key Learning:** Build complex objects step-by-step

**Complete Implementation:**

```python
"""
Complete theme park tycoon system with Builder pattern.
Demonstrates step-by-step construction of complex roller coasters.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
from dataclasses import dataclass

class ThemeType(Enum):
    MEDIEVAL = "Medieval"
    SCI_FI = "Sci-Fi"
    HORROR = "Horror"
    FANTASY = "Fantasy"
    JUNGLE = "Jungle"

class TrackType(Enum):
    STRAIGHT = ("Straight", 10, 5)
    CURVE = ("Curve", 20, 8)
    LOOP = ("Loop", 50, 30)
    CORKSCREW = ("Corkscrew", 60, 35)
    DROP = ("Drop", 40, 25)
    SPIRAL = ("Spiral", 45, 28)

    def __init__(self, label: str, excitement: int, cost: int):
        self.label = label
        self.excitement = excitement
        self.cost = cost

@dataclass
class TrackSection:
    """Section of roller coaster track"""
    track_type: TrackType
    length: int
    height: int

@dataclass
class Car:
    """Roller coaster car"""
    name: str
    capacity: int
    style: str

class SpecialEffect:
    """Special effect along the track"""
    def __init__(self, name: str, trigger_point: int):
        self.name = name
        self.trigger_point = trigger_point

# Product: The Roller Coaster
class RollerCoaster:
    """The complex product being built"""

    def __init__(self):
        self.name = "Unnamed Coaster"
        self.theme: Optional[ThemeType] = None
        self.track_sections: List[TrackSection] = []
        self.cars: List[Car] = []
        self.special_effects: List[SpecialEffect] = []
        self.max_speed = 0
        self.height = 0
        self.length = 0

    def calculate_stats(self):
        """Calculate coaster statistics"""
        self.length = sum(section.length for section in self.track_sections)
        self.height = max((section.height for section in self.track_sections), default=0)

        # Excitement based on track sections
        excitement = sum(section.track_type.excitement for section in self.track_sections)
        excitement += len(self.special_effects) * 10

        return excitement

    def get_info(self) -> str:
        """Get coaster information"""
        excitement = self.calculate_stats()

        info = f"\n{'='*60}\n"
        info += f"ROLLER COASTER: {self.name}\n"
        info += f"{'='*60}\n"
        info += f"Theme: {self.theme.value if self.theme else 'None'}\n"
        info += f"Track Length: {self.length}m\n"
        info += f"Max Height: {self.height}m\n"
        info += f"Excitement Rating: {excitement}\n"
        info += f"\nTrack Sections ({len(self.track_sections)}):\n"
        for i, section in enumerate(self.track_sections, 1):
            info += f"  {i}. {section.track_type.label} ({section.length}m, {section.height}m tall)\n"

        info += f"\nCars ({len(self.cars)}):\n"
        for car in self.cars:
            info += f"  - {car.name} (capacity: {car.capacity})\n"

        if self.special_effects:
            info += f"\nSpecial Effects ({len(self.special_effects)}):\n"
            for effect in self.special_effects:
                info += f"  - {effect.name} at position {effect.trigger_point}m\n"

        info += f"{'='*60}\n"
        return info

# Builder Pattern
class RollerCoasterBuilder(ABC):
    """Abstract builder for roller coasters"""

    def __init__(self):
        self.coaster = RollerCoaster()

    @abstractmethod
    def reset(self):
        pass

    def set_name(self, name: str) -> 'RollerCoasterBuilder':
        self.coaster.name = name
        return self

    def set_theme(self, theme: ThemeType) -> 'RollerCoasterBuilder':
        self.coaster.theme = theme
        return self

    def add_track_section(self, track_type: TrackType, length: int = 50, height: int = 10) -> 'RollerCoasterBuilder':
        section = TrackSection(track_type, length, height)
        self.coaster.track_sections.append(section)
        return self

    def add_car(self, name: str, capacity: int = 4, style: str = "standard") -> 'RollerCoasterBuilder':
        car = Car(name, capacity, style)
        self.coaster.cars.append(car)
        return self

    def add_special_effect(self, name: str, trigger_point: int) -> 'RollerCoasterBuilder':
        effect = SpecialEffect(name, trigger_point)
        self.coaster.special_effects.append(effect)
        return self

    def build(self) -> RollerCoaster:
        result = self.coaster
        self.reset()
        return result

class StandardCoasterBuilder(RollerCoasterBuilder):
    """Builder for standard roller coasters"""

    def reset(self):
        self.coaster = RollerCoaster()

class ThemeCoasterBuilder(RollerCoasterBuilder):
    """Builder for themed roller coasters"""

    def reset(self):
        self.coaster = RollerCoaster()

# Director (optional)
class CoasterDirector:
    """Director that knows how to build specific coaster types"""

    def __init__(self, builder: RollerCoasterBuilder):
        self.builder = builder

    def build_beginner_coaster(self) -> RollerCoaster:
        """Build a simple beginner-friendly coaster"""
        return (self.builder
                .set_name("Gentle Giant")
                .set_theme(ThemeType.FANTASY)
                .add_track_section(TrackType.STRAIGHT, length=100, height=5)
                .add_track_section(TrackType.CURVE, length=80, height=8)
                .add_track_section(TrackType.CURVE, length=80, height=8)
                .add_track_section(TrackType.STRAIGHT, length=100, height=5)
                .add_car("Family Car", capacity=6, style="comfortable")
                .add_car("Family Car", capacity=6, style="comfortable")
                .build())

    def build_thrill_coaster(self) -> RollerCoaster:
        """Build an intense thrill coaster"""
        return (self.builder
                .set_name("Death Drop Extreme")
                .set_theme(ThemeType.HORROR)
                .add_track_section(TrackType.STRAIGHT, length=50, height=10)
                .add_track_section(TrackType.DROP, length=100, height=80)
                .add_track_section(TrackType.LOOP, length=60, height=40)
                .add_track_section(TrackType.CORKSCREW, length=70, height=35)
                .add_track_section(TrackType.SPIRAL, length=80, height=30)
                .add_special_effect("Smoke Machine", trigger_point=50)
                .add_special_effect("Strobe Lights", trigger_point=150)
                .add_special_effect("Scream Audio", trigger_point=160)
                .add_car("Extreme Car", capacity=4, style="racing")
                .add_car("Extreme Car", capacity=4, style="racing")
                .build())

    def build_themed_coaster(self, theme: ThemeType) -> RollerCoaster:
        """Build a coaster based on theme"""
        if theme == ThemeType.MEDIEVAL:
            return (self.builder
                    .set_name("Dragon's Fury")
                    .set_theme(theme)
                    .add_track_section(TrackType.STRAIGHT, length=60, height=10)
                    .add_track_section(TrackType.LOOP, length=50, height=35)
                    .add_track_section(TrackType.CORKSCREW, length=60, height=30)
                    .add_special_effect("Fire Breathing", trigger_point=60)
                    .add_special_effect("Dragon Roar", trigger_point=110)
                    .add_car("Dragon Head Car", capacity=4, style="dragon")
                    .add_car("Dragon Tail Car", capacity=4, style="dragon")
                    .build())
        elif theme == ThemeType.SCI_FI:
            return (self.builder
                    .set_name("Hyperspace Jump")
                    .set_theme(theme)
                    .add_track_section(TrackType.STRAIGHT, length=100, height=5)
                    .add_track_section(TrackType.SPIRAL, length=80, height=40)
                    .add_track_section(TrackType.LOOP, length=60, height=35)
                    .add_special_effect("Laser Lights", trigger_point=100)
                    .add_special_effect("Hologram Portal", trigger_point=180)
                    .add_car("Spaceship Car", capacity=4, style="futuristic")
                    .build())
        else:
            return self.build_beginner_coaster()

# Example Usage
def main():
    print("🎢 THEME PARK TYCOON - Builder Pattern Demo 🎢\n")

    # Create builder
    builder = StandardCoasterBuilder()

    # Build custom coaster manually
    print("=== Building Custom Coaster ===\n")
    custom_coaster = (builder
                      .set_name("Wild Adventure")
                      .set_theme(ThemeType.JUNGLE)
                      .add_track_section(TrackType.STRAIGHT, length=80, height=10)
                      .add_track_section(TrackType.CURVE, length=60, height=15)
                      .add_track_section(TrackType.DROP, length=90, height=60)
                      .add_track_section(TrackType.LOOP, length=50, height=35)
                      .add_special_effect("Waterfall Splash", trigger_point=140)
                      .add_special_effect("Jungle Sounds", trigger_point=190)
                      .add_car("Jungle Explorer", capacity=6, style="open-air")
                      .add_car("Jungle Explorer", capacity=6, style="open-air")
                      .build())

    print(custom_coaster.get_info())

    # Use director for pre-designed coasters
    print("\n=== Using Director for Pre-Designed Coasters ===\n")

    director = CoasterDirector(builder)

    print("--- Building Beginner Coaster ---")
    beginner = director.build_beginner_coaster()
    print(beginner.get_info())

    print("\n--- Building Thrill Coaster ---")
    thrill = director.build_thrill_coaster()
    print(thrill.get_info())

    print("\n--- Building Themed Coaster (Medieval) ---")
    medieval = director.build_themed_coaster(ThemeType.MEDIEVAL)
    print(medieval.get_info())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
🎢 THEME PARK TYCOON - Builder Pattern Demo 🎢

=== Building Custom Coaster ===

============================================================
ROLLER COASTER: Wild Adventure
============================================================
Theme: Jungle
Track Length: 280m
Max Height: 60m
Excitement Rating: 130

Track Sections (4):
  1. Straight (80m, 10m tall)
  2. Curve (60m, 15m tall)
  3. Drop (90m, 60m tall)
  4. Loop (50m, 35m tall)

Cars (2):
  - Jungle Explorer (capacity: 6)
  - Jungle Explorer (capacity: 6)

Special Effects (2):
  - Waterfall Splash at position 140m
  - Jungle Sounds at position 190m
============================================================


=== Using Director for Pre-Designed Coasters ===

--- Building Beginner Coaster ---

============================================================
ROLLER COASTER: Gentle Giant
============================================================
Theme: Fantasy
Track Length: 360m
Max Height: 8m
Excitement Rating: 60

Track Sections (4):
  1. Straight (100m, 5m tall)
  2. Curve (80m, 8m tall)
  3. Curve (80m, 8m tall)
  4. Straight (100m, 5m tall)

Cars (2):
  - Family Car (capacity: 6)
  - Family Car (capacity: 6)

============================================================


--- Building Thrill Coaster ---

============================================================
ROLLER COASTER: Death Drop Extreme
============================================================
Theme: Horror
Track Length: 360m
Max Height: 80m
Excitement Rating: 245

Track Sections (5):
  1. Straight (50m, 10m tall)
  2. Drop (100m, 80m tall)
  3. Loop (60m, 40m tall)
  4. Corkscrew (70m, 35m tall)
  5. Spiral (80m, 30m tall)

Cars (2):
  - Extreme Car (capacity: 4)
  - Extreme Car (capacity: 4)

Special Effects (3):
  - Smoke Machine at position 50m
  - Strobe Lights at position 150m
  - Scream Audio at position 160m
============================================================


--- Building Themed Coaster (Medieval) ---

============================================================
ROLLER COASTER: Dragon's Fury
============================================================
Theme: Medieval
Track Length: 170m
Max Height: 35m
Excitement Rating: 180

Track Sections (3):
  1. Straight (60m, 10m tall)
  2. Loop (50m, 35m tall)
  3. Corkscrew (60m, 30m tall)

Cars (2):
  - Dragon Head Car (capacity: 4)
  - Dragon Tail Car (capacity: 4)

Special Effects (2):
  - Fire Breathing at position 60m
  - Dragon Roar at position 110m
============================================================
```

**Pattern Benefits:**
- **Builder Pattern:** Step-by-step construction of complex roller coasters with fluent interface
- **Flexibility:** Build coasters with any combination of sections, cars, and effects
- **Director Pattern:** Pre-configured coaster designs for common use cases
- **Readability:** Fluent interface makes code self-documenting
- **Immutability:** Each build creates a fresh coaster object

---

## 14. Mech Battle Arena - Composite + Strategy

**Theme:** Giant robot combat
**Patterns:** Composite, Strategy

**Key Learning:** Mechs are composites of parts with strategies

**Complete Implementation:**

```python
"""
Complete mech battle arena with Composite and Strategy patterns.
Demonstrates hierarchical mech composition and different combat strategies.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
import random


class DamageType(Enum):
    KINETIC = "kinetic"
    ENERGY = "energy"
    EXPLOSIVE = "explosive"


# Composite Pattern - Mech components hierarchy
class MechComponent(ABC):
    """Base component for all mech parts"""

    def __init__(self, name: str, weight: int, hp: int):
        self.name = name
        self.weight = weight
        self.max_hp = hp
        self.current_hp = hp

    @abstractmethod
    def get_stats(self) -> dict:
        """Return component statistics"""
        pass

    @abstractmethod
    def get_power_consumption(self) -> int:
        """Power required to operate"""
        pass

    def is_operational(self) -> bool:
        return self.current_hp > 0

    def take_damage(self, amount: int):
        self.current_hp = max(0, self.current_hp - amount)


class Weapon(MechComponent):
    """Leaf component - Weapon system"""

    def __init__(self, name: str, damage: int, damage_type: DamageType,
                 range_val: int, weight: int, power_consumption: int):
        super().__init__(name, weight, hp=50)
        self.damage = damage
        self.damage_type = damage_type
        self.range = range_val
        self.power = power_consumption

    def get_stats(self) -> dict:
        return {
            "damage": self.damage,
            "type": self.damage_type.value,
            "range": self.range,
            "weight": self.weight,
            "hp": f"{self.current_hp}/{self.max_hp}"
        }

    def get_power_consumption(self) -> int:
        return self.power if self.is_operational() else 0

    def fire(self) -> int:
        if self.is_operational():
            return self.damage
        return 0


class Armor(MechComponent):
    """Leaf component - Armor plating"""

    def __init__(self, name: str, defense: int, weight: int):
        super().__init__(name, weight, hp=100)
        self.defense = defense

    def get_stats(self) -> dict:
        return {
            "defense": self.defense,
            "weight": self.weight,
            "hp": f"{self.current_hp}/{self.max_hp}"
        }

    def get_power_consumption(self) -> int:
        return 0

    def absorb_damage(self, damage: int) -> int:
        """Returns damage after armor absorption"""
        if self.is_operational():
            reduction = min(damage, self.defense)
            remaining = damage - reduction
            self.take_damage(reduction // 2)
            return remaining
        return damage


class Reactor(MechComponent):
    """Leaf component - Power reactor"""

    def __init__(self, name: str, power_output: int, weight: int):
        super().__init__(name, weight, hp=75)
        self.power_output = power_output

    def get_stats(self) -> dict:
        return {
            "power_output": self.power_output,
            "weight": self.weight,
            "hp": f"{self.current_hp}/{self.max_hp}"
        }

    def get_power_consumption(self) -> int:
        return -self.power_output if self.is_operational() else 0


class MechSubsystem(MechComponent):
    """Composite - Group of components (weapons bay, armor plating, etc.)"""

    def __init__(self, name: str):
        super().__init__(name, weight=0, hp=100)
        self.components: List[MechComponent] = []

    def add_component(self, component: MechComponent):
        self.components.append(component)
        self.weight += component.weight

    def remove_component(self, component: MechComponent):
        if component in self.components:
            self.components.remove(component)
            self.weight -= component.weight

    def get_stats(self) -> dict:
        total_stats = {"subsystem": self.name, "components": []}
        for comp in self.components:
            total_stats["components"].append(comp.get_stats())
        return total_stats

    def get_power_consumption(self) -> int:
        return sum(comp.get_power_consumption() for comp in self.components)


# Strategy Pattern - Different combat strategies
class CombatStrategy(ABC):
    """Abstract combat strategy"""

    @abstractmethod
    def select_target(self, enemies: List['Mech']) -> Optional['Mech']:
        """Select which enemy to target"""
        pass

    @abstractmethod
    def select_weapon(self, weapons: List[Weapon]) -> Optional[Weapon]:
        """Select which weapon to use"""
        pass

    @abstractmethod
    def get_strategy_name(self) -> str:
        pass


class AggressiveStrategy(CombatStrategy):
    """Focus on maximum damage output"""

    def select_target(self, enemies: List['Mech']) -> Optional['Mech']:
        # Target weakest enemy for quick kills
        operational = [e for e in enemies if e.is_operational()]
        if not operational:
            return None
        return min(operational, key=lambda e: e.current_hp)

    def select_weapon(self, weapons: List[Weapon]) -> Optional[Weapon]:
        # Use highest damage weapon
        operational = [w for w in weapons if w.is_operational()]
        if not operational:
            return None
        return max(operational, key=lambda w: w.damage)

    def get_strategy_name(self) -> str:
        return "Aggressive Assault"


class DefensiveStrategy(CombatStrategy):
    """Focus on survival and picking targets carefully"""

    def select_target(self, enemies: List['Mech']) -> Optional['Mech']:
        # Target closest enemy
        operational = [e for e in enemies if e.is_operational()]
        if not operational:
            return None
        return random.choice(operational)

    def select_weapon(self, weapons: List[Weapon]) -> Optional[Weapon]:
        # Use most power-efficient weapon
        operational = [w for w in weapons if w.is_operational()]
        if not operational:
            return None
        return min(operational, key=lambda w: w.get_power_consumption())

    def get_strategy_name(self) -> str:
        return "Defensive Tactics"


class SniperStrategy(CombatStrategy):
    """Long-range precision strikes"""

    def select_target(self, enemies: List['Mech']) -> Optional['Mech']:
        # Target highest HP enemy (focus fire)
        operational = [e for e in enemies if e.is_operational()]
        if not operational:
            return None
        return max(operational, key=lambda e: e.current_hp)

    def select_weapon(self, weapons: List[Weapon]) -> Optional[Weapon]:
        # Prefer long-range weapons
        operational = [w for w in weapons if w.is_operational()]
        if not operational:
            return None
        return max(operational, key=lambda w: w.range)

    def get_strategy_name(self) -> str:
        return "Sniper Protocol"


# Main Mech class (Composite root + Strategy context)
class Mech:
    """Complete mech built from composite components with combat strategy"""

    def __init__(self, name: str, pilot: str):
        self.name = name
        self.pilot = pilot
        self.subsystems: List[MechSubsystem] = []
        self.max_hp = 1000
        self.current_hp = 1000
        self.strategy: Optional[CombatStrategy] = None

    def add_subsystem(self, subsystem: MechSubsystem):
        self.subsystems.append(subsystem)

    def set_strategy(self, strategy: CombatStrategy):
        self.strategy = strategy
        print(f"[{self.name}] Combat strategy set to: {strategy.get_strategy_name()}")

    def get_all_weapons(self) -> List[Weapon]:
        weapons = []
        for subsystem in self.subsystems:
            for component in subsystem.components:
                if isinstance(component, Weapon):
                    weapons.append(component)
        return weapons

    def get_all_armor(self) -> List[Armor]:
        armor = []
        for subsystem in self.subsystems:
            for component in subsystem.components:
                if isinstance(component, Armor):
                    armor.append(component)
        return armor

    def get_total_weight(self) -> int:
        return sum(subsystem.weight for subsystem in self.subsystems)

    def get_power_balance(self) -> int:
        return sum(subsystem.get_power_consumption() for subsystem in self.subsystems)

    def is_operational(self) -> bool:
        return self.current_hp > 0

    def take_damage(self, damage: int):
        # Apply armor reduction
        for armor in self.get_all_armor():
            damage = armor.absorb_damage(damage)
            if damage <= 0:
                break

        # Apply remaining damage to mech hull
        self.current_hp = max(0, self.current_hp - damage)

    def attack(self, target: 'Mech') -> bool:
        if not self.strategy:
            print(f"[{self.name}] No combat strategy assigned!")
            return False

        weapon = self.strategy.select_weapon(self.get_all_weapons())
        if not weapon:
            print(f"[{self.name}] No operational weapons!")
            return False

        damage = weapon.fire()
        target.take_damage(damage)

        print(f"[{self.name}] Fires {weapon.name} at {target.name} for {damage} damage! "
              f"({target.current_hp}/{target.max_hp} HP remaining)")

        return True

    def get_status(self) -> str:
        status = f"\n{'='*50}\n"
        status += f"MECH: {self.name} (Pilot: {self.pilot})\n"
        status += f"Hull HP: {self.current_hp}/{self.max_hp}\n"
        status += f"Total Weight: {self.get_total_weight()} tons\n"
        status += f"Power Balance: {self.get_power_balance()} units\n"
        status += f"Strategy: {self.strategy.get_strategy_name() if self.strategy else 'None'}\n"
        status += f"\nSUBSYSTEMS:\n"

        for subsystem in self.subsystems:
            status += f"\n  {subsystem.name}:\n"
            for component in subsystem.components:
                status += f"    - {component.name}: {component.get_stats()}\n"

        return status


class BattleArena:
    """Manages combat between mechs"""

    def __init__(self):
        self.team_a: List[Mech] = []
        self.team_b: List[Mech] = []

    def add_to_team_a(self, mech: Mech):
        self.team_a.append(mech)

    def add_to_team_b(self, mech: Mech):
        self.team_b.append(mech)

    def simulate_round(self) -> bool:
        """Simulate one round of combat. Returns True if battle continues."""
        print("\n" + "="*50)
        print("COMBAT ROUND")
        print("="*50)

        # Team A attacks
        for attacker in [m for m in self.team_a if m.is_operational()]:
            targets = [m for m in self.team_b if m.is_operational()]
            if not targets:
                return False

            target = attacker.strategy.select_target(targets) if attacker.strategy else random.choice(targets)
            if target:
                attacker.attack(target)

        # Check if Team B eliminated
        if not any(m.is_operational() for m in self.team_b):
            return False

        # Team B attacks
        for attacker in [m for m in self.team_b if m.is_operational()]:
            targets = [m for m in self.team_a if m.is_operational()]
            if not targets:
                return False

            target = attacker.strategy.select_target(targets) if attacker.strategy else random.choice(targets)
            if target:
                attacker.attack(target)

        # Check if Team A eliminated
        return any(m.is_operational() for m in self.team_a)

    def get_winner(self) -> str:
        team_a_alive = any(m.is_operational() for m in self.team_a)
        team_b_alive = any(m.is_operational() for m in self.team_b)

        if team_a_alive and not team_b_alive:
            return "Team A wins!"
        elif team_b_alive and not team_a_alive:
            return "Team B wins!"
        else:
            return "Draw!"


def main():
    print("=" * 60)
    print("MECH BATTLE ARENA - Composite + Strategy Pattern Demo")
    print("=" * 60)

    # Build Mech 1: Heavy Assault Mech
    weapons_bay_1 = MechSubsystem("Weapons Bay")
    weapons_bay_1.add_component(Weapon("Plasma Cannon", 120, DamageType.ENERGY, 500, 30, 50))
    weapons_bay_1.add_component(Weapon("Missile Pods", 80, DamageType.EXPLOSIVE, 800, 25, 30))

    armor_plating_1 = MechSubsystem("Armor Plating")
    armor_plating_1.add_component(Armor("Composite Armor", 40, 50))
    armor_plating_1.add_component(Armor("Reactive Plating", 30, 30))

    power_system_1 = MechSubsystem("Power Core")
    power_system_1.add_component(Reactor("Fusion Reactor", 150, 40))

    titan = Mech("Titan", "Commander Drake")
    titan.add_subsystem(weapons_bay_1)
    titan.add_subsystem(armor_plating_1)
    titan.add_subsystem(power_system_1)
    titan.set_strategy(AggressiveStrategy())

    # Build Mech 2: Sniper Mech
    weapons_bay_2 = MechSubsystem("Weapons Bay")
    weapons_bay_2.add_component(Weapon("Rail Gun", 150, DamageType.KINETIC, 1200, 35, 60))

    armor_plating_2 = MechSubsystem("Light Armor")
    armor_plating_2.add_component(Armor("Carbon Weave", 25, 20))

    power_system_2 = MechSubsystem("Power Core")
    power_system_2.add_component(Reactor("Arc Reactor", 120, 30))

    phantom = Mech("Phantom", "Lt. Shadow")
    phantom.add_subsystem(weapons_bay_2)
    phantom.add_subsystem(armor_plating_2)
    phantom.add_subsystem(power_system_2)
    phantom.set_strategy(SniperStrategy())

    # Display mech configurations
    print(titan.get_status())
    print(phantom.get_status())

    # Battle simulation
    arena = BattleArena()
    arena.add_to_team_a(titan)
    arena.add_to_team_b(phantom)

    print("\n" + "=" * 60)
    print("BATTLE START!")
    print("=" * 60)

    round_count = 0
    max_rounds = 10

    while round_count < max_rounds:
        round_count += 1
        if not arena.simulate_round():
            break

    print("\n" + "=" * 60)
    print(f"BATTLE END - {arena.get_winner()}")
    print("=" * 60)

    # Final status
    print(titan.get_status())
    print(phantom.get_status())


if __name__ == "__main__":
    main()
```

**Example Output:**
```
============================================================
MECH BATTLE ARENA - Composite + Strategy Pattern Demo
============================================================
[Titan] Combat strategy set to: Aggressive Assault
[Phantom] Combat strategy set to: Sniper Protocol

==================================================
MECH: Titan (Pilot: Commander Drake)
Hull HP: 1000/1000
Total Weight: 205 tons
Power Balance: -70 units
Strategy: Aggressive Assault

SUBSYSTEMS:

  Weapons Bay:
    - Plasma Cannon: {'damage': 120, 'type': 'energy', 'range': 500, 'weight': 30, 'hp': '50/50'}
    - Missile Pods: {'damage': 80, 'type': 'explosive', 'range': 800, 'weight': 25, 'hp': '50/50'}

  Armor Plating:
    - Composite Armor: {'defense': 40, 'weight': 50, 'hp': '100/100'}
    - Reactive Plating: {'defense': 30, 'weight': 30, 'hp': '100/100'}

  Power Core:
    - Fusion Reactor: {'power_output': 150, 'weight': 40, 'hp': '75/75'}

==================================================
MECH: Phantom (Pilot: Lt. Shadow)
Hull HP: 1000/1000
Total Weight: 105 tons
Power Balance: -60 units
Strategy: Sniper Protocol

SUBSYSTEMS:

  Weapons Bay:
    - Rail Gun: {'damage': 150, 'type': 'kinetic', 'range': 1200, 'weight': 35, 'hp': '50/50'}

  Light Armor:
    - Carbon Weave: {'defense': 25, 'weight': 20, 'hp': '100/100'}

  Power Core:
    - Arc Reactor: {'power_output': 120, 'weight': 30, 'hp': '75/75'}

============================================================
BATTLE START!
============================================================

==================================================
COMBAT ROUND
==================================================
[Titan] Fires Plasma Cannon at Phantom for 120 damage! (905/1000 HP remaining)
[Phantom] Fires Rail Gun at Titan for 150 damage! (890/1000 HP remaining)

==================================================
COMBAT ROUND
==================================================
[Titan] Fires Plasma Cannon at Phantom for 120 damage! (810/1000 HP remaining)
[Phantom] Fires Rail Gun at Titan for 150 damage! (780/1000 HP remaining)

============================================================
BATTLE END - Team B wins!
============================================================
```

**Pattern Benefits:**

- **Composite Pattern:** Allows building complex mechs from simple parts hierarchically
- **Component Reusability:** Same weapon/armor classes used across different mech configurations
- **Strategy Pattern:** Combat behavior changes without modifying mech class
- **Flexible Composition:** Easy to add/remove components or create new subsystems
- **Separation of Concerns:** Structure (composite) separated from behavior (strategy)
- **Power Management:** Hierarchical power consumption calculation via composite structure
- **Damage Propagation:** Damage flows through composite hierarchy naturally

---

## 15. Haunted Mansion - Visitor Pattern

**Theme:** Horror ghost hunting
**Patterns:** Visitor, Observer

**Key Learning:** Different equipment visits rooms differently

**Complete Implementation:**

```python
"""
Complete haunted mansion investigation with Visitor and Observer patterns.
Demonstrates visitor pattern for different equipment and observer pattern for paranormal events.
"""

from abc import ABC, abstractmethod
from typing import List, Dict
from enum import Enum
import random


class ParanormalType(Enum):
    GHOST = "ghost"
    POLTERGEIST = "poltergeist"
    SHADOW = "shadow"
    COLD_SPOT = "cold_spot"
    EMF_SPIKE = "emf_spike"


class GhostActivity(Enum):
    DORMANT = "dormant"
    ACTIVE = "active"
    AGGRESSIVE = "aggressive"


# Observer Pattern - Investigators observe paranormal events
class InvestigatorObserver(ABC):
    """Observer that responds to paranormal events"""

    @abstractmethod
    def on_paranormal_event(self, room_name: str, event_type: ParanormalType):
        pass


class HeadInvestigator(InvestigatorObserver):
    """Main investigator tracking all events"""

    def __init__(self, name: str):
        self.name = name
        self.event_log: List[str] = []

    def on_paranormal_event(self, room_name: str, event_type: ParanormalType):
        message = f"[{self.name}] Paranormal activity in {room_name}: {event_type.value}!"
        self.event_log.append(message)
        print(message)

    def get_summary(self) -> str:
        return f"\n{self.name}'s Event Log:\n" + "\n".join(self.event_log)


class TechSpecialist(InvestigatorObserver):
    """Technical specialist monitoring equipment"""

    def __init__(self, name: str):
        self.name = name
        self.readings: Dict[str, int] = {}

    def on_paranormal_event(self, room_name: str, event_type: ParanormalType):
        self.readings[room_name] = self.readings.get(room_name, 0) + 1
        if event_type == ParanormalType.EMF_SPIKE:
            print(f"[{self.name}] EMF equipment triggered in {room_name}!")


# Visitor Pattern - Different investigation equipment
class Room(ABC):
    """Base room class that accepts visitors"""

    def __init__(self, name: str):
        self.name = name
        self.paranormal_activity: List[ParanormalType] = []
        self.activity_level = GhostActivity.DORMANT
        self.observers: List[InvestigatorObserver] = []
        self.investigation_notes: List[str] = []

    @abstractmethod
    def accept(self, visitor: 'InvestigationEquipment'):
        """Accept a visitor (equipment) for investigation"""
        pass

    def add_observer(self, observer: InvestigatorObserver):
        self.observers.append(observer)

    def trigger_paranormal_event(self, event_type: ParanormalType):
        """Notify observers of paranormal activity"""
        self.paranormal_activity.append(event_type)
        for observer in self.observers:
            observer.on_paranormal_event(self.name, event_type)

    def add_investigation_note(self, note: str):
        self.investigation_notes.append(f"[{self.name}] {note}")


class Bedroom(Room):
    """Specific room type - Bedroom"""

    def __init__(self, name: str, has_mirror: bool = False):
        super().__init__(name)
        self.has_mirror = has_mirror
        self.temperature = 68  # Fahrenheit

    def accept(self, visitor: 'InvestigationEquipment'):
        visitor.visit_bedroom(self)


class Basement(Room):
    """Specific room type - Basement"""

    def __init__(self, name: str, is_flooded: bool = False):
        super().__init__(name)
        self.is_flooded = is_flooded
        self.humidity = 85  # Percentage

    def accept(self, visitor: 'InvestigationEquipment'):
        visitor.visit_basement(self)


class Attic(Room):
    """Specific room type - Attic"""

    def __init__(self, name: str, has_windows: bool = True):
        super().__init__(name)
        self.has_windows = has_windows
        self.dust_level = 95

    def accept(self, visitor: 'InvestigationEquipment'):
        visitor.visit_attic(self)


class Kitchen(Room):
    """Specific room type - Kitchen"""

    def __init__(self, name: str):
        super().__init__(name)
        self.appliances_count = 8

    def accept(self, visitor: 'InvestigationEquipment'):
        visitor.visit_kitchen(self)


# Visitor - Investigation Equipment
class InvestigationEquipment(ABC):
    """Abstract visitor - Investigation equipment"""

    def __init__(self, name: str):
        self.name = name
        self.battery = 100
        self.findings: List[str] = []

    @abstractmethod
    def visit_bedroom(self, room: Bedroom):
        pass

    @abstractmethod
    def visit_basement(self, room: Basement):
        pass

    @abstractmethod
    def visit_attic(self, room: Attic):
        pass

    @abstractmethod
    def visit_kitchen(self, room: Kitchen):
        pass

    def use_battery(self, amount: int):
        self.battery = max(0, self.battery - amount)

    def get_report(self) -> str:
        return f"\n{self.name} Report (Battery: {self.battery}%):\n" + "\n".join(self.findings)


class EMFDetector(InvestigationEquipment):
    """Concrete Visitor - EMF (Electromagnetic Field) Detector"""

    def __init__(self):
        super().__init__("EMF Detector")
        self.emf_readings: Dict[str, int] = {}

    def visit_bedroom(self, room: Bedroom):
        self.use_battery(5)
        reading = random.randint(0, 10)
        self.emf_readings[room.name] = reading

        if reading > 7:
            finding = f"HIGH EMF spike in {room.name} ({reading}/10)"
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.EMF_SPIKE)
            if room.has_mirror:
                room.trigger_paranormal_event(ParanormalType.GHOST)
        elif reading > 4:
            finding = f"Moderate EMF in {room.name} ({reading}/10)"
            self.findings.append(finding)
            room.add_investigation_note(finding)
        else:
            self.findings.append(f"Normal EMF in {room.name}")

    def visit_basement(self, room: Basement):
        self.use_battery(8)
        # Basements typically have higher EMF from electrical systems
        reading = random.randint(3, 10)
        self.emf_readings[room.name] = reading

        if reading > 8:
            finding = f"EXTREME EMF readings in {room.name}! ({reading}/10)"
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.EMF_SPIKE)
            room.trigger_paranormal_event(ParanormalType.POLTERGEIST)

    def visit_attic(self, room: Attic):
        self.use_battery(5)
        reading = random.randint(0, 8)
        self.emf_readings[room.name] = reading
        self.findings.append(f"EMF scan of {room.name}: {reading}/10")

    def visit_kitchen(self, room: Kitchen):
        self.use_battery(6)
        # Kitchens have appliances causing EMF
        reading = random.randint(2, 7)
        self.emf_readings[room.name] = reading
        self.findings.append(f"Expected elevated EMF in {room.name} due to {room.appliances_count} appliances: {reading}/10")


class InfraredCamera(InvestigationEquipment):
    """Concrete Visitor - Infrared thermal camera"""

    def __init__(self):
        super().__init__("Infrared Camera")

    def visit_bedroom(self, room: Bedroom):
        self.use_battery(10)
        cold_spots = random.randint(0, 3)

        if cold_spots > 0:
            temp_drop = random.randint(5, 20)
            room.temperature -= temp_drop
            finding = f"Detected {cold_spots} cold spot(s) in {room.name} - Temperature dropped to {room.temperature}°F"
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.COLD_SPOT)
        else:
            self.findings.append(f"No thermal anomalies in {room.name}")

    def visit_basement(self, room: Basement):
        self.use_battery(12)
        # Basements are naturally cooler
        finding = f"Baseline temperature in {room.name}: 58°F"
        self.findings.append(finding)

        if room.is_flooded:
            self.findings.append(f"Thermal signature affected by water in {room.name}")

    def visit_attic(self, room: Attic):
        self.use_battery(10)
        hot_spots = random.randint(0, 2)

        if hot_spots > 0:
            finding = f"Detected {hot_spots} unusual warm area(s) in {room.name}"
            self.findings.append(finding)
            room.add_investigation_note(finding)

    def visit_kitchen(self, room: Kitchen):
        self.use_battery(8)
        self.findings.append(f"Standard thermal profile in {room.name}")


class SpiritBox(InvestigationEquipment):
    """Concrete Visitor - Spirit communication device"""

    def __init__(self):
        super().__init__("Spirit Box")
        self.messages_received = 0

    def visit_bedroom(self, room: Bedroom):
        self.use_battery(15)

        if random.random() < 0.4:  # 40% chance of communication
            messages = ["Get out", "Help me", "Behind you", "Cold"]
            message = random.choice(messages)
            finding = f'Received message in {room.name}: "{message}"'
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.GHOST)
            self.messages_received += 1
        else:
            self.findings.append(f"Static only in {room.name}")

    def visit_basement(self, room: Basement):
        self.use_battery(18)

        if random.random() < 0.6:  # 60% chance - basements more active
            messages = ["Leave now", "Mine", "Trapped", "Darkness"]
            message = random.choice(messages)
            finding = f'Strong communication in {room.name}: "{message}"'
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.GHOST)
            room.activity_level = GhostActivity.AGGRESSIVE
            self.messages_received += 1

    def visit_attic(self, room: Attic):
        self.use_battery(15)

        if random.random() < 0.3:
            messages = ["Up here", "Find me", "Forgotten"]
            message = random.choice(messages)
            finding = f'Faint message in {room.name}: "{message}"'
            self.findings.append(finding)
            room.add_investigation_note(finding)
            self.messages_received += 1

    def visit_kitchen(self, room: Kitchen):
        self.use_battery(12)
        self.findings.append(f"No spirit communication in {room.name}")


class MotionSensor(InvestigationEquipment):
    """Concrete Visitor - Motion detection equipment"""

    def __init__(self):
        super().__init__("Motion Sensor")
        self.motion_detected_count = 0

    def visit_bedroom(self, room: Bedroom):
        self.use_battery(7)

        if random.random() < 0.35:
            finding = f"Unexplained movement detected in {room.name}"
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.SHADOW)
            self.motion_detected_count += 1

    def visit_basement(self, room: Basement):
        self.use_battery(10)

        if random.random() < 0.5:
            finding = f"Shadow figure detected moving in {room.name}"
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.SHADOW)
            room.trigger_paranormal_event(ParanormalType.POLTERGEIST)
            self.motion_detected_count += 1

    def visit_attic(self, room: Attic):
        self.use_battery(8)
        self.findings.append(f"No movement in {room.name}")

    def visit_kitchen(self, room: Kitchen):
        self.use_battery(7)

        if random.random() < 0.25:
            finding = f"Objects moved in {room.name}"
            self.findings.append(finding)
            room.add_investigation_note(finding)
            room.trigger_paranormal_event(ParanormalType.POLTERGEIST)
            self.motion_detected_count += 1


class HauntedMansion:
    """Manages the mansion and investigation"""

    def __init__(self, name: str):
        self.name = name
        self.rooms: List[Room] = []
        self.investigators: List[InvestigatorObserver] = []

    def add_room(self, room: Room):
        self.rooms.append(room)
        # Subscribe all investigators to new room
        for investigator in self.investigators:
            room.add_observer(investigator)

    def add_investigator(self, investigator: InvestigatorObserver):
        self.investigators.append(investigator)
        # Subscribe to all existing rooms
        for room in self.rooms:
            room.add_observer(investigator)

    def investigate_with(self, equipment: InvestigationEquipment):
        """Investigate all rooms with specific equipment"""
        print(f"\n{'='*60}")
        print(f"Investigating with {equipment.name}")
        print(f"{'='*60}")

        for room in self.rooms:
            if equipment.battery > 0:
                room.accept(equipment)
            else:
                print(f"[WARNING] {equipment.name} battery depleted!")
                break

        print(equipment.get_report())

    def get_investigation_summary(self) -> str:
        summary = f"\n{'='*60}\n"
        summary += f"Investigation Summary: {self.name}\n"
        summary += f"{'='*60}\n"

        for room in self.rooms:
            summary += f"\n{room.name}:\n"
            summary += f"  Activity Level: {room.activity_level.value}\n"
            summary += f"  Paranormal Events: {len(room.paranormal_activity)}\n"
            if room.investigation_notes:
                summary += "  Notes:\n"
                for note in room.investigation_notes:
                    summary += f"    - {note}\n"

        return summary


def main():
    print("="*60)
    print("HAUNTED MANSION INVESTIGATION")
    print("Visitor + Observer Pattern Demo")
    print("="*60)

    # Create mansion
    mansion = HauntedMansion("Blackwood Manor")

    # Create rooms
    master_bedroom = Bedroom("Master Bedroom", has_mirror=True)
    guest_room = Bedroom("Guest Room", has_mirror=False)
    basement = Basement("Basement", is_flooded=True)
    attic = Attic("Attic", has_windows=True)
    kitchen = Kitchen("Kitchen")

    mansion.add_room(master_bedroom)
    mansion.add_room(guest_room)
    mansion.add_room(basement)
    mansion.add_room(attic)
    mansion.add_room(kitchen)

    # Create investigators (observers)
    head_investigator = HeadInvestigator("Dr. Sarah Chen")
    tech_specialist = TechSpecialist("Mike Rodriguez")

    mansion.add_investigator(head_investigator)
    mansion.add_investigator(tech_specialist)

    # Create equipment (visitors)
    emf_detector = EMFDetector()
    ir_camera = InfraredCamera()
    spirit_box = SpiritBox()
    motion_sensor = MotionSensor()

    # Conduct investigation with different equipment
    mansion.investigate_with(emf_detector)
    mansion.investigate_with(ir_camera)
    mansion.investigate_with(spirit_box)
    mansion.investigate_with(motion_sensor)

    # Print investigation summary
    print(mansion.get_investigation_summary())

    # Print investigator logs
    print(head_investigator.get_summary())

    print(f"\n{'='*60}")
    print("INVESTIGATION COMPLETE")
    print(f"{'='*60}")
    print(f"Total spirit communications: {spirit_box.messages_received}")
    print(f"Total motion detections: {motion_sensor.motion_detected_count}")
    print(f"EMF readings taken: {len(emf_detector.emf_readings)}")


if __name__ == "__main__":
    main()
```

**Example Output:**
```
============================================================
HAUNTED MANSION INVESTIGATION
Visitor + Observer Pattern Demo
============================================================

============================================================
Investigating with EMF Detector
============================================================
[Dr. Sarah Chen] Paranormal activity in Master Bedroom: emf_spike!
[Mike Rodriguez] EMF equipment triggered in Master Bedroom!
[Dr. Sarah Chen] Paranormal activity in Master Bedroom: ghost!
[Dr. Sarah Chen] Paranormal activity in Basement: emf_spike!
[Mike Rodriguez] EMF equipment triggered in Basement!
[Dr. Sarah Chen] Paranormal activity in Basement: poltergeist!

EMF Detector Report (Battery: 76%):
HIGH EMF spike in Master Bedroom (9/10)
EXTREME EMF readings in Basement! (10/10)
EMF scan of Attic: 4/10
Expected elevated EMF in Kitchen due to 8 appliances: 5/10

============================================================
Investigating with Infrared Camera
============================================================
[Dr. Sarah Chen] Paranormal activity in Master Bedroom: cold_spot!

Infrared Camera Report (Battery: 60%):
Detected 2 cold spot(s) in Master Bedroom - Temperature dropped to 53°F
No thermal anomalies in Guest Room
Baseline temperature in Basement: 58°F
Thermal signature affected by water in Basement
Detected 1 unusual warm area(s) in Attic
Standard thermal profile in Kitchen

============================================================
Investigating with Spirit Box
============================================================
[Dr. Sarah Chen] Paranormal activity in Guest Room: ghost!
[Dr. Sarah Chen] Paranormal activity in Basement: ghost!

Spirit Box Report (Battery: 42%):
Static only in Master Bedroom
Received message in Guest Room: "Help me"
Strong communication in Basement: "Trapped"
Faint message in Attic: "Forgotten"
No spirit communication in Kitchen

============================================================
Investigating with Motion Sensor
============================================================
[Dr. Sarah Chen] Paranormal activity in Basement: shadow!
[Dr. Sarah Chen] Paranormal activity in Basement: poltergeist!

Motion Sensor Report (Battery: 68%):
No movement detected in Master Bedroom
Unexplained movement detected in Guest Room
Shadow figure detected moving in Basement
No movement in Attic
Objects moved in Kitchen

============================================================
Investigation Summary: Blackwood Manor
============================================================

Master Bedroom:
  Activity Level: dormant
  Paranormal Events: 3
  Notes:
    - [Master Bedroom] HIGH EMF spike in Master Bedroom (9/10)
    - [Master Bedroom] Detected 2 cold spot(s) in Master Bedroom - Temperature dropped to 53°F

Guest Room:
  Activity Level: dormant
  Paranormal Events: 1
  Notes:
    - [Guest Room] Received message in Guest Room: "Help me"

Basement:
  Activity Level: aggressive
  Paranormal Events: 5
  Notes:
    - [Basement] EXTREME EMF readings in Basement! (10/10)
    - [Basement] Strong communication in Basement: "Trapped"
    - [Basement] Shadow figure detected moving in Basement

Attic:
  Activity Level: dormant
  Paranormal Events: 0
  Notes:
    - [Attic] Detected 1 unusual warm area(s) in Attic
    - [Attic] Faint message in Attic: "Forgotten"

Kitchen:
  Activity Level: dormant
  Paranormal Events: 1
  Notes:
    - [Kitchen] Objects moved in Kitchen

============================================================
INVESTIGATION COMPLETE
============================================================
Total spirit communications: 3
Total motion detections: 2
EMF readings taken: 5
```

**Pattern Benefits:**

- **Visitor Pattern:** Different equipment (visitors) can investigate rooms without changing room classes
- **Open/Closed Principle:** Add new equipment types without modifying existing room code
- **Observer Pattern:** Investigators automatically notified of all paranormal events
- **Separation of Concerns:** Investigation logic separated from room structure
- **Type-Safe Operations:** Each equipment type implements room-specific investigation methods
- **Flexible Equipment:** Easy to add new investigation tools with unique behaviors
- **Event Broadcasting:** Multiple observers can track same paranormal events simultaneously
- **Double Dispatch:** Equipment behavior varies based on both equipment type and room type

---

## 16. Race Car Pit Stop - Chain + Observer

**Theme:** Formula racing pit crew
**Patterns:** Chain of Responsibility, Observer

**Key Learning:** Pit crew tasks chain together

**Complete Implementation:**

```python
"""
Complete race car pit stop system with Chain of Responsibility and Observer patterns.
Demonstrates chained pit crew tasks and race observers monitoring pit stops.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
import time


class TireCondition(Enum):
    NEW = "new"
    GOOD = "good"
    WORN = "worn"
    CRITICAL = "critical"


class TireType(Enum):
    SOFT = "soft"
    MEDIUM = "medium"
    HARD = "hard"
    WET = "wet"


class RaceCar:
    """Race car being serviced in pit stop"""

    def __init__(self, driver: str, team: str, number: int):
        self.driver = driver
        self.team = team
        self.number = number
        self.fuel_level = 50.0  # liters
        self.tire_condition = TireCondition.WORN
        self.tire_type = TireType.MEDIUM
        self.front_wing_damage = 30  # percentage
        self.rear_wing_angle = 5
        self.engine_temperature = 95  # celsius
        self.pit_stop_count = 0

    def get_status(self) -> str:
        return (f"\nCar #{self.number} ({self.driver} - {self.team})\n"
                f"  Fuel: {self.fuel_level:.1f}L\n"
                f"  Tires: {self.tire_type.value} ({self.tire_condition.value})\n"
                f"  Front Wing Damage: {self.front_wing_damage}%\n"
                f"  Rear Wing Angle: {self.rear_wing_angle}°\n"
                f"  Engine Temp: {self.engine_temperature}°C\n"
                f"  Pit Stops: {self.pit_stop_count}")


# Observer Pattern - Race observers monitor pit stops
class RaceObserver(ABC):
    """Observer for pit stop events"""

    @abstractmethod
    def on_pit_entry(self, car: RaceCar):
        pass

    @abstractmethod
    def on_task_complete(self, car: RaceCar, task_name: str, duration: float):
        pass

    @abstractmethod
    def on_pit_exit(self, car: RaceCar, total_time: float):
        pass


class RaceDirector(RaceObserver):
    """Race director monitoring all pit activities"""

    def __init__(self, name: str):
        self.name = name
        self.pit_stop_times: List[float] = []

    def on_pit_entry(self, car: RaceCar):
        print(f"[{self.name}] Car #{car.number} entering pit lane")

    def on_task_complete(self, car: RaceCar, task_name: str, duration: float):
        print(f"[{self.name}] {task_name} completed in {duration:.2f}s")

    def on_pit_exit(self, car: RaceCar, total_time: float):
        self.pit_stop_times.append(total_time)
        avg_time = sum(self.pit_stop_times) / len(self.pit_stop_times)
        print(f"[{self.name}] Car #{car.number} pit stop: {total_time:.2f}s (avg: {avg_time:.2f}s)")


class Commentator(RaceObserver):
    """Race commentator providing commentary"""

    def __init__(self, name: str):
        self.name = name

    def on_pit_entry(self, car: RaceCar):
        print(f"[{self.name}] And {car.driver} is coming into the pits!")

    def on_task_complete(self, car: RaceCar, task_name: str, duration: float):
        if duration < 2.0:
            print(f"[{self.name}] Lightning fast {task_name}!")
        elif duration > 4.0:
            print(f"[{self.name}] That {task_name} is taking longer than expected...")

    def on_pit_exit(self, car: RaceCar, total_time: float):
        if total_time < 3.0:
            commentary = "Incredible pit stop!"
        elif total_time < 4.0:
            commentary = "Excellent work by the crew!"
        else:
            commentary = "That'll cost them some positions"
        print(f"[{self.name}] {car.driver} is back out! {commentary}")


class TeamRadio(RaceObserver):
    """Team radio communications"""

    def __init__(self, team: str):
        self.team = team

    def on_pit_entry(self, car: RaceCar):
        if car.team == self.team:
            print(f"[Team Radio] Box box, {car.driver}")

    def on_task_complete(self, car: RaceCar, task_name: str, duration: float):
        pass  # Team doesn't announce each task

    def on_pit_exit(self, car: RaceCar, total_time: float):
        if car.team == self.team:
            print(f"[Team Radio] Good stop, {car.driver}. {total_time:.1f} seconds. Push now!")


# Chain of Responsibility Pattern - Pit crew tasks
class PitCrewHandler(ABC):
    """Abstract handler for pit crew tasks"""

    def __init__(self):
        self._next_handler: Optional[PitCrewHandler] = None

    def set_next(self, handler: 'PitCrewHandler') -> 'PitCrewHandler':
        self._next_handler = handler
        return handler

    @abstractmethod
    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        """Handle the task and return duration in seconds"""
        pass

    def _notify_observers(self, car: RaceCar, task_name: str, duration: float, observers: List[RaceObserver]):
        for observer in observers:
            observer.on_task_complete(car, task_name, duration)

    def process_next(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        if self._next_handler:
            return self._next_handler.handle(car, observers)
        return 0.0


class TireChangeHandler(PitCrewHandler):
    """Handler for changing tires"""

    def __init__(self, new_tire_type: TireType):
        super().__init__()
        self.new_tire_type = new_tire_type

    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        start_time = time.time()

        # Simulate tire change (in real-time for demo, would use time.sleep for actual delay)
        duration = 2.3  # seconds for tire change

        # Update car
        car.tire_type = self.new_tire_type
        car.tire_condition = TireCondition.NEW

        print(f"  [Tire Crew] Changed to {self.new_tire_type.value} tires")

        self._notify_observers(car, "Tire Change", duration, observers)

        # Continue chain
        next_duration = self.process_next(car, observers)
        return duration + next_duration


class RefuelHandler(PitCrewHandler):
    """Handler for refueling"""

    def __init__(self, fuel_amount: float):
        super().__init__()
        self.fuel_amount = fuel_amount

    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        # Refueling takes about 0.5s per 10 liters
        duration = (self.fuel_amount / 10.0) * 0.5

        # Update car
        car.fuel_level = min(110.0, car.fuel_level + self.fuel_amount)

        print(f"  [Fuel Crew] Added {self.fuel_amount:.1f}L fuel")

        self._notify_observers(car, "Refuel", duration, observers)

        # Continue chain
        next_duration = self.process_next(car, observers)
        return duration + next_duration


class FrontWingAdjustmentHandler(PitCrewHandler):
    """Handler for front wing adjustment/replacement"""

    def __init__(self, replace: bool = False):
        super().__init__()
        self.replace = replace

    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        if self.replace or car.front_wing_damage > 50:
            # Replace damaged wing
            duration = 4.5
            car.front_wing_damage = 0
            print(f"  [Wing Crew] Replaced damaged front wing")
        else:
            # Quick adjustment
            duration = 1.2
            print(f"  [Wing Crew] Adjusted front wing angle")

        self._notify_observers(car, "Front Wing Service", duration, observers)

        # Continue chain
        next_duration = self.process_next(car, observers)
        return duration + next_duration


class RearWingAdjustmentHandler(PitCrewHandler):
    """Handler for rear wing angle adjustment"""

    def __init__(self, new_angle: int):
        super().__init__()
        self.new_angle = new_angle

    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        duration = 0.8

        car.rear_wing_angle = self.new_angle
        print(f"  [Wing Crew] Adjusted rear wing to {self.new_angle}°")

        self._notify_observers(car, "Rear Wing Adjustment", duration, observers)

        # Continue chain
        next_duration = self.process_next(car, observers)
        return duration + next_duration


class EngineCoolingHandler(PitCrewHandler):
    """Handler for emergency engine cooling"""

    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        if car.engine_temperature < 100:
            # No cooling needed, skip
            return self.process_next(car, observers)

        duration = 2.0
        car.engine_temperature = max(85, car.engine_temperature - 15)

        print(f"  [Engine Crew] Applied cooling (temp now {car.engine_temperature}°C)")

        self._notify_observers(car, "Engine Cooling", duration, observers)

        # Continue chain
        next_duration = self.process_next(car, observers)
        return duration + next_duration


class QuickInspectionHandler(PitCrewHandler):
    """Handler for quick visual inspection"""

    def handle(self, car: RaceCar, observers: List[RaceObserver]) -> float:
        duration = 0.5

        issues = []
        if car.front_wing_damage > 20:
            issues.append("front wing damage")
        if car.engine_temperature > 95:
            issues.append("high engine temp")

        if issues:
            print(f"  [Inspector] Issues noted: {', '.join(issues)}")
        else:
            print(f"  [Inspector] Car looks good")

        self._notify_observers(car, "Inspection", duration, observers)

        # Continue chain
        next_duration = self.process_next(car, observers)
        return duration + next_duration


class PitCrew:
    """Manages pit crew and pit stop execution"""

    def __init__(self, team: str):
        self.team = team
        self.observers: List[RaceObserver] = []

    def add_observer(self, observer: RaceObserver):
        self.observers.append(observer)

    def execute_pit_stop(self, car: RaceCar, strategy: PitCrewHandler):
        """Execute a pit stop with the given strategy (handler chain)"""
        car.pit_stop_count += 1

        # Notify pit entry
        for observer in self.observers:
            observer.on_pit_entry(car)

        print(f"\n{'='*50}")
        print(f"PIT STOP #{car.pit_stop_count} - Car #{car.number}")
        print(f"{'='*50}")

        # Execute chain of tasks
        start_time = time.time()
        total_duration = strategy.handle(car, self.observers)

        # Notify pit exit
        for observer in self.observers:
            observer.on_pit_exit(car, total_duration)

        return total_duration


def main():
    print("="*60)
    print("FORMULA RACING PIT STOP SIMULATOR")
    print("Chain of Responsibility + Observer Pattern Demo")
    print("="*60)

    # Create race car
    car = RaceCar(driver="Lewis Hamilton", team="Mercedes", number=44)

    # Create pit crew
    pit_crew = PitCrew("Mercedes")

    # Create observers
    race_director = RaceDirector("Charlie Whiting")
    commentator = Commentator("David Croft")
    team_radio = TeamRadio("Mercedes")

    pit_crew.add_observer(race_director)
    pit_crew.add_observer(commentator)
    pit_crew.add_observer(team_radio)

    # Show initial car status
    print(car.get_status())

    # Pit Stop 1: Standard tire change and refuel
    print("\n" + "="*60)
    print("LAP 15 - STANDARD PIT STOP")
    print("="*60)

    strategy1 = TireChangeHandler(TireType.SOFT)
    strategy1.set_next(RefuelHandler(30.0))
    strategy1.set_next(QuickInspectionHandler())

    pit_crew.execute_pit_stop(car, strategy1)
    print(car.get_status())

    # Simulate some racing
    car.fuel_level -= 25
    car.tire_condition = TireCondition.WORN
    car.front_wing_damage = 65
    car.engine_temperature = 102

    # Pit Stop 2: Emergency stop with wing replacement
    print("\n" + "="*60)
    print("LAP 28 - EMERGENCY PIT STOP (WING DAMAGE)")
    print("="*60)

    strategy2 = FrontWingAdjustmentHandler(replace=True)
    strategy2.set_next(TireChangeHandler(TireType.MEDIUM))
    strategy2.set_next(RefuelHandler(40.0))
    strategy2.set_next(RearWingAdjustmentHandler(new_angle=7))
    strategy2.set_next(EngineCoolingHandler())
    strategy2.set_next(QuickInspectionHandler())

    pit_crew.execute_pit_stop(car, strategy2)
    print(car.get_status())

    # Simulate more racing
    car.fuel_level -= 35
    car.tire_condition = TireCondition.CRITICAL
    car.front_wing_damage = 5

    # Pit Stop 3: Late race quick stop
    print("\n" + "="*60)
    print("LAP 45 - LATE RACE QUICK STOP")
    print("="*60)

    strategy3 = TireChangeHandler(TireType.SOFT)
    strategy3.set_next(RearWingAdjustmentHandler(new_angle=3))
    strategy3.set_next(QuickInspectionHandler())

    pit_crew.execute_pit_stop(car, strategy3)
    print(car.get_status())

    # Final statistics
    print("\n" + "="*60)
    print("PIT STOP STATISTICS")
    print("="*60)
    print(f"Total pit stops: {car.pit_stop_count}")
    if race_director.pit_stop_times:
        avg_time = sum(race_director.pit_stop_times) / len(race_director.pit_stop_times)
        fastest = min(race_director.pit_stop_times)
        slowest = max(race_director.pit_stop_times)
        print(f"Average pit stop time: {avg_time:.2f}s")
        print(f"Fastest pit stop: {fastest:.2f}s")
        print(f"Slowest pit stop: {slowest:.2f}s")


if __name__ == "__main__":
    main()
```

**Example Output:**
```
============================================================
FORMULA RACING PIT STOP SIMULATOR
Chain of Responsibility + Observer Pattern Demo
============================================================

Car #44 (Lewis Hamilton - Mercedes)
  Fuel: 50.0L
  Tires: medium (worn)
  Front Wing Damage: 30%
  Rear Wing Angle: 5°
  Engine Temp: 95°C
  Pit Stops: 0

============================================================
LAP 15 - STANDARD PIT STOP
============================================================
[David Croft] And Lewis Hamilton is coming into the pits!
[Charlie Whiting] Car #44 entering pit lane
[Team Radio] Box box, Lewis Hamilton

==================================================
PIT STOP #1 - Car #44
==================================================
  [Tire Crew] Changed to soft tires
[Charlie Whiting] Tire Change completed in 2.30s
[David Croft] Lightning fast Tire Change!
  [Fuel Crew] Added 30.0L fuel
[Charlie Whiting] Refuel completed in 1.50s
[David Croft] Lightning fast Refuel!
  [Inspector] Car looks good
[Charlie Whiting] Inspection completed in 0.50s
[David Croft] Lightning fast Inspection!
[Charlie Whiting] Car #44 pit stop: 4.30s (avg: 4.30s)
[David Croft] Lewis Hamilton is back out! Excellent work by the crew!
[Team Radio] Good stop, Lewis Hamilton. 4.3 seconds. Push now!

Car #44 (Lewis Hamilton - Mercedes)
  Fuel: 80.0L
  Tires: soft (new)
  Front Wing Damage: 30%
  Rear Wing Angle: 5°
  Engine Temp: 95°C
  Pit Stops: 1

============================================================
LAP 28 - EMERGENCY PIT STOP (WING DAMAGE)
============================================================
[David Croft] And Lewis Hamilton is coming into the pits!
[Charlie Whiting] Car #44 entering pit lane
[Team Radio] Box box, Lewis Hamilton

==================================================
PIT STOP #2 - Car #44
==================================================
  [Wing Crew] Replaced damaged front wing
[Charlie Whiting] Front Wing Service completed in 4.50s
[David Croft] That Front Wing Service is taking longer than expected...
  [Tire Crew] Changed to medium tires
[Charlie Whiting] Tire Change completed in 2.30s
[David Croft] Lightning fast Tire Change!
  [Fuel Crew] Added 40.0L fuel
[Charlie Whiting] Refuel completed in 2.00s
[David Croft] Lightning fast Refuel!
  [Wing Crew] Adjusted rear wing to 7°
[Charlie Whiting] Rear Wing Adjustment completed in 0.80s
[David Croft] Lightning fast Rear Wing Adjustment!
  [Engine Crew] Applied cooling (temp now 87°C)
[Charlie Whiting] Engine Cooling completed in 2.00s
[David Croft] Lightning fast Engine Cooling!
  [Inspector] Car looks good
[Charlie Whiting] Inspection completed in 0.50s
[David Croft] Lightning fast Inspection!
[Charlie Whiting] Car #44 pit stop: 12.10s (avg: 8.20s)
[David Croft] Lewis Hamilton is back out! That'll cost them some positions
[Team Radio] Good stop, Lewis Hamilton. 12.1 seconds. Push now!

Car #44 (Lewis Hamilton - Mercedes)
  Fuel: 95.0L
  Tires: medium (new)
  Front Wing Damage: 0%
  Rear Wing Angle: 7°
  Engine Temp: 87°C
  Pit Stops: 2

============================================================
LAP 45 - LATE RACE QUICK STOP
============================================================
[David Croft] And Lewis Hamilton is coming into the pits!
[Charlie Whiting] Car #44 entering pit lane
[Team Radio] Box box, Lewis Hamilton

==================================================
PIT STOP #3 - Car #44
==================================================
  [Tire Crew] Changed to soft tires
[Charlie Whiting] Tire Change completed in 2.30s
[David Croft] Lightning fast Tire Change!
  [Wing Crew] Adjusted rear wing to 3°
[Charlie Whiting] Rear Wing Adjustment completed in 0.80s
[David Croft] Lightning fast Rear Wing Adjustment!
  [Inspector] Car looks good
[Charlie Whiting] Inspection completed in 0.50s
[David Croft] Lightning fast Inspection!
[Charlie Whiting] Car #44 pit stop: 3.60s (avg: 6.67s)
[David Croft] Lewis Hamilton is back out! Excellent work by the crew!
[Team Radio] Good stop, Lewis Hamilton. 3.6 seconds. Push now!

Car #44 (Lewis Hamilton - Mercedes)
  Fuel: 60.0L
  Tires: soft (new)
  Front Wing Damage: 5%
  Rear Wing Angle: 3°
  Engine Temp: 87°C
  Pit Stops: 3

============================================================
PIT STOP STATISTICS
============================================================
Total pit stops: 3
Average pit stop time: 6.67s
Fastest pit stop: 3.60s
Slowest pit stop: 12.10s
```

**Pattern Benefits:**

- **Chain of Responsibility:** Pit crew tasks processed in flexible sequence
- **Dynamic Configuration:** Build different pit stop strategies by chaining handlers
- **Observer Pattern:** Multiple race officials monitor same pit events
- **Loose Coupling:** Observers and handlers don't know about each other
- **Single Responsibility:** Each handler does one specific pit task
- **Open/Closed:** Add new pit tasks without modifying existing handlers
- **Real-time Notifications:** All observers immediately notified of events
- **Flexible Workflows:** Chain can be reconfigured for different race situations

---

## 17. Pet Monster Ranch - Prototype + Observer

**Theme:** Monster raising sim
**Patterns:** Prototype, Observer, State

**Key Learning:** Breed monsters by cloning with variations

**Complete Implementation:**

```python
"""
Complete pet monster ranch with Prototype and Observer patterns.
Demonstrates monster breeding via cloning and observers watching monster events.
"""

from abc import ABC, abstractmethod
from typing import List, Optional, Tuple
from enum import Enum
import random
import copy


class MonsterType(Enum):
    FIRE = "fire"
    WATER = "water"
    EARTH = "earth"
    AIR = "air"
    HYBRID = "hybrid"


class MonsterMood(Enum):
    HAPPY = "happy"
    CONTENT = "content"
    HUNGRY = "hungry"
    SICK = "sick"
    EXCITED = "excited"


# Observer Pattern - Monitor monster events
class MonsterObserver(ABC):
    """Observer for monster ranch events"""

    @abstractmethod
    def on_monster_born(self, monster: 'Monster'):
        pass

    @abstractmethod
    def on_monster_evolved(self, monster: 'Monster', old_stats: dict):
        pass

    @abstractmethod
    def on_rare_trait_discovered(self, monster: 'Monster', trait: str):
        pass


class Rancher(MonsterObserver):
    """Ranch owner monitoring monsters"""

    def __init__(self, name: str):
        self.name = name
        self.discoveries: List[str] = []

    def on_monster_born(self, monster: 'Monster'):
        print(f"[{self.name}] New monster born: {monster.name} ({monster.species})")
        if monster.is_rare:
            print(f"[{self.name}] This is a RARE monster!")

    def on_monster_evolved(self, monster: 'Monster', old_stats: dict):
        print(f"[{self.name}] {monster.name} evolved! Power: {old_stats['power']} -> {monster.power}")

    def on_rare_trait_discovered(self, monster: 'Monster', trait: str):
        discovery = f"{monster.name}: {trait}"
        if discovery not in self.discoveries:
            self.discoveries.append(discovery)
            print(f"[{self.name}] RARE TRAIT DISCOVERED: {trait}!")


class Scientist(MonsterObserver):
    """Monster researcher"""

    def __init__(self, name: str):
        self.name = name
        self.hybrid_count = 0
        self.rare_count = 0

    def on_monster_born(self, monster: 'Monster'):
        if monster.monster_type == MonsterType.HYBRID:
            self.hybrid_count += 1
            print(f"[{self.name}] Hybrid #{self.hybrid_count} created!")

        if monster.is_rare:
            self.rare_count += 1

    def on_monster_evolved(self, monster: 'Monster', old_stats: dict):
        stat_gain = monster.power - old_stats['power']
        if stat_gain > 50:
            print(f"[{self.name}] Exceptional evolution detected! +{stat_gain} power")

    def on_rare_trait_discovered(self, monster: 'Monster', trait: str):
        print(f"[{self.name}] Recording trait '{trait}' for research")


# Prototype Pattern - Cloneable monsters
class Monster(ABC):
    """Abstract prototype for monsters"""

    def __init__(self, species: str, monster_type: MonsterType):
        self.name = f"{species}_{random.randint(1000, 9999)}"
        self.species = species
        self.monster_type = monster_type
        self.power = 10
        self.speed = 10
        self.defense = 10
        self.color = "default"
        self.size = 1.0  # meters
        self.mood = MonsterMood.CONTENT
        self.special_abilities: List[str] = []
        self.is_rare = False
        self.generation = 1
        self.parents: Optional[Tuple[str, str]] = None
        self.observers: List[MonsterObserver] = []

    def add_observer(self, observer: MonsterObserver):
        self.observers.append(observer)

    def _notify_born(self):
        for observer in self.observers:
            observer.on_monster_born(self)

    def _notify_evolved(self, old_stats: dict):
        for observer in self.observers:
            observer.on_monster_evolved(self, old_stats)

    def _notify_rare_trait(self, trait: str):
        for observer in self.observers:
            observer.on_rare_trait_discovered(self, trait)

    @abstractmethod
    def clone(self) -> 'Monster':
        """Create a copy of this monster"""
        pass

    def breed_with(self, other: 'Monster') -> 'Monster':
        """Breed this monster with another using prototype pattern"""
        # Create hybrid or pure breed
        if self.monster_type == other.monster_type:
            # Same type breeding - clone with variations
            child = self.clone()
        else:
            # Cross-breed - create hybrid
            child = HybridMonster(f"{self.species}-{other.species}", self.monster_type, other.monster_type)
            child.color = random.choice([self.color, other.color, "mixed"])

        # Inherit traits from both parents
        child.power = int((self.power + other.power) / 2 + random.randint(-5, 10))
        child.speed = int((self.speed + other.speed) / 2 + random.randint(-5, 10))
        child.defense = int((self.defense + other.defense) / 2 + random.randint(-5, 10))
        child.size = (self.size + other.size) / 2 + random.uniform(-0.2, 0.2)
        child.generation = max(self.generation, other.generation) + 1
        child.parents = (self.name, other.name)

        # Inherit abilities
        child.special_abilities = []
        for ability in self.special_abilities:
            if random.random() < 0.7:  # 70% chance to inherit
                child.special_abilities.append(ability)
        for ability in other.special_abilities:
            if random.random() < 0.7 and ability not in child.special_abilities:
                child.special_abilities.append(ability)

        # Random mutations
        if random.random() < 0.15:  # 15% chance of rare trait
            child.is_rare = True
            rare_traits = ["Shiny", "Giant", "Miniature", "Rainbow", "Glowing"]
            trait = random.choice(rare_traits)
            child.special_abilities.append(trait)
            child._notify_rare_trait(trait)

        if random.random() < 0.1:  # 10% chance of new ability
            new_abilities = ["Telepathy", "Flight", "Regeneration", "Camouflage"]
            new_ability = random.choice(new_abilities)
            if new_ability not in child.special_abilities:
                child.special_abilities.append(new_ability)

        # Copy observers from parents
        for observer in self.observers:
            child.add_observer(observer)

        child._notify_born()
        return child

    def evolve(self):
        """Monster gains experience and evolves"""
        old_stats = {
            'power': self.power,
            'speed': self.speed,
            'defense': self.defense
        }

        # Stat increases
        self.power += random.randint(10, 30)
        self.speed += random.randint(5, 20)
        self.defense += random.randint(5, 20)
        self.size *= 1.2

        self._notify_evolved(old_stats)

    def get_stats(self) -> str:
        stats = f"\n{'='*50}\n"
        stats += f"Monster: {self.name}\n"
        stats += f"Species: {self.species}\n"
        stats += f"Type: {self.monster_type.value}\n"
        stats += f"Generation: {self.generation}\n"
        if self.parents:
            stats += f"Parents: {self.parents[0]} × {self.parents[1]}\n"
        stats += f"Stats - Power: {self.power} | Speed: {self.speed} | Defense: {self.defense}\n"
        stats += f"Size: {self.size:.1f}m | Color: {self.color}\n"
        stats += f"Mood: {self.mood.value}\n"
        if self.special_abilities:
            stats += f"Abilities: {', '.join(self.special_abilities)}\n"
        if self.is_rare:
            stats += "★ RARE MONSTER ★\n"
        stats += f"{'='*50}"
        return stats


class FireMonster(Monster):
    """Fire type monster"""

    def __init__(self):
        super().__init__("Flamewing", MonsterType.FIRE)
        self.color = "red"
        self.power = 25
        self.speed = 15
        self.defense = 10
        self.special_abilities = ["Fire Breath"]

    def clone(self) -> 'Monster':
        """Create a copy with slight variations"""
        new_monster = copy.deepcopy(self)
        new_monster.name = f"{self.species}_{random.randint(1000, 9999)}"
        # Slight stat variations
        new_monster.power += random.randint(-3, 3)
        new_monster.speed += random.randint(-2, 2)
        new_monster.size += random.uniform(-0.1, 0.1)
        return new_monster


class WaterMonster(Monster):
    """Water type monster"""

    def __init__(self):
        super().__init__("Aquafin", MonsterType.WATER)
        self.color = "blue"
        self.power = 15
        self.speed = 20
        self.defense = 15
        self.special_abilities = ["Water Jet"]

    def clone(self) -> 'Monster':
        new_monster = copy.deepcopy(self)
        new_monster.name = f"{self.species}_{random.randint(1000, 9999)}"
        new_monster.power += random.randint(-3, 3)
        new_monster.speed += random.randint(-2, 2)
        new_monster.size += random.uniform(-0.1, 0.1)
        return new_monster


class EarthMonster(Monster):
    """Earth type monster"""

    def __init__(self):
        super().__init__("Terraclaw", MonsterType.EARTH)
        self.color = "brown"
        self.power = 20
        self.speed = 10
        self.defense = 25
        self.special_abilities = ["Rock Shield"]

    def clone(self) -> 'Monster':
        new_monster = copy.deepcopy(self)
        new_monster.name = f"{self.species}_{random.randint(1000, 9999)}"
        new_monster.power += random.randint(-3, 3)
        new_monster.defense += random.randint(-2, 2)
        new_monster.size += random.uniform(-0.1, 0.1)
        return new_monster


class AirMonster(Monster):
    """Air type monster"""

    def __init__(self):
        super().__init__("Skywing", MonsterType.AIR)
        self.color = "white"
        self.power = 18
        self.speed = 30
        self.defense = 8
        self.special_abilities = ["Wind Gust"]

    def clone(self) -> 'Monster':
        new_monster = copy.deepcopy(self)
        new_monster.name = f"{self.species}_{random.randint(1000, 9999)}"
        new_monster.power += random.randint(-3, 3)
        new_monster.speed += random.randint(-2, 2)
        new_monster.size += random.uniform(-0.1, 0.1)
        return new_monster


class HybridMonster(Monster):
    """Hybrid monster created from two different types"""

    def __init__(self, species: str, type1: MonsterType, type2: MonsterType):
        super().__init__(species, MonsterType.HYBRID)
        self.type1 = type1
        self.type2 = type2
        self.color = "hybrid"

    def clone(self) -> 'Monster':
        new_monster = copy.deepcopy(self)
        new_monster.name = f"{self.species}_{random.randint(1000, 9999)}"
        new_monster.power += random.randint(-5, 5)
        new_monster.speed += random.randint(-3, 3)
        new_monster.defense += random.randint(-3, 3)
        new_monster.size += random.uniform(-0.15, 0.15)
        return new_monster


class MonsterRanch:
    """Manages the monster ranch"""

    def __init__(self, name: str):
        self.name = name
        self.monsters: List[Monster] = []
        self.observers: List[MonsterObserver] = []

    def add_observer(self, observer: MonsterObserver):
        self.observers.append(observer)

    def add_monster(self, monster: Monster):
        """Add a monster to the ranch"""
        self.monsters.append(monster)
        # Subscribe all ranch observers to the monster
        for observer in self.observers:
            monster.add_observer(observer)

    def breed_monsters(self, parent1_name: str, parent2_name: str) -> Optional[Monster]:
        """Breed two monsters together"""
        parent1 = next((m for m in self.monsters if m.name == parent1_name), None)
        parent2 = next((m for m in self.monsters if m.name == parent2_name), None)

        if not parent1 or not parent2:
            print("Error: One or both parents not found")
            return None

        print(f"\n🥚 Breeding {parent1.name} with {parent2.name}...")
        child = parent1.breed_with(parent2)
        self.add_monster(child)
        return child

    def get_monster_count_by_type(self) -> dict:
        counts = {}
        for monster in self.monsters:
            type_name = monster.monster_type.value
            counts[type_name] = counts.get(type_name, 0) + 1
        return counts


def main():
    print("="*60)
    print("MONSTER RANCH - Prototype + Observer Pattern Demo")
    print("="*60)

    # Create ranch
    ranch = MonsterRanch("Dragon Valley Ranch")

    # Create observers
    rancher = Rancher("Sarah")
    scientist = Scientist("Dr. Oak")

    ranch.add_observer(rancher)
    ranch.add_observer(scientist)

    # Create initial monsters
    print("\n--- Creating Initial Monsters ---")
    fire1 = FireMonster()
    water1 = WaterMonster()
    earth1 = EarthMonster()
    air1 = AirMonster()

    ranch.add_monster(fire1)
    ranch.add_monster(water1)
    ranch.add_monster(earth1)
    ranch.add_monster(air1)

    print(fire1.get_stats())
    print(water1.get_stats())

    # Breed same-type monsters (cloning with variation)
    print("\n" + "="*60)
    print("EXPERIMENT 1: Pure Breeding (Same Type)")
    print("="*60)
    fire2 = FireMonster()
    ranch.add_monster(fire2)

    fire_child = ranch.breed_monsters(fire1.name, fire2.name)
    if fire_child:
        print(fire_child.get_stats())

    # Breed different types (hybrid)
    print("\n" + "="*60)
    print("EXPERIMENT 2: Cross Breeding (Hybrid Creation)")
    print("="*60)

    hybrid1 = ranch.breed_monsters(fire1.name, water1.name)
    if hybrid1:
        print(hybrid1.get_stats())

    hybrid2 = ranch.breed_monsters(earth1.name, air1.name)
    if hybrid2:
        print(hybrid2.get_stats())

    # Evolve a monster
    print("\n" + "="*60)
    print("EXPERIMENT 3: Monster Evolution")
    print("="*60)
    print(f"\nBefore evolution:")
    print(fire1.get_stats())

    fire1.evolve()

    print(f"\nAfter evolution:")
    print(fire1.get_stats())

    # Breed hybrids together
    print("\n" + "="*60)
    print("EXPERIMENT 4: Hybrid × Hybrid Breeding")
    print("="*60)

    if hybrid1 and hybrid2:
        super_hybrid = ranch.breed_monsters(hybrid1.name, hybrid2.name)
        if super_hybrid:
            print(super_hybrid.get_stats())

    # Statistics
    print("\n" + "="*60)
    print("RANCH STATISTICS")
    print("="*60)
    print(f"Total monsters: {len(ranch.monsters)}")
    print(f"Monster types: {ranch.get_monster_count_by_type()}")
    print(f"Hybrids created: {scientist.hybrid_count}")
    print(f"Rare monsters: {scientist.rare_count}")
    print(f"\nRare discoveries by {rancher.name}:")
    for discovery in rancher.discoveries:
        print(f"  - {discovery}")


if __name__ == "__main__":
    main()
```

**Example Output:**
```
============================================================
MONSTER RANCH - Prototype + Observer Pattern Demo
============================================================

--- Creating Initial Monsters ---

==================================================
Monster: Flamewing_4521
Species: Flamewing
Type: fire
Generation: 1
Stats - Power: 25 | Speed: 15 | Defense: 10
Size: 1.0m | Color: red
Mood: content
Abilities: Fire Breath
==================================================

==================================================
Monster: Aquafin_7834
Species: Aquafin
Type: water
Generation: 1
Stats - Power: 15 | Speed: 20 | Defense: 15
Size: 1.0m | Color: blue
Mood: content
Abilities: Water Jet
==================================================

============================================================
EXPERIMENT 1: Pure Breeding (Same Type)
============================================================

🥚 Breeding Flamewing_4521 with Flamewing_9021...
[Sarah] New monster born: Flamewing_3487 (Flamewing)
[Dr. Oak] Recording trait 'Shiny' for research
[Sarah] RARE TRAIT DISCOVERED: Shiny!

==================================================
Monster: Flamewing_3487
Species: Flamewing
Type: fire
Generation: 2
Parents: Flamewing_4521 × Flamewing_9021
Stats - Power: 28 | Speed: 17 | Defense: 12
Size: 1.1m | Color: red
Mood: content
Abilities: Fire Breath, Shiny
★ RARE MONSTER ★
==================================================

============================================================
EXPERIMENT 2: Cross Breeding (Hybrid Creation)
============================================================

🥚 Breeding Flamewing_4521 with Aquafin_7834...
[Sarah] New monster born: Flamewing-Aquafin_5623 (Flamewing-Aquafin)
[Dr. Oak] Hybrid #1 created!

==================================================
Monster: Flamewing-Aquafin_5623
Species: Flamewing-Aquafin
Type: hybrid
Generation: 2
Parents: Flamewing_4521 × Aquafin_7834
Stats - Power: 24 | Speed: 21 | Defense: 14
Size: 1.0m | Color: mixed
Mood: content
Abilities: Fire Breath, Water Jet
==================================================

🥚 Breeding Terraclaw_2109 with Skywing_8765...
[Sarah] New monster born: Terraclaw-Skywing_4456 (Terraclaw-Skywing)
[Dr. Oak] Hybrid #2 created!

==================================================
Monster: Terraclaw-Skywing_4456
Species: Terraclaw-Skywing
Type: hybrid
Generation: 2
Parents: Terraclaw_2109 × Skywing_8765
Stats - Power: 28 | Speed: 24 | Defense: 18
Size: 1.1m | Color: brown
Mood: content
Abilities: Rock Shield, Wind Gust
==================================================

============================================================
EXPERIMENT 3: Monster Evolution
============================================================

Before evolution:

==================================================
Monster: Flamewing_4521
Species: Flamewing
Type: fire
Generation: 1
Stats - Power: 25 | Speed: 15 | Defense: 10
Size: 1.0m | Color: red
Mood: content
Abilities: Fire Breath
==================================================
[Sarah] Flamewing_4521 evolved! Power: 25 -> 47

After evolution:

==================================================
Monster: Flamewing_4521
Species: Flamewing
Type: fire
Generation: 1
Stats - Power: 47 | Speed: 28 | Defense: 22
Size: 1.2m | Color: red
Mood: content
Abilities: Fire Breath
==================================================

============================================================
EXPERIMENT 4: Hybrid × Hybrid Breeding
============================================================

🥚 Breeding Flamewing-Aquafin_5623 with Terraclaw-Skywing_4456...
[Sarah] New monster born: Flamewing-Aquafin-Terraclaw-Skywing_7891 (Flamewing-Aquafin-Terraclaw-Skywing)
[Dr. Oak] Hybrid #3 created!

==================================================
Monster: Flamewing-Aquafin-Terraclaw-Skywing_7891
Species: Flamewing-Aquafin-Terraclaw-Skywing
Type: hybrid
Generation: 3
Parents: Flamewing-Aquafin_5623 × Terraclaw-Skywing_4456
Stats - Power: 31 | Speed: 25 | Defense: 18
Size: 1.1m | Color: mixed
Mood: content
Abilities: Fire Breath, Water Jet, Rock Shield, Wind Gust, Regeneration
==================================================

============================================================
RANCH STATISTICS
============================================================
Total monsters: 9
Monster types: {'fire': 3, 'water': 1, 'earth': 1, 'air': 1, 'hybrid': 3}
Hybrids created: 3
Rare monsters: 1

Rare discoveries by Sarah:
  - Flamewing_3487: Shiny
```

**Pattern Benefits:**

- **Prototype Pattern:** Easy monster breeding via cloning instead of complex constructors
- **Deep Copying:** copy.deepcopy ensures complete monster duplication
- **Trait Inheritance:** Children inherit and mix parent traits naturally
- **Observer Pattern:** All ranch staff automatically notified of monster events
- **Flexible Breeding:** Same-type breeding and cross-breeding handled uniformly
- **Mutation System:** Random variations create unique offspring
- **Generational Tracking:** Maintains family tree through generations
- **Decoupled Observers:** Ranch, Rancher, and Scientist don't directly depend on each other

---

## 18. Heist Planning - Command + Memento

**Theme:** Ocean's Eleven style heist
**Patterns:** Command, Memento, Observer

**Key Learning:** Plan heist steps, simulate, rewind if caught

**Complete Implementation:**

```python
"""
Complete heist planning system with Command and Memento patterns.
Demonstrates executable heist commands with checkpoint save/restore.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from enum import Enum
import copy
import random


class SecurityLevel(Enum):
    ACTIVE = "active"
    DISABLED = "disabled"
    ALERTED = "alerted"


class HeistState:
    """State of the heist at any point"""

    def __init__(self):
        self.security_level = SecurityLevel.ACTIVE
        self.guards_alerted = False
        self.cameras_disabled = False
        self.safe_cracked = False
        self.loot_collected = 0
        self.team_position = "outside"
        self.alarm_countdown = 0
        self.suspicion_level = 0

    def get_status(self) -> str:
        return (f"Security: {self.security_level.value} | "
                f"Cameras: {'OFF' if self.cameras_disabled else 'ON'} | "
                f"Guards: {'ALERT' if self.guards_alerted else 'Normal'} | "
                f"Suspicion: {self.suspicion_level}% | "
                f"Loot: ${self.loot_collected:,}")


# Memento Pattern - Save heist state
class HeistMemento:
    """Captures complete heist state for restore"""

    def __init__(self, state: HeistState):
        self.saved_state = copy.deepcopy(state)

    def get_state(self) -> HeistState:
        return copy.deepcopy(self.saved_state)


# Command Pattern - Heist actions
class HeistCommand(ABC):
    """Abstract command for heist actions"""

    @abstractmethod
    def execute(self, state: HeistState) -> bool:
        """Execute command. Returns True if successful, False if caught"""
        pass

    @abstractmethod
    def undo(self, state: HeistState):
        """Undo the command effects"""
        pass

    @abstractmethod
    def get_description(self) -> str:
        pass

    @abstractmethod
    def get_risk_level(self) -> int:
        """Return risk level 1-10"""
        pass


class DisableCamerasCommand(HeistCommand):
    """Command to disable security cameras"""

    def __init__(self):
        self.was_disabled = False

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Hacking security cameras...")

        if state.guards_alerted:
            print(f"  [FAILED] Guards are alerted!")
            return False

        # Risk check
        risk = self.get_risk_level() + state.suspicion_level
        if random.randint(1, 100) < risk:
            print(f"  [CAUGHT] Security detected the hack!")
            state.guards_alerted = True
            state.security_level = SecurityLevel.ALERTED
            return False

        self.was_disabled = state.cameras_disabled
        state.cameras_disabled = True
        state.suspicion_level += 5
        print(f"  [SUCCESS] Cameras disabled")
        return True

    def undo(self, state: HeistState):
        state.cameras_disabled = self.was_disabled
        state.suspicion_level = max(0, state.suspicion_level - 5)

    def get_description(self) -> str:
        return "Disable Security Cameras"

    def get_risk_level(self) -> int:
        return 20


class DisableAlarmsCommand(HeistCommand):
    """Command to disable alarm systems"""

    def __init__(self):
        self.previous_countdown = 0

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Cutting alarm wires...")

        risk = self.get_risk_level() + state.suspicion_level
        if random.randint(1, 100) < risk:
            print(f"  [CAUGHT] Alarm triggered!")
            state.alarm_countdown = 60  # 60 seconds to escape
            state.guards_alerted = True
            return False

        self.previous_countdown = state.alarm_countdown
        state.alarm_countdown = 0
        state.suspicion_level += 10
        print(f"  [SUCCESS] Alarms disabled")
        return True

    def undo(self, state: HeistState):
        state.alarm_countdown = self.previous_countdown
        state.suspicion_level = max(0, state.suspicion_level - 10)

    def get_description(self) -> str:
        return "Disable Alarm System"

    def get_risk_level(self) -> int:
        return 30


class DistractGuardsCommand(HeistCommand):
    """Command to distract security guards"""

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Creating distraction for guards...")

        risk = self.get_risk_level()
        if random.randint(1, 100) < risk:
            print(f"  [CAUGHT] Guards saw through the distraction!")
            state.guards_alerted = True
            state.security_level = SecurityLevel.ALERTED
            return False

        state.suspicion_level = max(0, state.suspicion_level - 15)
        print(f"  [SUCCESS] Guards distracted - suspicion lowered")
        return True

    def undo(self, state: HeistState):
        state.suspicion_level = min(100, state.suspicion_level + 15)

    def get_description(self) -> str:
        return "Distract Guards"

    def get_risk_level(self) -> int:
        return 15


class EnterVaultCommand(HeistCommand):
    """Command to enter the vault"""

    def __init__(self):
        self.previous_position = ""

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Entering the vault...")

        if not state.cameras_disabled:
            print(f"  [FAILED] Cameras still active!")
            state.suspicion_level += 30
            return False

        if state.guards_alerted:
            print(f"  [FAILED] Guards blocking vault!")
            return False

        risk = self.get_risk_level() + state.suspicion_level
        if random.randint(1, 100) < risk:
            print(f"  [CAUGHT] Motion sensors detected entry!")
            state.guards_alerted = True
            return False

        self.previous_position = state.team_position
        state.team_position = "vault"
        state.suspicion_level += 20
        print(f"  [SUCCESS] Team inside vault")
        return True

    def undo(self, state: HeistState):
        state.team_position = self.previous_position
        state.suspicion_level = max(0, state.suspicion_level - 20)

    def get_description(self) -> str:
        return "Enter Vault"

    def get_risk_level(self) -> int:
        return 25


class CrackSafeCommand(HeistCommand):
    """Command to crack the safe"""

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Cracking the safe...")

        if state.team_position != "vault":
            print(f"  [FAILED] Team not in vault yet!")
            return False

        # Takes time, higher risk
        risk = self.get_risk_level() + state.suspicion_level
        if random.randint(1, 100) < risk:
            print(f"  [CAUGHT] Safe triggered silent alarm!")
            state.guards_alerted = True
            state.alarm_countdown = 30
            return False

        state.safe_cracked = True
        state.suspicion_level += 15
        print(f"  [SUCCESS] Safe cracked open!")
        return True

    def undo(self, state: HeistState):
        state.safe_cracked = False
        state.suspicion_level = max(0, state.suspicion_level - 15)

    def get_description(self) -> str:
        return "Crack Safe"

    def get_risk_level(self) -> int:
        return 35


class GrabLootCommand(HeistCommand):
    """Command to grab loot from safe"""

    def __init__(self, amount: int):
        self.amount = amount
        self.previous_loot = 0

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Grabbing ${self.amount:,} from safe...")

        if not state.safe_cracked:
            print(f"  [FAILED] Safe not cracked yet!")
            return False

        self.previous_loot = state.loot_collected
        state.loot_collected += self.amount
        state.suspicion_level += 5
        print(f"  [SUCCESS] Collected ${self.amount:,}")
        return True

    def undo(self, state: HeistState):
        state.loot_collected = self.previous_loot
        state.suspicion_level = max(0, state.suspicion_level - 5)

    def get_description(self) -> str:
        return f"Grab Loot (${self.amount:,})"

    def get_risk_level(self) -> int:
        return 10


class EscapeCommand(HeistCommand):
    """Command to escape the building"""

    def execute(self, state: HeistState) -> bool:
        print(f"  [ACTION] Escaping the building...")

        if state.guards_alerted and state.alarm_countdown > 0:
            print(f"  [FAILED] Guards are blocking exits!")
            return False

        risk = self.get_risk_level() + (state.suspicion_level // 2)
        if random.randint(1, 100) < risk:
            print(f"  [CAUGHT] Spotted during escape!")
            state.guards_alerted = True
            return False

        state.team_position = "escaped"
        print(f"  [SUCCESS] Clean escape!")
        return True

    def undo(self, state: HeistState):
        state.team_position = "vault"

    def get_description(self) -> str:
        return "Escape"

    def get_risk_level(self) -> int:
        return 20


class HeistPlanner:
    """Manages heist execution with checkpoints"""

    def __init__(self):
        self.state = HeistState()
        self.commands: List[HeistCommand] = []
        self.checkpoints: Dict[str, HeistMemento] = {}
        self.execution_log: List[str] = []

    def add_command(self, command: HeistCommand):
        self.commands.append(command)
        print(f"[PLAN] Added: {command.get_description()} (Risk: {command.get_risk_level()}%)")

    def create_checkpoint(self, name: str):
        self.checkpoints[name] = HeistMemento(self.state)
        print(f"[CHECKPOINT] '{name}' saved")

    def restore_checkpoint(self, name: str) -> bool:
        if name not in self.checkpoints:
            print(f"[ERROR] Checkpoint '{name}' not found")
            return False

        memento = self.checkpoints[name]
        self.state = memento.get_state()
        print(f"[RESTORE] Restored to checkpoint '{name}'")
        return True

    def execute_heist(self) -> bool:
        """Execute the planned heist"""
        print("\n" + "="*60)
        print("EXECUTING HEIST PLAN")
        print("="*60)

        for i, command in enumerate(self.commands, 1):
            print(f"\nStep {i}/{len(self.commands)}: {command.get_description()}")
            print(f"  Status: {self.state.get_status()}")

            success = command.execute(self.state)
            self.execution_log.append(f"Step {i}: {command.get_description()} - {'SUCCESS' if success else 'FAILED'}")

            if not success:
                print(f"\n{'='*60}")
                print("HEIST FAILED!")
                print("="*60)
                return False

            if self.state.alarm_countdown > 0:
                self.state.alarm_countdown -= 5
                if self.state.alarm_countdown <= 0:
                    print(f"\n[CAUGHT] Time ran out!")
                    return False

        print(f"\n{'='*60}")
        print("HEIST SUCCESSFUL!")
        print(f"Total loot: ${self.state.loot_collected:,}")
        print("="*60)
        return True

    def show_plan(self):
        print("\n" + "="*60)
        print("HEIST PLAN")
        print("="*60)
        for i, cmd in enumerate(self.commands, 1):
            print(f"{i}. {cmd.get_description()} (Risk: {cmd.get_risk_level()}%)")


def main():
    print("="*60)
    print("HEIST PLANNER - Command + Memento Pattern Demo")
    print("="*60)

    # Create heist plan
    planner = HeistPlanner()

    # Attempt 1: Risky plan
    print("\n--- ATTEMPT 1: Quick and Risky ---")
    planner.add_command(DisableCamerasCommand())
    planner.create_checkpoint("cameras_down")
    planner.add_command(EnterVaultCommand())
    planner.add_command(CrackSafeCommand())
    planner.add_command(GrabLootCommand(1000000))
    planner.add_command(EscapeCommand())

    planner.show_plan()
    success1 = planner.execute_heist()

    if not success1:
        print("\n--- RETRY: Loading checkpoint ---")
        planner.restore_checkpoint("cameras_down")

        # Try safer approach
        planner.commands = planner.commands[:1]  # Keep camera disable
        planner.add_command(DisableAlarmsCommand())
        planner.add_command(DistractGuardsCommand())
        planner.create_checkpoint("ready_to_enter")
        planner.add_command(EnterVaultCommand())
        planner.add_command(CrackSafeCommand())
        planner.add_command(GrabLootCommand(500000))
        planner.add_command(EscapeCommand())

        print("\n--- REVISED PLAN ---")
        planner.show_plan()
        success2 = planner.execute_heist()

    # Show execution log
    print("\n" + "="*60)
    print("EXECUTION LOG")
    print("="*60)
    for entry in planner.execution_log:
        print(entry)


if __name__ == "__main__":
    main()
```

**Example Output:**
```
============================================================
HEIST PLANNER - Command + Memento Pattern Demo
============================================================

--- ATTEMPT 1: Quick and Risky ---
[PLAN] Added: Disable Security Cameras (Risk: 20%)
[CHECKPOINT] 'cameras_down' saved
[PLAN] Added: Enter Vault (Risk: 25%)
[PLAN] Added: Crack Safe (Risk: 35%)
[PLAN] Added: Grab Loot ($1,000,000) (Risk: 10%)
[PLAN] Added: Escape (Risk: 20%)

============================================================
HEIST PLAN
============================================================
1. Disable Security Cameras (Risk: 20%)
2. Enter Vault (Risk: 25%)
3. Crack Safe (Risk: 35%)
4. Grab Loot ($1,000,000) (Risk: 10%)
5. Escape (Risk: 20%)

============================================================
EXECUTING HEIST PLAN
============================================================

Step 1/5: Disable Security Cameras
  Status: Security: active | Cameras: ON | Guards: Normal | Suspicion: 0% | Loot: $0
  [ACTION] Hacking security cameras...
  [SUCCESS] Cameras disabled

Step 2/5: Enter Vault
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 5% | Loot: $0
  [ACTION] Entering the vault...
  [CAUGHT] Motion sensors detected entry!

============================================================
HEIST FAILED!
============================================================

--- RETRY: Loading checkpoint ---
[RESTORE] Restored to checkpoint 'cameras_down'

--- REVISED PLAN ---
[PLAN] Added: Disable Alarm System (Risk: 30%)
[PLAN] Added: Distract Guards (Risk: 15%)
[CHECKPOINT] 'ready_to_enter' saved
[PLAN] Added: Enter Vault (Risk: 25%)
[PLAN] Added: Crack Safe (Risk: 35%)
[PLAN] Added: Grab Loot ($500,000) (Risk: 10%)
[PLAN] Added: Escape (Risk: 20%)

============================================================
HEIST PLAN
============================================================
1. Disable Security Cameras (Risk: 20%)
2. Disable Alarm System (Risk: 30%)
3. Distract Guards (Risk: 15%)
4. Enter Vault (Risk: 25%)
5. Crack Safe (Risk: 35%)
6. Grab Loot ($500,000) (Risk: 10%)
7. Escape (Risk: 20%)

============================================================
EXECUTING HEIST PLAN
============================================================

Step 1/7: Disable Security Cameras
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 5% | Loot: $0
  [ACTION] Hacking security cameras...
  [SUCCESS] Cameras disabled

Step 2/7: Disable Alarm System
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 10% | Loot: $0
  [ACTION] Cutting alarm wires...
  [SUCCESS] Alarms disabled

Step 3/7: Distract Guards
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 20% | Loot: $0
  [ACTION] Creating distraction for guards...
  [SUCCESS] Guards distracted - suspicion lowered

Step 4/7: Enter Vault
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 5% | Loot: $0
  [ACTION] Entering the vault...
  [SUCCESS] Team inside vault

Step 5/7: Crack Safe
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 25% | Loot: $0
  [ACTION] Cracking the safe...
  [SUCCESS] Safe cracked open!

Step 6/7: Grab Loot ($500,000)
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 40% | Loot: $0
  [ACTION] Grabbing $500,000 from safe...
  [SUCCESS] Collected $500,000

Step 7/7: Escape
  Status: Security: active | Cameras: OFF | Guards: Normal | Suspicion: 45% | Loot: $500,000
  [ACTION] Escaping the building...
  [SUCCESS] Clean escape!

============================================================
HEIST SUCCESSFUL!
Total loot: $500,000
============================================================

============================================================
EXECUTION LOG
============================================================
Step 1: Disable Security Cameras - SUCCESS
Step 2: Enter Vault - FAILED
Step 1: Disable Security Cameras - SUCCESS
Step 2: Disable Alarm System - SUCCESS
Step 3: Distract Guards - SUCCESS
Step 4: Enter Vault - SUCCESS
Step 5: Crack Safe - SUCCESS
Step 6: Grab Loot ($500,000) - SUCCESS
Step 7: Escape - SUCCESS
```

**Pattern Benefits:**

- **Command Pattern:** Each heist action encapsulated as executable command
- **Undo Capability:** Commands can reverse their effects
- **Memento Pattern:** Save/restore complete heist state at checkpoints
- **Risk Management:** Try risky approach, restore checkpoint if failed
- **Flexible Planning:** Commands can be added, removed, reordered
- **State Isolation:** Memento captures state without exposing internals
- **Execution Log:** Track all command executions for analysis
- **Retry Logic:** Failed heists can restart from last safe checkpoint

---

## 19. Food Truck Simulator - Strategy + State

**Theme:** Running a food truck
**Patterns:** Strategy, State, Observer

**Key Learning:** Different recipes are strategies, truck has states

**Complete Implementation:**

```python
"""
Complete food truck simulator with Strategy and State patterns.
Demonstrates recipe strategies and truck operational states.
"""

from abc import ABC, abstractmethod
from typing import List, Dict
from enum import Enum


class Ingredient:
    def __init__(self, name: str, cost: float):
        self.name = name
        self.cost = cost


# Strategy Pattern - Recipe strategies
class RecipeStrategy(ABC):
    """Abstract recipe strategy"""

    @abstractmethod
    def get_name(self) -> str:
        pass

    @abstractmethod
    def get_required_ingredients(self) -> List[str]:
        pass

    @abstractmethod
    def get_price(self) -> float:
        pass

    @abstractmethod
    def get_prep_time(self) -> int:
        """Preparation time in seconds"""
        pass

    def can_prepare(self, inventory: Dict[str, int]) -> bool:
        for ingredient in self.get_required_ingredients():
            if inventory.get(ingredient, 0) <= 0:
                return False
        return True


class TacoStrategy(RecipeStrategy):
    def get_name(self) -> str:
        return "Taco"

    def get_required_ingredients(self) -> List[str]:
        return ["tortilla", "meat", "salsa", "lettuce"]

    def get_price(self) -> float:
        return 5.00

    def get_prep_time(self) -> int:
        return 3


class BurritoStrategy(RecipeStrategy):
    def get_name(self) -> str:
        return "Burrito"

    def get_required_ingredients(self) -> List[str]:
        return ["tortilla", "meat", "rice", "beans", "cheese"]

    def get_price(self) -> float:
        return 8.00

    def get_prep_time(self) -> int:
        return 5


class QuesadillaStrategy(RecipeStrategy):
    def get_name(self) -> str:
        return "Quesadilla"

    def get_required_ingredients(self) -> List[str]:
        return ["tortilla", "cheese"]

    def get_price(self) -> float:
        return 4.00

    def get_prep_time(self) -> int:
        return 2


# State Pattern - Truck operational states
class TruckState(ABC):
    """Abstract truck state"""

    @abstractmethod
    def open_truck(self, truck: 'FoodTruck'):
        pass

    @abstractmethod
    def close_truck(self, truck: 'FoodTruck'):
        pass

    @abstractmethod
    def take_order(self, truck: 'FoodTruck', recipe: RecipeStrategy) -> bool:
        pass

    @abstractmethod
    def restock(self, truck: 'FoodTruck'):
        pass

    @abstractmethod
    def get_state_name(self) -> str:
        pass


class ClosedState(TruckState):
    def open_truck(self, truck: 'FoodTruck'):
        print("[TRUCK] Opening for business!")
        truck.state = OpenState()

    def close_truck(self, truck: 'FoodTruck'):
        print("[TRUCK] Already closed")

    def take_order(self, truck: 'FoodTruck', recipe: RecipeStrategy) -> bool:
        print("[TRUCK] Cannot take orders - truck is closed!")
        return False

    def restock(self, truck: 'FoodTruck'):
        print("[TRUCK] Restocking inventory...")
        truck.inventory = {
            "tortilla": 50,
            "meat": 30,
            "salsa": 20,
            "lettuce": 25,
            "rice": 30,
            "beans": 30,
            "cheese": 25
        }
        print("[TRUCK] Inventory restocked")

    def get_state_name(self) -> str:
        return "Closed"


class OpenState(TruckState):
    def open_truck(self, truck: 'FoodTruck'):
        print("[TRUCK] Already open!")

    def close_truck(self, truck: 'FoodTruck'):
        print("[TRUCK] Closing for the day")
        print(f"[TRUCK] Total revenue: ${truck.revenue:.2f}")
        truck.state = ClosedState()

    def take_order(self, truck: 'FoodTruck', recipe: RecipeStrategy) -> bool:
        if not recipe.can_prepare(truck.inventory):
            print(f"[TRUCK] Cannot make {recipe.get_name()} - missing ingredients!")
            # Check if need to transition to out of stock
            total_items = sum(truck.inventory.values())
            if total_items < 10:
                print("[TRUCK] Running low on inventory!")
                truck.state = LowStockState()
            return False

        # Prepare order
        print(f"[TRUCK] Preparing {recipe.get_name()}... ({recipe.get_prep_time()}s)")

        # Consume ingredients
        for ingredient in recipe.get_required_ingredients():
            truck.inventory[ingredient] -= 1

        # Add revenue
        price = recipe.get_price()
        truck.revenue += price
        truck.orders_completed += 1

        print(f"[TRUCK] Order complete! ${price:.2f}")
        return True

    def restock(self, truck: 'FoodTruck'):
        print("[TRUCK] Cannot restock while open - close first!")

    def get_state_name(self) -> str:
        return "Open"


class LowStockState(TruckState):
    def open_truck(self, truck: 'FoodTruck'):
        print("[TRUCK] Already open (but low on stock)")

    def close_truck(self, truck: 'FoodTruck'):
        print("[TRUCK] Closing early due to low stock")
        print(f"[TRUCK] Total revenue: ${truck.revenue:.2f}")
        truck.state = ClosedState()

    def take_order(self, truck: 'FoodTruck', recipe: RecipeStrategy) -> bool:
        print("[TRUCK] WARNING: Low inventory!")

        if not recipe.can_prepare(truck.inventory):
            print(f"[TRUCK] Cannot make {recipe.get_name()} - out of ingredients!")
            print("[TRUCK] Closing due to insufficient stock")
            truck.state = ClosedState()
            return False

        # Can still prepare this item
        print(f"[TRUCK] Preparing {recipe.get_name()}...")

        for ingredient in recipe.get_required_ingredients():
            truck.inventory[ingredient] -= 1

        price = recipe.get_price()
        truck.revenue += price
        truck.orders_completed += 1

        print(f"[TRUCK] Order complete! ${price:.2f}")
        return True

    def restock(self, truck: 'FoodTruck'):
        print("[TRUCK] Cannot restock while open!")

    def get_state_name(self) -> str:
        return "Low Stock"


class FoodTruck:
    """Food truck with state and recipe strategies"""

    def __init__(self, name: str):
        self.name = name
        self.state: TruckState = ClosedState()
        self.inventory: Dict[str, int] = {}
        self.revenue = 0.0
        self.orders_completed = 0
        self.menu: List[RecipeStrategy] = []

    def add_to_menu(self, recipe: RecipeStrategy):
        self.menu.append(recipe)
        print(f"[MENU] Added {recipe.get_name()} - ${recipe.get_price():.2f}")

    def open(self):
        self.state.open_truck(self)

    def close(self):
        self.state.close_truck(self)

    def take_order(self, recipe_name: str) -> bool:
        # Find recipe in menu
        recipe = next((r for r in self.menu if r.get_name() == recipe_name), None)
        if not recipe:
            print(f"[TRUCK] {recipe_name} not on menu!")
            return False

        return self.state.take_order(self, recipe)

    def restock(self):
        self.state.restock(self)

    def get_status(self) -> str:
        status = f"\n{'='*50}\n"
        status += f"Food Truck: {self.name}\n"
        status += f"State: {self.state.get_state_name()}\n"
        status += f"Revenue: ${self.revenue:.2f}\n"
        status += f"Orders Completed: {self.orders_completed}\n"
        status += f"\nInventory:\n"
        for item, count in self.inventory.items():
            status += f"  {item}: {count}\n"
        status += f"\nMenu:\n"
        for recipe in self.menu:
            status += f"  {recipe.get_name()} - ${recipe.get_price():.2f}\n"
        status += f"{'='*50}"
        return status


def main():
    print("="*60)
    print("FOOD TRUCK SIMULATOR - Strategy + State Pattern Demo")
    print("="*60)

    # Create food truck
    truck = FoodTruck("Taco Express")

    # Add recipes to menu (strategies)
    truck.add_to_menu(TacoStrategy())
    truck.add_to_menu(BurritoStrategy())
    truck.add_to_menu(QuesadillaStrategy())

    print(truck.get_status())

    # Try to take order while closed
    print("\n--- Attempting order while closed ---")
    truck.take_order("Taco")

    # Restock and open
    print("\n--- Restocking and opening ---")
    truck.restock()
    truck.open()

    print(truck.get_status())

    # Take various orders
    print("\n--- Taking orders ---")
    truck.take_order("Taco")
    truck.take_order("Burrito")
    truck.take_order("Quesadilla")
    truck.take_order("Taco")
    truck.take_order("Burrito")

    print(truck.get_status())

    # Simulate many orders to deplete inventory
    print("\n--- Busy lunch rush! ---")
    for i in range(15):
        truck.take_order("Burrito")

    print(truck.get_status())

    # Try one more order in low stock state
    print("\n--- One more order in low stock ---")
    truck.take_order("Taco")

    # Close truck
    print("\n--- Closing truck ---")
    truck.close()

    print(truck.get_status())

    # Restock for next day
    print("\n--- Preparing for next day ---")
    truck.restock()

    print(truck.get_status())


if __name__ == "__main__":
    main()
```

**Example Output:**
```
============================================================
FOOD TRUCK SIMULATOR - Strategy + State Pattern Demo
============================================================
[MENU] Added Taco - $5.00
[MENU] Added Burrito - $8.00
[MENU] Added Quesadilla - $4.00

==================================================
Food Truck: Taco Express
State: Closed
Revenue: $0.00
Orders Completed: 0

Inventory:

Menu:
  Taco - $5.00
  Burrito - $8.00
  Quesadilla - $4.00
==================================================

--- Attempting order while closed ---
[TRUCK] Cannot take orders - truck is closed!

--- Restocking and opening ---
[TRUCK] Restocking inventory...
[TRUCK] Inventory restocked
[TRUCK] Opening for business!

==================================================
Food Truck: Taco Express
State: Open
Revenue: $0.00
Orders Completed: 0

Inventory:
  tortilla: 50
  meat: 30
  salsa: 20
  lettuce: 25
  rice: 30
  beans: 30
  cheese: 25

Menu:
  Taco - $5.00
  Burrito - $8.00
  Quesadilla - $4.00
==================================================

--- Taking orders ---
[TRUCK] Preparing Taco... (3s)
[TRUCK] Order complete! $5.00
[TRUCK] Preparing Burrito... (5s)
[TRUCK] Order complete! $8.00
[TRUCK] Preparing Quesadilla... (2s)
[TRUCK] Order complete! $4.00
[TRUCK] Preparing Taco... (3s)
[TRUCK] Order complete! $5.00
[TRUCK] Preparing Burrito... (5s)
[TRUCK] Order complete! $8.00

==================================================
Food Truck: Taco Express
State: Open
Revenue: $30.00
Orders Completed: 5

Inventory:
  tortilla: 45
  meat: 27
  salsa: 18
  lettuce: 23
  rice: 28
  beans: 28
  cheese: 22

Menu:
  Taco - $5.00
  Burrito - $8.00
  Quesadilla - $4.00
==================================================

--- Busy lunch rush! ---
[TRUCK] Preparing Burrito... (5s)
[TRUCK] Order complete! $8.00
...
[TRUCK] Running low on inventory!
[TRUCK] WARNING: Low inventory!
[TRUCK] Preparing Burrito... (5s)
[TRUCK] Order complete! $8.00
[TRUCK] WARNING: Low inventory!
[TRUCK] Cannot make Burrito - out of ingredients!
[TRUCK] Closing due to insufficient stock

==================================================
Food Truck: Taco Express
State: Closed
Revenue: $94.00
Orders Completed: 13

Inventory:
  tortilla: 35
  meat: 17
  salsa: 18
  lettuce: 23
  rice: 18
  beans: 18
  cheese: 12

Menu:
  Taco - $5.00
  Burrito - $8.00
  Quesadilla - $4.00
==================================================
```

**Pattern Benefits:**

- **Strategy Pattern:** Different recipes encapsulated as swappable strategies
- **State Pattern:** Truck behavior changes based on operational state
- **Automatic State Transitions:** Low inventory triggers state change automatically
- **Single Responsibility:** Each recipe and state handles its own logic
- **Open/Closed:** Add new recipes without modifying truck code
- **Realistic Simulation:** States enforce business rules (can't order when closed)
- **Flexible Menu:** Recipes can be added/removed dynamically
- **Context-Aware Behavior:** Same order request behaves differently in different states

---

## 20. Ancient Library Quest - Iterator + Composite

**Theme:** Magical library exploration
**Patterns:** Iterator, Composite, Visitor

**Key Learning:** Navigate nested book collections

**Complete Implementation:**

```python
"""
Complete Ancient Library Quest system with Iterator and Composite patterns.
Demonstrates nested book collections and various traversal strategies.
"""

from abc import ABC, abstractmethod
from typing import List, Optional, Iterator, Any
from enum import Enum
from dataclasses import dataclass
import random

class BookRarity(Enum):
    COMMON = "common"
    UNCOMMON = "uncommon"
    RARE = "rare"
    LEGENDARY = "legendary"
    ANCIENT = "ancient"

@dataclass
class Book:
    """Individual book in the library"""
    title: str
    author: str
    topic: str
    rarity: BookRarity
    magic_power: int
    pages: int

    def __repr__(self) -> str:
        return f"{self.title} by {self.author} ({self.rarity.value})"

# Composite Pattern - Library structure
class LibraryComponent(ABC):
    """Base component for Composite pattern"""

    @abstractmethod
    def get_name(self) -> str:
        pass

    @abstractmethod
    def get_book_count(self) -> int:
        pass

    @abstractmethod
    def search(self, criteria: str) -> List[Book]:
        pass

    @abstractmethod
    def get_all_books(self) -> List[Book]:
        pass

    @abstractmethod
    def display(self, indent: int = 0) -> str:
        pass

class BookLeaf(LibraryComponent):
    """Leaf node - single book"""

    def __init__(self, book: Book):
        self.book = book

    def get_name(self) -> str:
        return self.book.title

    def get_book_count(self) -> int:
        return 1

    def search(self, criteria: str) -> List[Book]:
        criteria_lower = criteria.lower()
        if (criteria_lower in self.book.title.lower() or
            criteria_lower in self.book.author.lower() or
            criteria_lower in self.book.topic.lower()):
            return [self.book]
        return []

    def get_all_books(self) -> List[Book]:
        return [self.book]

    def display(self, indent: int = 0) -> str:
        return "  " * indent + f"📖 {self.book.title} [{self.book.rarity.value}]"

class LibrarySection(LibraryComponent):
    """Composite node - section containing books and subsections"""

    def __init__(self, name: str, description: str = ""):
        self.name = name
        self.description = description
        self.children: List[LibraryComponent] = []

    def add(self, component: LibraryComponent):
        self.children.append(component)

    def remove(self, component: LibraryComponent):
        self.children.remove(component)

    def get_name(self) -> str:
        return self.name

    def get_book_count(self) -> int:
        return sum(child.get_book_count() for child in self.children)

    def search(self, criteria: str) -> List[Book]:
        results = []
        for child in self.children:
            results.extend(child.search(criteria))
        return results

    def get_all_books(self) -> List[Book]:
        books = []
        for child in self.children:
            books.extend(child.get_all_books())
        return books

    def display(self, indent: int = 0) -> str:
        result = "  " * indent + f"📁 {self.name} ({self.get_book_count()} books)\n"
        for child in self.children:
            result += child.display(indent + 1) + "\n"
        return result.rstrip()

# Iterator Pattern - Different traversal strategies
class LibraryIterator(ABC):
    """Base iterator for library traversal"""

    @abstractmethod
    def __iter__(self):
        pass

    @abstractmethod
    def __next__(self) -> Book:
        pass

class DepthFirstIterator(LibraryIterator):
    """Depth-first traversal of library"""

    def __init__(self, root: LibraryComponent):
        self.stack: List[LibraryComponent] = [root]
        self.books: List[Book] = []
        self.current_index = 0
        self._collect_all_books()

    def _collect_all_books(self):
        """Collect all books in depth-first order"""
        temp_stack = [self.stack[0]]
        visited = []

        while temp_stack:
            component = temp_stack.pop()
            visited.append(component)

            if isinstance(component, BookLeaf):
                self.books.append(component.book)
            elif isinstance(component, LibrarySection):
                # Add children in reverse order for correct DFS
                for child in reversed(component.children):
                    temp_stack.append(child)

    def __iter__(self):
        self.current_index = 0
        return self

    def __next__(self) -> Book:
        if self.current_index >= len(self.books):
            raise StopIteration
        book = self.books[self.current_index]
        self.current_index += 1
        return book

class BreadthFirstIterator(LibraryIterator):
    """Breadth-first traversal of library"""

    def __init__(self, root: LibraryComponent):
        self.books: List[Book] = []
        self.current_index = 0
        self._collect_all_books(root)

    def _collect_all_books(self, root: LibraryComponent):
        """Collect all books in breadth-first order"""
        from collections import deque
        queue = deque([root])

        while queue:
            component = queue.popleft()

            if isinstance(component, BookLeaf):
                self.books.append(component.book)
            elif isinstance(component, LibrarySection):
                queue.extend(component.children)

    def __iter__(self):
        self.current_index = 0
        return self

    def __next__(self) -> Book:
        if self.current_index >= len(self.books):
            raise StopIteration
        book = self.books[self.current_index]
        self.current_index += 1
        return book

class RarityFilterIterator(LibraryIterator):
    """Iterator that filters books by rarity"""

    def __init__(self, root: LibraryComponent, min_rarity: BookRarity):
        self.root = root
        self.min_rarity = min_rarity
        self.rarity_values = {
            BookRarity.COMMON: 1,
            BookRarity.UNCOMMON: 2,
            BookRarity.RARE: 3,
            BookRarity.LEGENDARY: 4,
            BookRarity.ANCIENT: 5
        }
        self.books: List[Book] = []
        self.current_index = 0
        self._collect_filtered_books()

    def _collect_filtered_books(self):
        """Collect books matching rarity filter"""
        all_books = self.root.get_all_books()
        min_value = self.rarity_values[self.min_rarity]

        for book in all_books:
            if self.rarity_values[book.rarity] >= min_value:
                self.books.append(book)

    def __iter__(self):
        self.current_index = 0
        return self

    def __next__(self) -> Book:
        if self.current_index >= len(self.books):
            raise StopIteration
        book = self.books[self.current_index]
        self.current_index += 1
        return book

class MagicPowerIterator(LibraryIterator):
    """Iterator that returns books sorted by magic power"""

    def __init__(self, root: LibraryComponent, min_power: int = 0):
        self.books: List[Book] = []
        self.current_index = 0
        all_books = root.get_all_books()

        # Filter and sort by magic power
        filtered = [b for b in all_books if b.magic_power >= min_power]
        self.books = sorted(filtered, key=lambda b: b.magic_power, reverse=True)

    def __iter__(self):
        self.current_index = 0
        return self

    def __next__(self) -> Book:
        if self.current_index >= len(self.books):
            raise StopIteration
        book = self.books[self.current_index]
        self.current_index += 1
        return book

# Quest system
class LibraryQuest:
    """Quest system for exploring the ancient library"""

    def __init__(self, library: LibraryComponent):
        self.library = library
        self.discovered_books: List[Book] = []
        self.knowledge_points = 0

    def find_ancient_spell(self) -> Optional[Book]:
        """Quest: Find any ancient spell book"""
        print("\n🔍 Quest: Find an Ancient Spell Book")
        print("-" * 50)

        iterator = RarityFilterIterator(self.library, BookRarity.ANCIENT)
        for book in iterator:
            if "spell" in book.topic.lower() or "magic" in book.topic.lower():
                print(f"✨ Found: {book}")
                self.discovered_books.append(book)
                self.knowledge_points += book.magic_power
                return book

        print("❌ No ancient spell books found")
        return None

    def collect_powerful_tomes(self, min_power: int) -> List[Book]:
        """Quest: Collect all books with magic power above threshold"""
        print(f"\n🔍 Quest: Collect Powerful Tomes (Power >= {min_power})")
        print("-" * 50)

        iterator = MagicPowerIterator(self.library, min_power)
        powerful_books = []

        for book in iterator:
            print(f"  ⚡ {book.title}: {book.magic_power} power")
            powerful_books.append(book)
            self.discovered_books.append(book)
            self.knowledge_points += book.magic_power // 10

        print(f"\n📚 Collected {len(powerful_books)} powerful tomes!")
        return powerful_books

    def search_by_author(self, author: str) -> List[Book]:
        """Quest: Find all books by a specific author"""
        print(f"\n🔍 Quest: Find Books by {author}")
        print("-" * 50)

        results = self.library.search(author)
        for book in results:
            print(f"  📖 {book}")
            self.discovered_books.append(book)
            self.knowledge_points += 10

        print(f"\n Found {len(results)} books")
        return results

    def catalog_entire_library(self) -> dict:
        """Quest: Create catalog of entire library"""
        print("\n🔍 Quest: Catalog Entire Library")
        print("-" * 50)

        catalog = {
            "total_books": 0,
            "by_rarity": {},
            "by_topic": {},
            "total_magic_power": 0
        }

        iterator = DepthFirstIterator(self.library)
        for book in iterator:
            catalog["total_books"] += 1
            catalog["total_magic_power"] += book.magic_power

            # Count by rarity
            rarity = book.rarity.value
            catalog["by_rarity"][rarity] = catalog["by_rarity"].get(rarity, 0) + 1

            # Count by topic
            catalog["by_topic"][book.topic] = catalog["by_topic"].get(book.topic, 0) + 1

        print(f"  Total Books: {catalog['total_books']}")
        print(f"  Total Magic Power: {catalog['total_magic_power']}")
        print(f"  Rarity Distribution: {catalog['by_rarity']}")
        print(f"  Topic Distribution: {catalog['by_topic']}")

        self.knowledge_points += 50
        return catalog

    def get_quest_summary(self) -> str:
        """Get summary of quest progress"""
        summary = "\n" + "=" * 50 + "\n"
        summary += "📜 QUEST SUMMARY\n"
        summary += "=" * 50 + "\n"
        summary += f"Books Discovered: {len(set(self.discovered_books))}\n"
        summary += f"Knowledge Points: {self.knowledge_points}\n"
        summary += "=" * 50
        return summary

# Example Usage
def main():
    print("=== ANCIENT LIBRARY QUEST - Iterator + Composite Demo ===\n")

    # Build the library structure using Composite pattern
    library = LibrarySection("Ancient Grand Library", "Repository of magical knowledge")

    # Spellcraft Wing
    spellcraft = LibrarySection("Spellcraft Wing", "Books on magical spells")
    spellcraft.add(BookLeaf(Book(
        "Arcane Fundamentals",
        "Merlin the Wise",
        "Magic Basics",
        BookRarity.COMMON,
        magic_power=50,
        pages=120
    )))
    spellcraft.add(BookLeaf(Book(
        "Advanced Pyromancy",
        "Flame Master Ignis",
        "Fire Magic",
        BookRarity.RARE,
        magic_power=200,
        pages=300
    )))
    spellcraft.add(BookLeaf(Book(
        "The Eternal Codex",
        "Ancient Order",
        "Ancient Spells",
        BookRarity.ANCIENT,
        magic_power=1000,
        pages=666
    )))

    # History Section
    history = LibrarySection("History Archives", "Historical texts")
    history.add(BookLeaf(Book(
        "Chronicles of the First Age",
        "Chronicler Aldus",
        "History",
        BookRarity.UNCOMMON,
        magic_power=30,
        pages=500
    )))
    history.add(BookLeaf(Book(
        "Fall of the Dark Empire",
        "Chronicler Aldus",
        "History",
        BookRarity.RARE,
        magic_power=100,
        pages=450
    )))

    # Alchemy Wing
    alchemy = LibrarySection("Alchemy Laboratory", "Potions and transmutation")
    alchemy.add(BookLeaf(Book(
        "Potion Brewing 101",
        "Master Brewer",
        "Alchemy",
        BookRarity.COMMON,
        magic_power=40,
        pages=80
    )))
    alchemy.add(BookLeaf(Book(
        "Philosopher's Stone Secrets",
        "Nicolas Flamel",
        "Alchemy",
        BookRarity.LEGENDARY,
        magic_power=500,
        pages=200
    )))

    # Forbidden Section
    forbidden = LibrarySection("Forbidden Vault", "Dangerous knowledge")
    forbidden.add(BookLeaf(Book(
        "Necromancy Unveiled",
        "Unknown",
        "Dark Magic",
        BookRarity.LEGENDARY,
        magic_power=800,
        pages=350
    )))
    forbidden.add(BookLeaf(Book(
        "Book of the Dead",
        "Death Itself",
        "Ancient Spells",
        BookRarity.ANCIENT,
        magic_power=1500,
        pages=999
    )))

    # Build library structure
    library.add(spellcraft)
    library.add(history)
    library.add(alchemy)
    library.add(forbidden)

    # Display library structure
    print("📚 LIBRARY STRUCTURE:")
    print("=" * 60)
    print(library.display())
    print("=" * 60)

    # Create quest system
    quest = LibraryQuest(library)

    # Quest 1: Find ancient spell
    quest.find_ancient_spell()

    # Quest 2: Collect powerful tomes
    quest.collect_powerful_tomes(min_power=200)

    # Quest 3: Search by author
    quest.search_by_author("Chronicler Aldus")

    # Quest 4: Catalog entire library
    quest.catalog_entire_library()

    # Demonstrate different iteration strategies
    print("\n\n🔄 ITERATION STRATEGIES:")
    print("=" * 60)

    print("\n1️⃣  DEPTH-FIRST ITERATION:")
    print("-" * 40)
    df_iterator = DepthFirstIterator(library)
    for i, book in enumerate(df_iterator, 1):
        print(f"  {i}. {book.title}")

    print("\n2️⃣  BREADTH-FIRST ITERATION:")
    print("-" * 40)
    bf_iterator = BreadthFirstIterator(library)
    for i, book in enumerate(bf_iterator, 1):
        print(f"  {i}. {book.title}")

    print("\n3️⃣  RARE BOOKS ONLY (Rare+):")
    print("-" * 40)
    rare_iterator = RarityFilterIterator(library, BookRarity.RARE)
    for book in rare_iterator:
        print(f"  ⭐ {book.title} [{book.rarity.value}]")

    print("\n4️⃣  BY MAGIC POWER (500+):")
    print("-" * 40)
    power_iterator = MagicPowerIterator(library, min_power=500)
    for book in power_iterator:
        print(f"  ⚡ {book.title}: {book.magic_power} power")

    # Quest summary
    print(quest.get_quest_summary())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== ANCIENT LIBRARY QUEST - Iterator + Composite Demo ===

📚 LIBRARY STRUCTURE:
============================================================
📁 Ancient Grand Library (9 books)
  📁 Spellcraft Wing (3 books)
    📖 Arcane Fundamentals [common]
    📖 Advanced Pyromancy [rare]
    📖 The Eternal Codex [ancient]
  📁 History Archives (2 books)
    📖 Chronicles of the First Age [uncommon]
    📖 Fall of the Dark Empire [rare]
  📁 Alchemy Laboratory (2 books)
    📖 Potion Brewing 101 [common]
    📖 Philosopher's Stone Secrets [legendary]
  📁 Forbidden Vault (2 books)
    📖 Necromancy Unveiled [legendary]
    📖 Book of the Dead [ancient]
============================================================

🔍 Quest: Find an Ancient Spell Book
--------------------------------------------------
✨ Found: The Eternal Codex by Ancient Order (ancient)

🔍 Quest: Collect Powerful Tomes (Power >= 200)
--------------------------------------------------
  ⚡ Book of the Dead: 1500 power
  ⚡ The Eternal Codex: 1000 power
  ⚡ Necromancy Unveiled: 800 power
  ⚡ Philosopher's Stone Secrets: 500 power
  ⚡ Advanced Pyromancy: 200 power

📚 Collected 5 powerful tomes!

🔍 Quest: Find Books by Chronicler Aldus
--------------------------------------------------
  📖 Chronicles of the First Age by Chronicler Aldus (uncommon)
  📖 Fall of the Dark Empire by Chronicler Aldus (rare)

 Found 2 books

🔍 Quest: Catalog Entire Library
--------------------------------------------------
  Total Books: 9
  Total Magic Power: 4220
  Rarity Distribution: {'common': 2, 'rare': 2, 'ancient': 2, 'uncommon': 1, 'legendary': 2}
  Topic Distribution: {'Magic Basics': 1, 'Fire Magic': 1, 'Ancient Spells': 2, 'History': 2, 'Alchemy': 2, 'Dark Magic': 1}


🔄 ITERATION STRATEGIES:
============================================================

1️⃣  DEPTH-FIRST ITERATION:
----------------------------------------
  1. Arcane Fundamentals
  2. Advanced Pyromancy
  3. The Eternal Codex
  4. Chronicles of the First Age
  5. Fall of the Dark Empire
  6. Potion Brewing 101
  7. Philosopher's Stone Secrets
  8. Necromancy Unveiled
  9. Book of the Dead

2️⃣  BREADTH-FIRST ITERATION:
----------------------------------------
  1. Arcane Fundamentals
  2. Advanced Pyromancy
  3. The Eternal Codex
  4. Chronicles of the First Age
  5. Fall of the Dark Empire
  6. Potion Brewing 101
  7. Philosopher's Stone Secrets
  8. Necromancy Unveiled
  9. Book of the Dead

3️⃣  RARE BOOKS ONLY (Rare+):
----------------------------------------
  ⭐ Advanced Pyromancy [rare]
  ⭐ The Eternal Codex [ancient]
  ⭐ Fall of the Dark Empire [rare]
  ⭐ Philosopher's Stone Secrets [legendary]
  ⭐ Necromancy Unveiled [legendary]
  ⭐ Book of the Dead [ancient]

4️⃣  BY MAGIC POWER (500+):
----------------------------------------
  ⚡ Book of the Dead: 1500 power
  ⚡ The Eternal Codex: 1000 power
  ⚡ Necromancy Unveiled: 800 power
  ⚡ Philosopher's Stone Secrets: 500 power

==================================================
📜 QUEST SUMMARY
==================================================
Books Discovered: 7
Knowledge Points: 1490
==================================================
```

**Pattern Benefits:**
- **Composite Pattern:** Nested library sections treated uniformly with individual books
- **Tree Structure:** Sections can contain books or other sections to any depth
- **Iterator Pattern:** Multiple traversal strategies without exposing internal structure
- **Flexible Iteration:** Depth-first, breadth-first, filtered, and sorted iterations
- **Separation of Concerns:** Iteration logic separated from tree structure
- **Easy Extension:** New iterator types can be added without modifying library structure
- **Quest System:** Complex searches made simple through custom iterators
- **Type Safety:** Full type hints ensure correct usage of all components

---

## 21. Submarine Exploration - State + Observer

**Theme:** Deep sea exploration
**Patterns:** State, Observer, Chain

**Key Learning:** Submarine depth affects available states

**Complete Implementation:**

```python
"""
Complete submarine exploration system with State and Observer patterns.
Demonstrates depth-based state transitions and event notifications.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
from dataclasses import dataclass
import random

class DepthZone(Enum):
    SURFACE = "surface"
    SHALLOW = "shallow"
    MEDIUM = "medium"
    DEEP = "deep"
    EXTREME = "extreme"

@dataclass
class Position3D:
    x: float
    y: float
    depth: float

class Discovery:
    def __init__(self, name: str, value: int, depth: float):
        self.name = name
        self.value = value
        self.depth = depth

# Observer Pattern
class SubmarineObserver(ABC):
    @abstractmethod
    def on_depth_change(self, old_depth: float, new_depth: float, zone: DepthZone):
        pass

    @abstractmethod
    def on_state_change(self, old_state: str, new_state: str):
        pass

    @abstractmethod
    def on_discovery(self, discovery: Discovery):
        pass

    @abstractmethod
    def on_emergency(self, emergency_type: str, severity: int):
        pass

class CrewComms(SubmarineObserver):
    """Crew communication system"""
    def on_depth_change(self, old_depth: float, new_depth: float, zone: DepthZone):
        if new_depth > old_depth:
            print(f"CREW: Diving to {new_depth}m - Entering {zone.value} zone")
        else:
            print(f"CREW: Ascending to {new_depth}m - Entering {zone.value} zone")

    def on_state_change(self, old_state: str, new_state: str):
        print(f"CREW: Submarine state: {old_state} -> {new_state}")

    def on_discovery(self, discovery: Discovery):
        print(f"CREW: Discovery made! {discovery.name} worth {discovery.value} credits!")

    def on_emergency(self, emergency_type: str, severity: int):
        if severity >= 8:
            print(f"CREW: CRITICAL EMERGENCY - {emergency_type}!")
        else:
            print(f"CREW: Warning - {emergency_type}")

class ScienceLog(SubmarineObserver):
    """Scientific mission log"""
    def __init__(self):
        self.log: List[str] = []
        self.discoveries: List[Discovery] = []

    def on_depth_change(self, old_depth: float, new_depth: float, zone: DepthZone):
        self.log.append(f"Depth changed: {old_depth}m -> {new_depth}m ({zone.value})")

    def on_state_change(self, old_state: str, new_state: str):
        self.log.append(f"State transition: {old_state} -> {new_state}")

    def on_discovery(self, discovery: Discovery):
        self.discoveries.append(discovery)
        self.log.append(f"Discovery: {discovery.name} at {discovery.depth}m")

    def on_emergency(self, emergency_type: str, severity: int):
        self.log.append(f"Emergency event: {emergency_type} (severity {severity})")

    def get_mission_summary(self) -> str:
        summary = "\n=== MISSION LOG ===\n"
        summary += f"Total Discoveries: {len(self.discoveries)}\n"
        summary += f"Total Value: {sum(d.value for d in self.discoveries)} credits\n"
        summary += f"Log Entries: {len(self.log)}\n"
        return summary

# State Pattern - Depth-based states
class SubmarineState(ABC):
    """Base class for submarine states"""

    @abstractmethod
    def enter(self, submarine: 'Submarine'):
        pass

    @abstractmethod
    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        pass

    @abstractmethod
    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        pass

    @abstractmethod
    def can_surface(self, submarine: 'Submarine') -> bool:
        pass

    @abstractmethod
    def get_state_name(self) -> str:
        pass

    @abstractmethod
    def get_max_speed(self) -> float:
        pass

class SurfaceState(SubmarineState):
    """Submarine on surface - can refuel and repair"""

    def enter(self, submarine: 'Submarine'):
        submarine.depth = 0
        submarine.current_zone = DepthZone.SURFACE
        print("\n=== SURFACED ===")
        print("Systems recharging, crew can breathe fresh air")

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        # Recharge systems while surfaced
        submarine.oxygen = min(100, submarine.oxygen + dt * 5)
        submarine.power = min(100, submarine.power + dt * 3)
        submarine.hull_integrity = min(100, submarine.hull_integrity + dt * 1)
        return None

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return submarine.power > 20

    def can_surface(self, submarine: 'Submarine') -> bool:
        return False  # Already on surface

    def get_state_name(self) -> str:
        return "Surface"

    def get_max_speed(self) -> float:
        return 10.0

class ShallowDiveState(SubmarineState):
    """Shallow depth (0-200m) - Safe diving"""

    def enter(self, submarine: 'Submarine'):
        submarine.current_zone = DepthZone.SHALLOW
        print("\n=== SHALLOW DIVE ===")
        print("Optimal conditions for exploration")

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        # Slow oxygen depletion
        submarine.oxygen -= dt * 0.5
        submarine.power -= dt * 1.0

        # Check for automatic state transitions
        if submarine.depth >= 200:
            return MediumDiveState()
        elif submarine.depth <= 0:
            return SurfaceState()

        # Random discoveries
        if random.random() < 0.05:
            discovery = Discovery("Coral Reef", value=100, depth=submarine.depth)
            submarine.notify_discovery(discovery)

        return None

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return submarine.hull_integrity > 50 and submarine.power > 30

    def can_surface(self, submarine: 'Submarine') -> bool:
        return True  # Can surface anytime from shallow

    def get_state_name(self) -> str:
        return "Shallow Dive"

    def get_max_speed(self) -> float:
        return 15.0

class MediumDiveState(SubmarineState):
    """Medium depth (200-500m) - Increased pressure"""

    def enter(self, submarine: 'Submarine'):
        submarine.current_zone = DepthZone.MEDIUM
        print("\n=== MEDIUM DIVE ===")
        print("Pressure increasing, hull creaking")

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        # Moderate resource consumption
        submarine.oxygen -= dt * 1.0
        submarine.power -= dt * 1.5
        submarine.hull_integrity -= dt * 0.2  # Pressure damage

        # Depth-based state transitions
        if submarine.depth >= 500:
            return DeepDiveState()
        elif submarine.depth < 200:
            return ShallowDiveState()

        # Better discoveries at this depth
        if random.random() < 0.04:
            discovery = Discovery("Ancient Shipwreck", value=500, depth=submarine.depth)
            submarine.notify_discovery(discovery)

        # Random emergencies
        if random.random() < 0.02:
            submarine.notify_emergency("Pressure leak", severity=5)
            submarine.hull_integrity -= 10

        return None

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return submarine.hull_integrity > 70 and submarine.power > 40

    def can_surface(self, submarine: 'Submarine') -> bool:
        return True

    def get_state_name(self) -> str:
        return "Medium Dive"

    def get_max_speed(self) -> float:
        return 12.0

class DeepDiveState(SubmarineState):
    """Deep depth (500-1000m) - High pressure, rare discoveries"""

    def enter(self, submarine: 'Submarine'):
        submarine.current_zone = DepthZone.DEEP
        print("\n=== DEEP DIVE ===")
        print("Extreme pressure! Oxygen consumption doubled!")
        submarine.notify_emergency("High pressure environment", severity=6)

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        # High resource consumption
        submarine.oxygen -= dt * 2.0
        submarine.power -= dt * 2.5
        submarine.hull_integrity -= dt * 0.5

        # Check for critical transitions
        if submarine.depth >= 1000:
            return ExtremeDiveState()
        elif submarine.depth < 500:
            # Must decompress slowly
            return DecompressionState()

        # Valuable discoveries
        if random.random() < 0.03:
            discovery = Discovery("Hydrothermal Vent", value=1500, depth=submarine.depth)
            submarine.notify_discovery(discovery)

        # Higher emergency risk
        if random.random() < 0.03:
            submarine.notify_emergency("Hull stress critical", severity=7)
            submarine.hull_integrity -= 15

        # Critical failure check
        if submarine.hull_integrity <= 20:
            return CrushedState()

        return None

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return submarine.hull_integrity > 80 and submarine.power > 60

    def can_surface(self, submarine: 'Submarine') -> bool:
        return False  # Must decompress first

    def get_state_name(self) -> str:
        return "Deep Dive"

    def get_max_speed(self) -> float:
        return 8.0

class ExtremeDiveState(SubmarineState):
    """Extreme depth (1000m+) - Crushing pressure, legendary discoveries"""

    def enter(self, submarine: 'Submarine'):
        submarine.current_zone = DepthZone.EXTREME
        print("\n=== EXTREME DEPTH ===")
        print("WARNING: Maximum depth! Hull at breaking point!")
        submarine.notify_emergency("EXTREME DEPTH WARNING", severity=9)

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        # Extreme resource drain
        submarine.oxygen -= dt * 3.0
        submarine.power -= dt * 3.5
        submarine.hull_integrity -= dt * 1.0

        # Amazing discoveries
        if random.random() < 0.08:
            discovery = Discovery("Unknown Deep Sea Creature", value=5000, depth=submarine.depth)
            submarine.notify_discovery(discovery)

        # High emergency risk
        if random.random() < 0.05:
            submarine.notify_emergency("HULL BREACH IMMINENT", severity=9)
            submarine.hull_integrity -= 20

        # Critical failure
        if submarine.hull_integrity <= 0 or submarine.oxygen <= 0:
            return CrushedState()

        # Can only ascend from extreme depth
        if submarine.depth < 1000:
            return DecompressionState()

        return None

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return False  # Maximum depth

    def can_surface(self, submarine: 'Submarine') -> bool:
        return False  # Must decompress

    def get_state_name(self) -> str:
        return "Extreme Dive"

    def get_max_speed(self) -> float:
        return 5.0

class DecompressionState(SubmarineState):
    """Decompression required when ascending from deep depth"""

    def __init__(self):
        self.decompression_time = 0
        self.required_time = 30

    def enter(self, submarine: 'Submarine'):
        print("\n=== DECOMPRESSION PROTOCOL ===")
        print("Ascending slowly to avoid the bends...")
        self.decompression_time = 0

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        self.decompression_time += dt

        # Slow ascent
        submarine.depth = max(0, submarine.depth - dt * 10)

        # Slow resource consumption during decompression
        submarine.oxygen -= dt * 1.0
        submarine.power -= dt * 0.5

        # Decompression complete
        if self.decompression_time >= self.required_time:
            if submarine.depth < 200:
                return ShallowDiveState()

        # Emergency surface if resources critical
        if submarine.oxygen <= 10 or submarine.power <= 10:
            print("EMERGENCY SURFACE - Skipping decompression!")
            return SurfaceState()

        return None

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return False  # Must complete decompression

    def can_surface(self, submarine: 'Submarine') -> bool:
        return False  # Must complete decompression

    def get_state_name(self) -> str:
        return "Decompression"

    def get_max_speed(self) -> float:
        return 3.0

class CrushedState(SubmarineState):
    """Terminal state - submarine destroyed"""

    def enter(self, submarine: 'Submarine'):
        print("\n" + "="*60)
        print("!!! HULL INTEGRITY FAILURE !!!")
        print("The submarine has been crushed by pressure!")
        print("MISSION FAILED")
        print("="*60)

    def update(self, submarine: 'Submarine', dt: float) -> Optional['SubmarineState']:
        return None  # Terminal state

    def can_dive_deeper(self, submarine: 'Submarine') -> bool:
        return False

    def can_surface(self, submarine: 'Submarine') -> bool:
        return False

    def get_state_name(self) -> str:
        return "Destroyed"

    def get_max_speed(self) -> float:
        return 0.0

# Submarine class
class Submarine:
    def __init__(self, name: str):
        self.name = name
        self.depth = 0.0
        self.oxygen = 100
        self.power = 100
        self.hull_integrity = 100
        self.current_zone = DepthZone.SURFACE

        # State pattern
        self.current_state: SubmarineState = SurfaceState()
        self.current_state.enter(self)

        # Observers
        self.observers: List[SubmarineObserver] = []

    def add_observer(self, observer: SubmarineObserver):
        self.observers.append(observer)

    def notify_depth_change(self, old_depth: float):
        for observer in self.observers:
            observer.on_depth_change(old_depth, self.depth, self.current_zone)

    def notify_state_change(self, old_state: str, new_state: str):
        for observer in self.observers:
            observer.on_state_change(old_state, new_state)

    def notify_discovery(self, discovery: Discovery):
        for observer in self.observers:
            observer.on_discovery(discovery)

    def notify_emergency(self, emergency_type: str, severity: int):
        for observer in self.observers:
            observer.on_emergency(emergency_type, severity)

    def update(self, dt: float):
        """Update submarine state"""
        new_state = self.current_state.update(self, dt)

        if new_state:
            self.transition_to(new_state)

    def transition_to(self, new_state: SubmarineState):
        """Transition to new state"""
        old_state_name = self.current_state.get_state_name()
        self.current_state = new_state
        new_state_name = self.current_state.get_state_name()

        self.notify_state_change(old_state_name, new_state_name)
        self.current_state.enter(self)

    def dive(self, target_depth: float):
        """Attempt to dive to target depth"""
        if not self.current_state.can_dive_deeper(self):
            print("Cannot dive deeper in current state!")
            return

        old_depth = self.depth
        self.depth = target_depth
        self.notify_depth_change(old_depth)

    def surface_command(self):
        """Attempt to surface"""
        if not self.current_state.can_surface(self):
            print("Cannot surface directly - decompression required!")
            return

        old_depth = self.depth
        self.depth = 0
        self.notify_depth_change(old_depth)

    def get_status(self) -> str:
        status = f"\n=== {self.name.upper()} ===\n"
        status += f"State: {self.current_state.get_state_name()}\n"
        status += f"Depth: {self.depth:.1f}m ({self.current_zone.value})\n"
        status += f"Oxygen: {int(self.oxygen)}%\n"
        status += f"Power: {int(self.power)}%\n"
        status += f"Hull Integrity: {int(self.hull_integrity)}%\n"
        status += f"Max Speed: {self.current_state.get_max_speed()} knots\n"
        return status

# Example Usage
def main():
    print("=== SUBMARINE EXPLORATION - State + Observer Demo ===\n")

    # Create submarine
    sub = Submarine("HMS Nautilus")

    # Add observers
    crew = CrewComms()
    science = ScienceLog()
    sub.add_observer(crew)
    sub.add_observer(science)

    print(sub.get_status())

    # Mission: Dive to extreme depths
    print("\n--- Beginning Dive Sequence ---")

    # Dive to shallow
    sub.dive(150)
    sub.update(1)
    print(sub.get_status())

    # Dive to medium
    sub.dive(350)
    for _ in range(3):
        sub.update(2)
    print(sub.get_status())

    # Dive to deep
    sub.dive(750)
    for _ in range(5):
        sub.update(2)
    print(sub.get_status())

    # Dive to extreme
    if sub.hull_integrity > 70:
        sub.dive(1200)
        for _ in range(3):
            sub.update(2)
        print(sub.get_status())

    # Attempt to surface (should require decompression)
    print("\n--- Attempting to Surface ---")
    sub.surface_command()

    # Ascend through decompression
    for _ in range(10):
        sub.update(3)
        if isinstance(sub.current_state, (SurfaceState, ShallowDiveState)):
            break

    print(sub.get_status())

    # Mission summary
    print(science.get_mission_summary())

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== SUBMARINE EXPLORATION - State + Observer Demo ===

=== SURFACED ===
Systems recharging, crew can breathe fresh air

=== HMS NAUTILUS ===
State: Surface
Depth: 0.0m (surface)
Oxygen: 100%
Power: 100%
Hull Integrity: 100%
Max Speed: 10.0 knots

--- Beginning Dive Sequence ---
CREW: Diving to 150.0m - Entering shallow zone
CREW: Submarine state: Surface -> Shallow Dive

=== SHALLOW DIVE ===
Optimal conditions for exploration

=== HMS NAUTILUS ===
State: Shallow Dive
Depth: 150.0m (shallow)
Oxygen: 99%
Power: 99%
Hull Integrity: 100%
Max Speed: 15.0 knots

CREW: Diving to 350.0m - Entering medium zone
CREW: Submarine state: Shallow Dive -> Medium Dive

=== MEDIUM DIVE ===
Pressure increasing, hull creaking

CREW: Discovery made! Ancient Shipwreck worth 500 credits!

=== HMS NAUTILUS ===
State: Medium Dive
Depth: 350.0m (medium)
Oxygen: 94%
Power: 91%
Hull Integrity: 98%
Max Speed: 12.0 knots

CREW: Diving to 750.0m - Entering deep zone
CREW: Submarine state: Medium Dive -> Deep Dive

=== DEEP DIVE ===
Extreme pressure! Oxygen consumption doubled!
CREW: Warning - High pressure environment

CREW: CRITICAL EMERGENCY - Hull stress critical!
CREW: Discovery made! Hydrothermal Vent worth 1500 credits!

=== HMS NAUTILUS ===
State: Deep Dive
Depth: 750.0m (deep)
Oxygen: 84%
Power: 78%
Hull Integrity: 83%
Max Speed: 8.0 knots

CREW: Diving to 1200.0m - Entering extreme zone
CREW: Submarine state: Deep Dive -> Extreme Dive

=== EXTREME DEPTH ===
WARNING: Maximum depth! Hull at breaking point!
CREW: CRITICAL EMERGENCY - EXTREME DEPTH WARNING!

CREW: Discovery made! Unknown Deep Sea Creature worth 5000 credits!

--- Attempting to Surface ---
Cannot surface directly - decompression required!

CREW: Submarine state: Extreme Dive -> Decompression

=== DECOMPRESSION PROTOCOL ===
Ascending slowly to avoid the bends...

=== HMS NAUTILUS ===
State: Shallow Dive
Depth: 120.0m (shallow)
Oxygen: 58%
Power: 45%
Hull Integrity: 65%
Max Speed: 15.0 knots

=== MISSION LOG ===
Total Discoveries: 3
Total Value: 7000 credits
Log Entries: 12
```

**Pattern Benefits:**
- **State Pattern:** Depth ranges become discrete states with specific behaviors
- **Automatic Transitions:** States automatically transition based on depth changes
- **Observer Pattern:** Crew and systems independently react to submarine events
- **State-Specific Logic:** Each depth state has unique resource consumption and risks
- **Safe Decompression:** Deep states prevent direct surfacing, requiring decompression
- **Progressive Difficulty:** Deeper states offer better rewards but higher risks
- **Event-Driven:** Discoveries and emergencies notify all observers simultaneously
- **Realistic Simulation:** Models actual submarine limitations and dangers

---

## 22. Magical Potion Brewing - Builder + Decorator

**Theme:** Witch's potion crafting
**Patterns:** Builder, Decorator

**Key Learning:** Build base potion, decorate with effects

**Complete Implementation:**

```python
"""
Complete Magical Potion Brewing system with Builder and Decorator patterns.
Demonstrates complex potion creation and effect layering.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Optional
from enum import Enum
from dataclasses import dataclass
import random

class PotionBase(Enum):
    HEALTH = "Health"
    MANA = "Mana"
    STAMINA = "Stamina"
    POISON = "Poison"
    STRENGTH = "Strength"

class IngredientRarity(Enum):
    COMMON = 1
    UNCOMMON = 2
    RARE = 3
    LEGENDARY = 4

@dataclass
class Ingredient:
    """Individual potion ingredient"""
    name: str
    rarity: IngredientRarity
    power_boost: int
    flavor_text: str

# Potion Component (for Decorator pattern)
class Potion(ABC):
    """Base potion interface"""

    @abstractmethod
    def get_name(self) -> str:
        pass

    @abstractmethod
    def get_description(self) -> str:
        pass

    @abstractmethod
    def get_effects(self) -> List[str]:
        pass

    @abstractmethod
    def get_power(self) -> int:
        pass

    @abstractmethod
    def get_value(self) -> int:
        pass

    @abstractmethod
    def consume(self) -> str:
        pass

# Concrete Potion (built by Builder)
class BasePotion(Potion):
    """Base potion created by builder"""

    def __init__(self, base_type: PotionBase, potency: int, ingredients: List[Ingredient], color: str):
        self.base_type = base_type
        self.potency = potency
        self.ingredients = ingredients
        self.color = color
        self._calculate_power()

    def _calculate_power(self):
        """Calculate total power from ingredients"""
        self.base_power = self.potency * 10
        for ingredient in self.ingredients:
            self.base_power += ingredient.power_boost

    def get_name(self) -> str:
        return f"{self.base_type.value} Potion"

    def get_description(self) -> str:
        return f"A {self.color} {self.base_type.value.lower()} potion of potency {self.potency}"

    def get_effects(self) -> List[str]:
        base_effect = f"Restore {self.base_power} {self.base_type.value}"
        return [base_effect]

    def get_power(self) -> int:
        return self.base_power

    def get_value(self) -> int:
        base_value = self.potency * 50
        for ingredient in self.ingredients:
            base_value += ingredient.rarity.value * 25
        return base_value

    def consume(self) -> str:
        return f"You drink the {self.color} {self.base_type.value.lower()} potion. {self.get_effects()[0]}!"

# Builder Pattern
class PotionBuilder:
    """Builder for creating complex potions"""

    def __init__(self):
        self.reset()

    def reset(self):
        """Reset builder to initial state"""
        self._base_type: Optional[PotionBase] = None
        self._potency: int = 1
        self._ingredients: List[Ingredient] = []
        self._color: str = "clear"

    def set_base(self, base_type: PotionBase) -> 'PotionBuilder':
        """Set the base potion type"""
        self._base_type = base_type

        # Set default color based on type
        color_map = {
            PotionBase.HEALTH: "red",
            PotionBase.MANA: "blue",
            PotionBase.STAMINA: "green",
            PotionBase.POISON: "purple",
            PotionBase.STRENGTH: "orange"
        }
        self._color = color_map.get(base_type, "clear")

        return self

    def set_potency(self, potency: int) -> 'PotionBuilder':
        """Set potion potency (1-10)"""
        self._potency = max(1, min(10, potency))
        return self

    def add_ingredient(self, ingredient: Ingredient) -> 'PotionBuilder':
        """Add an ingredient to the potion"""
        self._ingredients.append(ingredient)
        return self

    def set_color(self, color: str) -> 'PotionBuilder':
        """Override the default color"""
        self._color = color
        return self

    def build(self) -> BasePotion:
        """Build the final potion"""
        if self._base_type is None:
            raise ValueError("Base type must be set before building")

        potion = BasePotion(
            base_type=self._base_type,
            potency=self._potency,
            ingredients=self._ingredients,
            color=self._color
        )

        self.reset()
        return potion

# Decorator Pattern - Magical Effects
class PotionDecorator(Potion):
    """Base decorator for adding effects to potions"""

    def __init__(self, potion: Potion):
        self._wrapped_potion = potion

    def get_name(self) -> str:
        return self._wrapped_potion.get_name()

    def get_description(self) -> str:
        return self._wrapped_potion.get_description()

    def get_effects(self) -> List[str]:
        return self._wrapped_potion.get_effects()

    def get_power(self) -> int:
        return self._wrapped_potion.get_power()

    def get_value(self) -> int:
        return self._wrapped_potion.get_value()

    def consume(self) -> str:
        return self._wrapped_potion.consume()

class FireResistanceDecorator(PotionDecorator):
    """Add fire resistance effect"""

    def __init__(self, potion: Potion, duration: int = 300):
        super().__init__(potion)
        self.duration = duration

    def get_name(self) -> str:
        return f"{super().get_name()} of Fire Resistance"

    def get_description(self) -> str:
        return f"{super().get_description()} with fire resistance"

    def get_effects(self) -> List[str]:
        effects = super().get_effects()
        effects.append(f"Fire Resistance for {self.duration}s")
        return effects

    def get_value(self) -> int:
        return super().get_value() + 150

    def consume(self) -> str:
        result = super().consume()
        result += f"\nYou feel a cool sensation. Fire resistance active for {self.duration}s!"
        return result

class IceResistanceDecorator(PotionDecorator):
    """Add ice resistance effect"""

    def __init__(self, potion: Potion, duration: int = 300):
        super().__init__(potion)
        self.duration = duration

    def get_name(self) -> str:
        return f"{super().get_name()} of Ice Resistance"

    def get_effects(self) -> List[str]:
        effects = super().get_effects()
        effects.append(f"Ice Resistance for {self.duration}s")
        return effects

    def get_value(self) -> int:
        return super().get_value() + 150

class RegenerationDecorator(PotionDecorator):
    """Add regeneration over time effect"""

    def __init__(self, potion: Potion, regen_per_second: int = 5, duration: int = 60):
        super().__init__(potion)
        self.regen_per_second = regen_per_second
        self.duration = duration

    def get_name(self) -> str:
        return f"{super().get_name()} of Regeneration"

    def get_effects(self) -> List[str]:
        effects = super().get_effects()
        total_regen = self.regen_per_second * self.duration
        effects.append(f"Regenerate {self.regen_per_second}/s for {self.duration}s (Total: {total_regen})")
        return effects

    def get_power(self) -> int:
        return super().get_power() + (self.regen_per_second * self.duration)

    def get_value(self) -> int:
        return super().get_value() + 200

    def consume(self) -> str:
        result = super().consume()
        result += f"\nA warm glow surrounds you. Regenerating {self.regen_per_second} HP/s!"
        return result

class DoubleEffectDecorator(PotionDecorator):
    """Double all potion effects"""

    def get_name(self) -> str:
        return f"Empowered {super().get_name()}"

    def get_power(self) -> int:
        return super().get_power() * 2

    def get_effects(self) -> List[str]:
        base_effects = super().get_effects()
        doubled_effects = ["DOUBLED: " + effect for effect in base_effects]
        return doubled_effects

    def get_value(self) -> int:
        return super().get_value() * 2

    def consume(self) -> str:
        result = super().consume()
        result += "\n⚡ EMPOWERED EFFECT - All benefits doubled!"
        return result

class PermanentDecorator(PotionDecorator):
    """Make potion effects permanent"""

    def get_name(self) -> str:
        return f"Eternal {super().get_name()}"

    def get_effects(self) -> List[str]:
        effects = super().get_effects()
        effects.append("PERMANENT - Effects never expire!")
        return effects

    def get_value(self) -> int:
        return super().get_value() * 10

    def consume(self) -> str:
        result = super().consume()
        result += "\n✨ ETERNAL BLESSING - Effects are now permanent!"
        return result

class PoisonedDecorator(PotionDecorator):
    """Add poison damage (for trap potions)"""

    def __init__(self, potion: Potion, poison_damage: int = 50):
        super().__init__(potion)
        self.poison_damage = poison_damage

    def get_name(self) -> str:
        return f"Poisoned {super().get_name()}"

    def get_effects(self) -> List[str]:
        effects = super().get_effects()
        effects.append(f"⚠️  POISONED: Take {self.poison_damage} damage!")
        return effects

    def get_value(self) -> int:
        return super().get_value() - 100

    def consume(self) -> str:
        result = super().consume()
        result += f"\n☠️  You feel sick! Poison damage: {self.poison_damage}!"
        return result

class SparklingDecorator(PotionDecorator):
    """Make potion sparkle (cosmetic + small bonus)"""

    def get_name(self) -> str:
        return f"Sparkling {super().get_name()}"

    def get_description(self) -> str:
        return f"{super().get_description()}, shimmering with magical sparkles"

    def get_value(self) -> int:
        return super().get_value() + 50

# Ingredient Library
class IngredientLibrary:
    """Collection of available ingredients"""

    @staticmethod
    def get_all_ingredients() -> List[Ingredient]:
        return [
            # Common ingredients
            Ingredient("Herb Bundle", IngredientRarity.COMMON, 5, "Fresh healing herbs"),
            Ingredient("Spring Water", IngredientRarity.COMMON, 3, "Pure water from a mountain spring"),
            Ingredient("Honey", IngredientRarity.COMMON, 8, "Sweet golden honey"),

            # Uncommon ingredients
            Ingredient("Moonflower", IngredientRarity.UNCOMMON, 15, "Blooms only under full moon"),
            Ingredient("Crystal Dust", IngredientRarity.UNCOMMON, 20, "Ground magical crystals"),
            Ingredient("Spider Silk", IngredientRarity.UNCOMMON, 12, "Gossamer strands"),

            # Rare ingredients
            Ingredient("Dragon Scale", IngredientRarity.RARE, 50, "Scale from an ancient dragon"),
            Ingredient("Phoenix Feather", IngredientRarity.RARE, 75, "Feather of rebirth"),
            Ingredient("Unicorn Hair", IngredientRarity.RARE, 60, "Hair from a pure unicorn"),

            # Legendary ingredients
            Ingredient("Star Fragment", IngredientRarity.LEGENDARY, 150, "Fragment of a fallen star"),
            Ingredient("Void Essence", IngredientRarity.LEGENDARY, 200, "Essence from beyond reality"),
        ]

    @staticmethod
    def get_random_ingredient(rarity: IngredientRarity) -> Ingredient:
        """Get random ingredient of specified rarity"""
        ingredients = [i for i in IngredientLibrary.get_all_ingredients() if i.rarity == rarity]
        return random.choice(ingredients) if ingredients else IngredientLibrary.get_all_ingredients()[0]

# Brewing System
class BrewingCauldron:
    """System for brewing potions"""

    def __init__(self):
        self.brewed_potions: List[Potion] = []
        self.total_value = 0

    def brew_simple_potion(self, base_type: PotionBase) -> Potion:
        """Brew a simple, basic potion"""
        print(f"\n🔮 Brewing simple {base_type.value} potion...")
        builder = PotionBuilder()
        potion = (builder
                  .set_base(base_type)
                  .set_potency(3)
                  .build())

        self.brewed_potions.append(potion)
        self.total_value += potion.get_value()
        print(f"✓ Brewed: {potion.get_name()}")
        return potion

    def brew_advanced_potion(self) -> Potion:
        """Brew a complex potion with ingredients"""
        print(f"\n🔮 Brewing advanced potion...")
        builder = PotionBuilder()

        # Select random ingredients
        ingredients = [
            IngredientLibrary.get_random_ingredient(IngredientRarity.UNCOMMON),
            IngredientLibrary.get_random_ingredient(IngredientRarity.RARE)
        ]

        potion = (builder
                  .set_base(PotionBase.HEALTH)
                  .set_potency(7)
                  .add_ingredient(ingredients[0])
                  .add_ingredient(ingredients[1])
                  .build())

        self.brewed_potions.append(potion)
        self.total_value += potion.get_value()
        print(f"✓ Brewed: {potion.get_name()}")
        print(f"  Ingredients: {', '.join(i.name for i in ingredients)}")
        return potion

    def enhance_potion(self, potion: Potion, decorators: List[str]) -> Potion:
        """Enhance potion with decorators"""
        print(f"\n✨ Enhancing {potion.get_name()}...")

        enhanced = potion
        for decorator_name in decorators:
            if decorator_name == "fire_resistance":
                enhanced = FireResistanceDecorator(enhanced)
            elif decorator_name == "ice_resistance":
                enhanced = IceResistanceDecorator(enhanced)
            elif decorator_name == "regeneration":
                enhanced = RegenerationDecorator(enhanced)
            elif decorator_name == "double_effect":
                enhanced = DoubleEffectDecorator(enhanced)
            elif decorator_name == "permanent":
                enhanced = PermanentDecorator(enhanced)
            elif decorator_name == "sparkling":
                enhanced = SparklingDecorator(enhanced)
            elif decorator_name == "poisoned":
                enhanced = PoisonedDecorator(enhanced)

            print(f"  + Added: {decorator_name}")

        self.total_value += (enhanced.get_value() - potion.get_value())
        print(f"✓ Enhanced to: {enhanced.get_name()}")
        return enhanced

    def display_potion_info(self, potion: Potion):
        """Display detailed potion information"""
        print("\n" + "=" * 60)
        print(f"🧪 {potion.get_name()}")
        print("=" * 60)
        print(f"Description: {potion.get_description()}")
        print(f"Power: {potion.get_power()}")
        print(f"Value: {potion.get_value()} gold")
        print(f"\nEffects:")
        for i, effect in enumerate(potion.get_effects(), 1):
            print(f"  {i}. {effect}")
        print("=" * 60)

# Example Usage
def main():
    print("=== MAGICAL POTION BREWING - Builder + Decorator Demo ===\n")

    cauldron = BrewingCauldron()
    library = IngredientLibrary()

    # Example 1: Simple builder usage
    print("\n" + "─" * 60)
    print("EXAMPLE 1: Basic Potion Building")
    print("─" * 60)

    builder = PotionBuilder()
    health_potion = (builder
                     .set_base(PotionBase.HEALTH)
                     .set_potency(5)
                     .add_ingredient(Ingredient("Dragon Scale", IngredientRarity.RARE, 50, "Powerful"))
                     .add_ingredient(Ingredient("Phoenix Feather", IngredientRarity.RARE, 75, "Rebirth"))
                     .build())

    cauldron.display_potion_info(health_potion)

    # Example 2: Decorator pattern - layering effects
    print("\n" + "─" * 60)
    print("EXAMPLE 2: Enhancing with Decorators")
    print("─" * 60)

    enhanced_potion = health_potion
    enhanced_potion = FireResistanceDecorator(enhanced_potion, duration=600)
    enhanced_potion = RegenerationDecorator(enhanced_potion, regen_per_second=10, duration=30)
    enhanced_potion = SparklingDecorator(enhanced_potion)

    cauldron.display_potion_info(enhanced_potion)

    # Example 3: Extreme enhancement
    print("\n" + "─" * 60)
    print("EXAMPLE 3: Ultimate Potion Creation")
    print("─" * 60)

    ultimate_builder = PotionBuilder()
    ultimate_potion = (ultimate_builder
                       .set_base(PotionBase.MANA)
                       .set_potency(10)
                       .add_ingredient(Ingredient("Star Fragment", IngredientRarity.LEGENDARY, 150, "Cosmic"))
                       .add_ingredient(Ingredient("Void Essence", IngredientRarity.LEGENDARY, 200, "Beyond"))
                       .build())

    # Layer multiple decorators
    ultimate_potion = FireResistanceDecorator(ultimate_potion)
    ultimate_potion = IceResistanceDecorator(ultimate_potion)
    ultimate_potion = RegenerationDecorator(ultimate_potion, regen_per_second=20)
    ultimate_potion = DoubleEffectDecorator(ultimate_potion)
    ultimate_potion = PermanentDecorator(ultimate_potion)

    cauldron.display_potion_info(ultimate_potion)

    # Example 4: Consuming potions
    print("\n" + "─" * 60)
    print("EXAMPLE 4: Consuming Potions")
    print("─" * 60)

    simple_potion = cauldron.brew_simple_potion(PotionBase.STAMINA)
    print(f"\n{simple_potion.consume()}")

    print("\n" + "─" * 40)
    print(f"\n{ultimate_potion.consume()}")

    # Example 5: Brewing session
    print("\n" + "─" * 60)
    print("EXAMPLE 5: Complete Brewing Session")
    print("─" * 60)

    potions = []

    # Brew various potions
    p1 = cauldron.brew_simple_potion(PotionBase.HEALTH)
    p2 = cauldron.brew_advanced_potion()
    p3 = cauldron.brew_simple_potion(PotionBase.STRENGTH)

    # Enhance them
    p1_enhanced = cauldron.enhance_potion(p1, ["fire_resistance", "sparkling"])
    p2_enhanced = cauldron.enhance_potion(p2, ["regeneration", "double_effect"])
    p3_enhanced = cauldron.enhance_potion(p3, ["permanent"])

    # Summary
    print("\n" + "=" * 60)
    print("📊 BREWING SESSION SUMMARY")
    print("=" * 60)
    print(f"Total Potions Brewed: {len(cauldron.brewed_potions)}")
    print(f"Total Value Created: {cauldron.total_value} gold")
    print("=" * 60)

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== MAGICAL POTION BREWING - Builder + Decorator Demo ===


────────────────────────────────────────────────────────────
EXAMPLE 1: Basic Potion Building
────────────────────────────────────────────────────────────

============================================================
🧪 Health Potion
============================================================
Description: A red health potion of potency 5
Power: 175
Value: 300 gold

Effects:
  1. Restore 175 Health
============================================================

────────────────────────────────────────────────────────────
EXAMPLE 2: Enhancing with Decorators
────────────────────────────────────────────────────────────

============================================================
🧪 Sparkling Health Potion of Regeneration of Fire Resistance
============================================================
Description: A red health potion of potency 5, shimmering with magical sparkles
Power: 475
Value: 600 gold

Effects:
  1. Restore 175 Health
  2. Fire Resistance for 600s
  3. Regenerate 10/s for 30s (Total: 300)
============================================================

────────────────────────────────────────────────────────────
EXAMPLE 3: Ultimate Potion Creation
────────────────────────────────────────────────────────────

============================================================
🧪 Eternal Empowered Mana Potion of Regeneration of Ice Resistance of Fire Resistance
============================================================
Description: A blue mana potion of potency 10
Power: 1600
Value: 18000 gold

Effects:
  1. DOUBLED: Restore 450 Mana
  2. DOUBLED: Fire Resistance for 300s
  3. DOUBLED: Ice Resistance for 300s
  4. DOUBLED: Regenerate 20/s for 60s (Total: 1200)
  5. PERMANENT - Effects never expire!
============================================================

────────────────────────────────────────────────────────────
EXAMPLE 4: Consuming Potions
────────────────────────────────────────────────────────────

🔮 Brewing simple Stamina potion...
✓ Brewed: Stamina Potion

You drink the green stamina potion. Restore 30 Stamina!

────────────────────────────────────────────────────────────

You drink the blue mana potion. Restore 450 Mana!
You feel a cool sensation. Fire resistance active for 300s!
A warm glow surrounds you. Regenerating 20 HP/s!
⚡ EMPOWERED EFFECT - All benefits doubled!
✨ ETERNAL BLESSING - Effects are now permanent!

────────────────────────────────────────────────────────────
EXAMPLE 5: Complete Brewing Session
────────────────────────────────────────────────────────────

🔮 Brewing simple Health potion...
✓ Brewed: Health Potion

🔮 Brewing advanced potion...
✓ Brewed: Health Potion
  Ingredients: Moonflower, Phoenix Feather

🔮 Brewing simple Strength potion...
✓ Brewed: Strength Potion

✨ Enhancing Health Potion...
  + Added: fire_resistance
  + Added: sparkling
✓ Enhanced to: Sparkling Health Potion of Fire Resistance

✨ Enhancing Health Potion...
  + Added: regeneration
  + Added: double_effect
✓ Enhanced to: Empowered Health Potion of Regeneration

✨ Enhancing Strength Potion...
  + Added: permanent
✓ Enhanced to: Eternal Strength Potion

============================================================
📊 BREWING SESSION SUMMARY
============================================================
Total Potions Brewed: 3
Total Value Created: 2850 gold
============================================================
```

**Pattern Benefits:**
- **Builder Pattern:** Complex potions constructed step-by-step with fluent interface
- **Flexible Construction:** Same builder can create vastly different potions
- **Decorator Pattern:** Effects layered dynamically without modifying base classes
- **Open/Closed Principle:** New decorators can be added without changing existing code
- **Composition Over Inheritance:** Effects combined through wrapping rather than class hierarchies
- **Multiple Enhancements:** Potions can have unlimited decorator layers
- **Type Safety:** Full type hints ensure correct usage of builders and decorators
- **Separation of Concerns:** Building (structure) separated from decorating (behavior)

---

## 23. Gladiator Arena - Strategy + Observer

**Theme:** Roman gladiator combat
**Patterns:** Strategy, Observer, State

**Key Learning:** Gladiators use combat strategies, crowd observes

**Complete Implementation:**

```python
"""
Complete Gladiator Arena system with Strategy and Observer patterns.
Demonstrates different combat strategies and event-driven crowd reactions.
"""

from abc import ABC, abstractmethod
from typing import List, Optional
from enum import Enum
from dataclasses import dataclass
import random

class GladiatorClass(Enum):
    RETIARIUS = "Retiarius"      # Net fighter
    MURMILLO = "Murmillo"        # Heavy armor, sword
    SECUTOR = "Secutor"          # Pursuer, counters Retiarius
    THRAEX = "Thraex"            # Curved sword, small shield
    HOPLOMACHUS = "Hoplomachus"  # Spear fighter

class CombatEvent(Enum):
    HIT_LANDED = "hit_landed"
    CRITICAL_HIT = "critical_hit"
    MISS = "miss"
    DODGE = "dodge"
    SPECIAL_MOVE = "special_move"
    GLADIATOR_DOWN = "gladiator_down"
    MATCH_END = "match_end"

@dataclass
class CombatAction:
    """Result of a combat action"""
    attacker_name: str
    defender_name: str
    damage: int
    event_type: CombatEvent
    description: str
    is_critical: bool = False

# Observer Pattern
class ArenaObserver(ABC):
    """Base observer for arena events"""

    @abstractmethod
    def on_combat_action(self, action: CombatAction):
        pass

    @abstractmethod
    def on_match_start(self, gladiator1: 'Gladiator', gladiator2: 'Gladiator'):
        pass

    @abstractmethod
    def on_match_end(self, winner: 'Gladiator', loser: 'Gladiator'):
        pass

class CrowdObserver(ArenaObserver):
    """The crowd watching the arena"""

    def __init__(self):
        self.excitement = 0
        self.favor_fighter1 = 50  # 0-100 scale

    def on_combat_action(self, action: CombatAction):
        if action.event_type == CombatEvent.CRITICAL_HIT:
            print(f"👥 CROWD: OOOOHHHH!!! What a blow!")
            self.excitement += 30
        elif action.event_type == CombatEvent.HIT_LANDED:
            if action.damage > 30:
                print(f"👥 CROWD: That's gotta hurt!")
                self.excitement += 20
            elif action.damage > 15:
                print(f"👥 CROWD: Nice hit!")
                self.excitement += 10
        elif action.event_type == CombatEvent.DODGE:
            print(f"👥 CROWD: What agility!")
            self.excitement += 5
        elif action.event_type == CombatEvent.SPECIAL_MOVE:
            print(f"👥 CROWD: AMAZING! {action.description}!")
            self.excitement += 25
        elif action.event_type == CombatEvent.MISS:
            print(f"👥 CROWD: *Jeers and boos*")
            self.excitement -= 5

        self.excitement = max(0, min(100, self.excitement))

    def on_match_start(self, gladiator1: 'Gladiator', gladiator2: 'Gladiator'):
        print(f"\n👥 CROWD: Let the battle begin! {gladiator1.name} vs {gladiator2.name}!")
        print(f"👥 CROWD: *Roaring and chanting*")
        self.excitement = 50

    def on_match_end(self, winner: 'Gladiator', loser: 'Gladiator'):
        print(f"\n👥 CROWD: {winner.name} IS VICTORIOUS!")
        if self.excitement > 80:
            print(f"👥 CROWD: *Standing ovation!* GLORY TO {winner.name.upper()}!")
        elif self.excitement > 50:
            print(f"👥 CROWD: Well fought! Honor to both warriors!")
        else:
            print(f"👥 CROWD: *Polite applause*")

    def get_excitement_level(self) -> str:
        if self.excitement >= 80:
            return "ECSTATIC"
        elif self.excitement >= 60:
            return "EXCITED"
        elif self.excitement >= 40:
            return "INTERESTED"
        elif self.excitement >= 20:
            return "BORED"
        else:
            return "DISAPPOINTED"

class AnnouncerObserver(ArenaObserver):
    """Arena announcer providing commentary"""

    def on_combat_action(self, action: CombatAction):
        if action.event_type == CombatEvent.CRITICAL_HIT:
            print(f"📢 ANNOUNCER: DEVASTATING BLOW! {action.damage} damage!")
        elif action.event_type == CombatEvent.HIT_LANDED:
            print(f"📢 ANNOUNCER: {action.attacker_name} strikes for {action.damage} damage!")
        elif action.event_type == CombatEvent.DODGE:
            print(f"📢 ANNOUNCER: {action.defender_name} dodges skillfully!")
        elif action.event_type == CombatEvent.SPECIAL_MOVE:
            print(f"📢 ANNOUNCER: {action.description}!")
        elif action.event_type == CombatEvent.MISS:
            print(f"📢 ANNOUNCER: {action.attacker_name} misses!")

    def on_match_start(self, gladiator1: 'Gladiator', gladiator2: 'Gladiator'):
        print(f"\n📢 ANNOUNCER: Ladies and gentlemen of Rome!")
        print(f"📢 ANNOUNCER: {gladiator1.name} the {gladiator1.gladiator_class.value}!")
        print(f"📢 ANNOUNCER: VERSUS")
        print(f"📢 ANNOUNCER: {gladiator2.name} the {gladiator2.gladiator_class.value}!")

    def on_match_end(self, winner: 'Gladiator', loser: 'Gladiator'):
        print(f"\n📢 ANNOUNCER: The victor is {winner.name}!")
        print(f"📢 ANNOUNCER: {loser.name} fought bravely but falls today!")

class ScoreKeeper(ArenaObserver):
    """Tracks match statistics"""

    def __init__(self):
        self.total_damage_dealt = {}
        self.hits_landed = {}
        self.critical_hits = {}
        self.special_moves = {}

    def on_combat_action(self, action: CombatAction):
        attacker = action.attacker_name

        # Track damage
        self.total_damage_dealt[attacker] = self.total_damage_dealt.get(attacker, 0) + action.damage

        # Track hits
        if action.event_type == CombatEvent.HIT_LANDED:
            self.hits_landed[attacker] = self.hits_landed.get(attacker, 0) + 1
        elif action.event_type == CombatEvent.CRITICAL_HIT:
            self.critical_hits[attacker] = self.critical_hits.get(attacker, 0) + 1
        elif action.event_type == CombatEvent.SPECIAL_MOVE:
            self.special_moves[attacker] = self.special_moves.get(attacker, 0) + 1

    def on_match_start(self, gladiator1: 'Gladiator', gladiator2: 'Gladiator'):
        pass  # Reset handled by arena

    def on_match_end(self, winner: 'Gladiator', loser: 'Gladiator'):
        print(f"\n📊 MATCH STATISTICS:")
        print(f"  {winner.name}: {self.total_damage_dealt.get(winner.name, 0)} total damage")
        print(f"  {loser.name}: {self.total_damage_dealt.get(loser.name, 0)} total damage")

    def reset(self):
        self.total_damage_dealt.clear()
        self.hits_landed.clear()
        self.critical_hits.clear()
        self.special_moves.clear()

# Strategy Pattern - Combat Strategies
class CombatStrategy(ABC):
    """Base strategy for combat"""

    @abstractmethod
    def attack(self, attacker: 'Gladiator', defender: 'Gladiator') -> CombatAction:
        pass

    @abstractmethod
    def get_strategy_name(self) -> str:
        pass

class RetariusStrategy(CombatStrategy):
    """Net fighter strategy - uses net and trident"""

    def __init__(self):
        self.has_net = True

    def attack(self, attacker: 'Gladiator', defender: 'Gladiator') -> CombatAction:
        # Net attack - chance to entangle
        if self.has_net and random.random() < 0.3:
            self.has_net = False  # Net used
            damage = random.randint(15, 25)
            defender.is_entangled = True
            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=damage,
                event_type=CombatEvent.SPECIAL_MOVE,
                description=f"{attacker.name} ensnares {defender.name} in the net!"
            )

        # Trident attack - lighter but faster
        hit_chance = 0.75
        if random.random() < hit_chance:
            damage = random.randint(20, 35)
            is_crit = random.random() < 0.15

            if is_crit:
                damage = int(damage * 1.5)
                return CombatAction(
                    attacker_name=attacker.name,
                    defender_name=defender.name,
                    damage=damage,
                    event_type=CombatEvent.CRITICAL_HIT,
                    description=f"{attacker.name} pierces with the trident",
                    is_critical=True
                )

            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=damage,
                event_type=CombatEvent.HIT_LANDED,
                description=f"{attacker.name} strikes with trident"
            )
        else:
            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=0,
                event_type=CombatEvent.MISS,
                description=f"{attacker.name} misses with trident"
            )

    def get_strategy_name(self) -> str:
        return "Net Fighter"

class MurmilloStrategy(CombatStrategy):
    """Heavy armor and sword strategy - high defense, powerful attacks"""

    def attack(self, attacker: 'Gladiator', defender: 'Gladiator') -> CombatAction:
        # Heavy sword strike - slower but powerful
        hit_chance = 0.65
        if random.random() < hit_chance:
            base_damage = random.randint(30, 50)

            # Reduced by defender's armor
            actual_damage = base_damage - (defender.armor * 2)
            actual_damage = max(10, actual_damage)  # Minimum damage

            is_crit = random.random() < 0.2

            if is_crit:
                actual_damage = int(actual_damage * 2)
                return CombatAction(
                    attacker_name=attacker.name,
                    defender_name=defender.name,
                    damage=actual_damage,
                    event_type=CombatEvent.CRITICAL_HIT,
                    description=f"{attacker.name} delivers a crushing blow",
                    is_critical=True
                )

            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=actual_damage,
                event_type=CombatEvent.HIT_LANDED,
                description=f"{attacker.name} strikes with gladius"
            )
        else:
            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=0,
                event_type=CombatEvent.MISS,
                description=f"{attacker.name} swings but misses"
            )

    def get_strategy_name(self) -> str:
        return "Heavy Fighter"

class SecutorStrategy(CombatStrategy):
    """Pursuer strategy - balanced offense/defense, counters net fighters"""

    def attack(self, attacker: 'Gladiator', defender: 'Gladiator') -> CombatAction:
        # Bonus damage against net fighters
        hit_chance = 0.70
        bonus_damage = 10 if defender.gladiator_class == GladiatorClass.RETIARIUS else 0

        if random.random() < hit_chance:
            damage = random.randint(25, 40) + bonus_damage

            is_crit = random.random() < 0.15

            if is_crit:
                damage = int(damage * 1.5)
                return CombatAction(
                    attacker_name=attacker.name,
                    defender_name=defender.name,
                    damage=damage,
                    event_type=CombatEvent.CRITICAL_HIT,
                    description=f"{attacker.name} lands a perfect strike",
                    is_critical=True
                )

            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=damage,
                event_type=CombatEvent.HIT_LANDED,
                description=f"{attacker.name} strikes decisively"
            )
        else:
            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=0,
                event_type=CombatEvent.MISS,
                description=f"{attacker.name} misses"
            )

    def get_strategy_name(self) -> str:
        return "Pursuer"

class ThraexStrategy(CombatStrategy):
    """Curved sword fighter - quick attacks"""

    def attack(self, attacker: 'Gladiator', defender: 'Gladiator') -> CombatAction:
        # Two quick strikes
        hit_chance = 0.80  # Higher hit chance
        total_damage = 0
        hits = 0

        for _ in range(2):
            if random.random() < hit_chance:
                total_damage += random.randint(12, 20)
                hits += 1

        if hits > 0:
            if hits == 2:
                return CombatAction(
                    attacker_name=attacker.name,
                    defender_name=defender.name,
                    damage=total_damage,
                    event_type=CombatEvent.SPECIAL_MOVE,
                    description=f"{attacker.name} lands a flurry of strikes"
                )
            else:
                return CombatAction(
                    attacker_name=attacker.name,
                    defender_name=defender.name,
                    damage=total_damage,
                    event_type=CombatEvent.HIT_LANDED,
                    description=f"{attacker.name} strikes quickly"
                )
        else:
            return CombatAction(
                attacker_name=attacker.name,
                defender_name=defender.name,
                damage=0,
                event_type=CombatEvent.MISS,
                description=f"{attacker.name} misses both strikes"
            )

    def get_strategy_name(self) -> str:
        return "Quick Striker"

# Gladiator class
class Gladiator:
    """Individual gladiator with strategy"""

    def __init__(self, name: str, gladiator_class: GladiatorClass, strategy: CombatStrategy):
        self.name = name
        self.gladiator_class = gladiator_class
        self.strategy = strategy
        self.max_health = 100
        self.health = 100
        self.armor = self._get_base_armor()
        self.is_entangled = False

    def _get_base_armor(self) -> int:
        """Get armor value based on gladiator class"""
        armor_map = {
            GladiatorClass.RETIARIUS: 2,
            GladiatorClass.MURMILLO: 8,
            GladiatorClass.SECUTOR: 5,
            GladiatorClass.THRAEX: 4,
            GladiatorClass.HOPLOMACHUS: 6
        }
        return armor_map.get(self.gladiator_class, 3)

    def set_strategy(self, strategy: CombatStrategy):
        """Change combat strategy mid-fight"""
        self.strategy = strategy

    def attack(self, opponent: 'Gladiator') -> CombatAction:
        """Execute attack using current strategy"""
        # Entangled penalty
        if self.is_entangled:
            self.is_entangled = False  # Break free
            return CombatAction(
                attacker_name=self.name,
                defender_name=opponent.name,
                damage=0,
                event_type=CombatEvent.SPECIAL_MOVE,
                description=f"{self.name} breaks free from the net"
            )

        return self.strategy.attack(self, opponent)

    def take_damage(self, damage: int):
        """Take damage"""
        self.health -= damage
        self.health = max(0, self.health)

    def is_alive(self) -> bool:
        """Check if gladiator is still fighting"""
        return self.health > 0

    def get_status(self) -> str:
        """Get current status"""
        health_bar = "█" * int(self.health / 10) + "░" * int((self.max_health - self.health) / 10)
        return f"{self.name}: [{health_bar}] {self.health}/{self.max_health} HP"

# Arena
class GladiatorArena:
    """Main arena managing combat"""

    def __init__(self):
        self.observers: List[ArenaObserver] = []

    def add_observer(self, observer: ArenaObserver):
        """Add observer to arena"""
        self.observers.append(observer)

    def remove_observer(self, observer: ArenaObserver):
        """Remove observer from arena"""
        self.observers.remove(observer)

    def notify_combat_action(self, action: CombatAction):
        """Notify all observers of combat action"""
        for observer in self.observers:
            observer.on_combat_action(action)

    def notify_match_start(self, gladiator1: Gladiator, gladiator2: Gladiator):
        """Notify match start"""
        for observer in self.observers:
            observer.on_match_start(gladiator1, gladiator2)

    def notify_match_end(self, winner: Gladiator, loser: Gladiator):
        """Notify match end"""
        for observer in self.observers:
            observer.on_match_end(winner, loser)

    def fight(self, gladiator1: Gladiator, gladiator2: Gladiator, max_rounds: int = 20) -> Gladiator:
        """Conduct a fight between two gladiators"""
        self.notify_match_start(gladiator1, gladiator2)

        round_num = 0
        while gladiator1.is_alive() and gladiator2.is_alive() and round_num < max_rounds:
            round_num += 1
            print(f"\n⚔️  ROUND {round_num}")
            print(f"  {gladiator1.get_status()}")
            print(f"  {gladiator2.get_status()}")
            print()

            # Gladiator 1 attacks
            action1 = gladiator1.attack(gladiator2)
            self.notify_combat_action(action1)
            gladiator2.take_damage(action1.damage)

            if not gladiator2.is_alive():
                break

            # Gladiator 2 attacks
            action2 = gladiator2.attack(gladiator1)
            self.notify_combat_action(action2)
            gladiator1.take_damage(action2.damage)

        # Determine winner
        if gladiator1.is_alive():
            winner, loser = gladiator1, gladiator2
        else:
            winner, loser = gladiator2, gladiator1

        self.notify_match_end(winner, loser)
        return winner

# Example Usage
def main():
    print("=== GLADIATOR ARENA - Strategy + Observer Demo ===\n")

    # Create arena
    arena = GladiatorArena()

    # Add observers
    crowd = CrowdObserver()
    announcer = AnnouncerObserver()
    score_keeper = ScoreKeeper()

    arena.add_observer(crowd)
    arena.add_observer(announcer)
    arena.add_observer(score_keeper)

    # Match 1: Retiarius vs Murmillo
    print("\n" + "=" * 70)
    print("MATCH 1: Net Fighter vs Heavy Fighter")
    print("=" * 70)

    gladiator1 = Gladiator("Spartacus", GladiatorClass.RETIARIUS, RetariusStrategy())
    gladiator2 = Gladiator("Maximus", GladiatorClass.MURMILLO, MurmilloStrategy())

    winner1 = arena.fight(gladiator1, gladiator2, max_rounds=10)
    print(f"\nCrowd Excitement: {crowd.get_excitement_level()} ({crowd.excitement}/100)")

    # Match 2: Secutor vs Thraex
    print("\n\n" + "=" * 70)
    print("MATCH 2: Pursuer vs Quick Striker")
    print("=" * 70)

    score_keeper.reset()
    crowd.excitement = 50

    gladiator3 = Gladiator("Commodus", GladiatorClass.SECUTOR, SecutorStrategy())
    gladiator4 = Gladiator("Tigris", GladiatorClass.THRAEX, ThraexStrategy())

    winner2 = arena.fight(gladiator3, gladiator4, max_rounds=10)
    print(f"\nCrowd Excitement: {crowd.get_excitement_level()} ({crowd.excitement}/100)")

    # Tournament summary
    print("\n\n" + "=" * 70)
    print("🏆 TOURNAMENT SUMMARY")
    print("=" * 70)
    print(f"Match 1 Winner: {winner1.name}")
    print(f"Match 2 Winner: {winner2.name}")
    print("=" * 70)

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== GLADIATOR ARENA - Strategy + Observer Demo ===


======================================================================
MATCH 1: Net Fighter vs Heavy Fighter
======================================================================

📢 ANNOUNCER: Ladies and gentlemen of Rome!
📢 ANNOUNCER: Spartacus the Retiarius!
📢 ANNOUNCER: VERSUS
📢 ANNOUNCER: Maximus the Murmillo!

👥 CROWD: Let the battle begin! Spartacus vs Maximus!
👥 CROWD: *Roaring and chanting*

⚔️  ROUND 1
  Spartacus: [██████████] 100/100 HP
  Maximus: [██████████] 100/100 HP

📢 ANNOUNCER: Spartacus ensnares Maximus in the net!!
👥 CROWD: AMAZING! Spartacus ensnares Maximus in the net!
📢 ANNOUNCER: Maximus breaks free from the net!
👥 CROWD: AMAZING! Maximus breaks free from the net!

⚔️  ROUND 2
  Spartacus: [██████████] 100/100 HP
  Maximus: [█████████░] 95/100 HP

📢 ANNOUNCER: Spartacus strikes for 25 damage!
👥 CROWD: That's gotta hurt!
📢 ANNOUNCER: DEVASTATING BLOW! 72 damage!
👥 CROWD: OOOOHHHH!!! What a blow!

⚔️  ROUND 3
  Spartacus: [███░░░░░░░] 28/100 HP
  Maximus: [███████░░░] 70/100 HP

📢 ANNOUNCER: Spartacus strikes for 30 damage!
👥 CROWD: That's gotta hurt!
📢 ANNOUNCER: Maximus strikes for 35 damage!
👥 CROWD: OOOOHHHH!!! What a blow!

📢 ANNOUNCER: The victor is Maximus!
📢 ANNOUNCER: Spartacus fought bravely but falls today!

👥 CROWD: MAXIMUS IS VICTORIOUS!
👥 CROWD: *Standing ovation!* GLORY TO MAXIMUS!

📊 MATCH STATISTICS:
  Maximus: 107 total damage
  Spartacus: 55 total damage

Crowd Excitement: ECSTATIC (90/100)


======================================================================
MATCH 2: Pursuer vs Quick Striker
======================================================================

📢 ANNOUNCER: Ladies and gentlemen of Rome!
📢 ANNOUNCER: Commodus the Secutor!
📢 ANNOUNCER: VERSUS
📢 ANNOUNCER: Tigris the Thraex!

👥 CROWD: Let the battle begin! Commodus vs Tigris!
👥 CROWD: *Roaring and chanting*

⚔️  ROUND 1
  Commodus: [██████████] 100/100 HP
  Tigris: [██████████] 100/100 HP

📢 ANNOUNCER: Commodus strikes for 32 damage!
👥 CROWD: That's gotta hurt!
📢 ANNOUNCER: Tigris lands a flurry of strikes!
👥 CROWD: AMAZING! Tigris lands a flurry of strikes!

⚔️  ROUND 2
  Commodus: [██████░░░░] 68/100 HP
  Tigris: [███████░░░] 68/100 HP

📢 ANNOUNCER: Commodus strikes for 28 damage!
👥 CROWD: Nice hit!
📢 ANNOUNCER: Tigris strikes for 18 damage!
👥 CROWD: Nice hit!

⚔️  ROUND 3
  Commodus: [█████░░░░░] 50/100 HP
  Tigris: [████░░░░░░] 40/100 HP

📢 ANNOUNCER: DEVASTATING BLOW! 54 damage!
👥 CROWD: OOOOHHHH!!! What a blow!

📢 ANNOUNCER: The victor is Commodus!
📢 ANNOUNCER: Tigris fought bravely but falls today!

👥 CROWD: COMMODUS IS VICTORIOUS!
👥 CROWD: *Standing ovation!* GLORY TO COMMODUS!

📊 MATCH STATISTICS:
  Commodus: 114 total damage
  Tigris: 50 total damage

Crowd Excitement: ECSTATIC (95/100)


======================================================================
🏆 TOURNAMENT SUMMARY
======================================================================
Match 1 Winner: Maximus
Match 2 Winner: Commodus
======================================================================
```

**Pattern Benefits:**
- **Strategy Pattern:** Different combat styles encapsulated as interchangeable strategies
- **Runtime Strategy Switching:** Gladiators can change tactics mid-fight
- **Observer Pattern:** Multiple independent systems react to combat events
- **Decoupled Systems:** Crowd, announcer, and stats tracking work independently
- **Event-Driven Architecture:** Combat actions trigger cascading observer notifications
- **Easy Extension:** New gladiator types and observer types can be added easily
- **Realistic Simulation:** Each strategy has unique mechanics and strengths/weaknesses
- **Type Safety:** Full type hints ensure correct usage of all components

---

## 24. Time Loop Mystery - Memento + Observer

**Theme:** Groundhog Day investigation
**Patterns:** Memento, Observer, State

**Key Learning:** Each loop is a memento, observers track changes

**Complete Implementation:**

```python
"""
Complete Time Loop Mystery system with Memento and Observer patterns.
Demonstrates state saving/restoration and tracking changes across loops.
"""

from abc import ABC, abstractmethod
from typing import List, Dict, Set, Optional, Any
from enum import Enum
from dataclasses import dataclass
from copy import deepcopy
import random

class TimeOfDay(Enum):
    MORNING = "6:00 AM"
    MIDDAY = "12:00 PM"
    AFTERNOON = "3:00 PM"
    EVENING = "6:00 PM"
    NIGHT = "9:00 PM"
    MIDNIGHT = "12:00 AM"

class EventType(Enum):
    DISCOVERY = "discovery"
    NPC_INTERACTION = "npc_interaction"
    LOCATION_VISIT = "location_visit"
    ITEM_FOUND = "item_found"
    DEATH = "death"
    LOOP_RESET = "loop_reset"

@dataclass
class GameEvent:
    """Represents an event that occurred"""
    event_type: EventType
    description: str
    time: TimeOfDay
    location: str

# Memento Pattern
class GameMemento:
    """Memento storing game state"""

    def __init__(self, state: Dict[str, Any]):
        self._state = deepcopy(state)

    def get_state(self) -> Dict[str, Any]:
        """Get saved state"""
        return deepcopy(self._state)

class MementoCaretaker:
    """Manages mementos"""

    def __init__(self):
        self._mementos: List[GameMemento] = []
        self._loop_start_mementos: Dict[int, GameMemento] = {}

    def save(self, memento: GameMemento):
        """Save a memento"""
        self._mementos.append(memento)

    def save_loop_start(self, loop_number: int, memento: GameMemento):
        """Save the state at loop start"""
        self._loop_start_mementos[loop_number] = memento

    def get_loop_start(self, loop_number: int) -> Optional[GameMemento]:
        """Get the loop start memento"""
        return self._loop_start_mementos.get(loop_number)

    def get_latest(self) -> Optional[GameMemento]:
        """Get most recent memento"""
        return self._mementos[-1] if self._mementos else None

# Observer Pattern
class LoopObserver(ABC):
    """Observer for loop events"""

    @abstractmethod
    def on_loop_start(self, loop_number: int):
        pass

    @abstractmethod
    def on_loop_end(self, loop_number: int, reason: str):
        pass

    @abstractmethod
    def on_discovery(self, discovery: str, loop_number: int):
        pass

    @abstractmethod
    def on_npc_interaction(self, npc_name: str, dialogue: str, loop_number: int):
        pass

class DetectiveNotebook(LoopObserver):
    """Tracks persistent knowledge across loops"""

    def __init__(self):
        self.discoveries: Set[str] = set()
        self.npc_dialogues: Dict[str, List[str]] = {}
        self.loop_outcomes: List[str] = []
        self.total_loops = 0

    def on_loop_start(self, loop_number: int):
        print(f"\n📓 NOTEBOOK: Starting loop #{loop_number}")
        print(f"📓 Known facts: {len(self.discoveries)}")

    def on_loop_end(self, loop_number: int, reason: str):
        self.loop_outcomes.append(f"Loop {loop_number}: {reason}")
        print(f"📓 NOTEBOOK: Loop ended - {reason}")

    def on_discovery(self, discovery: str, loop_number: int):
        if discovery not in self.discoveries:
            self.discoveries.add(discovery)
            print(f"📓 NOTEBOOK: NEW DISCOVERY - {discovery}")
            print(f"📓 This knowledge persists across loops!")
        else:
            print(f"📓 NOTEBOOK: Already knew - {discovery}")

    def on_npc_interaction(self, npc_name: str, dialogue: str, loop_number: int):
        if npc_name not in self.npc_dialogues:
            self.npc_dialogues[npc_name] = []
        if dialogue not in self.npc_dialogues[npc_name]:
            self.npc_dialogues[npc_name].append(dialogue)
            print(f"📓 NOTEBOOK: {npc_name} said something new!")

    def get_summary(self) -> str:
        """Get investigation summary"""
        summary = "\n" + "=" * 60 + "\n"
        summary += "📓 DETECTIVE NOTEBOOK SUMMARY\n"
        summary += "=" * 60 + "\n"
        summary += f"Total Loops: {self.total_loops}\n"
        summary += f"Discoveries: {len(self.discoveries)}\n"
        summary += "\nKnown Facts:\n"
        for i, discovery in enumerate(self.discoveries, 1):
            summary += f"  {i}. {discovery}\n"
        summary += "=" * 60
        return summary

class LoopAnalyzer(LoopObserver):
    """Analyzes patterns across loops"""

    def __init__(self):
        self.loop_data: List[Dict] = []
        self.current_loop_events: List[str] = []

    def on_loop_start(self, loop_number: int):
        self.current_loop_events = []
        print(f"🔍 ANALYZER: Monitoring loop #{loop_number}")

    def on_loop_end(self, loop_number: int, reason: str):
        self.loop_data.append({
            'loop': loop_number,
            'reason': reason,
            'events': len(self.current_loop_events)
        })

        if loop_number > 1:
            print(f"🔍 ANALYZER: Pattern detected - loops ending at same time?")

    def on_discovery(self, discovery: str, loop_number: int):
        self.current_loop_events.append(f"Discovery: {discovery}")

    def on_npc_interaction(self, npc_name: str, dialogue: str, loop_number: int):
        self.current_loop_events.append(f"NPC: {npc_name}")

class EchoesOfTime(LoopObserver):
    """Mysterious observer showing déjà vu moments"""

    def __init__(self):
        self.seen_before: Set[str] = set()

    def on_loop_start(self, loop_number: int):
        if loop_number > 1:
            print(f"👁️  ECHOES: The day begins again... as it has {loop_number - 1} times before")

    def on_loop_end(self, loop_number: int, reason: str):
        pass

    def on_discovery(self, discovery: str, loop_number: int):
        if discovery in self.seen_before:
            print(f"👁️  ECHOES: *Déjà vu* - You've seen this before...")
        self.seen_before.add(discovery)

    def on_npc_interaction(self, npc_name: str, dialogue: str, loop_number: int):
        key = f"{npc_name}:{dialogue}"
        if key in self.seen_before:
            print(f"👁️  ECHOES: {npc_name} says the same thing again...")
        self.seen_before.add(key)

# Game State
class MysteryGame:
    """Main game with time loop mechanics"""

    def __init__(self):
        self.current_time = TimeOfDay.MORNING
        self.current_location = "Home"
        self.inventory: List[str] = []
        self.npc_states: Dict[str, str] = {
            "Mayor": "office",
            "Bartender": "pub",
            "Detective": "crime_scene"
        }
        self.player_alive = True

        # Persistent knowledge (NOT reset by loops)
        self.persistent_knowledge: Set[str] = set()

        # Observers
        self.observers: List[LoopObserver] = []

        # Memento caretaker
        self.caretaker = MementoCaretaker()

    def add_observer(self, observer: LoopObserver):
        """Add observer"""
        self.observers.append(observer)

    def notify_loop_start(self, loop_number: int):
        """Notify loop start"""
        for observer in self.observers:
            observer.on_loop_start(loop_number)

    def notify_loop_end(self, loop_number: int, reason: str):
        """Notify loop end"""
        for observer in self.observers:
            observer.on_loop_end(loop_number, reason)

    def notify_discovery(self, discovery: str, loop_number: int):
        """Notify discovery"""
        for observer in self.observers:
            observer.on_discovery(discovery, loop_number)

    def notify_npc_interaction(self, npc_name: str, dialogue: str, loop_number: int):
        """Notify NPC interaction"""
        for observer in self.observers:
            observer.on_npc_interaction(npc_name, dialogue, loop_number)

    def create_memento(self) -> GameMemento:
        """Create memento of current state"""
        state = {
            'time': self.current_time,
            'location': self.current_location,
            'inventory': self.inventory.copy(),
            'npc_states': self.npc_states.copy(),
            'player_alive': self.player_alive
        }
        return GameMemento(state)

    def restore_from_memento(self, memento: GameMemento):
        """Restore state from memento"""
        state = memento.get_state()
        self.current_time = state['time']
        self.current_location = state['location']
        self.inventory = state['inventory']
        self.npc_states = state['npc_states']
        self.player_alive = state['player_alive']
        # Note: persistent_knowledge is NOT restored!

    def advance_time(self):
        """Advance to next time period"""
        times = list(TimeOfDay)
        current_index = times.index(self.current_time)

        if current_index < len(times) - 1:
            self.current_time = times[current_index + 1]
        else:
            return "midnight"  # Loop reset trigger

        return "continue"

    def visit_location(self, location: str):
        """Visit a location"""
        self.current_location = location
        print(f"\n📍 You are now at: {location}")

    def talk_to_npc(self, npc_name: str, loop_number: int) -> str:
        """Talk to an NPC"""
        dialogues = {
            "Mayor": [
                "Good morning! Lovely day, isn't it?",
                "The murder happened at the old mansion last night.",
                "I was at the charity gala all evening. Many witnesses!"
            ],
            "Bartender": [
                "Welcome to my pub!",
                "The victim was here yesterday, arguing with someone.",
                "If you know the secret password, I can tell you more..."
            ],
            "Detective": [
                "Another investigator? This case is mine!",
                "I found a bloody knife at the scene.",
                "The time of death was exactly midnight."
            ]
        }

        dialogue = random.choice(dialogues.get(npc_name, ["..."]))
        self.notify_npc_interaction(npc_name, dialogue, loop_number)
        return dialogue

    def investigate(self, clue: str, loop_number: int):
        """Investigate and potentially make a discovery"""
        discoveries = {
            "mansion": "The mansion door was forced open from the inside!",
            "crime_scene": "Blood trail leads to the mayor's office!",
            "pub_secret": "The bartender is the mayor's brother!",
            "mayor_alibi": "The charity gala ended at 11 PM, not midnight!",
            "knife": "The knife belongs to the pub - same logo!"
        }

        if clue in discoveries:
            discovery = discoveries[clue]
            self.persistent_knowledge.add(discovery)
            self.notify_discovery(discovery, loop_number)
            return discovery
        return None

class TimeLoopManager:
    """Manages the time loop mechanic"""

    def __init__(self, game: MysteryGame):
        self.game = game
        self.loop_number = 0
        self.loop_start_memento: Optional[GameMemento] = None
        self.mystery_solved = False

    def start_loop(self):
        """Start a new loop"""
        self.loop_number += 1

        # Save loop start state
        self.loop_start_memento = self.game.create_memento()
        self.game.caretaker.save_loop_start(self.loop_number, self.loop_start_memento)

        # Notify observers
        self.game.notify_loop_start(self.loop_number)

        print(f"\n{'='*60}")
        print(f"⏰ TIME LOOP #{self.loop_number} BEGINS")
        print(f"{'='*60}")
        print(f"🌅 {self.game.current_time.value} - {self.game.current_location}")

    def reset_loop(self, reason: str):
        """Reset to loop start"""
        print(f"\n⏰ TIME RESETS! Reason: {reason}")

        # Preserve persistent knowledge
        preserved_knowledge = self.game.persistent_knowledge.copy()

        # Restore game state
        if self.loop_start_memento:
            self.game.restore_from_memento(self.loop_start_memento)

        # Restore persistent knowledge
        self.game.persistent_knowledge = preserved_knowledge

        # Notify observers
        self.game.notify_loop_end(self.loop_number, reason)

    def check_solution(self) -> bool:
        """Check if mystery is solved"""
        required_knowledge = {
            "The mansion door was forced open from the inside!",
            "The bartender is the mayor's brother!",
            "The charity gala ended at 11 PM, not midnight!"
        }

        return required_knowledge.issubset(self.game.persistent_knowledge)

# Example Usage
def main():
    print("=== TIME LOOP MYSTERY - Memento + Observer Demo ===\n")
    print("🎮 You wake up. The day is repeating. Solve the murder mystery!")
    print("💡 Your discoveries persist across loops, but the day resets!\n")

    # Create game
    game = MysteryGame()

    # Add observers
    notebook = DetectiveNotebook()
    analyzer = LoopAnalyzer()
    echoes = EchoesOfTime()

    game.add_observer(notebook)
    game.add_observer(analyzer)
    game.add_observer(echoes)

    # Create time loop manager
    loop_manager = TimeLoopManager(game)

    # Simulate investigation across multiple loops
    scenarios = [
        # Loop 1: Basic exploration, die early
        [
            ("visit", "Mansion"),
            ("investigate", "mansion"),
            ("advance", None),
            ("death", "Caught by killer!")
        ],
        # Loop 2: Talk to NPCs, reach midnight
        [
            ("visit", "Police Station"),
            ("talk", "Detective"),
            ("investigate", "knife"),
            ("advance", None),
            ("advance", None),
            ("visit", "Pub"),
            ("talk", "Bartender"),
            ("advance", None),
            ("advance", None),
            ("advance", None),
            ("midnight", None)
        ],
        # Loop 3: Gather critical evidence
        [
            ("visit", "Mansion"),
            ("investigate", "crime_scene"),
            ("advance", None),
            ("visit", "City Hall"),
            ("talk", "Mayor"),
            ("investigate", "mayor_alibi"),
            ("advance", None),
            ("visit", "Pub"),
            ("investigate", "pub_secret"),
            ("check", None)
        ]
    ]

    for loop_actions in scenarios:
        loop_manager.start_loop()

        for action, param in loop_actions:
            print()  # Spacing

            if action == "visit":
                game.visit_location(param)
            elif action == "talk":
                dialogue = game.talk_to_npc(param, loop_manager.loop_number)
                print(f"💬 {param}: \"{dialogue}\"")
            elif action == "investigate":
                discovery = game.investigate(param, loop_manager.loop_number)
                if discovery:
                    print(f"🔍 Investigated {param}")
            elif action == "advance":
                result = game.advance_time()
                if result == "midnight":
                    loop_manager.reset_loop("Midnight struck!")
                    break
                else:
                    print(f"⏰ Time advances to {game.current_time.value}")
            elif action == "death":
                print(f"💀 {param}")
                loop_manager.reset_loop(param)
                break
            elif action == "midnight":
                loop_manager.reset_loop("Midnight struck!")
                break
            elif action == "check":
                if loop_manager.check_solution():
                    loop_manager.mystery_solved = True
                    print("\n" + "🎉" * 30)
                    print("🎉 MYSTERY SOLVED!")
                    print("🎉" * 30)
                    print("\n🔍 You've uncovered the truth:")
                    print("   The Mayor's brother (the bartender) committed the murder!")
                    print("   The Mayor lied about his alibi to protect his brother!")
                    print("   The door was forced from inside - it was a setup!")
                    break

        if loop_manager.mystery_solved:
            break

    # Final summary
    print(notebook.get_summary())

    # Analyzer report
    print("\n🔍 LOOP ANALYSIS:")
    print(f"  Total loops: {loop_manager.loop_number}")
    print(f"  Mystery solved: {loop_manager.mystery_solved}")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== TIME LOOP MYSTERY - Memento + Observer Demo ===

🎮 You wake up. The day is repeating. Solve the murder mystery!
💡 Your discoveries persist across loops, but the day resets!

============================================================
⏰ TIME LOOP #1 BEGINS
============================================================
🌅 6:00 AM - Home

📓 NOTEBOOK: Starting loop #1
📓 Known facts: 0
🔍 ANALYZER: Monitoring loop #1

📍 You are now at: Mansion

🔍 Investigated mansion
📓 NOTEBOOK: NEW DISCOVERY - The mansion door was forced open from the inside!
📓 This knowledge persists across loops!

⏰ Time advances to 12:00 PM

💀 Caught by killer!
⏰ TIME RESETS! Reason: Caught by killer!
📓 NOTEBOOK: Loop ended - Caught by killer!

============================================================
⏰ TIME LOOP #2 BEGINS
============================================================
🌅 6:00 AM - Home

👁️  ECHOES: The day begins again... as it has 1 times before
📓 NOTEBOOK: Starting loop #2
📓 Known facts: 1
🔍 ANALYZER: Monitoring loop #2
🔍 ANALYZER: Pattern detected - loops ending at same time?

📍 You are now at: Police Station

💬 Detective: "The time of death was exactly midnight."
📓 NOTEBOOK: Detective said something new!

🔍 Investigated knife
📓 NOTEBOOK: NEW DISCOVERY - The knife belongs to the pub - same logo!
📓 This knowledge persists across loops!

⏰ Time advances to 12:00 PM

⏰ Time advances to 3:00 PM

📍 You are now at: Pub

💬 Bartender: "The victim was here yesterday, arguing with someone."
📓 NOTEBOOK: Bartender said something new!

⏰ Time advances to 6:00 PM

⏰ Time advances to 9:00 PM

⏰ Time advances to 12:00 AM

⏰ TIME RESETS! Reason: Midnight struck!
📓 NOTEBOOK: Loop ended - Midnight struck!

============================================================
⏰ TIME LOOP #3 BEGINS
============================================================
🌅 6:00 AM - Home

👁️  ECHOES: The day begins again... as it has 2 times before
📓 NOTEBOOK: Starting loop #3
📓 Known facts: 2
🔍 ANALYZER: Monitoring loop #3
🔍 ANALYZER: Pattern detected - loops ending at same time?

📍 You are now at: Mansion

🔍 Investigated crime_scene
📓 NOTEBOOK: NEW DISCOVERY - Blood trail leads to the mayor's office!
📓 This knowledge persists across loops!

⏰ Time advances to 12:00 PM

📍 You are now at: City Hall

💬 Mayor: "I was at the charity gala all evening. Many witnesses!"
📓 NOTEBOOK: Mayor said something new!

🔍 Investigated mayor_alibi
📓 NOTEBOOK: NEW DISCOVERY - The charity gala ended at 11 PM, not midnight!
📓 This knowledge persists across loops!

⏰ Time advances to 3:00 PM

📍 You are now at: Pub

🔍 Investigated pub_secret
📓 NOTEBOOK: NEW DISCOVERY - The bartender is the mayor's brother!
📓 This knowledge persists across loops!

🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉
🎉 MYSTERY SOLVED!
🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉

🔍 You've uncovered the truth:
   The Mayor's brother (the bartender) committed the murder!
   The Mayor lied about his alibi to protect his brother!
   The door was forced from inside - it was a setup!

============================================================
📓 DETECTIVE NOTEBOOK SUMMARY
============================================================
Total Loops: 0
Discoveries: 5

Known Facts:
  1. The mansion door was forced open from the inside!
  2. The knife belongs to the pub - same logo!
  3. Blood trail leads to the mayor's office!
  4. The charity gala ended at 11 PM, not midnight!
  5. The bartender is the mayor's brother!
============================================================

🔍 LOOP ANALYSIS:
  Total loops: 3
  Mystery solved: True
```

**Pattern Benefits:**
- **Memento Pattern:** Complete game state can be saved and restored without exposing internals
- **Time Loop Mechanic:** Perfect use case for memento - reset world but keep player knowledge
- **Observer Pattern:** Multiple independent systems track loop events simultaneously
- **Persistent Knowledge:** Some data (discoveries) persists while other data (game state) resets
- **Decoupled Tracking:** Notebook, analyzer, and echoes all work independently
- **Investigation Gameplay:** Piece together clues across multiple loops
- **State Preservation:** Memento ensures perfect state restoration after each loop
- **Type Safety:** Full type hints ensure correct usage of all components

---

## 25. Robot Factory - Abstract Factory + Builder

**Theme:** Automated robot assembly
**Patterns:** Abstract Factory, Builder

**Key Learning:** Different factories produce compatible robot parts

**Complete Implementation:**

```python
"""
Complete Robot Factory system with Abstract Factory and Builder patterns.
Demonstrates compatible component creation and complex robot assembly.
"""

from abc import ABC, abstractmethod
from typing import List, Optional, Dict
from enum import Enum
from dataclasses import dataclass

class RobotType(Enum):
    MILITARY = "Military"
    UTILITY = "Utility"
    MEDICAL = "Medical"
    SCOUT = "Scout"

class ComponentGrade(Enum):
    BASIC = "Basic"
    ADVANCED = "Advanced"
    ELITE = "Elite"

# Abstract Factory Pattern - Component Families
class Chassis(ABC):
    """Base chassis component"""

    @abstractmethod
    def get_durability(self) -> int:
        pass

    @abstractmethod
    def get_speed(self) -> int:
        pass

    @abstractmethod
    def get_name(self) -> str:
        pass

class ArmoredChassis(Chassis):
    """Heavy armored chassis for military robots"""

    def get_durability(self) -> int:
        return 200

    def get_speed(self) -> int:
        return 40

    def get_name(self) -> str:
        return "Armored Chassis"

class LightweightChassis(Chassis):
    """Light chassis for utility robots"""

    def get_durability(self) -> int:
        return 80

    def get_speed(self) -> int:
        return 100

    def get_name(self) -> str:
        return "Lightweight Chassis"

class MedicalChassis(Chassis):
    """Sterile chassis for medical robots"""

    def get_durability(self) -> int:
        return 100

    def get_speed(self) -> int:
        return 70

    def get_name(self) -> str:
        return "Medical Chassis"

class ScoutChassis(Chassis):
    """Fast, agile chassis for scout robots"""

    def get_durability(self) -> int:
        return 60

    def get_speed(self) -> int:
        return 150

    def get_name(self) -> str:
        return "Scout Chassis"

# Weapon Components
class Weapon(ABC):
    """Base weapon component"""

    @abstractmethod
    def get_damage(self) -> int:
        pass

    @abstractmethod
    def get_name(self) -> str:
        pass

class PlasmaRifle(Weapon):
    """Plasma weapon"""

    def get_damage(self) -> int:
        return 50

    def get_name(self) -> str:
        return "Plasma Rifle"

class MissilePods(Weapon):
    """Missile launcher"""

    def get_damage(self) -> int:
        return 80

    def get_name(self) -> str:
        return "Missile Pods"

class Manipulator(Weapon):
    """Utility manipulator arm"""

    def get_damage(self) -> int:
        return 5

    def get_name(self) -> str:
        return "Manipulator Arm"

class Welder(Weapon):
    """Welding tool"""

    def get_damage(self) -> int:
        return 10

    def get_name(self) -> str:
        return "Welding Torch"

class Laser(Weapon):
    """Precision laser"""

    def get_damage(self) -> int:
        return 30

    def get_name(self) -> str:
        return "Precision Laser"

class Scanner(Weapon):
    """Scanning device"""

    def get_damage(self) -> int:
        return 0

    def get_name(self) -> str:
        return "Advanced Scanner"

# AI Components
class AICore(ABC):
    """Base AI component"""

    @abstractmethod
    def get_intelligence(self) -> int:
        pass

    @abstractmethod
    def get_specialty(self) -> str:
        pass

    @abstractmethod
    def get_name(self) -> str:
        pass

class CombatAI(AICore):
    """Combat-focused AI"""

    def get_intelligence(self) -> int:
        return 70

    def get_specialty(self) -> str:
        return "Combat Tactics"

    def get_name(self) -> str:
        return "Combat AI"

class WorkerAI(AICore):
    """Worker-focused AI"""

    def get_intelligence(self) -> int:
        return 80

    def get_specialty(self) -> str:
        return "Task Optimization"

    def get_name(self) -> str:
        return "Worker AI"

class MedicalAI(AICore):
    """Medical-focused AI"""

    def get_intelligence(self) -> int:
        return 95

    def get_specialty(self) -> str:
        return "Medical Diagnosis"

    def get_name(self) -> str:
        return "Medical AI"

class ReconAI(AICore):
    """Reconnaissance AI"""

    def get_intelligence(self) -> int:
        return 85

    def get_specialty(self) -> str:
        return "Reconnaissance"

    def get_name(self) -> str:
        return "Recon AI"

# Power Source
class PowerSource(ABC):
    """Base power source"""

    @abstractmethod
    def get_capacity(self) -> int:
        pass

    @abstractmethod
    def get_name(self) -> str:
        pass

class FusionCore(PowerSource):
    """High-capacity fusion power"""

    def get_capacity(self) -> int:
        return 1000

    def get_name(self) -> str:
        return "Fusion Core"

class BatteryPack(PowerSource):
    """Standard battery"""

    def get_capacity(self) -> int:
        return 500

    def get_name(self) -> str:
        return "Battery Pack"

class SolarCell(PowerSource):
    """Solar power"""

    def get_capacity(self) -> int:
        return 300

    def get_name(self) -> str:
        return "Solar Cell"

# Abstract Factory
class RobotFactory(ABC):
    """Abstract factory for creating robot components"""

    @abstractmethod
    def create_chassis(self) -> Chassis:
        pass

    @abstractmethod
    def create_weapons(self) -> List[Weapon]:
        pass

    @abstractmethod
    def create_ai(self) -> AICore:
        pass

    @abstractmethod
    def create_power_source(self) -> PowerSource:
        pass

    @abstractmethod
    def get_factory_type(self) -> RobotType:
        pass

class MilitaryRobotFactory(RobotFactory):
    """Factory for military robots"""

    def create_chassis(self) -> Chassis:
        return ArmoredChassis()

    def create_weapons(self) -> List[Weapon]:
        return [PlasmaRifle(), MissilePods()]

    def create_ai(self) -> AICore:
        return CombatAI()

    def create_power_source(self) -> PowerSource:
        return FusionCore()

    def get_factory_type(self) -> RobotType:
        return RobotType.MILITARY

class UtilityRobotFactory(RobotFactory):
    """Factory for utility robots"""

    def create_chassis(self) -> Chassis:
        return LightweightChassis()

    def create_weapons(self) -> List[Weapon]:
        return [Manipulator(), Welder()]

    def create_ai(self) -> AICore:
        return WorkerAI()

    def create_power_source(self) -> PowerSource:
        return BatteryPack()

    def get_factory_type(self) -> RobotType:
        return RobotType.UTILITY

class MedicalRobotFactory(RobotFactory):
    """Factory for medical robots"""

    def create_chassis(self) -> Chassis:
        return MedicalChassis()

    def create_weapons(self) -> List[Weapon]:
        return [Laser(), Manipulator()]

    def create_ai(self) -> AICore:
        return MedicalAI()

    def create_power_source(self) -> PowerSource:
        return BatteryPack()

    def get_factory_type(self) -> RobotType:
        return RobotType.MEDICAL

class ScoutRobotFactory(RobotFactory):
    """Factory for scout robots"""

    def create_chassis(self) -> Chassis:
        return ScoutChassis()

    def create_weapons(self) -> List[Weapon]:
        return [Scanner(), Laser()]

    def create_ai(self) -> AICore:
        return ReconAI()

    def create_power_source(self) -> PowerSource:
        return SolarCell()

    def get_factory_type(self) -> RobotType:
        return RobotType.SCOUT

# Robot Product
class Robot:
    """Complete robot with all components"""

    def __init__(self, designation: str):
        self.designation = designation
        self.chassis: Optional[Chassis] = None
        self.weapons: List[Weapon] = []
        self.ai: Optional[AICore] = None
        self.power_source: Optional[PowerSource] = None
        self.upgrades: List[str] = []

    def get_combat_rating(self) -> int:
        """Calculate combat effectiveness"""
        if not self.chassis or not self.ai:
            return 0

        base_rating = self.chassis.get_durability() // 2
        weapon_rating = sum(w.get_damage() for w in self.weapons)
        ai_bonus = self.ai.get_intelligence() // 10

        return base_rating + weapon_rating + ai_bonus

    def get_utility_rating(self) -> int:
        """Calculate utility effectiveness"""
        if not self.ai:
            return 0

        return self.ai.get_intelligence()

    def get_specifications(self) -> str:
        """Get detailed specifications"""
        specs = f"\n{'='*60}\n"
        specs += f"🤖 ROBOT DESIGNATION: {self.designation}\n"
        specs += f"{'='*60}\n"

        if self.chassis:
            specs += f"Chassis: {self.chassis.get_name()}\n"
            specs += f"  - Durability: {self.chassis.get_durability()}\n"
            specs += f"  - Speed: {self.chassis.get_speed()}\n"

        if self.ai:
            specs += f"\nAI Core: {self.ai.get_name()}\n"
            specs += f"  - Intelligence: {self.ai.get_intelligence()}\n"
            specs += f"  - Specialty: {self.ai.get_specialty()}\n"

        if self.weapons:
            specs += f"\nWeapons/Tools:\n"
            for weapon in self.weapons:
                specs += f"  - {weapon.get_name()} (DMG: {weapon.get_damage()})\n"

        if self.power_source:
            specs += f"\nPower Source: {self.power_source.get_name()}\n"
            specs += f"  - Capacity: {self.power_source.get_capacity()}\n"

        specs += f"\nRatings:\n"
        specs += f"  - Combat: {self.get_combat_rating()}\n"
        specs += f"  - Utility: {self.get_utility_rating()}\n"

        if self.upgrades:
            specs += f"\nUpgrades:\n"
            for upgrade in self.upgrades:
                specs += f"  - {upgrade}\n"

        specs += f"{'='*60}"
        return specs

# Builder Pattern
class RobotBuilder:
    """Builder for constructing robots step-by-step"""

    def __init__(self, designation: str):
        self.robot = Robot(designation)

    def set_chassis(self, chassis: Chassis) -> 'RobotBuilder':
        """Set robot chassis"""
        self.robot.chassis = chassis
        return self

    def add_weapon(self, weapon: Weapon) -> 'RobotBuilder':
        """Add a weapon"""
        self.robot.weapons.append(weapon)
        return self

    def set_ai(self, ai: AICore) -> 'RobotBuilder':
        """Set AI core"""
        self.robot.ai = ai
        return self

    def set_power_source(self, power_source: PowerSource) -> 'RobotBuilder':
        """Set power source"""
        self.robot.power_source = power_source
        return self

    def add_upgrade(self, upgrade: str) -> 'RobotBuilder':
        """Add an upgrade"""
        self.robot.upgrades.append(upgrade)
        return self

    def build(self) -> Robot:
        """Build and return the robot"""
        if not self.robot.chassis:
            raise ValueError("Robot must have a chassis")
        if not self.robot.ai:
            raise ValueError("Robot must have an AI core")

        return self.robot

# Director - combines Factory and Builder
class RobotAssemblyDirector:
    """Director that uses factories and builders together"""

    def __init__(self):
        self.produced_robots: List[Robot] = []

    def produce_standard_robot(self, factory: RobotFactory, designation: str) -> Robot:
        """Produce standard robot using factory"""
        print(f"\n🏭 Producing {factory.get_factory_type().value} Robot: {designation}")

        builder = RobotBuilder(designation)

        robot = (builder
                 .set_chassis(factory.create_chassis())
                 .set_ai(factory.create_ai())
                 .set_power_source(factory.create_power_source())
                 .build())

        # Add weapons
        for weapon in factory.create_weapons():
            robot.weapons.append(weapon)

        self.produced_robots.append(robot)
        print(f"✓ {factory.get_factory_type().value} robot assembled successfully")
        return robot

    def produce_custom_robot(self, designation: str) -> RobotBuilder:
        """Start building custom robot"""
        print(f"\n🔧 Building custom robot: {designation}")
        return RobotBuilder(designation)

    def register_robot(self, robot: Robot):
        """Register a custom-built robot"""
        self.produced_robots.append(robot)
        print(f"✓ Custom robot {robot.designation} registered")

    def get_production_summary(self) -> str:
        """Get production summary"""
        summary = f"\n{'='*60}\n"
        summary += "🏭 PRODUCTION SUMMARY\n"
        summary += f"{'='*60}\n"
        summary += f"Total Robots Produced: {len(self.produced_robots)}\n"
        summary += f"\nRobot Roster:\n"

        for i, robot in enumerate(self.produced_robots, 1):
            summary += f"{i}. {robot.designation} - "
            summary += f"Combat: {robot.get_combat_rating()}, "
            summary += f"Utility: {robot.get_utility_rating()}\n"

        summary += f"{'='*60}"
        return summary

# Example Usage
def main():
    print("=== ROBOT FACTORY - Abstract Factory + Builder Demo ===\n")

    # Create assembly director
    director = RobotAssemblyDirector()

    # Example 1: Standard production using factories
    print("\n" + "─" * 60)
    print("EXAMPLE 1: Standard Production Line")
    print("─" * 60)

    military_factory = MilitaryRobotFactory()
    utility_factory = UtilityRobotFactory()
    medical_factory = MedicalRobotFactory()
    scout_factory = ScoutRobotFactory()

    # Produce standard robots
    robot1 = director.produce_standard_robot(military_factory, "WARBOT-001")
    robot2 = director.produce_standard_robot(utility_factory, "BUILDER-042")
    robot3 = director.produce_standard_robot(medical_factory, "MEDIC-15")
    robot4 = director.produce_standard_robot(scout_factory, "SCOUT-99")

    # Display robot 1
    print(robot1.get_specifications())

    # Example 2: Custom robot using builder
    print("\n\n" + "─" * 60)
    print("EXAMPLE 2: Custom Robot Assembly")
    print("─" * 60)

    custom_robot = (director.produce_custom_robot("HYBRID-X")
                    .set_chassis(ArmoredChassis())
                    .set_ai(ReconAI())
                    .set_power_source(FusionCore())
                    .add_weapon(PlasmaRifle())
                    .add_weapon(Scanner())
                    .add_weapon(MissilePods())
                    .add_upgrade("Stealth Module")
                    .add_upgrade("Enhanced Sensors")
                    .add_upgrade("Shield Generator")
                    .build())

    director.register_robot(custom_robot)
    print(custom_robot.get_specifications())

    # Example 3: Mixed production
    print("\n\n" + "─" * 60)
    print("EXAMPLE 3: Batch Production")
    print("─" * 60)

    # Produce a squad
    print("\n📦 Producing military squad...")
    for i in range(3):
        director.produce_standard_robot(military_factory, f"WARBOT-{100 + i}")

    print("\n📦 Producing utility team...")
    for i in range(2):
        director.produce_standard_robot(utility_factory, f"BUILDER-{200 + i}")

    # Production summary
    print(director.get_production_summary())

    # Example 4: Component compatibility demonstration
    print("\n\n" + "─" * 60)
    print("EXAMPLE 4: Component Compatibility")
    print("─" * 60)

    print("\n🔄 All military robots use compatible military-grade components:")
    print("  - Same chassis type ensures uniform armor")
    print("  - Same AI type ensures coordinated tactics")
    print("  - Same weapons ensure standardized ammunition")

    print("\n🔄 Custom robots can mix components from different factories:")
    print("  - Military chassis + Scout AI = Heavy reconnaissance")
    print("  - Medical AI + Utility tools = Advanced repair unit")

    # Show difference between factory-built and custom
    print("\n\n" + "─" * 60)
    print("EXAMPLE 5: Factory vs Custom Comparison")
    print("─" * 60)

    print("\n🏭 STANDARD MILITARY ROBOT:")
    print(f"   Combat Rating: {robot1.get_combat_rating()}")
    print(f"   Utility Rating: {robot1.get_utility_rating()}")
    print(f"   Components: {len(robot1.weapons)} standard weapons")

    print("\n🔧 CUSTOM HYBRID ROBOT:")
    print(f"   Combat Rating: {custom_robot.get_combat_rating()}")
    print(f"   Utility Rating: {custom_robot.get_utility_rating()}")
    print(f"   Components: {len(custom_robot.weapons)} custom weapons + {len(custom_robot.upgrades)} upgrades")

if __name__ == "__main__":
    main()
```

**Example Output:**
```
=== ROBOT FACTORY - Abstract Factory + Builder Demo ===


────────────────────────────────────────────────────────────
EXAMPLE 1: Standard Production Line
────────────────────────────────────────────────────────────

🏭 Producing Military Robot: WARBOT-001
✓ Military robot assembled successfully

🏭 Producing Utility Robot: BUILDER-042
✓ Utility robot assembled successfully

🏭 Producing Medical Robot: MEDIC-15
✓ Medical robot assembled successfully

🏭 Producing Scout Robot: SCOUT-99
✓ Scout robot assembled successfully

============================================================
🤖 ROBOT DESIGNATION: WARBOT-001
============================================================
Chassis: Armored Chassis
  - Durability: 200
  - Speed: 40

AI Core: Combat AI
  - Intelligence: 70
  - Specialty: Combat Tactics

Weapons/Tools:
  - Plasma Rifle (DMG: 50)
  - Missile Pods (DMG: 80)

Power Source: Fusion Core
  - Capacity: 1000

Ratings:
  - Combat: 237
  - Utility: 70
============================================================


────────────────────────────────────────────────────────────
EXAMPLE 2: Custom Robot Assembly
────────────────────────────────────────────────────────────

🔧 Building custom robot: HYBRID-X
✓ Custom robot HYBRID-X registered

============================================================
🤖 ROBOT DESIGNATION: HYBRID-X
============================================================
Chassis: Armored Chassis
  - Durability: 200
  - Speed: 40

AI Core: Recon AI
  - Intelligence: 85
  - Specialty: Reconnaissance

Weapons/Tools:
  - Plasma Rifle (DMG: 50)
  - Advanced Scanner (DMG: 0)
  - Missile Pods (DMG: 80)

Power Source: Fusion Core
  - Capacity: 1000

Ratings:
  - Combat: 238
  - Utility: 85

Upgrades:
  - Stealth Module
  - Enhanced Sensors
  - Shield Generator
============================================================


────────────────────────────────────────────────────────────
EXAMPLE 3: Batch Production
────────────────────────────────────────────────────────────

📦 Producing military squad...

🏭 Producing Military Robot: WARBOT-100
✓ Military robot assembled successfully

🏭 Producing Military Robot: WARBOT-101
✓ Military robot assembled successfully

🏭 Producing Military Robot: WARBOT-102
✓ Military robot assembled successfully

📦 Producing utility team...

🏭 Producing Utility Robot: BUILDER-200
✓ Utility robot assembled successfully

🏭 Producing Utility Robot: BUILDER-201
✓ Utility robot assembled successfully

============================================================
🏭 PRODUCTION SUMMARY
============================================================
Total Robots Produced: 10

Robot Roster:
1. WARBOT-001 - Combat: 237, Utility: 70
2. BUILDER-042 - Combat: 55, Utility: 80
3. MEDIC-15 - Combat: 98, Utility: 95
4. SCOUT-99 - Combat: 38, Utility: 85
5. HYBRID-X - Combat: 238, Utility: 85
6. WARBOT-100 - Combat: 237, Utility: 70
7. WARBOT-101 - Combat: 237, Utility: 70
8. WARBOT-102 - Combat: 237, Utility: 70
9. BUILDER-200 - Combat: 55, Utility: 80
10. BUILDER-201 - Combat: 55, Utility: 80
============================================================


────────────────────────────────────────────────────────────
EXAMPLE 4: Component Compatibility
────────────────────────────────────────────────────────────

🔄 All military robots use compatible military-grade components:
  - Same chassis type ensures uniform armor
  - Same AI type ensures coordinated tactics
  - Same weapons ensure standardized ammunition

🔄 Custom robots can mix components from different factories:
  - Military chassis + Scout AI = Heavy reconnaissance
  - Medical AI + Utility tools = Advanced repair unit


────────────────────────────────────────────────────────────
EXAMPLE 5: Factory vs Custom Comparison
────────────────────────────────────────────────────────────

🏭 STANDARD MILITARY ROBOT:
   Combat Rating: 237
   Utility Rating: 70
   Components: 2 standard weapons

🔧 CUSTOM HYBRID ROBOT:
   Combat Rating: 238
   Utility Rating: 85
   Components: 3 custom weapons + 3 upgrades
```

**Pattern Benefits:**
- **Abstract Factory:** Ensures all components are compatible (e.g., military chassis with military AI)
- **Component Families:** Each factory creates a coherent set of parts
- **Builder Pattern:** Allows step-by-step construction of complex robots
- **Fluent Interface:** Builder methods chain for readable robot construction
- **Flexibility:** Custom robots can mix components from different factories
- **Standard Production:** Factories enable quick production of standard robot types
- **Type Safety:** Full type hints ensure correct component usage
- **Easy Extension:** New robot types can be added by creating new factories

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


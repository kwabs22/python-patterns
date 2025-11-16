# AI & NPC Behavior Patterns

## Overview
Game AI needs to be responsive, predictable, and performant. These patterns help organize NPC behavior and decision-making.

## State Pattern for AI

**Reference:** [state](../patterns/behavioral/state.py)

### Basic AI State Machine

```python
class AIState:
    """Base class for AI states"""
    def enter(self, ai):
        """Called when entering this state"""
        pass

    def exit(self, ai):
        """Called when leaving this state"""
        pass

    def update(self, ai, dt):
        """Called every frame, returns next state or None to stay"""
        return None

class IdleState(AIState):
    def enter(self, ai):
        ai.animation = "idle"
        ai.idle_timer = 0

    def update(self, ai, dt):
        ai.idle_timer += dt

        # Check for player in range
        if ai.can_see_player():
            return ChaseState()

        # Wander after idle timeout
        if ai.idle_timer > 3.0:
            return PatrolState()

        return None

class PatrolState(AIState):
    def enter(self, ai):
        ai.animation = "walk"
        ai.pick_random_patrol_point()

    def update(self, ai, dt):
        # Move toward patrol point
        if ai.reached_patrol_point():
            return IdleState()

        # Spot player
        if ai.can_see_player():
            return ChaseState()

        ai.move_toward_patrol_point(dt)
        return None

class ChaseState(AIState):
    def enter(self, ai):
        ai.animation = "run"
        ai.alert_nearby_enemies()

    def update(self, ai, dt):
        # Lost sight of player
        if not ai.can_see_player():
            return SearchState()

        # Close enough to attack
        if ai.distance_to_player() < ai.attack_range:
            return AttackState()

        ai.move_toward_player(dt)
        return None

class AttackState(AIState):
    def enter(self, ai):
        ai.animation = "attack"
        ai.attack_cooldown = 0

    def update(self, ai, dt):
        ai.attack_cooldown -= dt

        # Player moved away
        if ai.distance_to_player() > ai.attack_range:
            return ChaseState()

        # Attack when ready
        if ai.attack_cooldown <= 0:
            ai.perform_attack()
            ai.attack_cooldown = 1.0  # 1 second between attacks

        return None

class SearchState(AIState):
    """Look for player after losing sight"""
    def enter(self, ai):
        ai.animation = "walk"
        ai.last_known_position = ai.get_player_position()
        ai.search_timer = 5.0

    def update(self, ai, dt):
        ai.search_timer -= dt

        # Found player again
        if ai.can_see_player():
            return ChaseState()

        # Give up search
        if ai.search_timer <= 0:
            return PatrolState()

        ai.move_toward(ai.last_known_position, dt)
        return None

class FleeState(AIState):
    """Run away when low health"""
    def enter(self, ai):
        ai.animation = "run"
        ai.flee_timer = 3.0

    def update(self, ai, dt):
        ai.flee_timer -= dt

        # Healed or timer expired
        if ai.health > ai.max_health * 0.5 or ai.flee_timer <= 0:
            return IdleState()

        ai.move_away_from_player(dt)
        return None

class AIController:
    """Main AI controller using state pattern"""
    def __init__(self):
        self.current_state = IdleState()
        self.animation = "idle"
        self.health = 100
        self.max_health = 100
        self.attack_range = 50
        self.current_state.enter(self)

    def update(self, dt):
        # Check if we should flee (low health)
        if self.health < self.max_health * 0.3:
            if not isinstance(self.current_state, FleeState):
                self.change_state(FleeState())

        # Update current state
        next_state = self.current_state.update(self, dt)

        # Change state if needed
        if next_state:
            self.change_state(next_state)

    def change_state(self, new_state):
        self.current_state.exit(self)
        self.current_state = new_state
        self.current_state.enter(self)

    # Helper methods (would interact with game world)
    def can_see_player(self):
        # Raycast or distance check
        return False

    def distance_to_player(self):
        return 100

    def move_toward_player(self, dt):
        pass

    def perform_attack(self):
        pass
```

## Strategy Pattern for AI Behaviors

**Reference:** [strategy](../patterns/behavioral/strategy.py)

### Interchangeable AI Behaviors

```python
class AIStrategy:
    """Base strategy for AI behavior"""
    def decide_action(self, ai, game_state):
        raise NotImplementedError

class AggressiveStrategy(AIStrategy):
    """Always attack, never retreat"""
    def decide_action(self, ai, game_state):
        player = game_state.get_player()

        if ai.can_see(player):
            if ai.distance_to(player) > ai.attack_range:
                return ("move_toward", player)
            else:
                return ("attack", player)

        return ("search", ai.last_known_player_pos)

class DefensiveStrategy(AIStrategy):
    """Maintain distance, retreat when hurt"""
    def decide_action(self, ai, game_state):
        player = game_state.get_player()

        # Low health - flee
        if ai.health < ai.max_health * 0.5:
            return ("flee", player)

        if ai.can_see(player):
            distance = ai.distance_to(player)

            # Too close - back away
            if distance < ai.preferred_range * 0.8:
                return ("move_away", player)

            # Too far - move closer
            elif distance > ai.preferred_range * 1.2:
                return ("move_toward", player)

            # Just right - attack
            else:
                return ("attack", player)

        return ("patrol", None)

class SupportStrategy(AIStrategy):
    """Stay near allies, buff them, avoid combat"""
    def decide_action(self, ai, game_state):
        allies = game_state.get_allies(ai)
        player = game_state.get_player()

        # Find wounded ally
        wounded_ally = min(allies, key=lambda a: a.health) if allies else None

        if wounded_ally and wounded_ally.health < wounded_ally.max_health * 0.7:
            if ai.distance_to(wounded_ally) < ai.heal_range:
                return ("heal", wounded_ally)
            else:
                return ("move_toward", wounded_ally)

        # Stay near allies
        if allies:
            nearest_ally = min(allies, key=lambda a: ai.distance_to(a))
            if ai.distance_to(nearest_ally) > 100:
                return ("move_toward", nearest_ally)

        # Avoid player
        if ai.can_see(player) and ai.distance_to(player) < 150:
            return ("move_away", player)

        return ("patrol", None)

class Enemy:
    def __init__(self, strategy=AggressiveStrategy()):
        self.strategy = strategy
        self.health = 100
        self.max_health = 100
        self.attack_range = 50
        self.preferred_range = 100

    def set_strategy(self, strategy):
        """Change behavior at runtime"""
        self.strategy = strategy

    def update(self, game_state, dt):
        # Get decision from strategy
        action, target = self.strategy.decide_action(self, game_state)

        # Execute action
        if action == "move_toward":
            self.move_toward(target, dt)
        elif action == "move_away":
            self.move_away_from(target, dt)
        elif action == "attack":
            self.attack(target)
        elif action == "heal":
            self.heal_target(target)
        elif action == "flee":
            self.flee_from(target, dt)
        elif action == "patrol":
            self.patrol(dt)

# Usage: Create enemies with different personalities
grunt = Enemy(AggressiveStrategy())  # Rushes player
sniper = Enemy(DefensiveStrategy())  # Keeps distance
medic = Enemy(SupportStrategy())     # Heals allies

# Can change strategy dynamically
if boss.health < boss.max_health * 0.3:
    boss.set_strategy(AggressiveStrategy())  # Berserk mode!
```

## Command Pattern for AI Actions

**Reference:** [command](../patterns/behavioral/command.py)

### Queued AI Commands

```python
class AICommand:
    """Base class for AI commands"""
    def execute(self, ai):
        raise NotImplementedError

    def is_complete(self, ai):
        return True

class MoveToCommand(AICommand):
    def __init__(self, target_position):
        self.target_position = target_position
        self.threshold = 10  # How close is "close enough"

    def execute(self, ai):
        ai.move_toward_position(self.target_position)

    def is_complete(self, ai):
        distance = ai.distance_to_position(self.target_position)
        return distance < self.threshold

class AttackCommand(AICommand):
    def __init__(self, target):
        self.target = target
        self.attack_started = False

    def execute(self, ai):
        if not self.attack_started:
            ai.start_attack_animation(self.target)
            self.attack_started = True

    def is_complete(self, ai):
        return ai.attack_animation_finished()

class WaitCommand(AICommand):
    def __init__(self, duration):
        self.duration = duration
        self.elapsed = 0

    def execute(self, ai):
        self.elapsed += ai.delta_time

    def is_complete(self, ai):
        return self.elapsed >= self.duration

class PlayAnimationCommand(AICommand):
    def __init__(self, animation_name):
        self.animation_name = animation_name
        self.started = False

    def execute(self, ai):
        if not self.started:
            ai.play_animation(self.animation_name)
            self.started = True

    def is_complete(self, ai):
        return ai.is_animation_finished(self.animation_name)

class AICommandQueue:
    """Execute AI commands in sequence"""
    def __init__(self):
        self.commands = []

    def add_command(self, command):
        self.commands.append(command)

    def update(self, ai):
        if not self.commands:
            return

        # Execute current command
        current = self.commands[0]
        current.execute(ai)

        # Remove if complete
        if current.is_complete(ai):
            self.commands.pop(0)

    def clear(self):
        self.commands.clear()

    def is_idle(self):
        return len(self.commands) == 0

# Usage: Scripted AI sequence
boss_queue = AICommandQueue()

# Boss enters arena
boss_queue.add_command(MoveToCommand((400, 300)))
boss_queue.add_command(PlayAnimationCommand("roar"))
boss_queue.add_command(WaitCommand(2.0))

# Attack sequence
for _ in range(3):
    boss_queue.add_command(AttackCommand(player))
    boss_queue.add_command(WaitCommand(1.0))

# Retreat
boss_queue.add_command(MoveToCommand((400, 100)))
```

## Blackboard Pattern for Shared AI Knowledge

**Reference:** [blackboard](../patterns/other/blackboard.py)

### AI Knowledge Sharing

```python
class Blackboard:
    """Shared knowledge between AI agents"""
    def __init__(self):
        self._data = {}

    def set(self, key, value):
        self._data[key] = value

    def get(self, key, default=None):
        return self._data.get(key, default)

    def has(self, key):
        return key in self._data

    def remove(self, key):
        if key in self._data:
            del self._data[key]

# Global AI blackboard
ai_blackboard = Blackboard()

class SmartEnemy:
    def __init__(self, blackboard):
        self.blackboard = blackboard

    def update(self, dt):
        # Check if any ally has spotted the player
        if self.blackboard.has("player_last_seen"):
            position = self.blackboard.get("player_last_seen")
            self.investigate(position)

        # If I see the player, update blackboard
        if self.can_see_player():
            player_pos = self.get_player_position()
            self.blackboard.set("player_last_seen", player_pos)
            self.blackboard.set("player_last_seen_time", current_time())

            # Alert others
            self.trigger_alert()

# All enemies share knowledge
enemy1 = SmartEnemy(ai_blackboard)
enemy2 = SmartEnemy(ai_blackboard)
enemy3 = SmartEnemy(ai_blackboard)

# When one sees player, all know about it!
```

## Observer Pattern for AI Events

**Reference:** [observer](../patterns/behavioral/observer.py)

### AI Communication

```python
class AIEvent:
    """Events that AI can trigger/listen to"""
    ENEMY_DIED = "enemy_died"
    PLAYER_SPOTTED = "player_spotted"
    ALARM_TRIGGERED = "alarm_triggered"
    ALLY_HURT = "ally_hurt"

class AIObserver:
    def on_ai_event(self, event_type, data):
        raise NotImplementedError

class AIEventManager:
    def __init__(self):
        self.observers = []

    def subscribe(self, observer):
        self.observers.append(observer)

    def unsubscribe(self, observer):
        if observer in self.observers:
            self.observers.remove(observer)

    def notify(self, event_type, data=None):
        for observer in self.observers:
            observer.on_ai_event(event_type, data)

class AlertEnemy(AIObserver):
    """Enemy that responds to events"""
    def __init__(self, event_manager):
        self.event_manager = event_manager
        self.event_manager.subscribe(self)
        self.state = "patrol"

    def on_ai_event(self, event_type, data):
        if event_type == AIEvent.PLAYER_SPOTTED:
            # Another enemy spotted player
            self.state = "alert"
            self.last_known_player_pos = data["position"]

        elif event_type == AIEvent.ALARM_TRIGGERED:
            # Alarm triggered - all enemies alert
            self.state = "search"

        elif event_type == AIEvent.ALLY_HURT:
            # Ally hurt nearby
            if self.distance_to(data["position"]) < 200:
                self.state = "alert"

    def spot_player(self, player_pos):
        # Notify all other AI
        self.event_manager.notify(AIEvent.PLAYER_SPOTTED, {
            "position": player_pos,
            "spotter": self
        })

# Usage
event_manager = AIEventManager()
enemy1 = AlertEnemy(event_manager)
enemy2 = AlertEnemy(event_manager)
enemy3 = AlertEnemy(event_manager)

# When one spots player, all react
enemy1.spot_player((100, 200))
```

## Utility AI (Alternative to FSM)

```python
class Consideration:
    """Evaluates how desirable an action is"""
    def evaluate(self, ai, game_state):
        """Returns 0.0 to 1.0"""
        raise NotImplementedError

class DistanceToPlayerConsideration(Consideration):
    def __init__(self, max_distance=500):
        self.max_distance = max_distance

    def evaluate(self, ai, game_state):
        distance = ai.distance_to(game_state.player)
        # Closer = higher score
        return 1.0 - min(distance / self.max_distance, 1.0)

class HealthConsideration(Consideration):
    def evaluate(self, ai, game_state):
        # Lower health = higher score (for healing actions)
        return 1.0 - (ai.health / ai.max_health)

class Action:
    """An action the AI can take"""
    def __init__(self, name):
        self.name = name
        self.considerations = []

    def add_consideration(self, consideration, weight=1.0):
        self.considerations.append((consideration, weight))

    def calculate_score(self, ai, game_state):
        if not self.considerations:
            return 0.0

        total_score = 1.0
        for consideration, weight in self.considerations:
            score = consideration.evaluate(ai, game_state)
            # Weighted geometric mean
            total_score *= pow(score, weight)

        return total_score

    def execute(self, ai, game_state):
        raise NotImplementedError

class AttackAction(Action):
    def __init__(self):
        super().__init__("attack")
        self.add_consideration(DistanceToPlayerConsideration(max_distance=100), weight=2.0)

    def execute(self, ai, game_state):
        ai.attack(game_state.player)

class FleeAction(Action):
    def __init__(self):
        super().__init__("flee")
        self.add_consideration(HealthConsideration(), weight=3.0)
        # Flee when low health

    def execute(self, ai, game_state):
        ai.flee_from(game_state.player)

class UtilityAI:
    def __init__(self):
        self.actions = []

    def add_action(self, action):
        self.actions.append(action)

    def decide(self, ai, game_state):
        """Pick best action based on utility scores"""
        best_action = None
        best_score = -1.0

        for action in self.actions:
            score = action.calculate_score(ai, game_state)
            if score > best_score:
                best_score = score
                best_action = action

        return best_action

    def update(self, ai, game_state):
        action = self.decide(ai, game_state)
        if action:
            action.execute(ai, game_state)

# Usage
utility_ai = UtilityAI()
utility_ai.add_action(AttackAction())
utility_ai.add_action(FleeAction())
# AI automatically picks best action based on context
```

## When to Use Each Pattern

- **State Machine**: Simple AI with clear states (patrol, chase, attack)
- **Strategy**: Different AI personalities with swappable behaviors
- **Command**: Scripted sequences, cutscenes, boss patterns
- **Blackboard**: Coordinated AI that shares knowledge
- **Observer**: Event-driven AI responses
- **Utility AI**: Complex decision-making with many factors

## Resources

- [AI for Games (Ian Millington)](https://www.routledge.com/AI-for-Games/Millington-Funge/p/book/9780123747310)
- [Game AI Pro series](http://www.gameaipro.com/)
- [Behavior Trees vs FSM](https://www.gamedeveloper.com/programming/behavior-trees-for-ai-how-they-work)

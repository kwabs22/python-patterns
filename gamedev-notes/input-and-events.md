# Input Handling & Event Patterns

## Overview
Handling input and game events cleanly is essential for maintainable games. These patterns help decouple input from game logic.

## Command Pattern for Input Binding

**Reference:** [command](../patterns/behavioral/command.py)

### Rebindable Controls

```python
class InputCommand:
    """Represents an action that can be triggered by input"""
    def execute(self, actor):
        raise NotImplementedError

    def undo(self, actor):
        """Optional: for undo functionality"""
        pass

class JumpCommand(InputCommand):
    def execute(self, actor):
        if actor.is_grounded():
            actor.jump()

class MoveLeftCommand(InputCommand):
    def execute(self, actor):
        actor.move_left()

class MoveRightCommand(InputCommand):
    def execute(self, actor):
        actor.move_right()

class AttackCommand(InputCommand):
    def execute(self, actor):
        actor.attack()

class UseItemCommand(InputCommand):
    def __init__(self, item_slot):
        self.item_slot = item_slot

    def execute(self, actor):
        actor.use_item(self.item_slot)

class InputHandler:
    """Maps inputs to commands - allows rebinding"""
    def __init__(self):
        # Default key bindings
        self.key_bindings = {
            "W": JumpCommand(),
            "A": MoveLeftCommand(),
            "D": MoveRightCommand(),
            "SPACE": AttackCommand(),
            "1": UseItemCommand(0),
            "2": UseItemCommand(1),
        }

        # Gamepad bindings
        self.gamepad_bindings = {
            "BUTTON_A": JumpCommand(),
            "BUTTON_X": AttackCommand(),
            "DPAD_LEFT": MoveLeftCommand(),
            "DPAD_RIGHT": MoveRightCommand(),
        }

    def handle_keyboard(self, key, actor):
        """Execute command for key press"""
        if key in self.key_bindings:
            command = self.key_bindings[key]
            command.execute(actor)

    def handle_gamepad(self, button, actor):
        """Execute command for gamepad button"""
        if button in self.gamepad_bindings:
            command = self.gamepad_bindings[button]
            command.execute(actor)

    def rebind_key(self, old_key, new_key):
        """Allow player to rebind controls"""
        if old_key in self.key_bindings:
            command = self.key_bindings[old_key]
            del self.key_bindings[old_key]
            self.key_bindings[new_key] = command

    def set_binding(self, key, command):
        """Set a specific key to a command"""
        self.key_bindings[key] = command

# Usage
input_handler = InputHandler()
player = Player()

# Game loop
for event in get_input_events():
    if event.type == "KEY_PRESS":
        input_handler.handle_keyboard(event.key, player)

# Allow rebinding
input_handler.rebind_key("W", "SPACE")  # Jump is now SPACE
```

### Input Buffering for Fighting Games

```python
class BufferedInputHandler:
    """Buffers inputs for combo/special move detection"""
    def __init__(self, buffer_time=0.2):
        self.buffer_time = buffer_time
        self.input_buffer = []
        self.command_sequences = {}

    def register_sequence(self, name, sequence, command):
        """Register a combo/special move sequence"""
        self.command_sequences[name] = {
            "sequence": sequence,
            "command": command
        }

    def add_input(self, input_event, timestamp):
        """Add input to buffer"""
        self.input_buffer.append({
            "input": input_event,
            "time": timestamp
        })

        # Remove old inputs
        self._clean_buffer(timestamp)

    def _clean_buffer(self, current_time):
        """Remove inputs older than buffer time"""
        self.input_buffer = [
            inp for inp in self.input_buffer
            if current_time - inp["time"] <= self.buffer_time
        ]

    def check_sequences(self, actor):
        """Check if any command sequence matches buffer"""
        # Get just the input sequence
        buffered_inputs = [inp["input"] for inp in self.input_buffer]

        for name, seq_data in self.command_sequences.items():
            sequence = seq_data["sequence"]
            command = seq_data["command"]

            # Check if sequence matches end of buffer
            if len(buffered_inputs) >= len(sequence):
                recent = buffered_inputs[-len(sequence):]
                if recent == sequence:
                    command.execute(actor)
                    self.input_buffer.clear()
                    return True

        return False

class HadoukenCommand(InputCommand):
    """Quarter circle forward + punch"""
    def execute(self, actor):
        actor.perform_hadouken()

# Usage
buffered_handler = BufferedInputHandler(buffer_time=0.5)

# Register Hadouken: Down, Down-Forward, Forward, Punch
buffered_handler.register_sequence(
    "hadouken",
    ["DOWN", "DOWN_FORWARD", "FORWARD", "PUNCH"],
    HadoukenCommand()
)

# In game loop
current_time = get_time()
for event in get_input_events():
    buffered_handler.add_input(event.input, current_time)
    buffered_handler.check_sequences(player)
```

## Observer Pattern for Events

**Reference:** [observer](../patterns/behavioral/observer.py)

### Event System

```python
class Observer:
    """Base observer interface"""
    def on_notify(self, event_type, event_data):
        raise NotImplementedError

class GameEvent:
    """Event types"""
    PLAYER_DIED = "player_died"
    ENEMY_KILLED = "enemy_killed"
    ITEM_COLLECTED = "item_collected"
    LEVEL_COMPLETE = "level_complete"
    ACHIEVEMENT_UNLOCKED = "achievement_unlocked"
    DAMAGE_TAKEN = "damage_taken"

class EventManager:
    """Manages observers and notifications"""
    def __init__(self):
        self.observers = {}  # event_type -> list of observers

    def subscribe(self, event_type, observer):
        """Subscribe to specific event type"""
        if event_type not in self.observers:
            self.observers[event_type] = []
        self.observers[event_type].append(observer)

    def unsubscribe(self, event_type, observer):
        """Unsubscribe from event type"""
        if event_type in self.observers:
            if observer in self.observers[event_type]:
                self.observers[event_type].remove(observer)

    def notify(self, event_type, event_data=None):
        """Notify all observers of event"""
        if event_type in self.observers:
            for observer in self.observers[event_type]:
                observer.on_notify(event_type, event_data)

# Observers
class UIObserver(Observer):
    """Updates UI based on events"""
    def on_notify(self, event_type, event_data):
        if event_type == GameEvent.DAMAGE_TAKEN:
            self.update_health_bar(event_data["new_health"])
        elif event_type == GameEvent.ITEM_COLLECTED:
            self.show_item_notification(event_data["item_name"])
        elif event_type == GameEvent.ENEMY_KILLED:
            self.update_score(event_data["points"])

class AchievementObserver(Observer):
    """Tracks achievements"""
    def on_notify(self, event_type, event_data):
        if event_type == GameEvent.ENEMY_KILLED:
            self.increment_kill_count(event_data["enemy_type"])
            self.check_kill_achievements()
        elif event_type == GameEvent.LEVEL_COMPLETE:
            self.check_speed_run_achievement(event_data["time"])

class AudioObserver(Observer):
    """Plays sounds for events"""
    def on_notify(self, event_type, event_data):
        if event_type == GameEvent.DAMAGE_TAKEN:
            self.play_sound("hurt")
        elif event_type == GameEvent.ITEM_COLLECTED:
            self.play_sound("pickup")
        elif event_type == GameEvent.ENEMY_KILLED:
            self.play_sound("enemy_death")

class AnalyticsObserver(Observer):
    """Logs telemetry"""
    def on_notify(self, event_type, event_data):
        self.log_event(event_type, event_data)
        self.send_to_analytics_service(event_type, event_data)

# Setup
event_manager = EventManager()

ui_observer = UIObserver()
achievement_observer = AchievementObserver()
audio_observer = AudioObserver()
analytics_observer = AnalyticsObserver()

# Subscribe observers to events
event_manager.subscribe(GameEvent.DAMAGE_TAKEN, ui_observer)
event_manager.subscribe(GameEvent.DAMAGE_TAKEN, audio_observer)
event_manager.subscribe(GameEvent.ITEM_COLLECTED, ui_observer)
event_manager.subscribe(GameEvent.ITEM_COLLECTED, audio_observer)
event_manager.subscribe(GameEvent.ENEMY_KILLED, ui_observer)
event_manager.subscribe(GameEvent.ENEMY_KILLED, achievement_observer)
event_manager.subscribe(GameEvent.ENEMY_KILLED, audio_observer)

# All observers get all events
event_manager.subscribe(GameEvent.PLAYER_DIED, analytics_observer)
event_manager.subscribe(GameEvent.ENEMY_KILLED, analytics_observer)
event_manager.subscribe(GameEvent.ITEM_COLLECTED, analytics_observer)

# Trigger events
def player_take_damage(player, damage):
    player.health -= damage
    event_manager.notify(GameEvent.DAMAGE_TAKEN, {
        "damage": damage,
        "new_health": player.health
    })

def kill_enemy(enemy):
    enemy.alive = False
    event_manager.notify(GameEvent.ENEMY_KILLED, {
        "enemy_type": enemy.type,
        "points": enemy.point_value,
        "position": enemy.position
    })
```

## Publish-Subscribe Pattern

**Reference:** [publish_subscribe](../patterns/behavioral/publish_subscribe.py)

### Message Bus for Decoupled Systems

```python
class Message:
    """Base message class"""
    def __init__(self, topic, data=None):
        self.topic = topic
        self.data = data

class MessageBus:
    """Central message hub"""
    def __init__(self):
        self.subscribers = {}  # topic -> list of callbacks

    def subscribe(self, topic, callback):
        """Subscribe to a topic"""
        if topic not in self.subscribers:
            self.subscribers[topic] = []
        self.subscribers[topic].append(callback)

    def unsubscribe(self, topic, callback):
        """Unsubscribe from topic"""
        if topic in self.subscribers:
            if callback in self.subscribers[topic]:
                self.subscribers[topic].remove(callback)

    def publish(self, message):
        """Publish message to all subscribers"""
        if message.topic in self.subscribers:
            for callback in self.subscribers[topic]:
                callback(message)

    def publish_immediate(self, topic, data=None):
        """Convenience method"""
        self.publish(Message(topic, data))

# Global message bus
message_bus = MessageBus()

# Game systems subscribe
class ParticleSystem:
    def __init__(self, bus):
        bus.subscribe("explosion", self.on_explosion)
        bus.subscribe("footstep", self.on_footstep)

    def on_explosion(self, message):
        pos = message.data["position"]
        self.spawn_explosion_particles(pos)

    def on_footstep(self, message):
        pos = message.data["position"]
        self.spawn_dust_particles(pos)

class SoundSystem:
    def __init__(self, bus):
        bus.subscribe("explosion", self.on_explosion)
        bus.subscribe("footstep", self.on_footstep)

    def on_explosion(self, message):
        pos = message.data["position"]
        self.play_3d_sound("explosion", pos)

    def on_footstep(self, message):
        surface = message.data.get("surface", "default")
        self.play_sound(f"footstep_{surface}")

class CameraShake:
    def __init__(self, bus):
        bus.subscribe("explosion", self.on_explosion)
        bus.subscribe("player_hit", self.on_player_hit)

    def on_explosion(self, message):
        intensity = message.data.get("intensity", 1.0)
        self.shake_camera(intensity)

    def on_player_hit(self, message):
        self.shake_camera(0.5)

# Initialize systems
particle_system = ParticleSystem(message_bus)
sound_system = SoundSystem(message_bus)
camera_shake = CameraShake(message_bus)

# Publish events
def explode_at(position, intensity=1.0):
    message_bus.publish_immediate("explosion", {
        "position": position,
        "intensity": intensity
    })
    # Particle, sound, and camera shake all react!

def player_footstep(position, surface="grass"):
    message_bus.publish_immediate("footstep", {
        "position": position,
        "surface": surface
    })
```

## Chain of Responsibility for Input

**Reference:** [chain_of_responsibility](../patterns/behavioral/chain_of_responsibility.py)

### Input Handling Hierarchy

```python
class InputHandler:
    """Base handler in chain"""
    def __init__(self):
        self.next_handler = None

    def set_next(self, handler):
        self.next_handler = handler
        return handler  # Allow chaining

    def handle_input(self, input_event):
        """Handle input or pass to next handler"""
        if self.next_handler:
            return self.next_handler.handle_input(input_event)
        return False  # Input not handled

class UIInputHandler(InputHandler):
    """Handles UI interactions (highest priority)"""
    def __init__(self, ui_system):
        super().__init__()
        self.ui_system = ui_system

    def handle_input(self, input_event):
        # If UI is active and clicked on UI element
        if self.ui_system.is_active():
            if self.ui_system.handle_click(input_event):
                return True  # Input consumed

        # Pass to next handler
        return super().handle_input(input_event)

class PlayerInputHandler(InputHandler):
    """Handles player controls"""
    def __init__(self, player):
        super().__init__()
        self.player = player

    def handle_input(self, input_event):
        # Only handle if player can move
        if not self.player.can_move():
            return super().handle_input(input_event)

        # Handle player movement/actions
        if input_event.type == "KEY_PRESS":
            if input_event.key == "W":
                self.player.jump()
                return True
            elif input_event.key == "A":
                self.player.move_left()
                return True
            elif input_event.key == "D":
                self.player.move_right()
                return True
            elif input_event.key == "SPACE":
                self.player.attack()
                return True

        return super().handle_input(input_event)

class CameraInputHandler(InputHandler):
    """Handles camera controls (lowest priority)"""
    def __init__(self, camera):
        super().__init__()
        self.camera = camera

    def handle_input(self, input_event):
        if input_event.type == "MOUSE_MOVE":
            self.camera.rotate(input_event.delta_x, input_event.delta_y)
            return True

        if input_event.type == "SCROLL":
            self.camera.zoom(input_event.delta)
            return True

        return super().handle_input(input_event)

# Setup chain: UI -> Player -> Camera
ui_handler = UIInputHandler(ui_system)
player_handler = PlayerInputHandler(player)
camera_handler = CameraInputHandler(camera)

ui_handler.set_next(player_handler).set_next(camera_handler)

# Process input through chain
for event in get_input_events():
    ui_handler.handle_input(event)
    # If UI handles it, player and camera won't receive it
```

## Mediator Pattern for Input Coordination

**Reference:** [mediator](../patterns/behavioral/mediator.py)

### Coordinating Multiple Input Sources

```python
class InputMediator:
    """Coordinates input from multiple sources"""
    def __init__(self):
        self.keyboard_enabled = True
        self.gamepad_enabled = True
        self.touch_enabled = False

        self.active_input_method = None

    def handle_keyboard_input(self, key):
        if not self.keyboard_enabled:
            return

        # Switch active input method
        self.active_input_method = "keyboard"

        # Disable conflicting inputs temporarily
        self.gamepad_enabled = False

        # Process input
        return self._process_input("keyboard", key)

    def handle_gamepad_input(self, button):
        if not self.gamepad_enabled:
            return

        self.active_input_method = "gamepad"
        self.keyboard_enabled = False

        return self._process_input("gamepad", button)

    def handle_touch_input(self, touch_pos):
        if not self.touch_enabled:
            return

        self.active_input_method = "touch"

        return self._process_input("touch", touch_pos)

    def _process_input(self, source, data):
        # Central input processing logic
        # Translate different input types to common actions
        pass

    def reset_input_methods(self):
        """Re-enable all input methods"""
        self.keyboard_enabled = True
        self.gamepad_enabled = True
        self.touch_enabled = True
```

## Best Practices

### 1. Separate Input from Game Logic

```python
# BAD: Input directly in game logic
class Player:
    def update(self):
        if keyboard.is_pressed("W"):
            self.jump()

# GOOD: Input generates commands
class Player:
    def update(self, commands):
        for command in commands:
            command.execute(self)
```

### 2. Input Mapping Layer

```python
class InputMapper:
    """Translates raw input to game actions"""
    def __init__(self):
        self.action_states = {
            "jump": False,
            "attack": False,
            "move_x": 0.0,
            "move_y": 0.0
        }

    def update(self, raw_inputs):
        """Update action states from raw inputs"""
        self.action_states["jump"] = raw_inputs.is_pressed("SPACE")
        self.action_states["attack"] = raw_inputs.is_pressed("MOUSE_LEFT")

        move_x = 0.0
        if raw_inputs.is_pressed("A"):
            move_x -= 1.0
        if raw_inputs.is_pressed("D"):
            move_x += 1.0

        self.action_states["move_x"] = move_x

    def get_action(self, action_name):
        return self.action_states.get(action_name, 0)

# Usage
input_mapper = InputMapper()

# Game loop
raw_inputs = get_raw_input()
input_mapper.update(raw_inputs)

# Game logic uses actions, not keys
if input_mapper.get_action("jump"):
    player.jump()

player.velocity_x = input_mapper.get_action("move_x") * player.speed
```

### 3. Event Queuing

```python
class EventQueue:
    """Queue events for processing"""
    def __init__(self):
        self.queue = []

    def enqueue(self, event):
        self.queue.append(event)

    def process_all(self, handler):
        """Process all queued events"""
        while self.queue:
            event = self.queue.pop(0)
            handler(event)

    def clear(self):
        self.queue.clear()

# Collect events during frame
event_queue = EventQueue()

for event in get_input_events():
    event_queue.enqueue(event)

# Process at appropriate time
event_queue.process_all(input_handler.handle)
```

## Common Patterns Summary

| Pattern | Best For |
|---------|----------|
| **Command** | Rebindable controls, input buffering, undo/replay |
| **Observer** | Decoupled event responses (UI, audio, achievements) |
| **Pub-Sub** | System-wide events, message bus architecture |
| **Chain of Responsibility** | Input priority (UI > Player > Camera) |
| **Mediator** | Coordinating multiple input devices |

## Performance Tips

1. **Pool command objects** instead of creating new ones
2. **Limit observers** - too many can slow event processing
3. **Use event queues** to batch processing
4. **Profile event systems** - they can become bottlenecks
5. **Consider immediate vs deferred** event processing

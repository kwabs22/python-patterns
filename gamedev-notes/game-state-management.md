# Game State Management

## Overview
Managing game states (menu, playing, paused, game over) and character states is critical for organized game code.

## State Pattern for Game Flow

**Reference:** [state](../patterns/behavioral/state.py)

### Game States

```python
class GameState:
    """Base class for game states"""
    def enter(self, game):
        """Called when entering this state"""
        pass

    def exit(self, game):
        """Called when leaving this state"""
        pass

    def update(self, game, dt):
        """Update logic, returns next state or None"""
        return None

    def handle_input(self, game, event):
        """Handle input events"""
        pass

    def render(self, game):
        """Render this state"""
        pass

class MainMenuState(GameState):
    def enter(self, game):
        game.load_menu_ui()
        game.play_menu_music()

    def handle_input(self, game, event):
        if event.type == "button_click":
            if event.button == "play":
                return PlayingState()
            elif event.button == "options":
                return OptionsState()
            elif event.button == "quit":
                game.quit()
        return None

    def render(self, game):
        game.render_background("menu_bg")
        game.render_ui()

class PlayingState(GameState):
    def enter(self, game):
        if not game.level_loaded:
            game.load_level()
        game.play_level_music()
        game.resume_game_time()

    def exit(self, game):
        game.pause_game_time()

    def update(self, game, dt):
        # Update game logic
        game.update_player(dt)
        game.update_enemies(dt)
        game.update_physics(dt)

        # Check for state transitions
        if game.player.is_dead():
            return GameOverState()

        if game.level_complete():
            return LevelCompleteState()

        return None

    def handle_input(self, game, event):
        if event.type == "key_press" and event.key == "escape":
            return PausedState()

        # Pass input to player
        game.player.handle_input(event)
        return None

    def render(self, game):
        game.render_level()
        game.render_entities()
        game.render_hud()

class PausedState(GameState):
    def enter(self, game):
        game.pause_game_time()
        game.show_pause_menu()
        game.pause_audio()

    def exit(self, game):
        game.hide_pause_menu()
        game.resume_audio()

    def handle_input(self, game, event):
        if event.type == "key_press" and event.key == "escape":
            return PlayingState()

        if event.type == "button_click":
            if event.button == "resume":
                return PlayingState()
            elif event.button == "main_menu":
                return MainMenuState()
            elif event.button == "quit":
                game.quit()

        return None

    def render(self, game):
        # Render game in background (frozen)
        game.render_level()
        game.render_entities()
        # Overlay pause menu
        game.render_pause_overlay()
        game.render_ui()

class GameOverState(GameState):
    def enter(self, game):
        game.show_game_over_screen()
        game.save_high_score()
        game.play_game_over_music()

    def handle_input(self, game, event):
        if event.type == "button_click":
            if event.button == "retry":
                game.reset_level()
                return PlayingState()
            elif event.button == "main_menu":
                return MainMenuState()

        return None

    def render(self, game):
        game.render_game_over_screen()

class LevelCompleteState(GameState):
    def enter(self, game):
        game.calculate_score()
        game.unlock_next_level()
        game.save_progress()

    def handle_input(self, game, event):
        if event.type == "button_click":
            if event.button == "next_level":
                game.load_next_level()
                return PlayingState()
            elif event.button == "main_menu":
                return MainMenuState()

        return None

class Game:
    def __init__(self):
        self.current_state = MainMenuState()
        self.current_state.enter(self)
        self.running = True

    def change_state(self, new_state):
        if new_state:
            self.current_state.exit(self)
            self.current_state = new_state
            self.current_state.enter(self)

    def update(self, dt):
        next_state = self.current_state.update(self, dt)
        if next_state:
            self.change_state(next_state)

    def handle_input(self, event):
        next_state = self.current_state.handle_input(self, event)
        if next_state:
            self.change_state(next_state)

    def render(self):
        self.current_state.render(self)

    def run(self):
        while self.running:
            dt = self.get_delta_time()
            events = self.get_input_events()

            for event in events:
                self.handle_input(event)

            self.update(dt)
            self.render()
```

## Stack-Based State Management

For states that can be layered (e.g., pause menu over gameplay):

```python
class StateManager:
    """Manages a stack of game states"""
    def __init__(self):
        self.states = []

    def push(self, state):
        """Add a new state on top"""
        if self.states:
            self.states[-1].on_pause()
        self.states.append(state)
        state.on_enter()

    def pop(self):
        """Remove top state"""
        if self.states:
            state = self.states.pop()
            state.on_exit()

            if self.states:
                self.states[-1].on_resume()

    def change(self, state):
        """Replace top state"""
        if self.states:
            self.states[-1].on_exit()
            self.states.pop()

        self.states.append(state)
        state.on_enter()

    def clear(self):
        """Remove all states"""
        for state in reversed(self.states):
            state.on_exit()
        self.states.clear()

    def update(self, dt):
        """Update only the top state"""
        if self.states:
            self.states[-1].update(dt)

    def render(self):
        """Render all states (bottom to top)"""
        for state in self.states:
            state.render()

class GameState:
    def on_enter(self):
        """Called when state becomes active"""
        pass

    def on_exit(self):
        """Called when state is removed"""
        pass

    def on_pause(self):
        """Called when another state is pushed on top"""
        pass

    def on_resume(self):
        """Called when state on top is popped"""
        pass

    def update(self, dt):
        pass

    def render(self):
        pass

class PlayState(GameState):
    def on_enter(self):
        print("Entering play state")

    def on_pause(self):
        print("Pausing gameplay")

    def on_resume(self):
        print("Resuming gameplay")

    def update(self, dt):
        # Game logic runs
        pass

    def render(self):
        # Render game world
        pass

class PauseMenuState(GameState):
    def on_enter(self):
        print("Opening pause menu")

    def render(self):
        # Don't clear screen - render over gameplay
        # Render semi-transparent overlay
        # Render menu options
        pass

# Usage
state_manager = StateManager()
state_manager.push(PlayState())  # Start playing

# Player pauses
state_manager.push(PauseMenuState())  # Pause menu appears

# Player resumes
state_manager.pop()  # Back to gameplay

# Player opens inventory while playing
state_manager.push(InventoryState())  # Inventory over gameplay
```

## Character State Management

### Finite State Machine for Characters

```python
class CharacterState:
    """Base state for character"""
    def __init__(self, name):
        self.name = name

    def enter(self, character):
        pass

    def exit(self, character):
        pass

    def update(self, character, dt):
        return None  # Return next state or None

    def handle_input(self, character, input_state):
        return None

class IdleState(CharacterState):
    def __init__(self):
        super().__init__("idle")

    def enter(self, character):
        character.play_animation("idle")
        character.velocity_x = 0

    def handle_input(self, character, input_state):
        if input_state.jump:
            return JumpingState()
        if input_state.move_x != 0:
            return WalkingState()
        if input_state.attack:
            return AttackingState()
        return None

class WalkingState(CharacterState):
    def __init__(self):
        super().__init__("walking")

    def enter(self, character):
        character.play_animation("walk")

    def update(self, character, dt):
        # Apply movement
        character.velocity_x = character.walk_speed * character.facing

        return None

    def handle_input(self, character, input_state):
        if input_state.jump:
            return JumpingState()
        if input_state.move_x == 0:
            return IdleState()
        if input_state.attack:
            return AttackingState()

        # Update facing direction
        if input_state.move_x != 0:
            character.facing = input_state.move_x

        return None

class JumpingState(CharacterState):
    def __init__(self):
        super().__init__("jumping")

    def enter(self, character):
        character.play_animation("jump")
        character.velocity_y = character.jump_force
        character.play_sound("jump")

    def update(self, character, dt):
        # Check if landed
        if character.is_grounded():
            if character.velocity_x == 0:
                return IdleState()
            else:
                return WalkingState()

        return None

    def handle_input(self, character, input_state):
        # Air control
        if input_state.move_x != 0:
            character.velocity_x = character.walk_speed * input_state.move_x * 0.5
            character.facing = input_state.move_x

        if input_state.attack:
            return AirAttackState()

        return None

class AttackingState(CharacterState):
    def __init__(self):
        super().__init__("attacking")
        self.can_cancel = False

    def enter(self, character):
        character.play_animation("attack")
        character.velocity_x = 0
        self.attack_frame = 0

    def update(self, character, dt):
        self.attack_frame += 1

        # Hit frame
        if self.attack_frame == 10:
            character.spawn_hitbox()

        # Animation complete
        if character.animation_finished():
            return IdleState()

        # Allow canceling after certain frame
        if self.attack_frame > 15:
            self.can_cancel = True

        return None

    def handle_input(self, character, input_state):
        # Can't cancel early
        if not self.can_cancel:
            return None

        if input_state.jump:
            return JumpingState()

        return None

class Character:
    def __init__(self):
        self.state = IdleState()
        self.state.enter(self)

        self.velocity_x = 0
        self.velocity_y = 0
        self.facing = 1  # 1 = right, -1 = left
        self.walk_speed = 200
        self.jump_force = -500

    def change_state(self, new_state):
        if new_state:
            self.state.exit(self)
            self.state = new_state
            self.state.enter(self)

    def update(self, dt, input_state):
        # Handle input
        next_state = self.state.handle_input(self, input_state)
        if next_state:
            self.change_state(next_state)

        # Update state
        next_state = self.state.update(self, dt)
        if next_state:
            self.change_state(next_state)

        # Apply physics
        self.apply_physics(dt)

    def get_state_name(self):
        return self.state.name
```

## Memento Pattern for Save States

**Reference:** [memento](../patterns/behavioral/memento.py)

### Save/Load Game State

```python
class GameMemento:
    """Snapshot of game state"""
    def __init__(self, player_data, level_data, progress_data):
        self._player_data = player_data.copy()
        self._level_data = level_data.copy()
        self._progress_data = progress_data.copy()

    def get_player_data(self):
        return self._player_data.copy()

    def get_level_data(self):
        return self._level_data.copy()

    def get_progress_data(self):
        return self._progress_data.copy()

class SaveSystem:
    def __init__(self):
        self.save_slots = {}

    def save_game(self, slot_id, game):
        """Create and store a memento"""
        memento = game.create_memento()
        self.save_slots[slot_id] = memento
        self._write_to_disk(slot_id, memento)

    def load_game(self, slot_id, game):
        """Restore from memento"""
        if slot_id in self.save_slots:
            memento = self.save_slots[slot_id]
            game.restore_from_memento(memento)
            return True
        return False

    def _write_to_disk(self, slot_id, memento):
        # Serialize memento to file
        import json
        data = {
            "player": memento.get_player_data(),
            "level": memento.get_level_data(),
            "progress": memento.get_progress_data()
        }
        with open(f"save_{slot_id}.json", "w") as f:
            json.dump(data, f)

class Game:
    def __init__(self):
        self.player_data = {
            "health": 100,
            "position": [0, 0],
            "inventory": []
        }
        self.level_data = {
            "current_level": 1,
            "checkpoints": []
        }
        self.progress_data = {
            "levels_unlocked": [1],
            "achievements": []
        }

    def create_memento(self):
        """Create snapshot of current state"""
        return GameMemento(
            self.player_data,
            self.level_data,
            self.progress_data
        )

    def restore_from_memento(self, memento):
        """Restore state from snapshot"""
        self.player_data = memento.get_player_data()
        self.level_data = memento.get_level_data()
        self.progress_data = memento.get_progress_data()

# Usage
game = Game()
save_system = SaveSystem()

# Play game...
game.player_data["health"] = 50
game.player_data["position"] = [100, 200]

# Save
save_system.save_game(slot_id=1, game=game)

# Continue playing...
game.player_data["health"] = 0  # Player dies

# Load
save_system.load_game(slot_id=1, game=game)
# Game state restored to save point
```

## Command Pattern for Undo/Replay

**Reference:** [command](../patterns/behavioral/command.py)

### Replay System

```python
class GameCommand:
    """Recordable game action"""
    def __init__(self, frame_number):
        self.frame_number = frame_number

    def execute(self, game):
        raise NotImplementedError

class MoveCommand(GameCommand):
    def __init__(self, frame_number, player_id, direction):
        super().__init__(frame_number)
        self.player_id = player_id
        self.direction = direction

    def execute(self, game):
        player = game.get_player(self.player_id)
        player.move(self.direction)

class AttackCommand(GameCommand):
    def __init__(self, frame_number, player_id):
        super().__init__(frame_number)
        self.player_id = player_id

    def execute(self, game):
        player = game.get_player(self.player_id)
        player.attack()

class ReplaySystem:
    def __init__(self):
        self.commands = []
        self.is_recording = False
        self.is_replaying = False
        self.current_frame = 0

    def start_recording(self):
        self.commands.clear()
        self.is_recording = True
        self.current_frame = 0

    def stop_recording(self):
        self.is_recording = False

    def record_command(self, command):
        if self.is_recording:
            command.frame_number = self.current_frame
            self.commands.append(command)

    def start_replay(self, game):
        self.is_replaying = True
        self.current_frame = 0
        game.reset()  # Reset to initial state

    def update_replay(self, game):
        if not self.is_replaying:
            return

        # Execute all commands for this frame
        for command in self.commands:
            if command.frame_number == self.current_frame:
                command.execute(game)

        self.current_frame += 1

        # End of replay
        if self.current_frame > max(c.frame_number for c in self.commands):
            self.is_replaying = False

# Usage for "replay your death" feature
replay = ReplaySystem()
replay.start_recording()

# During gameplay
player_input = get_input()
move_cmd = MoveCommand(0, player_id=1, direction=player_input.direction)
move_cmd.execute(game)
replay.record_command(move_cmd)

# Player dies
if player.is_dead():
    replay.stop_recording()
    # Show replay of last 5 seconds
    replay.start_replay(game)
```

## Best Practices

1. **Keep States Independent**: Each state should be self-contained
2. **Clear Transitions**: Define what triggers state changes
3. **State Entry/Exit**: Use enter/exit for setup/cleanup
4. **Avoid Deep Hierarchies**: Keep state machine flat when possible
5. **Consider Stack-Based**: For overlay states (menus, dialogs)
6. **Test State Transitions**: Ensure all paths work correctly

## Common Pitfalls

- **Forgetting to initialize**: Always call enter() when changing states
- **Memory leaks**: Clean up in exit() method
- **Complex transitions**: Keep transition logic simple
- **Too many states**: Consider grouping or using sub-states

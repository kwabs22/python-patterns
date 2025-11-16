# Game Mechanics Analysis

This document analyzes the unique game mechanics for each of the 25 game scene examples, showing how design patterns enable specific gameplay systems.

---

## 1. Cyberpunk Hacking - Command Pattern

### Core Gameplay Loop
Player executes hacking commands against corporate systems, trying to extract data before ICE (security) detects them.

### Unique Mechanics

**1. ICE Level System**
- Security alert level that rises with each action
- Creates tension and time pressure
- Forces risk/reward decisions

**2. Undo/Rewind Mechanic**
- If detected, player can undo recent hacks
- Limited number of rewinds per mission
- Tactical resource management

**3. Command Sequencing**
- Some hacks only work after others (dependencies)
- Order matters: bypass firewall → decrypt data → upload virus
- Puzzle-like planning element

**4. Replay System**
- Record successful hack sequences
- Replay them in future missions
- Speedrunning optimization

### Pattern Enablement
- **Command Pattern**: Each hack is a reversible command
- **Memento Pattern**: Save states for rewinding
- Commands store their previous state for undo
- Command history enables replay feature

### Player Decision Points
- Which firewall to breach first?
- When to use limited rewind charges?
- Accept high ICE risk for valuable data?
- Which commands to chain together?

---

## 2. Pirate Ship Battle - Strategy Pattern

### Core Gameplay Loop
Naval combat where players command a ship, choosing battle tactics against enemy vessels.

### Unique Mechanics

**1. Wind Direction System**
- Wind affects ship speed and maneuverability
- Must position relative to wind for advantages
- "Crossing the T" and other naval tactics

**2. Range-Based Combat**
- Different weapons effective at different ranges
- Cannons: 100-200 yards
- Ramming: <20 yards
- Sniping: 300-400 yards

**3. Ship Component Damage**
- Hull integrity (sinking)
- Sail integrity (mobility)
- Crew morale (effectiveness)
- Each affects different capabilities

**4. Dynamic Strategy Switching**
- Ship can change tactics mid-battle
- Broadside → Ramming when hull damaged
- Flee when crew morale low
- Adaptive AI opponents

**5. Boarding Mechanic**
- Close-range action to capture ships
- Crew vs crew combat mini-game
- Risk/reward: capture ship vs lose crew

### Pattern Enablement
- **Strategy Pattern**: Swappable battle tactics
- **Observer Pattern**: Crew/systems react to damage
- Each ship can change behavior dynamically
- Strategies make decisions based on context

### Player Decision Points
- Which battle strategy to employ?
- When to switch from offense to defense?
- Risk ramming for big damage vs safe distance?
- Target enemy hull or sails first?

---

## 3. Wizard Tower Defense - Factory Pattern

### Core Gameplay Loop
Summon magical creatures to defend tower from waves of enemies.

### Unique Mechanics

**1. Mana Economy**
- Mana regenerates over time
- Different summons have different costs
- Resource management is core challenge

**2. Minion Synergies**
- Ice Golem slows → Lightning Wisp chains easier
- Fire Elemental weakens → Earth Guardian tanks
- Arcane Sentinel buffs nearby allies
- Composition matters, not just quantity

**3. Special Ability Timing**
- Each minion has powerful cooldown ability
- Ice Nova freeze
- Chain Lightning
- Fire Elemental explosion
- Manual activation adds skill element

**4. Positional Strategy**
- Where you summon matters
- Tanks in front, DPS behind
- Area denial with Fire Elementals
- Chokepoint control

**5. Wave Progression**
- Enemies get stronger each wave
- Need to adapt composition
- Flying enemies require anti-air (Lightning Wisps)
- Boss waves need specialized builds

### Pattern Enablement
- **Factory Pattern**: Create different minion types on demand
- **Pool Pattern**: Reuse minion objects for performance
- Factory allows easy addition of new minion types
- Costs/stats defined per minion class

### Player Decision Points
- Which minions to summon given mana?
- Save mana for expensive tank or spam cheap DPS?
- When to use special abilities?
- Composition for upcoming wave type?

---

## 4. Space Station Emergency - State Pattern

### Core Gameplay Loop
Manage cascading emergencies on a space station, making decisions to prevent total failure.

### Unique Mechanics

**1. State Transitions**
- Station moves between emergency states
- Minor → Major → Critical → Destroyed
- Some states can be recovered, others are fatal
- Creates escalating tension

**2. Resource Depletion**
- Life Support drains during breaches
- Power fails without repairs
- Oxygen consumed by crew
- Multiple resources to balance

**3. Emergency Triage**
- Multiple simultaneous problems
- Must prioritize: fix power or seal breach?
- Some choices lock out others
- Moral dilemmas (seal section with crew inside?)

**4. Repair Progress System**
- Repairs take time
- Can be interrupted by new emergencies
- Risk letting one problem worsen to fix another

**5. Cascade Failures**
- Fire spreads to life support
- Power failure disables airlocks
- One system failure triggers others
- Systemic complexity

**6. Evacuation Timing**
- When to give up and evacuate?
- Partial evacuation to save some crew?
- Balance saving crew vs saving station

### Pattern Enablement
- **State Pattern**: Each emergency is a distinct state
- **Observer Pattern**: Systems react to failures
- States define available actions
- Transitions model escalation

### Player Decision Points
- Which emergency to address first?
- Sacrifice crew to save station?
- Risk quick fix vs safe but slow repair?
- When to abandon station?

---

## 5. Time-Traveling Detective - Memento Pattern

### Core Gameplay Loop
Investigate a mystery by traveling to different time points, gathering clues, and rewinding when stuck.

### Unique Mechanics

**1. Timeline Anchoring**
- Create save points at key moments
- Can return to any anchor
- Strategic anchor placement matters
- Limited anchors add strategy

**2. Persistent Knowledge**
- Clues carry across timelines
- Detective remembers even after rewind
- Metaprogression through loops
- Information is the real currency

**3. Branching Timelines**
- Different actions in past create different futures
- Talk to butler early → he's suspicious later
- Search office → find business card → new dialogue
- Butterfly effect mechanics

**4. NPC State Tracking**
- Each NPC has location, alive/dead, suspicious state
- States change based on player actions
- Complex relationship web

**5. Evidence Collection**
- Physical evidence (knife, card)
- Testimonial evidence (interviews)
- Environmental evidence (muddy footprints)
- Need multiple evidence types for accusation

**6. Accusation System**
- Requires specific clue combinations
- Wrong accusation = case goes cold
- Must explore multiple timelines to gather all clues

### Pattern Enablement
- **Memento Pattern**: Save/restore complete game state
- **Command Pattern**: Record actions for replay
- State includes NPC positions, inventory, clues
- Can rewind time while keeping meta-knowledge

### Player Decision Points
- When to create a timeline anchor?
- Which timeline to explore next?
- Is evidence sufficient for accusation?
- Which suspects to interview in which order?

---

## 6. Kitchen Nightmare - Observer Pattern

### Core Gameplay Loop
Manage a restaurant kitchen where multiple stations must coordinate to fulfill orders.

### Unique Mechanics

**1. Order Broadcasting**
- Single order goes to all stations
- Each station filters for relevant items
- Parallel processing of components
- Coordination challenge

**2. Station Queues**
- Each station has its own work queue
- Bottlenecks at slow stations
- Must balance station speeds
- Queue management is core skill

**3. Timing Synchronization**
- All items must finish together
- Steak takes 5 min, salad takes 1 min
- Cold food reheated, hot food cooled
- Timing windows for quality

**4. Rush Hour Mode**
- Dramatically increased order rate
- Stations work faster but more mistakes
- Stress mechanic affects quality
- Time pressure intensifies

**5. Quality Degradation**
- Food quality drops if delayed
- Affects customer satisfaction
- Balance speed vs quality
- Perfect timing = bonuses

**6. Station Upgrades**
- Improve grill speed
- Add extra fry baskets
- Better prep tools
- Optimization metagame

### Pattern Enablement
- **Observer Pattern**: Stations subscribe to order events
- **Chain of Responsibility**: Orders pass through stations
- Decoupled communication between systems
- Easy to add new station types

### Player Decision Points
- Which station to upgrade first?
- Prioritize complex or simple orders?
- When to trigger rush mode bonuses?
- Sacrifice quality for speed?

---

## 7. Dungeon Ecosystem - Composite Pattern

### Core Gameplay Loop
Explore a living dungeon where rooms, floors, and the entire complex can be treated uniformly.

### Unique Mechanics

**1. Nested Threat Calculation**
- Query any level: monster, room, floor, dungeon
- Same interface at all scales
- Helps player gauge difficulty
- Risk assessment tool

**2. Composite Operations**
- Buff entire floor's monsters
- Curse all rooms in wing
- Operations propagate down tree
- Systemic interactions

**3. Dynamic Restructuring**
- Rooms can be added/removed
- Floor layout changes
- Dungeon is mutable structure
- Roguelike generation

**4. Visitor-Based Systems**
- Map generation visitor
- Loot collection visitor
- Combat simulation visitor
- Different traversals for different purposes

**5. Hierarchical Search**
- Find all dragons in dungeon
- Locate all traps on floor
- Search at any granularity
- Efficient queries

**6. Threat Levels**
- Individual monsters have threat
- Rooms sum contained threats
- Floors sum room threats
- Guides player progression

### Pattern Enablement
- **Composite Pattern**: Uniform interface for all dungeon elements
- **Visitor Pattern**: Different operations on same structure
- Tree structure for hierarchy
- Recursive operations

### Player Decision Points
- Which floor to explore first?
- Clear easy rooms or hard rooms?
- Risk high-threat room for loot?
- When to retreat to safety?

---

## 8. Zombie Survival - Object Pool Pattern

### Core Gameplay Loop
Survive waves of zombies using pooled resources (bullets, zombies) for performance.

### Unique Mechanics

**1. Wave Spawning**
- Zombies spawn from pool, not created
- Wave difficulty scales with number
- Spawn rate increases over time
- Pressure builds gradually

**2. Enemy Type Variety**
- Walkers: slow, tanky
- Runners: fast, fragile
- Tanks: very slow, massive HP
- Crawlers: hard to hit
- Different counter-strategies

**3. Ammo Scarcity**
- Limited bullets per wave
- Must make shots count
- Reload between waves
- Resource management

**4. Combo System**
- Headshots give bonus points
- Kill streaks multiply score
- Accuracy affects combo
- Skill-based scoring

**5. Pool Exhaustion**
- Pool can run out of zombies/bullets
- Warnings when pool stressed
- Encourages smart pooling
- Performance-aware gameplay

**6. Bullet Collision**
- Each bullet checks all zombies
- Spatial partitioning for performance
- Physics optimization
- Technical challenge exposed as mechanic

### Pattern Enablement
- **Pool Pattern**: Reuse zombie/bullet objects
- **Observer Pattern**: Events for kills, hits
- No allocations during gameplay
- Predictable performance

### Player Decision Points
- Prioritize which zombie type?
- Conserve ammo or shoot freely?
- Go for combo or safe kills?
- Position for crowd control?

---

## 9. Musical Rhythm Game - Command + Observer

### Core Gameplay Loop
Hit notes in time with music, building combos for high scores.

### Unique Mechanics

**1. Timing Windows**
- Perfect: <50ms
- Good: <100ms
- OK: <150ms
- Miss: >150ms
- Skill-based precision

**2. Combo System**
- Consecutive hits build combo
- Combo multiplies score
- Break on miss
- Risk/reward tension

**3. Judgment Feedback**
- Visual/audio feedback per note
- Helps player improve timing
- Learning through play
- Skill progression

**4. Combo Milestones**
- 10 combo: stage lights
- 25 combo: crowd cheer
- 50 combo: fireworks
- 100 combo: legendary status
- Motivational targets

**5. Score Multiplier**
- Base score per judgment
- Multiplied by combo count
- Exponential scaling
- Incentivizes perfect play

**6. Song Difficulty Curves**
- Slow intro
- Build to chorus
- Hard sections
- Rest periods
- Dynamic pacing

### Pattern Enablement
- **Command Pattern**: Each beat is a command
- **Observer Pattern**: UI/effects react to hits
- Commands judge timing accuracy
- Observers trigger combo effects

### Player Decision Points
- Focus on perfects or just hit notes?
- Risk hard sections for combo?
- When to use special abilities (if any)?
- Play safe or go for high score?

---

## 10. Garden Simulator - Decorator Pattern

### Core Gameplay Loop
Grow plants by stacking enhancement decorators for optimal yields.

### Unique Mechanics

**1. Decorator Stacking**
- Each decorator adds/multiplies stats
- Order sometimes matters
- Diminishing returns on some
- Optimization puzzle

**2. Growth Rate System**
- Base rate modified by decorators
- Water: additive
- Fertilizer: multiplicative
- Sunlight: time-based
- Complex interactions

**3. Resource Investment**
- Decorators cost money/resources
- Over-invest = wasted resources
- Under-invest = slow growth
- Economic optimization

**4. Plant Lifecycle**
- Seedling → Growing → Mature → Harvesting
- Decorators effective at different stages
- Timing when to apply enhancements
- Strategic enhancement

**5. Value Calculation**
- Base plant value
- Enhanced by decorators
- Quality tiers (normal, great, perfect)
- Market prices fluctuate

**6. Seasonal Effects**
- Sunlight varies by season
- Water needs change
- Temperature affects growth
- Time-based strategy

### Pattern Enablement
- **Decorator Pattern**: Stack enhancements on plants
- **Observer Pattern**: React to growth stages
- Decorators modify behavior transparently
- Easy to add new enhancement types

### Player Decision Points
- Which enhancements to apply?
- When to harvest vs keep growing?
- Invest heavily in few plants or spread thin?
- Which plant varieties to grow?

---

## 11. Trading Card Game - Prototype Pattern

### Core Gameplay Loop
Build deck by cloning card prototypes, then play cards in strategic battles.

### Unique Mechanics

**1. Card Cloning**
- All cards cloned from prototypes
- Modify clones without affecting template
- Deck building from registry
- Efficient card creation

**2. Mana Curve**
- Cards cost different mana amounts
- Balance cheap and expensive cards
- Deck construction strategy
- Resource curve optimization

**3. Card Types**
- Creatures: persistent board presence
- Spells: instant effects
- Enchantments: ongoing effects
- Type interactions

**4. Combat System**
- Creatures attack/block
- Health and damage
- Abilities (flying, first strike)
- Tactical positioning

**5. Deck Archetypes**
- Aggro: cheap creatures
- Control: removal spells
- Combo: specific card interactions
- Strategic diversity

**6. Limited Resources**
- Mana per turn
- Cards in hand
- Deck size limit
- Constraint-based decisions

### Pattern Enablement
- **Prototype Pattern**: Clone cards from templates
- **Factory Pattern**: Create cards by ID
- Registry stores all card templates
- Deep copy for card instances

### Player Decision Points
- Which cards to include in deck?
- Play creature or hold for next turn?
- Use removal now or save it?
- Attack or hold back blockers?

---

## 12. Stealth Espionage - Chain of Responsibility

### Core Gameplay Loop
Infiltrate facility by bypassing layered security systems.

### Unique Mechanics

**1. Security Layers**
- Motion detectors
- Cameras
- Guards
- Biometric scanners
- Laser grids
- Sequential challenges

**2. Gadget System**
- Each gadget counters specific security
- Motion dampener for motion sensors
- Invisibility cloak for cameras
- Guard uniform for guards
- Limited inventory slots

**3. Detection Thresholds**
- Each layer has sensitivity
- Can partially bypass some
- Full detection triggers alarm
- Stealth rating system

**4. Patrol Patterns**
- Guards follow routes
- Timing-based challenges
- Windows of opportunity
- Spatial puzzle

**5. Alert Escalation**
- Suspicious activity raises alert
- Higher alert = tighter security
- Can lower alert by hiding
- Tension system

**6. Multiple Paths**
- Vent system (avoids cameras)
- Front door (needs disguise)
- Rooftop (requires climbing)
- Replayability through routes

### Pattern Enablement
- **Chain of Responsibility**: Security layers chain together
- **State Pattern**: Agent has state (seen, hidden)
- Each layer can detect or pass along
- Order of layers matters

### Player Decision Points
- Which route to take?
- Which gadgets to bring?
- Risk fast route with high detection?
- When to use consumable gadgets?

---

## 13. Theme Park Tycoon - Builder Pattern

### Core Gameplay Loop
Design and build custom roller coasters using fluent builder interface.

### Unique Mechanics

**1. Modular Construction**
- Track sections (loop, drop, turn)
- Cars (design affects capacity)
- Effects (fire, water, lights)
- Component-based design

**2. Rating System**
- Excitement: how thrilling
- Fear: how scary
- Nausea: how sick-inducing
- Balance ratings for target audience

**3. Track Physics**
- Loops need minimum speed
- Drops increase speed
- Turns create g-forces
- Realistic constraints

**4. Guest Preferences**
- Families want low fear
- Thrill-seekers want high excitement
- Market segments
- Target audience design

**5. Pre-built Templates**
- Family coaster template
- Thrill coaster template
- Water ride template
- Customization from base

**6. Economic Simulation**
- Build cost vs revenue
- More complex = more expensive
- Higher ratings = more riders
- Profit optimization

### Pattern Enablement
- **Builder Pattern**: Fluent interface for construction
- **Composite Pattern**: Coasters contain sections
- Director provides templates
- Builder validates constraints

### Player Decision Points
- Template or custom build?
- Maximize one rating or balance all?
- Expensive effects worth the cost?
- Target audience: families or thrill-seekers?

---

## 14. Mech Battle Arena - Composite + Strategy

### Core Gameplay Loop
Build mechs from components and use battle strategies in arena combat.

### Unique Mechanics

**1. Component System**
- Weapons (guns, missiles, melee)
- Armor (heavy, light, shield)
- Engine (speed, power)
- Customization depth

**2. Weight Classes**
- Light: fast, fragile
- Medium: balanced
- Heavy: slow, tanky
- Strategic tradeoffs

**3. Loadout Limits**
- Weight limit
- Power consumption limit
- Slot restrictions
- Constraint-based building

**4. Heat Management**
- Weapons generate heat
- Overheating shuts down
- Heat sinks dissipate
- Resource management in combat

**5. Strategy Modes**
- Brawler: close range aggro
- Sniper: long range control
- Hit & Run: mobility focus
- Defensive: turtle mode
- Switch mid-battle

**6. Damage Localization**
- Target specific components
- Destroy weapons to disarm
- Destroy legs to immobilize
- Tactical targeting

### Pattern Enablement
- **Composite Pattern**: Mech composed of parts
- **Strategy Pattern**: Swappable battle AI
- Parts contribute to whole
- Calculate total stats from components

### Player Decision Points
- Which components to equip?
- Specialize or generalize?
- Which strategy for opponent type?
- Target arms, legs, or torso?

---

## 15. Haunted Mansion - Visitor Pattern

### Core Gameplay Loop
Investigate haunted mansion with different ghost-hunting equipment.

### Unique Mechanics

**1. Equipment Types**
- EMF Detector (electromagnetic)
- IR Camera (thermal)
- Spirit Box (audio)
- UV Light (ectoplasm)
- Different data per tool

**2. Ghost Evidence**
- Each ghost type has signature
- Need multiple evidence types
- Cross-reference to identify
- Deduction puzzle

**3. Room Investigation**
- Each room can be scanned
- Different rooms have different evidence
- Must explore entire mansion
- Spatial exploration

**4. Ghost Behavior**
- Ghosts react to investigation
- Can become aggressive
- Sanity drain near ghosts
- Risk/reward for evidence

**5. Evidence Journal**
- Record findings
- Track what's been found
- Narrow down ghost type
- Detective work

**6. Time Pressure**
- Sanity depletes over time
- Must find evidence before insane
- Rushed vs thorough investigation
- Time management

### Pattern Enablement
- **Visitor Pattern**: Each tool visits rooms differently
- **Observer Pattern**: Ghosts react to investigation
- Same room structure, different operations
- Easy to add new equipment

### Player Decision Points
- Which tool to use in which room?
- Risk staying for more evidence?
- Which ghost type is it?
- When to flee vs investigate?

---

## 16. Race Car Pit Stop - Chain + Observer

### Core Gameplay Loop
Manage F1-style pit crew executing sequential tasks rapidly.

### Unique Mechanics

**1. Task Chaining**
- Tire change → Refuel → Wing adjust → Go
- Must complete in order
- Each task has duration
- Sequential optimization

**2. Crew Management**
- Different crew member skills
- Assign best person per task
- Training improves speed
- Resource management

**3. Timing Optimization**
- Pit stop time directly affects race
- Every second counts
- Perfect execution = bonus time
- High-pressure minigame

**4. Parallel Operations**
- Front and rear tires simultaneously
- Left and right sides
- Multi-threading concept
- Efficiency challenge

**5. Damage Assessment**
- Inspect car for damage
- Decide what to repair
- Trade repair time vs performance
- Strategic decisions under pressure

**6. Fuel Strategy**
- How much fuel to add?
- More fuel = heavier car
- Less fuel = risk running out
- Strategic calculation

### Pattern Enablement
- **Chain of Responsibility**: Tasks handled in sequence
- **Observer Pattern**: Race team watches pit stop
- Each handler does its job, passes along
- Clear task delegation

### Player Decision Points
- Which crew member for which task?
- Repair damage or skip for speed?
- How much fuel to add?
- Risk quick tire change or safe slow one?

---

## 17. Pet Monster Ranch - Prototype + Observer

### Core Gameplay Loop
Breed and raise monsters by cloning parents with trait variations.

### Unique Mechanics

**1. Breeding System**
- Combine two parent monsters
- Child cloned from one parent
- Inherits mixed traits
- Genetic-like system

**2. Trait Inheritance**
- Color, size, ability, stats
- Random selection from parents
- Dominant/recessive traits
- Breeding optimization

**3. Mutation System**
- 10% chance rare ability
- Completely new traits
- Breeding for mutations
- RNG excitement

**4. Growth Stages**
- Egg → Baby → Teen → Adult
- Different care at each stage
- Affects final stats
- Time investment

**5. Monster Care**
- Feed, play, train
- Affects happiness and stats
- Neglect causes problems
- Tamagotchi-like mechanics

**6. Competitive Battling**
- Enter monsters in competitions
- Stats determine outcomes
- Prize money for winners
- Goal for breeding

### Pattern Enablement
- **Prototype Pattern**: Clone parent monsters
- **Observer Pattern**: React to growth stages
- Deep copy with modifications
- Genetic algorithm simulation

### Player Decision Points
- Which monsters to breed?
- Keep offspring or sell?
- Focus on stats or rare abilities?
- When to enter competitions?

---

## 18. Heist Planning - Command + Memento

### Core Gameplay Loop
Plan elaborate heists step-by-step with checkpoints for when things go wrong.

### Unique Mechanics

**1. Heist Sequencing**
- Plan steps in advance
- Execute in order
- Each step has success chance
- Strategic planning

**2. Checkpoint System**
- Create save points mid-heist
- Restore if detected
- Limited checkpoints
- Risk management tool

**3. Detection Chance**
- Each action has risk %
- Accumulates over heist
- Higher skill = lower risk
- Probability management

**4. Crew Management**
- Different specialists (hacker, driver, muscle)
- Assign to steps
- Specialist bonuses
- Team composition matters

**5. Contingency Planning**
- Plan A and Plan B
- Switch if detected
- Alternative routes
- Adaptive strategy

**6. Loot Grading**
- Perfect heist = all loot
- Detected = less loot
- Injured crew = medical costs
- Score based on execution

### Pattern Enablement
- **Command Pattern**: Each heist step is a command
- **Memento Pattern**: Checkpoints save state
- Can undo to checkpoint
- Replay successful heists

### Player Decision Points
- Which steps to include?
- Where to place checkpoints?
- Which specialist for which task?
- Risk high-reward or safe approach?

---

## 19. Food Truck Simulator - Strategy + State

### Core Gameplay Loop
Run a food truck, switching recipes and managing operational states.

### Unique Mechanics

**1. Recipe Strategies**
- Different food types
- Each has prep time, ingredients, profit
- Switch recipes based on demand
- Strategic menu management

**2. Truck States**
- Closed (prep time)
- Open (serving)
- Moving (travel)
- Broken (repairs)
- State transitions affect gameplay

**3. Location System**
- Different locations, different crowds
- Office park: lunch rush
- Park: families
- Events: varied
- Location affects demand

**4. Ingredient Management**
- Buy ingredients in bulk
- Spoilage over time
- Balance supply and demand
- Inventory optimization

**5. Reputation System**
- Quality affects reputation
- Reputation affects customers
- Bad food = bad reviews
- Long-term consequences

**6. Weather Effects**
- Rain reduces customers
- Heat increases drink sales
- Adapt to conditions
- Environmental challenge

### Pattern Enablement
- **Strategy Pattern**: Different recipe algorithms
- **State Pattern**: Truck operational states
- **Observer Pattern**: Weather affects sales
- Swappable behaviors

### Player Decision Points
- Which recipe to serve where?
- How much ingredients to buy?
- Where to park today?
- Sacrifice quality for speed?

---

## 20. Ancient Library Quest - Iterator + Composite

### Core Gameplay Loop
Navigate nested library structure searching for ancient knowledge.

### Unique Mechanics

**1. Hierarchical Structure**
- Library → Sections → Shelves → Books
- Nested organization
- Complex topology
- Exploration challenge

**2. Iterator Types**
- Depth-first (thorough)
- Breadth-first (shallow)
- Random (chaos)
- Different search patterns

**3. Book Properties**
- Title, author, topic, magic
- Search by properties
- Pattern matching
- Query system

**4. Restricted Sections**
- Locked areas
- Require keys/permissions
- Higher-value knowledge
- Gated progression

**5. Knowledge Collection**
- Spells learned from books
- Lore for quests
- Recipes for crafting
- Information as currency

**6. Librarian AI**
- Helps narrow search
- Gives hints
- Can be bribed
- NPC interaction

### Pattern Enablement
- **Iterator Pattern**: Traverse library structure
- **Composite Pattern**: Nested sections
- **Visitor Pattern**: Different search algorithms
- Uniform traversal interface

### Player Decision Points
- Which search pattern to use?
- Explore thoroughly or quickly?
- Risk restricted section?
- Which knowledge to prioritize?

---

## 21. Submarine Exploration - State + Observer

### Core Gameplay Loop
Explore ocean depths while managing submarine systems that change with depth.

### Unique Mechanics

**1. Depth Pressure System**
- Deeper = more pressure
- Affects hull integrity
- Limits maximum depth
- Progressive challenge

**2. State-Based Abilities**
- Can't fire torpedoes on surface
- Can't surface from deep without decompression
- Abilities locked/unlocked by depth
- Context-sensitive actions

**3. Oxygen Management**
- Limited oxygen supply
- Consumption increases in deep water
- Must return to surface
- Time pressure mechanic

**4. Sonar System**
- Reveals environment
- Active sonar attracts hostiles
- Passive sonar is slow
- Risk/reward trade-off

**5. Marine Life Encounters**
- Fish schools (harmless)
- Whales (avoid)
- Hostile creatures (combat)
- Environmental hazards

**6. Discovery Rewards**
- Shipwrecks to explore
- Treasure to collect
- Scientific specimens
- Exploration incentive

### Pattern Enablement
- **State Pattern**: Depth states change behavior
- **Observer Pattern**: Systems react to depth changes
- State transitions enforce decompression
- Context-aware mechanics

### Player Decision Points
- How deep to dive?
- Active or passive sonar?
- Explore or return to surface?
- Engage hostile or flee?

---

## 22. Magical Potion Brewing - Builder + Decorator

### Core Gameplay Loop
Craft potions step-by-step, then enhance with magical decorators.

### Unique Mechanics

**1. Recipe Building**
- Base potion type
- Add ingredients sequentially
- Order affects outcome
- Crafting puzzle

**2. Ingredient Properties**
- Dragon Scale: fire resistance
- Phoenix Feather: resurrection
- Mandrake Root: confusion
- Combination effects

**3. Potion Potency**
- Ingredient quality affects power
- Rare ingredients = stronger effects
- Diminishing returns
- Quality vs quantity

**4. Decorator Enhancements**
- Duration extension
- Effect doubling
- Permanence
- Stack multiple decorators

**5. Experimentation System**
- Try unknown combinations
- Discover new recipes
- Risk explosions
- Research mechanic

**6. Potion Market**
- Sell potions for profit
- Prices vary by demand
- Rare recipes worth more
- Economic gameplay

### Pattern Enablement
- **Builder Pattern**: Step-by-step potion construction
- **Decorator Pattern**: Add magical effects
- Fluent interface for recipe
- Layers of enhancement

### Player Decision Points
- Which ingredients to use?
- Experiment or follow recipe?
- Which decorators to apply?
- Sell or keep rare potions?

---

## 23. Gladiator Arena - Strategy + Observer

### Core Gameplay Loop
Fight in Roman arena using different combat strategies while crowd watches.

### Unique Mechanics

**1. Fighting Styles**
- Retiarius: net and trident
- Murmillo: sword and shield
- Thraex: curved sword
- Secutor: heavy armor
- Style determines tactics

**2. Weapon/Armor Trade-offs**
- Heavy armor: slow but protected
- Light armor: fast but vulnerable
- Weapons: reach vs damage
- Equipment strategy

**3. Stamina System**
- Actions consume stamina
- Heavy attacks cost more
- Must manage throughout fight
- Resource in combat

**4. Crowd Favor**
- Crowd excitement affects rewards
- Flashy moves = more favor
- Crowd can grant mercy
- Popularity mechanic

**5. Wound System**
- Light wounds: minor penalty
- Heavy wounds: major penalty
- Bleeding damage over time
- Injury management

**6. Glory Points**
- Win matches for glory
- Buy better equipment
- Unlock new styles
- Progression system

### Pattern Enablement
- **Strategy Pattern**: Different fighting styles
- **Observer Pattern**: Crowd reacts to combat
- **State Pattern**: Gladiator has stamina/health states
- Swappable combat AI

### Player Decision Points
- Which fighting style to train?
- Heavy attack or light attack?
- Play to crowd or play to win?
- Risk injury for glory?

---

## 24. Time Loop Mystery - Memento + Observer

### Core Gameplay Loop
Solve mystery by repeating same day, retaining knowledge across loops.

### Unique Mechanics

**1. Loop Reset**
- Die or midnight = loop restarts
- World state resets
- Player knowledge persists
- Metaprogression

**2. Knowledge Accumulation**
- Learn NPC schedules
- Discover secret locations
- Uncover passwords/codes
- Information permanence

**3. Changing Actions**
- Different choices each loop
- See consequences
- Find optimal path
- Experimentation encouraged

**4. Time Limits**
- Must accomplish goals before midnight
- Routing optimization
- Speedrunning elements
- Pressure mechanic

**5. NPC Tracking**
- NPCs follow schedules
- Interrupt at key moments
- Butterfly effects
- Systemic interactions

**6. Mystery Pieces**
- Each loop reveals one piece
- Must connect pieces across loops
- Deduction puzzle
- Persistence required

### Pattern Enablement
- **Memento Pattern**: Save state at loop start
- **Observer Pattern**: Track what changes
- **Command Pattern**: Record player actions
- Perfect loop replay

### Player Decision Points
- What to investigate this loop?
- Which NPC to follow?
- When to trigger events?
- Sufficient knowledge to solve?

---

## 25. Robot Factory - Abstract Factory + Builder

### Core Gameplay Loop
Assemble robots from compatible parts produced by different factories.

### Unique Mechanics

**1. Factory Types**
- Military: combat robots
- Utility: worker robots
- Scout: spy robots
- Each produces compatible parts

**2. Part Compatibility**
- Military chassis needs military AI
- Can't mix factory types
- Ensures coherent designs
- Constraint system

**3. Assembly Line**
- Parts arrive sequentially
- Must assemble in order
- Time management
- Production optimization

**4. Quality Control**
- Inspect assembled robots
- Reject defective units
- Affects reputation
- Quality vs speed

**5. Custom Orders**
- Customers request specific types
- Build to specification
- Bonus for perfection
- Order fulfillment

**6. Factory Upgrades**
- Better parts over time
- Faster production
- New robot types
- Progression system

### Pattern Enablement
- **Abstract Factory**: Different robot families
- **Builder Pattern**: Step-by-step assembly
- Factories ensure compatibility
- Builder handles complex construction

### Player Decision Points
- Which factory to focus on?
- Reject defective or ship anyway?
- Fulfill custom orders or mass produce?
- Upgrade speed or quality?

---

## Cross-Cutting Mechanics Patterns

### Common Mechanics Across Games

**Resource Management**
- Appears in: 8, 11, 13, 14, 17, 19, 22
- Pattern: Pool, Factory, Builder
- Manage limited resources efficiently

**Risk/Reward Systems**
- Appears in: 1, 4, 5, 12, 15, 18, 21
- Pattern: Memento, State, Command
- Balance danger vs benefit

**Progression Systems**
- Appears in: 3, 10, 13, 17, 23, 25
- Pattern: Decorator, Builder, Prototype
- Improve over time

**Time Pressure**
- Appears in: 1, 4, 6, 9, 16, 24
- Pattern: State, Observer, Command
- Decisions under pressure

**Strategic Depth**
- Appears in: 2, 11, 14, 18, 19, 20
- Pattern: Strategy, Composite
- Multiple valid approaches

**Information Gathering**
- Appears in: 5, 15, 20, 24
- Pattern: Memento, Visitor, Iterator
- Knowledge is power

---

## Mechanics That Benefit From Patterns

### Observer Pattern Mechanics
- Event-driven gameplay
- Reactive systems
- Combo counters
- Achievement tracking
- Dynamic difficulty

### State Pattern Mechanics
- Context-sensitive actions
- Progressive difficulty
- Character transformations
- Game mode switching
- AI behavior changes

### Command Pattern Mechanics
- Undo/redo systems
- Replay functionality
- Input recording
- Macro systems
- Network synchronization

### Composite Pattern Mechanics
- Hierarchical structures
- Recursive operations
- Nested inventories
- Skill trees
- Tech trees

### Strategy Pattern Mechanics
- AI personalities
- Difficulty settings
- Play styles
- Adaptive enemies
- Player choices

### Builder Pattern Mechanics
- Crafting systems
- Character creation
- Base building
- Procedural generation
- Complex configuration

### Decorator Pattern Mechanics
- Power-ups
- Equipment systems
- Buff/debuff mechanics
- Skill modifications
- Temporary effects

---

## Design Lessons

1. **Patterns Enable Complexity**: Sophisticated mechanics need solid patterns
2. **Multiple Patterns Together**: Best games combine several patterns
3. **Patterns Support Emergence**: Simple patterns create complex gameplay
4. **Player-Facing Mechanics**: Patterns should enable fun, not just structure
5. **Flexibility Matters**: Patterns allow iteration and experimentation

Each example shows how design patterns aren't just code organization—they enable specific gameplay mechanics that would be difficult to implement otherwise.

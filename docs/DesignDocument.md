# Dungeon Crawler Game - Design Document

## Project Overview
A text-based dungeon crawler RPG built in C# to demonstrate Object-Oriented Programming principles and UML design. The game features combat, inventory management, character progression, multiple rooms, and save/load functionality.

## Core Design Principles

### 1. **Extensibility First**
The combat system starts simple but is designed to easily add:
- Defense/armor mechanics (already built into Player class)
- Critical hits (Weapon class has criticalChance property)
- Special abilities (can extend Character class with Ability system)
- Status effects (future: add StatusEffect class)

### 2. **Single Responsibility**
Each class has one clear job:
- `Game`: Manages game state and main loop
- `Character`: Handles all combat mechanics
- `Inventory`: Manages items and equipment
- `Room`: Represents locations and connections
- `Item` hierarchy: Different item behaviors

### 3. **Inheritance for Code Reuse**
```
Character (abstract base)
├── Player (adds inventory, leveling, equipment)
└── Enemy (adds loot drops, experience rewards)

Item (abstract base)
├── Weapon (adds attack bonus, crit chance)
├── Armor (adds defense bonus)
└── Potion (adds healing)
```

---

## Class Breakdown

### **Game Class** - The Controller
**Purpose**: Orchestrates the entire game flow

**Key Responsibilities:**
- Initialize the game world (rooms, enemies, items)
- Run the main game loop
- Process player commands
- Handle save/load functionality

**Properties:**
- `player`: The player character
- `rooms`: All rooms in the game
- `currentRoom`: Where the player currently is
- `isRunning`: Controls the game loop

**Methods:**
- `Start()`: Initialize new game or load saved game
- `GameLoop()`: Main game loop - display room, get command, process
- `ProcessCommand(string)`: Parse and execute player commands
- `SaveGame()`: Serialize game state to file
- `LoadGame()`: Deserialize game state from file

**Implementation Notes:**
- Use a Dictionary<string, Action> for command processing (scalable)
- Save game state as JSON (easy to read/debug)
- Consider using a Room dictionary with string keys for easy lookup

---

### **Character Class** - Combat Foundation
**Purpose**: Abstract base class for all combatants (Player and Enemy)

**Why Abstract?**
- You never create a "Character" directly - only Players or Enemies
- Provides shared combat logic both use
- Allows polymorphism (treat Player and Enemy the same during combat)

**Properties:**
- `name`: Character's name
- `maxHealth`: Maximum HP
- `currentHealth`: Current HP (can't exceed maxHealth)
- `attackPower`: Base damage
- `level`: Current level

**Methods:**
- `Attack(Character target)`: Deal damage to target, return damage dealt
- `TakeDamage(int damage)`: Reduce health, ensure it doesn't go below 0
- `IsAlive()`: Returns true if currentHealth > 0
- `Heal(int amount)`: Restore health, don't exceed maxHealth

**Key Design Decision:**
- Properties are `protected` (not private) so Player and Enemy can access them
- Methods are `public` so any code can call them
- `Attack()` is `virtual` so Player can override to add weapon bonuses

**Example Logic:**
```
Attack(target):
    damage = this.attackPower
    target.TakeDamage(damage)
    return damage

TakeDamage(damage):
    currentHealth -= damage
    if currentHealth < 0:
        currentHealth = 0
```

---

### **Player Class** - The Hero
**Purpose**: Extends Character with player-specific features

**Additional Properties:**
- `experience`: Current XP
- `experienceToNextLevel`: XP needed to level up
- `inventory`: Player's Inventory object
- `defense`: Damage reduction (for future armor system)

**Additional Methods:**
- `GainExperience(int exp)`: Add XP, check for level up
- `LevelUp()`: Increase stats, reset XP requirement
- `EquipWeapon(Weapon)`: Equip weapon, add to attack power
- `EquipArmor(Armor)`: Equip armor, add to defense
- `UseItem(Item)`: Consume item from inventory
- `DisplayStats()`: Show health, level, XP, equipment

**Attack Override:**
```csharp
public override int Attack(Character target)
{
    int baseDamage = base.Attack(target); // Call Character's Attack
    
    // Add weapon bonus
    if (inventory.equippedWeapon != null)
    {
        baseDamage += inventory.equippedWeapon.GetAttackBonus();
        
        // Check for critical hit
        if (RollCritical(inventory.equippedWeapon.GetCriticalChance()))
        {
            baseDamage *= 2; // Critical hit doubles damage
        }
    }
    
    target.TakeDamage(baseDamage);
    return baseDamage;
}
```

**Leveling System:**
```
Level 1 → 2: 100 XP
Level 2 → 3: 200 XP  (exponential growth)
Level 3 → 4: 400 XP
etc.

On Level Up:
- maxHealth += 10
- attackPower += 2
- Fully heal player
- Display level up message
```

---

### **Enemy Class** - The Monsters
**Purpose**: Extends Character with enemy-specific features

**Additional Properties:**
- `experienceReward`: XP given when defeated
- `dropItem`: Item dropped when killed (can be null)

**Methods:**
- `GetExperienceReward()`: Return XP value
- `GetDropItem()`: Return the item to drop

**Enemy Types Examples:**
```
Goblin: 
- Health: 20
- Attack: 5
- XP Reward: 50
- Drop: Health Potion (50% chance)

Orc Warrior:
- Health: 40
- Attack: 10
- XP Reward: 100
- Drop: Iron Sword (guaranteed)

Dragon:
- Health: 100
- Attack: 25
- XP Reward: 500
- Drop: Legendary Armor (guaranteed)
```

---

### **Inventory Class** - Item Management
**Purpose**: Manage player's items and equipment

**Properties:**
- `items`: List of all items in inventory
- `equippedWeapon`: Currently equipped weapon (null if none)
- `equippedArmor`: Currently equipped armor (null if none)
- `maxCapacity`: Maximum items (e.g., 20)

**Methods:**
- `AddItem(Item)`: Add to inventory if not full, return success/fail
- `RemoveItem(Item)`: Remove from inventory
- `UseItem(Item)`: Use consumable, remove if used up
- `EquipWeapon(Weapon)`: Unequip old weapon, equip new one
- `EquipArmor(Armor)`: Unequip old armor, equip new one
- `DisplayInventory()`: Show all items, mark equipped items
- `HasItem(string name)`: Check if item exists

**Important Design:**
- When equipping new weapon, old weapon goes back to inventory
- Equipped items don't count toward capacity
- Consumables (potions) are removed after use

---

### **Item Hierarchy** - Collectibles

#### **Item (Abstract Base Class)**
**Purpose**: Foundation for all items

**Properties:**
- `name`: Item name
- `description`: What it does
- `value`: Worth (for future shop system)

**Methods:**
- `GetName()`: Return name
- `GetDescription()`: Return description
- `Use(Player)`: Abstract method - each item type implements differently

#### **Weapon Class**
**Properties:**
- `attackBonus`: Added to player's attack
- `criticalChance`: % chance for critical hit (0-100)

**Use Method:**
```csharp
public override void Use(Player player)
{
    player.EquipWeapon(this);
    Console.WriteLine($"Equipped {name}!");
}
```

**Example Weapons:**
```
Rusty Dagger:
- Attack Bonus: +2
- Crit Chance: 5%

Iron Sword:
- Attack Bonus: +5
- Crit Chance: 10%

Legendary Blade:
- Attack Bonus: +15
- Crit Chance: 25%
```

#### **Armor Class**
**Properties:**
- `defenseBonus`: Reduces damage taken

**Use Method:**
```csharp
public override void Use(Player player)
{
    player.EquipArmor(this);
    Console.WriteLine($"Equipped {name}!");
}
```

#### **Potion Class**
**Properties:**
- `healAmount`: HP restored

**Use Method:**
```csharp
public override void Use(Player player)
{
    player.Heal(healAmount);
    Console.WriteLine($"Used {name}! Restored {healAmount} HP.");
    // Note: Game should remove this from inventory after use
}
```

---

### **Room Class** - The World
**Purpose**: Represent locations in the dungeon

**Properties:**
- `name`: Room name (e.g., "Dark Hallway")
- `description`: What the player sees
- `enemies`: List of enemies in the room
- `items`: List of items on the ground
- `exits`: Dictionary<string, Room> for navigation (e.g., "north" -> NextRoom)
- `isCleared`: True if all enemies defeated

**Methods:**
- `AddEnemy(Enemy)`: Place enemy in room
- `AddItem(Item)`: Place item in room
- `AddExit(direction, Room)`: Connect to another room
- `GetEnemies()`: Return enemy list
- `GetItems()`: Return item list
- `GetExit(direction)`: Return room in that direction (or null)
- `DisplayRoom()`: Show room description, enemies, items, exits
- `ClearEnemies()`: Mark room as cleared

**Example Room Setup:**
```csharp
Room entrance = new Room("Dungeon Entrance", 
    "A dark stone entrance. Torches flicker on the walls.");

Room hallway = new Room("Dark Hallway",
    "A narrow corridor. You hear growling in the distance.");

entrance.AddExit("north", hallway);
hallway.AddExit("south", entrance);

Enemy goblin = new Enemy("Goblin", 20, 5, 50);
hallway.AddEnemy(goblin);

Item potion = new Potion("Health Potion", "Restores 25 HP", 10, 25);
hallway.AddItem(potion);
```

**Display Output:**
```
=== Dark Hallway ===
A narrow corridor. You hear growling in the distance.

Enemies:
- Goblin (20/20 HP)

Items:
- Health Potion

Exits: north, south
```

---

## Game Flow Detailed

### **1. Game Initialization**
```
1. Display title screen
2. Ask: New Game or Load Game?
3. If New:
   - Create player (enter name)
   - Initialize player stats (health: 100, attack: 10, level: 1)
   - Create all rooms
   - Populate rooms with enemies and items
   - Set player in starting room
4. If Load:
   - Read save file (JSON)
   - Deserialize all game state
5. Enter main game loop
```

### **2. Main Game Loop**
```
while (isRunning):
    1. Display current room
    2. Show player HP and level
    3. Get command from player
    4. Process command:
       - look: Show room details again
       - stats: Show player stats
       - inventory: Show inventory
       - move [direction]: Change rooms
       - attack: Enter combat
       - take [item]: Pick up item
       - use [item]: Use item from inventory
       - equip [item]: Equip weapon/armor
       - save: Save game
       - help: Show commands
       - quit: Exit game
    5. Check win condition (all rooms cleared?)
```

### **3. Combat System**
```
Combat Begins:
1. Display all enemies in room
2. If multiple enemies, ask which to target
3. Combat loop:
   
   PLAYER TURN:
   a. Player attacks selected enemy
   b. Calculate damage (base attack + weapon bonus)
   c. Check for critical hit
   d. Apply damage to enemy
   e. If enemy dead:
      - Award experience
      - Check for level up
      - Drop item
      - Remove from room
      - If no more enemies: return to main loop
   
   ENEMY TURN (if alive):
   a. Enemy attacks player
   b. Calculate damage (enemy attack - player defense)
   c. Apply damage to player
   d. If player dead: Game Over
   
4. Repeat until player dies or all enemies defeated
```

### **4. Save/Load System**
**Save Format (JSON):**
```json
{
  "player": {
    "name": "Kyle",
    "health": 85,
    "maxHealth": 110,
    "attackPower": 14,
    "defense": 5,
    "level": 3,
    "experience": 150,
    "inventory": {
      "items": ["Health Potion", "Mana Potion"],
      "equippedWeapon": "Iron Sword",
      "equippedArmor": "Leather Armor"
    }
  },
  "currentRoom": "Dark Hallway",
  "rooms": [
    {
      "name": "Dark Hallway",
      "isCleared": false,
      "enemies": ["Goblin"],
      "items": []
    }
  ]
}
```

---

## Implementation Order (Recommended)

### **Phase 1: Core Foundation**
1. **Character class** (base combat)
2. **Player class** (without inventory first)
3. **Enemy class**
4. Test combat between Player and Enemy in Main()

### **Phase 2: Items & Inventory**
5. **Item base class**
6. **Potion class** (simplest item)
7. **Weapon class**
8. **Armor class**
9. **Inventory class**
10. Add inventory to Player class
11. Test item usage

### **Phase 3: World Building**
12. **Room class**
13. Create 3-5 connected rooms
14. Test movement between rooms

### **Phase 4: Game Controller**
15. **Game class** (main loop)
16. Command processing
17. Integrate combat into game loop

### **Phase 5: Progression & Persistence**
18. Experience/leveling system
19. Save game functionality
20. Load game functionality

### **Phase 6: Polish**
21. Win/lose conditions
22. Better UI formatting
23. Testing and bug fixes
24. Add more content (rooms, enemies, items)

---

## Extension Ideas (After Core Complete)

### Easy Extensions:
- More enemy types
- More item types (keys, quest items)
- More rooms
- Boss battles
- Difficulty levels

### Medium Extensions:
- Status effects (poison, burn, stun)
- Magic system (spells, mana)
- Shop system (buy/sell items)
- Character classes (Warrior, Mage, Rogue)

### Advanced Extensions:
- Procedurally generated dungeons
- Multiple save slots
- Achievements system
- Combat abilities/skills
- Companion system

---

## Testing Checklist

- [ ] Player can attack and defeat enemy
- [ ] Enemy can damage player
- [ ] Player death triggers game over
- [ ] Experience awards correctly
- [ ] Level up increases stats
- [ ] Items can be picked up
- [ ] Potions restore health correctly
- [ ] Weapons increase attack power
- [ ] Armor reduces damage taken
- [ ] Inventory respects capacity limit
- [ ] Movement between rooms works
- [ ] Room descriptions display correctly
- [ ] Save game preserves all state
- [ ] Load game restores correctly
- [ ] Combat with multiple enemies works
- [ ] All commands execute correctly

---

## Code Style Guidelines

### Naming Conventions:
- Classes: `PascalCase` (Player, Enemy, HealthPotion)
- Methods: `PascalCase` (Attack, TakeDamage, DisplayStats)
- Properties: `PascalCase` (CurrentHealth, MaxHealth)
- Private fields: `camelCase` (maxHealth, attackPower)
- Constants: `UPPER_SNAKE_CASE` (MAX_INVENTORY_SIZE)

### Documentation:
```csharp
/// <summary>
/// Deals damage to a target character.
/// </summary>
/// <param name="target">The character to attack</param>
/// <returns>The amount of damage dealt</returns>
public virtual int Attack(Character target)
{
    // Implementation
}
```

### Error Handling:
- Validate user input (check for null, bounds)
- Handle file I/O exceptions for save/load
- Prevent negative health
- Prevent division by zero

---

## Portfolio Presentation Tips

### README.md should include:
1. Game description
2. Features list
3. UML diagrams (embedded images)
4. How to run
5. Technologies used (C#, .NET, OOP)
6. What you learned

### GitHub Repository Structure:
```
DungeonCrawler/
├── README.md
├── docs/
│   ├── diagrams/
│   │   ├── ClassDiagram.png
│   │   └── GameFlowchart.png
│   └── DesignDocument.md
├── src/
│   ├── DungeonCrawler.sln
│   └── DungeonCrawler/
│       ├── Game.cs
│       ├── Character.cs
│       ├── Player.cs
│       ├── Enemy.cs
│       ├── Inventory.cs
│       ├── Items/
│       │   ├── Item.cs
│       │   ├── Weapon.cs
│       │   ├── Armor.cs
│       │   └── Potion.cs
│       ├── Room.cs
│       └── Program.cs
└── saves/
    └── savegame.json
```

---

## Questions to Consider While Building

1. **How should critical hits be calculated?**
   - Random number 1-100, if <= critChance, it's a crit?
   
2. **Should enemies drop items 100% of the time?**
   - Maybe use a drop chance percentage?

3. **How should leveling curve work?**
   - Linear (100, 200, 300...) or exponential (100, 200, 400...)?

4. **Should player heal fully on level up?**
   - Makes leveling during combat more strategic

5. **Maximum inventory size?**
   - 10? 20? Unlimited?

6. **Can player flee from combat?**
   - Add a "flee" command that has a success chance?

---

## Resources for When You Get Stuck

- **C# Documentation**: docs.microsoft.com
- **Inheritance**: Learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/inheritance
- **File I/O**: System.IO namespace
- **JSON Serialization**: System.Text.Json or Newtonsoft.Json
- **Collections**: List<T>, Dictionary<TKey, TValue>

Good luck! Build one class at a time, test frequently, and don't hesitate to check in with me when you need help!

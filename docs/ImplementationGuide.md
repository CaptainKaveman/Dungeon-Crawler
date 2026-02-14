# Dungeon Crawler - Project Structure & Implementation Guide

## 📁 Recommended File Structure

```
DungeonCrawler/
├── README.md                          # Main project documentation
├── docs/
│   ├── diagrams/
│   │   ├── ClassDiagram.mermaid       # UML class diagram
│   │   ├── ClassDiagram.png           # (Generate from .mermaid)
│   │   └── GameFlowchart.mermaid      # Game flow diagram
│   └── screenshots/                   # Game screenshots for README
├── design/
│   ├── DesignDocument.md              # Complete design specification
│   ├── Pseudocode.md                  # Algorithm pseudocode
│   └── ImplementationGuide.md         # This file
├── src/
│   ├── DungeonCrawler.sln             # Visual Studio solution
│   └── DungeonCrawler/                # Main project folder
│       ├── DungeonCrawler.csproj
│       ├── Program.cs                 # Entry point (Main method)
│       ├── Game.cs                    # Game controller
│       ├── Characters/
│       │   ├── Character.cs           # Abstract base class
│       │   ├── Player.cs              # Player class
│       │   └── Enemy.cs               # Enemy class
│       ├── Items/
│       │   ├── Item.cs                # Abstract base class
│       │   ├── Weapon.cs              # Weapon implementation
│       │   ├── Armor.cs               # Armor implementation
│       │   └── Potion.cs              # Potion implementation
│       ├── World/
│       │   └── Room.cs                # Room class
│       ├── Systems/
│       │   └── Inventory.cs           # Inventory management
│       └── Utilities/
│           ├── GameData.cs            # For save/load serialization
│           └── Display.cs             # UI formatting helpers
└── saves/
    └── savegame.json                  # Save file location
```

---

## 🎯 Implementation Order

### **Phase 1: Foundation (Days 1-2)**
Build the core combat system without any extras.

#### Step 1.1: Character Class
**File**: `src/DungeonCrawler/Characters/Character.cs`

```csharp
// Start with this basic structure:
public abstract class Character
{
    protected string name;
    protected int maxHealth;
    protected int currentHealth;
    protected int attackPower;
    protected int level;
    
    // Constructor
    // Attack method
    // TakeDamage method
    // IsAlive method
    // Heal method
}
```

**Test it**: In Program.cs, create a simple Character subclass and test attack/damage.

#### Step 1.2: Player Class
**File**: `src/DungeonCrawler/Characters/Player.cs`

```csharp
// Inherit from Character, add:
public class Player : Character
{
    private int experience;
    private int experienceToNextLevel;
    private int defense;
    // Don't add inventory yet - keep it simple
    
    // Constructor
    // Override Attack if needed
    // DisplayStats method
}
```

**Test it**: Create a Player, display stats, make sure it works.

#### Step 1.3: Enemy Class
**File**: `src/DungeonCrawler/Characters/Enemy.cs`

```csharp
public class Enemy : Character
{
    private int experienceReward;
    
    // Constructor
    // GetExperienceReward method
}
```

**Test it**: Create a Player and Enemy, have them fight until one dies.

#### Step 1.4: Simple Combat Test
**File**: `src/DungeonCrawler/Program.cs`

```csharp
static void Main(string[] args)
{
    // Create a player and enemy
    Player player = new Player("Kyle");
    Enemy goblin = new Enemy("Goblin", 20, 5, 50);
    
    // Simple combat loop
    while (player.IsAlive() && goblin.IsAlive())
    {
        // Player attacks
        Console.WriteLine("\nYour turn!");
        player.Attack(goblin);
        
        if (!goblin.IsAlive())
        {
            Console.WriteLine("Victory!");
            break;
        }
        
        // Enemy attacks
        Console.WriteLine("\nEnemy turn!");
        goblin.Attack(player);
        
        if (!player.IsAlive())
        {
            Console.WriteLine("Game Over!");
            break;
        }
    }
}
```

✅ **Checkpoint**: Combat works, characters can die

---

### **Phase 2: Items & Inventory (Days 3-4)**

#### Step 2.1: Item Base Class
**File**: `src/DungeonCrawler/Items/Item.cs`

```csharp
public abstract class Item
{
    protected string name;
    protected string description;
    protected int value;
    
    // Constructor
    // GetName, GetDescription
    // Abstract Use method
}
```

#### Step 2.2: Potion Class (Simplest Item)
**File**: `src/DungeonCrawler/Items/Potion.cs`

```csharp
public class Potion : Item
{
    private int healAmount;
    
    // Constructor
    // Override Use method
}
```

**Test it**: Create a potion, have player use it, verify health increases.

#### Step 2.3: Weapon Class
**File**: `src/DungeonCrawler/Items/Weapon.cs`

```csharp
public class Weapon : Item
{
    private int attackBonus;
    private int criticalChance;
    
    // Constructor
    // GetAttackBonus, GetCriticalChance
    // Override Use method (equips weapon)
}
```

#### Step 2.4: Armor Class
**File**: `src/DungeonCrawler/Items/Armor.cs`

```csharp
public class Armor : Item
{
    private int defenseBonus;
    
    // Constructor
    // GetDefenseBonus
    // Override Use method
}
```

#### Step 2.5: Inventory Class
**File**: `src/DungeonCrawler/Systems/Inventory.cs`

```csharp
public class Inventory
{
    private List<Item> items;
    private Weapon equippedWeapon;
    private Armor equippedArmor;
    private int maxCapacity;
    
    // AddItem (check capacity)
    // RemoveItem
    // EquipWeapon
    // EquipArmor
    // DisplayInventory
}
```

#### Step 2.6: Add Inventory to Player
**Update**: `src/DungeonCrawler/Characters/Player.cs`

```csharp
public class Player : Character
{
    // ... existing properties ...
    private Inventory inventory;
    
    // Add in constructor:
    // inventory = new Inventory(20);
    
    // Add methods:
    // EquipWeapon, EquipArmor, UseItem
    
    // Update Attack to use weapon bonus
}
```

**Test it**: 
- Create items
- Add to inventory
- Equip weapon
- Verify attack power increases
- Use potion, verify healing

✅ **Checkpoint**: Inventory system works, items can be used/equipped

---

### **Phase 3: World & Rooms (Days 5-6)**

#### Step 3.1: Room Class
**File**: `src/DungeonCrawler/World/Room.cs`

```csharp
public class Room
{
    private string name;
    private string description;
    private List<Enemy> enemies;
    private List<Item> items;
    private Dictionary<string, Room> exits;
    private bool isCleared;
    
    // Constructor
    // AddEnemy, AddItem, AddExit
    // GetEnemies, GetItems, GetExit
    // DisplayRoom
    // ClearEnemies
}
```

#### Step 3.2: Create Initial Rooms
**In**: `Program.cs` or separate method

```csharp
private static List<Room> CreateDungeon()
{
    List<Room> rooms = new List<Room>();
    
    // Create 3-5 rooms
    Room entrance = new Room("Entrance", "Starting area");
    Room hallway = new Room("Hallway", "Dark corridor");
    Room treasury = new Room("Treasury", "Gold everywhere");
    
    // Connect them
    entrance.AddExit("north", hallway);
    hallway.AddExit("south", entrance);
    hallway.AddExit("east", treasury);
    treasury.AddExit("west", hallway);
    
    // Add enemies and items
    Enemy goblin = new Enemy("Goblin", 20, 5, 50);
    hallway.AddEnemy(goblin);
    
    Weapon sword = new Weapon("Iron Sword", "Sturdy blade", 50, 5, 10);
    treasury.AddItem(sword);
    
    rooms.Add(entrance);
    rooms.Add(hallway);
    rooms.Add(treasury);
    
    return rooms;
}
```

**Test it**:
- Create rooms
- Display room
- Verify exits shown correctly
- Verify enemies/items listed

✅ **Checkpoint**: Rooms exist and connect properly

---

### **Phase 4: Game Controller (Days 7-8)**

#### Step 4.1: Game Class
**File**: `src/DungeonCrawler/Game.cs`

```csharp
public class Game
{
    private Player player;
    private List<Room> rooms;
    private Room currentRoom;
    private bool isRunning;
    
    // Constructor
    public Game(Player player, List<Room> rooms, Room startingRoom)
    {
        this.player = player;
        this.rooms = rooms;
        this.currentRoom = startingRoom;
        this.isRunning = true;
    }
    
    // Start method
    // GameLoop method
    // ProcessCommand method
}
```

#### Step 4.2: Basic Game Loop
```csharp
public void GameLoop()
{
    while (isRunning && player.IsAlive())
    {
        // Display current room
        currentRoom.DisplayRoom();
        
        // Show player health
        Console.WriteLine($"\nHP: {player.GetHealth()}");
        
        // Get command
        Console.Write("> ");
        string command = Console.ReadLine().ToLower();
        
        // Process command
        ProcessCommand(command);
        
        // Check win condition
        if (AllRoomsCleared())
        {
            Console.WriteLine("You win!");
            isRunning = false;
        }
    }
    
    if (!player.IsAlive())
    {
        Console.WriteLine("Game Over!");
    }
}
```

#### Step 4.3: Command Processing
Start with basic commands:

```csharp
private void ProcessCommand(string input)
{
    string[] parts = input.Split(' ');
    string command = parts[0];
    
    switch (command)
    {
        case "look":
            currentRoom.DisplayRoom();
            break;
            
        case "stats":
            player.DisplayStats();
            break;
            
        case "inventory":
        case "inv":
            player.inventory.DisplayInventory();
            break;
            
        case "move":
        case "go":
            if (parts.Length > 1)
            {
                MoveToRoom(parts[1]);
            }
            break;
            
        case "attack":
            StartCombat();
            break;
            
        case "help":
            DisplayHelp();
            break;
            
        case "quit":
            isRunning = false;
            break;
            
        default:
            Console.WriteLine("Unknown command. Type 'help' for commands.");
            break;
    }
}
```

#### Step 4.4: Movement Method
```csharp
private void MoveToRoom(string direction)
{
    Room nextRoom = currentRoom.GetExit(direction);
    
    if (nextRoom == null)
    {
        Console.WriteLine("You can't go that way.");
        return;
    }
    
    currentRoom = nextRoom;
    Console.WriteLine($"\nYou move {direction}...\n");
}
```

#### Step 4.5: Combat Integration
```csharp
private void StartCombat()
{
    List<Enemy> enemies = currentRoom.GetEnemies();
    
    if (enemies.Count == 0)
    {
        Console.WriteLine("There are no enemies to fight.");
        return;
    }
    
    // Combat loop (copy logic from Phase 1, integrate with room)
    while (enemies.Count > 0 && player.IsAlive())
    {
        // ... combat logic ...
    }
}
```

#### Step 4.6: Update Program.cs
```csharp
static void Main(string[] args)
{
    Console.WriteLine("=== DUNGEON CRAWLER ===\n");
    
    // Create player
    Console.Write("Enter your name: ");
    string playerName = Console.ReadLine();
    Player player = new Player(playerName);
    
    // Create dungeon
    List<Room> rooms = CreateDungeon();
    
    // Create and start game
    Game game = new Game(player, rooms, rooms[0]);
    game.GameLoop();
    
    Console.WriteLine("\nThanks for playing!");
}
```

✅ **Checkpoint**: Full game loop works, you can explore and fight

---

### **Phase 5: Experience & Leveling (Days 9-10)**

#### Step 5.1: Add Leveling to Player
**Update**: `src/DungeonCrawler/Characters/Player.cs`

```csharp
public void GainExperience(int exp)
{
    Console.WriteLine($"Gained {exp} experience!");
    experience += exp;
    
    while (experience >= experienceToNextLevel)
    {
        LevelUp();
    }
}

private void LevelUp()
{
    level++;
    experience -= experienceToNextLevel;
    
    maxHealth += 10;
    attackPower += 2;
    currentHealth = maxHealth; // Full heal
    
    experienceToNextLevel *= 2; // Exponential
    
    Console.WriteLine("\n╔═══════════════╗");
    Console.WriteLine("║  LEVEL UP!    ║");
    Console.WriteLine("╚═══════════════╝");
    Console.WriteLine($"You are now level {level}!");
    Console.WriteLine($"Health: {maxHealth}");
    Console.WriteLine($"Attack: {attackPower}");
}
```

#### Step 5.2: Award XP After Combat
**Update**: `Game.cs` combat method

```csharp
// After enemy dies:
if (!enemy.IsAlive())
{
    Console.WriteLine($"{enemy.GetName()} defeated!");
    
    int expReward = enemy.GetExperienceReward();
    player.GainExperience(expReward);
    
    enemies.Remove(enemy);
}
```

✅ **Checkpoint**: Player levels up after gaining XP

---

### **Phase 6: Save/Load System (Days 11-12)**

#### Step 6.1: Install JSON Library
Right-click project → Manage NuGet Packages → Install `System.Text.Json`

OR use .NET's built-in `System.Text.Json` (comes with .NET Core 3.0+)

#### Step 6.2: Create Save Data Class
**File**: `src/DungeonCrawler/Utilities/GameData.cs`

```csharp
using System.Text.Json.Serialization;

public class GameData
{
    public PlayerData Player { get; set; }
    public string CurrentRoom { get; set; }
    public List<RoomData> Rooms { get; set; }
}

public class PlayerData
{
    public string Name { get; set; }
    public int CurrentHealth { get; set; }
    public int MaxHealth { get; set; }
    public int AttackPower { get; set; }
    public int Defense { get; set; }
    public int Level { get; set; }
    public int Experience { get; set; }
    public int ExperienceToNextLevel { get; set; }
    public List<string> InventoryItems { get; set; }
    public string EquippedWeapon { get; set; }
    public string EquippedArmor { get; set; }
}

public class RoomData
{
    public string Name { get; set; }
    public bool IsCleared { get; set; }
    public List<string> Enemies { get; set; }
    public List<string> Items { get; set; }
}
```

#### Step 6.3: Add Save Method to Game
```csharp
using System.Text.Json;
using System.IO;

public void SaveGame(string filename)
{
    try
    {
        GameData saveData = new GameData
        {
            Player = new PlayerData
            {
                Name = player.GetName(),
                CurrentHealth = player.GetHealth(),
                // ... all other player properties ...
            },
            CurrentRoom = currentRoom.GetName(),
            Rooms = new List<RoomData>()
            // ... populate room data ...
        };
        
        string jsonString = JsonSerializer.Serialize(saveData, new JsonSerializerOptions { WriteIndented = true });
        File.WriteAllText(filename, jsonString);
        
        Console.WriteLine("Game saved successfully!");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Failed to save: {ex.Message}");
    }
}
```

#### Step 6.4: Add Load Method
```csharp
public static Game LoadGame(string filename)
{
    try
    {
        string jsonString = File.ReadAllText(filename);
        GameData saveData = JsonSerializer.Deserialize<GameData>(jsonString);
        
        // Reconstruct player
        Player player = new Player(saveData.Player.Name);
        player.SetHealth(saveData.Player.CurrentHealth);
        // ... restore all properties ...
        
        // Reconstruct rooms
        // ... restore room state ...
        
        Console.WriteLine("Game loaded successfully!");
        return new Game(player, rooms, currentRoom);
    }
    catch (FileNotFoundException)
    {
        Console.WriteLine("Save file not found.");
        return null;
    }
}
```

#### Step 6.5: Add Save/Load Commands
Update `ProcessCommand`:

```csharp
case "save":
    SaveGame("saves/savegame.json");
    break;
```

Update `Main` to offer load option at start.

✅ **Checkpoint**: Can save and load game state

---

## 🎨 Polish & Testing (Days 13-14)

### Add Better UI Formatting
**Create**: `src/DungeonCrawler/Utilities/Display.cs`

```csharp
public static class Display
{
    public static void PrintHeader(string text)
    {
        Console.WriteLine($"\n╔{'═'.PadRight(text.Length + 2, '═')}╗");
        Console.WriteLine($"║ {text} ║");
        Console.WriteLine($"╚{'═'.PadRight(text.Length + 2, '═')}╝");
    }
    
    public static void PrintDivider()
    {
        Console.WriteLine(new string('─', 40));
    }
}
```

### Test Everything
Create a test checklist:
- [ ] Combat works correctly
- [ ] Leveling system works
- [ ] Items can be picked up
- [ ] Equipment modifies stats
- [ ] Room navigation works
- [ ] Save/load preserves state
- [ ] Win condition triggers

---

## 📝 Common Mistakes to Avoid

1. **Trying to do everything at once**
   - Build one class at a time
   - Test each piece before moving on

2. **Not using access modifiers correctly**
   - Use `private` for internal details
   - Use `protected` for things subclasses need
   - Use `public` for external interface

3. **Forgetting null checks**
   ```csharp
   if (inventory.equippedWeapon != null)
   {
       // Safe to use weapon
   }
   ```

4. **Not testing edge cases**
   - What if player has no items?
   - What if room has no exits?
   - What if player tries to move to invalid direction?

5. **Magic numbers in code**
   ```csharp
   // Bad
   if (experience >= 100) { levelUp(); }
   
   // Good
   const int EXPERIENCE_PER_LEVEL = 100;
   if (experience >= EXPERIENCE_PER_LEVEL) { levelUp(); }
   ```

---

## 🆘 When to Check In With Me

- **Stuck on implementing a class**: Show me your code, describe the problem
- **Not sure how to connect two classes**: Ask about the relationship
- **Getting errors you don't understand**: Share the error message
- **Want to verify your approach**: Show pseudocode before coding
- **Finished a phase**: Get feedback before continuing

---

## 🎯 Final Deliverable Checklist

- [ ] All classes implemented
- [ ] Combat system works
- [ ] Inventory/equipment works
- [ ] Room navigation works
- [ ] Experience/leveling works
- [ ] Save/load works
- [ ] At least 5 rooms created
- [ ] At least 3 enemy types
- [ ] At least 5 different items
- [ ] Win condition implemented
- [ ] Code is commented
- [ ] README.md written
- [ ] UML diagrams included
- [ ] Screenshots added
- [ ] Pushed to GitHub

Good luck! You've got this! 🎮

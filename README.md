# Drift Script
Drift Script is a modified version of Squirrel 3.2 language. \
Please refer to the language reference manual here: \
http://squirrel-lang.org/squirreldoc/reference/language.html


# Units, Players, Teams, Locations, Unit Definitions, IDs...
Integer IDs are used to represent `units`, `players`, `teams`, and other things.. \
Invalid IDs are equal to 0. \
Some objects such as `Locations` and `Unit Definitions` are identified with strings. \
Invalid string identifiers are equal to an empty or null string.

# gx_create_unit
```c
int gx_create_unit(table params)
```
```c
table params = {
    string m_unitType,              // Required
    int m_playerID,                 // Required
    Vec2 m_position = {},           // Optional
    string m_location = {}          // Optional
}
```

```lua
local new_unit = gx_create_unit({ m_unitType = "Brute", m_playerID=1, m_location = "my_cool_location" })
```

- Will create unit of type `m_unitType` at position `m_position` or at location `m_location` for player `m_playerID`.
- It is undefined behavior to have both `m_position` and `m_location` set.
- If neither `m_position` nor `m_location` is set, unit will be created at `(0, 0)`

# gx_get_sim_tick
```c
int gx_get_sim_tick()
```
- Returns the current sim tick number in simulation.
- The first `gx_update` call will have tick = 1
- The second `gx_update` call will have tick = 2, etc..
- every tick corresponds to 50ms real time

# gx_get_units
```c
int[] gx_get_units(table params)
```
```c
table params = {
    int m_playerID = {},                            // Optional, Filter for player_id
    int m_teamID = {},                              // Optional, Filter for team_id
    string m_unitType = {}                          // Optional, Filter for unit type
    string m_location = {},                         // Optional, location to get units from
    BoundsCheck m_boundsCheck = BoundsCheck.Center  // Optional, used if m_location is set. (default = Center)
    bool m_bIncludeAirUnits = true,                 // Optional, Set to false if you want to exclude air units
    bool m_bIncludeGroundUnits = true               // Optional, set to false if you want to exclude ground units
    bool m_bIncludeKilledUnits = false,             // Optional, Set to true if you want to include killed units
    bool m_bIncludeRemovedUnits = false             // Optional, set to true if you want to include removed units
    bool m_bIncludeProjectiles = false              // Optional, include projectiles (default: false)
}
```
Example:
```lua
local the_units = gx_get_units({
    m_location = "my_cool_location",
    m_unitType = "Brute",
    m_playerID=1
})

foreach (unit in the_units) {
    gx_kill_unit(unit)
}
```

- If m_location is not set, will grab units from entire map.

# gx_get_unit_count
```c
int gx_get_unit_count(table params)
```
```c
table params = {
    int m_playerID = {},                            // Optional, filter by player_id
    int m_teamID = {},                              // Optional, filter by team_id
    string m_unitType = {},                         // Optional, filter by unit_type
    string m_location = {},                         // Optional, only return units within specified location
    BoundsCheck m_boundsCheck = BoundsCheck.Center  // Optional, used only if m_location is set (default: Center)
    bool m_bCountAirUnits = true,                   // Optional, include air units in query (default: true)
    bool m_bCountGroundUnits = true,                // Optional, include ground units in query (default: true)
    bool m_bCountKilledUnits = false,               // Optional, include killed units (default: false)
    bool m_bCountRemovedUnits = false               // Optional, include removed units (default: false)
    bool m_bCountProjectiles = false                // Optional, include projectiles (default: false)
}
```

Example:

```lua
local numBrutesAtMyLocation = gx_get_unit_count({
    m_unitType = "Brute",
    m_location = "MyLocation"
})

gx_print(numBrutesAtMyLocation)
```

- if `m_location` is not set, will return number of units on entire map

# gx_create_explosion
```c
void gx_create_explosion(table params)
```

```c
table params = {
    float m_size = {},              // Optional, Diameter of explosion
    Vec3 m_color = Vec3(1,1,0)      // Optional, ColorSRGB of explosion.
    string m_location = {},         // Optional, Location for explosion
    Vec2 m_pos2d = {}               // Optional, Position for explosion
}
```

Example:
```lua
gx_create_explosion( {
    m_color = Vec3(1, 0, .5),
    m_location = "my_location"
} )
```

- it is undefined behavior to set both `m_location` and `m_pos2d`
- if `m_size` is not set and `m_location` is set, the resolved size will be the minimum width/height of `m_location`
- if `m_size` is not set and `m_pos2d` is set, the resolved size will be `1`
- Explosions are purely visual. They do not do any damage.
- Default value for `m_color` is `Vec3(1,1,0)` aka `0xFFFF00` (yellow)

# gx_kill_unit
```c
void gx_kill_unit(int unit_id)
```

- kills the unit `unit_id`.
- It is safe to call this function on already killed units

# gx_is_unit_killed
```c
void gx_is_unit_killed(int unit_id)
```
- returns if the unit `unit_id` is killed
- returns `true` if unit does not exist
- equivalent to calling `!gx_is_unit_alive(unit_id)`

# gx_remove_unit
```c
void gx_remove_unit(int unit_id)
```

- marks the unit to be removed by game
- `unit_id` will be removed from game before the next gx_update call
- It is safe to call this function on already killed or removed units

# gx_is_unit_removed
```c
void gx_is_unit_removed(int unit_id)
```

- returns if unit is marked to be removed
- will still return `true` if unit_id does not exist

# gx_unit_exists
```c
bool gx_unit_exists(int unit_id)
```

- checks if unit still exists in the game

# gx_is_unit_alive_and_constructed
```c
bool gx_is_unit_alive_and_constructed(int unit_id)
```
- returns `true` if unit is alive and constructed
- returns `false` if unit_id is invalid or unit no longer exists in game

# gx_is_unit_alive
```c
bool gx_is_unit_alive(int unit_id)
```
- returns `true` if unit is alive
- returns `false` if unit_id is invalid or unit no longer exists in game
- Note: This function still returns `true` if unit is not yet fully constructed
- equivalent to calling `!gx_is_unit_killed(unit_id)`

# gx_get_unit_position
```c
Vec2 gx_get_unit_position(int unit_id)
```

- returns position of unit
- returns Vec2(0, 0) if unit no longer exists

# gx_set_unit_position
```c
void gx_set_unit_position(int unit_id, table params)
```

```c
table params = {
    string m_location = {},     // Optional, location to put unit
    Vec2 m_pos2d = {}           // Optional, position to put unit
}
```
Example:
```lua
gx_set_unit_position(some_unit, { m_location = "location_to_teleport_to" } )
```

- it is undefined to set both `m_location` and `m_pos2d`
- if neither is defined, unit will be teleported to Vec2(0, 0)

# gx_is_ground_unit
```c
bool gx_is_ground_unit(int unit_id)
```
- returns if unit is currently a `ground` unit
- units are always in either the `ground` or `air` state
- note: knock-back effect can cause `ground` units to temporarily become `air` units
- equivalent to calling `!gx_is_air_unit(unit_id)`

# gx_is_air_unit
```c
bool gx_is_air_unit(int unit_id)
```
- returns if unit is currently an `air` unit
- units are always in either the `ground` or `air` state
- note: knock-back effect can cause `ground` units to temporarily become `air` units
- equivalent to calling `!gx_is_ground_unit(unit_id)`

# gx_get_player
```c
int gx_get_player(int unit_id)
```
- returns the `player_id` for unit `unit_id`

# gx_print
```c
void gx_print(string message, table params = {})
```

```c
table params = {
    int m_teamID = {},      // Optional, send message to only team_id
    int m_playerID = {}     // Optional, send message to only player_id
}
```

Example
```c
gx_print("Hello World!", { m_playerID = 3 } ) // display chat message 'Hello World!' to player 3
```

- outputs text to game chat
- useful for debugging as well

# gx_set_victory
```c
void gx_set_victory(table params)
```

```c
table params = {
    int m_playerID = {},    // Ignored in team games
    int m_teamID = {}       // Ignored in non-team games
}
```
- `m_playerID` is ignored in team games.
- `m_teamID` is ignored in non-team games
- once a player or team is set to `victory`, future calls to `gx_set_victory`/`gx_set_defeat` for that player/team will be ignored

# gx_set_defeat
```c
void gx_set_defeat(table params)
```

```c
table params = {
    int m_playerID = {},            // Ignored in team games
    int m_teamID = {},              // Ignored in non-team games
    bool m_bKillAllUnits = true     // Optional, (default true)
}
```
- `m_playerID` is ignored in team games.
- `m_teamID` is ignored in non-team games
- if `m_bKillAllUnits` is `true`, all units for player (or team) will be killed.
- once a player or team is set to `defeat`, future calls to `gx_set_victory`/`gx_set_defeat` for that player/team will be ignored


# gx_encode_text
```c
string gx_encode_text(string text)
```

Example
```lua
local someText = gx_encode_text("^23Rainbow Text")
gx_print(someText)
```

# BoundsCheck enum

The BoundsCheck enum is used in unit search queries within locations.\
Most notably in `gx_get_units` and `gx_get_unit_count`

```c
enum BoundsCheck
{
    Center = 1,     // Unit's center position is in location
    Touching = 2,   // Unit is fully inside or touching location
    Inside = 3      // Unit fully inside a location
}
```
- NOTE: Do not rely on enum values to remain the same.


# gx_copy_ud

Creates a copy of a `unit_definition`.\
Required for creating new custom unit types.

```c
void gx_copy_ud(string unit_type, string new_unit_type)
```

The example below creates a new type of unit called "_BabyBrute".\
It copies the existing definition of "Brute" to "_BabyBrute".

Example:

```c
gx_copy_ud("Brute", "_BabyBrute")
```
- new unit type name MUST begin with `_`. This is to prevent naming collisions for future added official units.
- this function will be a no-op if `new_unit_type` name does not begin with `_`
- `ud` is short for `unit_definition` 
- this function can only be called in the `gx_map_init` stage
- any attempt to call this outside of the `gx_map_init` will be ignored.
- in the future, it should be possible to call this during `gx_update`


# gx_modify_ud_props
```c
void gx_modify_ud_props(string unit_type, table params = {})
```

```c
table params = {
    string m_friendlyName = {},     // Optional, set unit's friendly name
    int m_maxHealth = {},           // Optional, set max health
    float m_maxSpeed = {},          // Optional, set max speed
    int m_baseArmor = {},           // Optional, sets base armor
    int m_size = {}                 // Optional, set unit size
}
```
The example below creates a new type of unit called "_BabyBrute".\
It copies the existing `unit_definition` of "Brute" to "_BabyBrute". \
It then sets properties such as friendly name, maxHealth, baseArmor, and size.

Example:

```c
gx_copy_ud("Brute", "_BabyBrute")
gx_modify_ud_props("_BabyBrute", {
    m_friendlyName = "Baby Brute",
    m_maxHealth = 30,
    m_baseArmor = 0,
    m_size = 1
})
```

- Sets properties for a specific `unit_type`
- `ud` is short for `unit_definition` 
- this function can only be called in the `gx_map_init` stage
- any attempt to call this outside of the `gx_map_init` will be ignored.
- in the future, it should be possible to call this during `gx_update`


# gx_fling_unit

throws the unit

```c
void gx_fling_unit(int unit_id, table params = {})
```

```c
table params = {
    Vec2 m_dir2d = {},      // Optional, 2d direction to throw unit. Does not need to be normalized.
    Vec3 m_dir3d = {},      // Optional, 3d direction to throw unit. Does not need to be normalized.
    float m_force = 1       // Optional, velocity to throw unit
}
```

- If neither `m_dir2d` nor `m_dir3d` is set, unit will be thrown in random direction
- Only `m_dir2d` or `m_dir3d` should be set. Setting both is undefined behavior.

# TerrainTypes enum
```c
// Primary Terrain Types
enum TerrainTypes
{
    Normal = 0,         // See SecondaryTerrainTypeNormal for valid secondary types
	Glow = 1,           // valid secondary types are [0 - 31]
	Water = 2,          // valid secondary types are [0 - 2]
	Lava = 3,           // valid secondary types are [0 - 2]
	Diamond = 4,        // valid secondary types is just 0 (TODO: verify, 1 might be valid as well)
	PlayerColor = 8,    // valid secondary types are [1 - 16] (i.e. player_id)
	Unpassable = 9,     // !! Not a dynamic terrain type! Cannot dynamically change or be set to!
	Space = 10,         // valid secondary type is just 0
	CliffClosed = 14,   // !! Not a dynamic terrain type! Cannot dynamically change or be set to!
	CliffBorder = 15    // !! Not a dynamic terrain type! Cannot dynamically change or be set to!
}
```
- Used in `gx_set_terrain_type` and `gx_get_terrain_type`

# SecondaryTerrainTypeNormal enum
```c
enum SecondaryTerrainTypeNormal
{
	Normal = 0,         // Units are normal on this type (no effects)
	Speed = 1,          // Units move faster on this type
	AttackRate = 2,     // Units have faster attack rate on this type
	Heal = 3,           // Units heal faster on this type
	Forbidden = 4,      // Units insta-die on this type
	Sniper = 5,         // Units have increased range on this type
	MeleeOnly = 6,      // Units have decreased range on this type
	Pacifist = 7        // Units are unable to attack on this type
}
```
- Used in `gx_set_terrain_type` and `gx_get_terrain_type`

# gx_set_terrain_type
```c
void gx_set_terrain_type(params = {})
```

```c
table params = {
    TerrainTypes m_type         // Required. The type of terrain to change to. See TerrainTypes enum. 
    int m_secondary = 0         // Secondary terrain type. (default = 0)
    Vec2 m_index = {},          // 2d index of square to change terrain type of
    int m_index2,               // 0 or 1, 0 indicates bottom triangle, 1 indicates top.
                                // If index2 is not defined, entire square specified by m_index
                                // will be set to terrain type (i.e. both triangles, top and bottom).
                                // m_index and index2 are ignored if m_location is set.
    string m_location = {}      // location to set terrain tile types.
}
```

Example that sets terrain at location `my_location` to Pacifist type:
```c
gx_set_terrain_type({
    m_type = TerrainTypes.Normal,
    m_secondary = SecondaryTerrainTypeNormal.Pacifist,
    m_location = "my_location"
})
```

# gx_get_terrain_type
```c
Vec2 gx_get_terrain_type(Vec2 index, int index2 = 0)
````
- returns primary terrain type in `vec.x` and returns secondary terrain type in `vec.y`

# gx_set_player_camera_look_at
```c
void gx_set_camera_look_at(params = {})
```

```c
local params = {
    int m_playerID = {},
    int m_teamID = {},
    int m_unitID = {},
    string m_location = {}
}
```

# get_get_unit_user_data
```c
table gx_get_unit_user_data(int unit_id)
```

Example:
```c
local myData = gx_get_unit_user_data(my_unit_id)
myData["abc"] <- "haha"
```

# PlayerProps enum

The PlayerProps enum is used in `gx_get_player_prop` and `gx_set_player_prop`

```c
enum PlayerProps
{
	Fungus = 1,         // Read / Write     (float)
	Gemstone = 2,       / Read / Write     (float)
	Supply = 3,         // Read-Only        (int)
	MaxSupply = 4,      // Read-Only        (int)
	NumKills = 5,       // Read-Only        (int)
	NumDeaths = 6       // Read-Only        (int)
    PlayerName = 7      // Read-Only        (string)
}
```

# gx_get_player_prop
```c
mixed gx_get_player_prop(PlayerProps prop, int playerID)
```

# gx_set_player_prop
```c
void gx_get_player_prop(PlayerProps prop, int playerID, mixed val)
```

# gx_add_player_prop
```c
void gx_add_player_prop(PlayerProps prop, int playerID, mixed val)
```

# LocationProps enum
```c
enum LocationProps
{
	TopLeft = 1,        // Currently Read-Only (vec2)
	TopRight = 2,       // Currently Read-Only (vec2)
	BottomLeft = 3,     // Currently Read-Only (vec2)
	BottomRight = 4,    // Currently Read-Only (vec2)
	Center = 5,         // Currently Read-Only (vec2)
	Size = 6            // Currently Read-Only (vec2)
}
```
- Used in `gx_get_location_prop` and `gx_set_location_prop`

# gx_get_location_prop
```c
mixed gx_get_location_prop(LocationProps.TopLeft, string location)
```

# gx_set_location_prop
- none currently

# UnitProps enum

The UnitProps enum is used in `gx_get_unit_prop` and `gx_set_unit_prop`

```c
enum UnitProps
{
	MaxHealth = 1,  // Read-Only        (int)
	Health = 2,     // Read / Write     (float)
	MaxSpeed = 3,   // Read-Only        (float)
	Size = 4,       // Read-Only        (float)
	UnitType = 5,   // Read-Only        (string)
    IsOnFire = 6    // Read-Only        (bool)
    GetParentJeep = 7,          // Read-Only (int)
	GetParentDropship = 8,      // Read-Only (int)
	GetParentStarShip = 9,      // Read-Only (int)
	GetParentSpinnerShip = 10   // Read-Only (int)
}
```

# gx_set_unit_prop
```c
void gx_set_unit_prop(UnitProps prop, int unit_id, mixed val)
```

- Refer to `UnitProps` enum for valid `UnitProps` and `type` vals.
- Only enums that have `Write` property can be used in this function

# gx_get_unit_prop
```c
mixed gx_get_unit_prop(UnitProps prop, int unit_id)
```

- Refer to `UnitProps` enum for valid `UnitProps` and `return types`.
- Only enums that have `Read` property can be used in this function

Examples:
```c
local jeep_unit_id = gx_get_unit_prop(UnitProps.GetParentJeep, my_unit_id)
if (jeep_unit_id == 0) {
    gx_print("my_unit is NOT riding a jeep...")
} else {
    gx_print("my_unit is riding a jeep, yay :) !!")
}

local bUnitOnFire = gx_get_unit_prop(UnitProps.IsOnFire, my_unit_id)
if (bUnitOnFire) {
    gx_print("oh noes! my unit is burning!")
}

local my_unit_health = gx_get_unit_prop(UnitProps.Health, my_unit_id)
gx_print("my unit's health is " + my_unit_health)
```
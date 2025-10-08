# Drift Script
Drift Script is a modified version of Squirrel 3.2 language. \
Please refer to the language reference manual here: \
http://squirrel-lang.org/squirreldoc/reference/language.html

Main differences from squirrel:
- `float` type internally modified to be a 64-bit `fixed` type.
- Changes to make table key,value iteration deterministic
- Standard squirrel library is not supported
- Custom math and other functions added
- `gx_include` added to include external source files

Misc:
- `int` types are signed 64-bit
- `float` types are 64-bit `Q31.32` fixed point types
- encoding `utf-8`

# Units, Players, Teams, Locations, Unit Definitions, IDs...
Integer IDs are used to represent `Units`, `Players`, `Teams`, and other things.. \
Invalid Integer IDs are equal to `0`. \
String identifiers are used to represent `Locations`, `TriangleGroups`, `UnitDatas`, `UnitGroups` and other things.. \
Invalid string identifiers are equal to `""`.

# Script Entry points
Your script code has 3 entry points.

`function gx_map_init()`
- This is called before minimap creation.
- This is the only place currently where you are allowed to modify and copy unit datas (i.e. make your own units).
- This function is only called once. Global variables defined in this script are not accessible from the `gx_sim_*` functions

`function gx_sim_init()`
- Called once when the game simulation is setup and ready.
- This is where your initialization code goes.
- Global variables defined here are accessible to `gx_sim_update`

`function gx_sim_update()`
- Called once every simulation update. This is where your main script code goes.
- Global variables persist across `gx_sim_update` calls.

# gx_include
`gx_include(string filename)`
- Includes `filename` in current compilation.
- A file is only included once by `gx_include`; subsequent calls are ignored.

# Supported math functions

- `sqrt(x)` - returns square root of x
- `sin(x)` - return sin of x
- `cos(x)` - return cos of x
- `tan(x)` - return tan of x
- `rand()` - return random integer from [0, `RAND_MAX`]
- `abs(x)` - return absolute value of x.
- `min(x, y)` - returns the minimum of x and y.
- `max(x, y)` - returns the maximum of x and y.
- `clamp(val, min, max)` - clamps `val` to be between `min` and `max`
- `RAND_MAX` - Constant value that is set to `0xFFFFFFFF` (do not rely on this staying the same)
- `PI` - Constant value to represent PI (3.14159...)

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
local new_unit = gx_create_unit({ m_unitType = "Brute", m_playerID = 1, m_location = "my_cool_location" })
```

- Will create unit of type `m_unitType` at position `m_position` or at location `m_location` for player `m_playerID`.
- It is undefined behavior to have both `m_position` and `m_location` set.
- If neither `m_position` nor `m_location` is set, unit will be created at `(0, 0)`

# gx_get_sim_tick
```c
int gx_get_sim_tick()
```
- Returns the current sim tick number in simulation.
- The `gx_sim_init` call will have `tick = 0`
- The first `gx_sim_update` call will have `tick = 1`
- The second `gx_sim_update` call will have `tick = 2`, etc..
- every tick corresponds to 50ms real time

# gx_get_units
```c
int[] gx_get_units(table params)
```
```c
table params = {
    int m_playerIDs[],                              // Optional, Filter for player_id
    int m_forceIDs[],                               // Optional, Filter for force_id
    string m_unitTypes[],                           // Optional, Filter for certain unit types
    string m_exceptUnitTypes[],                     // Optional, ignore certain unit types
    string m_locations[],                           // Optional, locations to get units from
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
    m_playerID = 1
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
    int m_forceID = {},                              // Optional, filter by force_id
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
    Vec2 m_pos = {}               // Optional, Position for explosion
}
```

Example:
```lua
gx_create_explosion( {
    m_color = Vec3(1, 0, .5),
    m_location = "my_location"
} )
```

- it is undefined behavior to set both `m_location` and `m_pos`
- if `m_size` is not set and `m_location` is set, the resolved size will be the minimum width/height of `m_location`
- if `m_size` is not set and `m_pos` is set, the resolved size will be `1`
- Explosions are purely visual. They do not do any damage.
- Default value for `m_color` is `Vec3(1,1,0)` aka `0xFFFF00` (yellow)

# gx_kill_unit
```c
void gx_kill_unit(int unit_id)
```

- kills the unit `unit_id`.
- It is safe to call this function on already killed units

# gx_get_kills
```c
void gx_get_kills(int player_id, table params = {})
```
```
params = {
    m_playerID = {},
    m_bAlliedKills = {},
    m_bSelfKills = {},
    m_bNonAlliedKills = {},
}
```

- if `m_playerID` is set, none of `m_bAlliedKills`, `m_bSelfKills`, `m_bNonAlliedKills` should be set
- if empty {} is passed in for params, all kills will be returned, equivalent to: \
```{ m_bAlliedKills = true, m_bSelfKills = rue, m_bNonAlliedKills = true }```
- Allied kills does not include self kills


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
- `unit_id` will be removed from game before the next `gx_sim_update` call
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
}
```
Example:
```lua
gx_set_unit_position(some_unit, { m_location = "location_to_teleport_to" } )
```

- it is undefined to set both `m_location` and `m_pos`
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

# gx_get_players
```c
int[] gx_get_players(table params = {})
```

```c
table params = {
	bool m_bIncludeNormalPlayers = true;
	bool m_bIncludeDefeatedPlayers = false;
	bool m_bIncludeNeutralPlayer = false;
	bool m_bIncludeRescuePlayer = false;
	bool m_bIncludeHostilePlayer = false;
	bool m_bPlayerMustBeInGame = true;
}
```


# gx_get_player
```c
int gx_get_player(int unit_id)
```
- returns the `player_id` for unit `unit_id`

```c
table params = {
    int m_forceID = {},      // Optional, send message to only force_id
    int m_playerID = {}     // Optional, send message to only player_id
}
```

# gx_print
```c
void gx_print(string message, table params = {})
```

```c
table params = {
    int m_forceID = {},      // Optional, send message to only force_id
    int m_playerID = {}     // Optional, send message to only player_id
}
```
- Should only set `m_forceID` or `m_playerID`, it is undefined to set both.

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
    int m_playerID = {},
    int m_forceID = {}
}
```
- once a player or team is set to `victory`, future calls to `gx_set_victory`/`gx_set_defeat` for that player/team will be ignored
- should only set one of `m_playerID` or `m_forceID`, setting both is undefined
- setting `m_forceID` will set victory for all players within that force

# gx_set_defeat
```c
void gx_set_defeat(table params)
```

```c
table params = {
    int m_playerID = {},
    int m_forceID = {},
    bool m_bKillAllUnits = true     // Optional, (default true)
}
```
- if `m_bKillAllUnits` is `true`, all units for player (or team) will be killed.
- once a player or team is set to `defeat`, future calls to `gx_set_victory`/`gx_set_defeat` for that player/team will be ignored
- should only set one of `m_playerID` or `m_forceID`, setting both is undefined
- setting `m_forceID` will set defeat for all players within that force

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

Creates a copy of a `unit_data`.\
A `unit_data` serves as a 'definition' for a type of unit. \
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
- in the future, it should be possible to call this during `gx_sim_update`

# UnitDataType enum
```c
enum UnitDataType
{
    Invalid = "",
    Myrmidon,
    Chariot,
    Brute,
    Amputee,
    Dungeon
}
```

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
    int m_size = {}                 // Optional, set unit size,
    int m_gemstoneCost = {},
    int m_fungusCost = {},
    int m_supplyCost = {},
    int m_buildTime = {}
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
- in the future, it should be possible to call this during `gx_sim_update`

# gx_add_build_structure_item
```c
void gx_add_build_structure_item(string unitType, table params = {})
```
```
table params = {
    string m_structure = {},    // name of structure to build, i.e. "Microwave"
    Vec2 m_position = {},           // position in command card to place at, leaving empty will use default
    bool bBuildAdvanced = false     // default is false, if set to True, will be placed in 'build advanced' tab
}
```

# gx_remove_all_build_structure_items
```c
void gx_remove_all_build_structure_items(string unitType)
```

- Remove all build structure items from worker

# gx_add_build_item
```c
void gx_add_build_item(string unitType, table params)
```

```c
table params = {
    string m_unitType = {},     // unit type to add
    string m_research ={},      // research to add
    string m_position = {}      // position in command card
}
```
- Only one `m_unitType` or `m_research` should be set

- Remove all build items from structure

# gx_remove_all_build_items
```c
void gx_remove_all_build_items(string unitType)
```

- Remove all build items from structure

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


# gx_set_area_vision
```c
void gx_set_area_vision(int player_id, table params)
```

```c
table params = {
    string m_location = {},         // location to give or take away vision of (depending on m_bSet)
    string m_triangleGroup = {},    // triangle group to give or take away vision of (depending on m_bSet)
    bool m_bFullMapVision = {}      // If set to true, will give or take away vision of entire map (depending on m_bSet)
    bool m_bSet = true              // If true gives vision, else takes away vision. (default: true)
}
```
- This function gives or removes permanent vision of a `m_location`, `m_triangleGroup`, or `entire map`.
- Only one of `m_location`, `m_triangleGroup`, or `m_bFullMapVision` should be set.
- The only valid value of `m_bFullMapVision` is `true`. Setting to `false` is `undefined`.
- If you want to give `Full Map Vision` to a player, set `m_bFullMapVision` to `true` and `m_bSet` to `true`.
- If you want to remove `Full Map Vision` from a player, set `m_bFullMapVision` to `true` and `m_bSet` to `false`.
- The value of `m_bSet` determines if vision is given or taken away from player.

# gx_get_area_vision
```c
bool gx_get_area_vision(int player_id, table params)
```

```c
table params = {
    string m_location = {},         // location to query vision for
    string m_triangleGroup = {},    // triangle group to query vision for
    bool m_bFullMapVision = {}      // If set to true, queries if player has full map vision
}
```

- This function queries if the player has permanent vision of a `location`, `triangleGroup`, or `entire map`.
- Only one of `m_location`, `m_triangleGroup`, or `m_bFullMapVision` should be set.
- The only valid value of `m_bFullMapVision` is `true`. Setting to `false` is `undefined`.

# TerrainTypes enum
```c
// Primary Terrain Types
enum TerrainTypes
{
    Normal = 0,         // See SecondaryTerrainTypeNormal for valid secondary types
	Water = 2,          // valid secondary types are 0 and 2
	Lava = 3,           // valid secondary types are 0 and 2
	Diamond = 4,        // valid secondary types is just 0
    Glow = 6,           // valid secondary types are [0 - 31]
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
    string m_location = {}      // location to set terrain tile types.,
    string m_triangleGroup = {} // triangle group to set terrian tile stype
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

- if only `m_index` is set, the square at `m_index` is set to terrain type (i.e. both bottom and top triangle)
- if `m_location` is set, the triangles within the `m_location` are set to the new terrain type
- if `m_triangleGroup` is set, the triangles within `m_triangleGroup` are set to the new terrain type
- if `m_location` is set, it is `undefined behavior` to also set  `m_index`, `m_index2`, or `m_triangleGroup`
- if `m_triangleGroup` is set, is it `undefined behavior` to also set `m_index`, `m_index2`, or `m_location`
- if `m_index` is set, it is `undefined behavior` to also set `m_location`, and `m_triangleGroup`



# gx_get_terrain_type
```c
Vec2 gx_get_terrain_type(Vec2 index, int index2 = 0)
````
- returns primary terrain type in `vec.x` and returns secondary terrain type in `vec.y`

# gx_set_player_camera_look_at
```c
void gx_set_player_camera_look_at(int player_id, table params)
```

```c
local params = {
    int m_unitID = {},
    string m_location = {}
}
```

- Moves camera of `player_id` to look at `m_unit` or `m_location`
- One of `m_unit` or `m_location` should be set. Not both.

# gx_lock_player_camera
```c
void gx_lock_player_camera(int player_id, table params = {})
```

```c
local params = {
    int m_unitID = {},
    string m_location = {}
}
```

- locks `player_id` camera to look at `m_unitID` or `m_location`.
- camera will follow `m_unitID` or `m_location` until `m_unitID` is removed from game
- can unlock camera by calling `gx_unlock_player_camera(player_id)`
- passing empty args for `params` will unlock the camera for `player_id`.

# gx_unlock_player_camera
```c
void gx_unlock_player_camera(int player_id)
```

- unlocks `player_id` camera position set by `gx_lock_player_camera`.
- equivalent to calling `gx_lock_player_camera(player_id, {})`

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
	Fungus = 1,             // Read / Write     (float)
	Gemstone = 2,           // Read / Write     (float)
	Supply = 3,             // Read-Only        (int)
	MaxSupply = 4,          // Read-Only        (int)
	NumKills = 5,           // Read-Only        (int)
	NumDeaths = 6,          // Read-Only        (int)
    PlayerName = 7,         // Read-Only        (string)
    FullMapVision = 8,      // Read / Write     (bool)
                            // When set to true, player is given vision of entire map
    NumUnitsProduced = 9,   // Read-Only        (int)
    TagID = 10,             // Read-Only        (int)
    ChoseRandom = 11,       // Read-Only        (bool)
    Race = 12               // Read-Only        (int)
    WeaponsLevel = 13       // Read-Write       (int)
    ArmorLevel = 14         // Read-Write       (int)
    SpeedLevel = 15         // Read-Write       (int) (not implemented atm)
    StartLocationPosition = 16  // Read         (vec2) (not implemented atm)
}
```

# gx_get_player_prop
```c
mixed gx_get_player_prop(PlayerProps prop, int playerID)
```

# gx_set_player_prop
```c
void gx_set_player_prop(PlayerProps prop, int playerID, mixed val)
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

# GunShipState enum
```
enum GunShipState
{
    Normal = 0,
    StarShot = 1,
    BigGunLevel1 = 2
    BigGunLevel2 = 3,
    ChainGunLevel1 = 4,
    ChainGunLevel2 = 5
}
```
- to be used with `gx_set_unit_prop` and `gx_get_unit_prop`
- `gx_set_unit_prop(unit_id, UnitProps.GunShipState, GunShipState.ChainGunLevel2)`
- the above would give a gunship two chainguns

# CommandType enum
```c
enum CommandType : string
{
    Attack,             // valid params: [m_unitID, m_location, m_pos]
    Move,               // valid params: [m_unitID, m_location, m_pos]
    Hold,               // valid params: []
    Stop                // valid params: []
    RightClick          // valid params: [m_unitID, m_location, m_pos]
}
```
- `CommandType` enums that are meant to be used with `gx_queue_command`
- more to be added later

# gx_queue_command
```c
gx_queue_command(int unit_ids[], CommandType command, table params = {})
```

```c
table params = {
    int m_unitID,            // unit to target
    string m_location = {},     // location to target
    string m_pos = {},          // position to target
}
```

- only one (or zero) unit_id, m_location, m_pos should be set
- some commands/spells only work when certain params are set
- (i.e., a spell that can only target units cannot target a `m_pos` or `m_location`)
- `CommandType` can also be a spell identifier

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
    GetParentBunker = 11        // Read-Only (int),
    GunShipState = 12           // Read-Write (GunShipState enum)
}
```

- Setting `Health` to `<= 0` will cause unit to be set to `killed` state.

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
# gx_set_player_variable
```c
void gx_set_player_variable(int playerID, string varName, mixed varValue)
```
- Valid Types for `varValue` are (int, float, string)

# gx_get_player_variable
```c
mixed gx_get_player_variable(int playerID, string varName)
```

# gx_set_unit_variable
```c
void gx_set_unit_variable(int unitID, string varName, mixed varValue)
```
- Valid Types for `varValue` are (int, float, string)

# gx_get_unit_variable
```c
mixed gx_get_unit_variable(int unitID, string varName)
```

# gx_set_force_variable
```c
void gx_set_force_variable(int forceID, string varName, mixed varValue)
```
- Valid Types for `varValue` are (int, float, string)

# gx_get_force_variable
```c
mixed gx_get_force_variable(int forceID, string varName)
```

# gx_set_sim_variable
```c
void gx_set_sim_variable(string varName, mixed varValue)
```
- Valid Types for `varValue` are (int, float, string)

# gx_get_sim_variable
```c
mixed gx_get_sim_variable(string varName)
```

# Event Queue

Event Types:
```c
enum EventType
{
    Invalid = 0,            // Invalid Event
    PlayerNameChanged = 1,  // Populates m_playerID, m_playerName, and m_oldPlayerName of the Event structure
    PlayerLeftGame = 2,     // Populates m_playerID, and m_playerName of the Event structure
    TextCommand             // Populates m_playerID, m_playerName, and m_cmd of Event structure
}
```

Event Structure:
```c
table Event
{
    EventType m_type        // Always populated, specifies type of event.
    int m_playerID = {}
    string m_playerName = {}
    string m_oldPlayerName = {}
    string m_cmd = {}
}
```

- Look at the comments in the definition of `EventType` enum to see which fields each `EventType` populates.

# gx_is_event_queue_empty
```c
bool gx_is_event_queue_empty()
```

# gx_pop_event_from_queue()
```c
Event gx_pop_event_from_queue()
```

Example of reading events from queue

```c
function gx_sim_update()
{
    while (!gx_is_event_queue_empty())
    {
        local ev = gx_pop_event_from_queue()
        if (ev.m_type == EventType.PlayerLeftGame)
        {
            gx_print("Player " + ev.m_playerID + " has left the game!")
        }
        else if (ev.m_type == EventType.PlayerNameChanged)
        {
            gx_print(ev.m_oldPlayerName + " changed name to " + ev.m_playerName)
        }
    }

    // do rest of game logic
}
```

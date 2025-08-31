# `int gx_create_unit(table params)`
```c
int gx_create_unit(table params)
```
```c
table params = {
    string m_unitType = {}          # Required
    int m_playerID = {},            # Required
    Vec2 m_position = {},           # Optional
    std::string m_location = {}     # Optional
}
```

```c
local new_unit = gx_create_unit({ m_unitType = "Brute", m_playerID=1, m_location="my_cool_location" })
```

- Will create unit of type `m_unitType` at position `m_position` or at location `m_location` for player `m_playerID`.
- It is undefined behavior to have both `m_position` and `m_location` set.
- If neither `m_position` nor `m_location` is set, unit will be created at `(0, 0)`

# `int gx_get_sim_tick()`
```c
int gx_get_sim_tick()
```
- Returns the current sim tick number in simulation.
- The first `gx_update` call will have tick = 1
- every tick corresponds to 50ms real time

# `int[] gx_get_units_at_location(table params = {})`
```c
int[] gx_get_units_at_location(table params)
```
```c
table params = {
    string m_location = {}, # Required, location to get units from
    string m_unitType = {}  # Optional, Filter for unit type
    int m_playerID = {},    # Optional, Filter for player_id
    int m_teamID = {},      # Optional, Filter for team_id
    bool m_bReturnAirUnits = true,      # Optional, Set to false if you want to exclude air units
    bool m_bReturnGroundUnits = true    # Optional, set to false if you want to exclude ground units
}
```
Example:
```c
local the_units = gx_get_units_at_location({
    m_location = "my_cool_location",
    m_unitType = "Brute",
    m_playerID=1
})
foreach (unit in the_units) {
    gx_kill_unit(unit)
}
```

# `void gx_create_explosion(table params)`
```c
void gx_create_explosion(table params)
```

```c
table params = {
    float m_size = {},              # Optional, Diameter of explosion
    Vec3 m_color = Vec3(1,1,0)      # Optional, ColorSRGB of explosion.
    std::string m_location = {},    # Optional, Location for explosion
    Vec2 m_pos2d = {}               # Optional, Position for explosion
}
```

Example:
```
gx_create_explosion( {
    m_color = Vec3(1, 0, .5),
    m_location = "my_location"
} )
```

- it is undefined behavior to set both `m_location` and `m_pos2d`
- if `m_size` is not set and `m_location` is set, the resolved size will be the minimum width/height of `m_location`
- if `m_size` is not set and `m_pos2d` is set, the resolved size will be `1`
- Default value for `m_color` is `Vec3(1,1,0)` aka `0xFFFF00` (yellow)

# `void gx_kill_unit(int unitID)`
```c
void gx_kill_unit(int unitID)
```

- kills the unit `unitID`.
- It is safe to call this function on already killed units

# `bool gx_unit_exists(int unitID)`
```c
bool gx_unit_exists(int unitID)
```

- checks if unit still exists in the game
- Note: this will still return `true` if unit is killed but not yet removed, since some units may not be removed immediately (i.e. they have a death animation)

# `void gx_remove_unit(int unitID)`
```c
void gx_remove_unit(int unitID)
```

- sets the unit to be removed by game
- `unit_id` will be removed from game before the next gx_update call
- It is safe to call this function on already killed or removed units

# `bool gx_is_unit_alive_and_constructed(int unitID)`
```c
bool gx_is_unit_alive_and_constructed(int unitID)
```
- returns `true` if unit is alive and constructed

# `bool gx_is_unit_alive(int unitID)`
```c
bool gx_is_unit_alive(int unitID)
```
- returns `true` if unit is alive
- Note: This function still returns `true` if unit is not yet fully constructed

# `vec2 gx_get_unit_position(int unitID)`
```c
vec2 gx_get_unit_position(int unitID)
```

- returns position of unit
- returns Vec2(0, 0) if unit no longer exists

# `void gx_set_unit_position(int unitID, table params)`
```c
void gx_set_unit_position(int unitID, table params)
```

```c
table params = {
    string m_location = {}, # Optional, location to put unit
    Vec2 m_pos2d = {}  # Optional, position to put unit
}
```
example
```c
gx_set_unit_position(some_unit, { m_location = "location_to_teleport_to" } )
```

- it is undefined to set both `m_location` and `m_pos2d`
- if neither is defined, unit will be teleported to Vec2(0, 0)

# `bool gx_is_ground_unit(int unitID)`
```c
bool gx_is_ground_unit(int unitID)
```
- returns if unit is currently a `ground` unit
- units are always in either the `ground` or `air` state
- note: knock-back effect can cause `ground` units to temporarily become `air` units
- equivalent to calling `!gx_is_air_unit(unitID)`

# `bool gx_is_air_unit(unit_id)`
```c
bool gx_is_air_unit(int unitID)
```
- returns if unit is currently an `air` unit
- units are always in either the `ground` or `air` state
- note: knock-back effect can cause `ground` units to temporarily become `air` units
- equivalent to calling `!gx_is_ground_unit(unitID)`

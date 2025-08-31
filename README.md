DriftScript API

int gx_get_sim_tick()

int gx_create_unit(table params)

ints gx_get_units_at_location(str locationName)

void gx_create_explosion(table params)

void gx_kill_unit(int unit_id)

bool gx_unit_exists(int unit_id)

void gx_remove_unit(int unit_id)

bool gx_is_unit_alive_and_constructed(unit_id)
    
bool gx_is_unit_alive(unit_id)

vec2 gx_get_unit_position(unit_id)

void get_set_unit_position(int unitID, table params)

bool gx_is_ground_unit(unit_id)

bool gx_is_air_unit(unit_id)

int get_player_id(unit_id)

void gx_set_player_gemstone(player, gemstone)

void gx_set_player_fungus(player, fungus)

void gx_add_player_gemstone(player, gemstone)

void gx_add_player_fungus(player, fungus)

int gx_get_player_gemstone(player)

int gx_get_player_fungus(player)

int gx_get_player_supply(player)

int gx_set_achievement_for_player(player, achievementID)

void gx_output_text(text)

ids gx_get_players_in_game()

void gx_ud_copy(name, newName)

void gx_fling_unit(unit, params)

vec2 gx_get_location_position(locationName)

int gx_get_unit_count_for_player(player_id, params)

void gx_teleport_unit(unit, params)

string gx_encode_text(string text)

void gx_set_defeat(table params)

void gx_set_victory(table params)

void gx_modify_ud_props(string unitName, table params)

void gx_print(string msg, table params = {})

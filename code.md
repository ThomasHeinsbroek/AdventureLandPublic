    var monster_targets = ["crab"];
    var state = "farm";
    var min_potions = 50; 
    var purchase_amount = 50;
    var potion_types = ["hpot0", "mpot0"];

    setInterval(function(){
	
        function handle_death(){
            setTimeout(respawn,15000);
        }
        if(character.rip) {respawn();}
        
        if(can_heal(parent.character) &&
           parent.character.hp / parent.character.max_hp < 0.7){
            heal(parent.character);
            game_log("Healed self!")
        }
        
        if (character.hp / character.max_hp < 0.75 
            || character.mp / character.max_mp < 0.75) {
            use_hp_or_mp();
        }
        
        state_controller();
        
        switch(state)
        {
            case "farm":
                farm();
                break;
            case "resupply_potions":
                resupply_potions();
                break;
        }
    }, 500);

    function state_controller(){
        //Default to farming
        var new_state = "farm";
                
        //Do we need potions?
        for(type_id in potion_types){
            var type = potion_types[type_id];
            var num_potions = num_items(type);			
            if(num_potions < min_potions){
                new_state = "resupply_potions";
                break;
            }
        }
        if(state != new_state) state = new_state
    }


    function farm(){
        
        loot();
        party_heal()
        
        var target=get_targeted_monster();
        if(!target){
            target=get_nearest_monster({type: "crab"});
            if(target) change_target(target);
                else if (!smart.moving) {
                    smart_move({to: "crab"});
                    game_log("smart move to crab")
                }
        }
        if(target){
            if(!is_in_range(target)){
                move(
                    character.x+(target.x-character.x)/2,
                    character.y+(target.y-character.y)/2
                );// Walk half the distance
            } else if (can_attack(target)) {
                set_message("Attacking");
                attack(target);
            }
        }	
    }

    function party_heal(){
            var warrior = get_player("Makker");
            var ranger = get_player("boogman")
            if(warrior != null && can_heal(warrior) 
                && warrior.hp / warrior.max_hp < 0.7) {
                heal(warrior);
                game_log("Healed Makker!")
            } else if(ranger != null && can_heal(ranger) 
                      && ranger.hp / ranger.max_hp < 0.7) {
                heal(ranger);
                game_log("Healed Boogman!")
            }
    }

    function resupply_potions(){
        var potion_merchant = get_npc("fancypots");
        var distance_to_merchant = null;
        
        if(potion_merchant != null) {
            distance_to_merchant = distance_to_point(
                potion_merchant.position[0],
                potion_merchant.position[1]
            )
        }
        
        if (!smart.moving 
            && (distance_to_merchant == null
                || distance_to_merchant > 250)) {
                smart_move({ to:"potions"});
        }
        
        if(distance_to_merchant != null 
           && distance_to_merchant < 250) {
            buy_potions()
        }
    }

    function buy_potions() {
        if(empty_slots() > 0) {
            for(type_id in potion_types) {
                var type = potion_types[type_id];
                var item_def = parent.G.items[type];
                
                if(item_def != null) {
                    var cost = item_def.g * purchase_amount;
    
                    if(character.gold >= cost)
                    {
                        var num_potions = num_items(type);
    
                        if(num_potions < min_potions)
                        {
                            buy(type, purchase_amount)
                        }
                    }
                    else
                    {
                        game_log("Not Enough Gold!");
                    }
                }
            }
        } else {
            game_log("Inventory Full!");
        }
    }


    function num_items(name){
    	var item_count = character.items.filter(
    		item => item != null && item.name == name
    	).reduce(function(a,b){ return a + (b["q"] || 1)}, 0);
    	
    	return item_count;
    }

    function empty_slots()
    {
        return character.esize;
    }
    
    function get_npc(name)
    {
        var npc = parent.G.maps[character.map].npcs.filter(npc => npc.id == name);
        
        if(npc.length > 0)
        {
            return npc[0];
        }
        
        return null;
    }
    
    function get_monster(type){
        var monster = parent.G.maps[character.map].monsters.filter(
            x => x.type == type
        );
        if(monster.length > 0){
            return monster[0]
        }
        
        game_log("could't find monster with type: " + type)
        
        return null
    }
    
    function distance_to_point(x, y) {
        return Math.sqrt(Math.pow(character.real_x - x, 2) 
                         + Math.pow(character.real_y - y, 2));
    }
    
    function move_to_target(target) {
        if (can_move_to(target.real_x, target.real_y)) {
            smart.moving = false;
            smart.searching = false;
            move(
                character.real_x + (target.real_x - character.real_x) / 2,
                character.real_y + (target.real_y - character.real_y) / 2
            );
        }
        else {
            if (!smart.moving) {
                smart_move({ x: target.real_x, y: target.real_y });
            }
        }
    }

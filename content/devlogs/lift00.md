---
draft: false 
title: "Lift00"
date: 2025-09-11T21:52:05-07:00
# IMPORTANT: Set this to the 'uid' of the project this dev log belongs to.
project_uid: "lift" 
hidden: false
archived: false
---
> [*Description*]
> this new movement system makes it almost feel like quake live movement. momentum acquired!

> [*Screenshots*]
> ![](/projects/lift/lift-main-menu.png)

> [*Code*]
{{< code language="gdscript" open="true">}}
# Source-Like Movement
func _ready:
    var h_vel = velocity
    h_vel.y = 0
    
    var current_speed_limit = SPRINT_SPEED if Input.is_action_pressed("sprint") else SPEED_DEFAULT

    if is_on_floor():
        # --- Ground Movement ---
        # Apply friction
        var speed = h_vel.length()
        if speed > 0:
            var drop = speed * friction * delta
            h_vel *= max(speed - drop, 0) / speed
        
        # Apply ground acceleration
        h_vel = _accelerate(h_vel, wish_dir, current_speed_limit, ground_accel, delta)
    else:
        # --- Air Movement ---
        # Use the _accelerate function for air movement too.
        # The key is the `air_strafe_speed` which dictates how much "purchase" your strafe has.
        h_vel = _accelerate(h_vel, wish_dir, air_strafe_speed, air_accel, delta)

    velocity.x = h_vel.x
    velocity.z = h_vel.z

    

    # --- APPLY MOVEMENT ---
    move_and_slide()

    # --- POST-MOVEMENT JUICE ---
    var cam_tilt_input_dir = Input.get_vector("move_left","move_right","move_forward","move_backward",)
    cam_tilt(cam_tilt_input_dir.x, delta)
    weapon_tilt(cam_tilt_input_dir.x, cam_tilt_input_dir.y, delta)
    jump_weapon_tilt(delta)
    weapon_bob(velocity.length(), delta)

    if footstep_component:
        footstep_component.update_state(is_on_floor(), velocity, delta)

func _accelerate(current_velocity: Vector3, wish_dir: Vector3, wish_speed: float, accel: float, delta: float) -> Vector3:
    var current_speed_in_wish_dir = current_velocity.dot(wish_dir)
    var add_speed = wish_speed - current_speed_in_wish_dir
    
    if add_speed <= 0:
        return current_velocity
        
    var accel_speed = accel * delta * wish_speed
    accel_speed = min(accel_speed, add_speed)
    
    return current_velocity + wish_dir * accel_speed
{{< /code >}}

### Introduction

This document provides an in-depth analysis of the mechanics behind surfing in Counter-Strike: Source (CS:S), based on the HL2 engine SDK [code](https://github.com/lua9520/source-engine-2018-hl2_src/blob/master/game/shared/gamemovement.cpp).

**What is Surfing?**

Surfing in CS:S is a player movement technique that allows players to glide along sloped surfaces (ramps) to gain speed and navigate the map in ways not possible through normal movement. By utilizing the game's physics, players can achieve high speeds and perform complex maneuvers, making surfing a popular aspect of CS:S gameplay and custom maps.

---

### Surface Steepness

For a surface to be viable for surfing, it must meet specific criteria related to its angle:

- The surface must be steep enough that the player is considered "in the air" while interacting with it.
- Technically, the Z component of the surface's normal vector must be less than 0.7.
- **Mathematical Interpretation**:
  - A Z normal less than 0.7 corresponds to surfaces steeper than approximately 45 degrees.
- This condition ensures that the player is treated as "in the air" rather than "on the ground" when touching the surface.

---

### How Surfing Works

The fundamental concept in surfing is the interaction between the player and inclined surfaces:

- When a player collides with a sloped surface, their velocity is adjusted using the `ClipVelocity` function.
- `ClipVelocity` removes the component of velocity perpendicular to the surface, leaving only the component parallel to the surface.
- Gravity is applied every tick, adding downward (negative Z) velocity.
- **Conversion of Vertical to Horizontal Motion**:
  - The constant downward acceleration from gravity, when interacting with the inclined plane of the ramp, results in the conversion of vertical velocity into horizontal acceleration along the ramp's surface.
  - Since gravity consistently adds negative Z velocity, and the ramp's normal has a non-zero Z component, `ClipVelocity` will always remove some of this downward velocity, effectively converting it into horizontal motion along the ramp.
- Air acceleration allows the player to control their movement along the ramp and gain speed.

---

### Key Functions

Several key functions and concepts work together to enable surfing:

- **`ProcessMovement`**: The main function that orchestrates player movement processing and collision handling.
- **`TryPlayerMove`**: Handles collision detection and response, allowing the player to move through the game world while interacting with surfaces and obstacles.
- **`AirMove`**: Manages player movement while in the air, calculating desired movement direction and applying air acceleration.
- **`AirAccelerate`**: Applies acceleration to the player while airborne, allowing for speed gain and maneuverability.
- **`ClipVelocity`**: Adjusts player velocity upon collision with surfaces by removing the component of velocity into the surface.
- **`StartGravity`** and **`FinishGravity`**: Continuously add downward velocity to the player, applying gravity in two halves each frame.

**Note**: While an `AddGravity` function exists, it's typically not used in player movement. Instead, gravity is applied in halves using `StartGravity` and `FinishGravity`.

---

### Movement and Physics Terms

#### Velocity

- **Definition**: The player's current speed and direction in 3D space, represented as a vector with X, Y, and Z components.
- **Note**: In the context of surfing, "units" often refers to the magnitude of the horizontal (X and Y) components of velocity, while velocity itself is the full 3D vector.

#### Wishdir

- **Definition**: The normalized direction vector in which the player wishes to move, based on their input and view direction.
- **Significance**: Determines the direction of the player's intended movement.

#### Wishspeed

- **Definition**: The scalar speed at which the player intends to move, calculated based on the player's input.
- **Significance**: Determines the magnitude of the player's intended movement, used to scale the wishdir to form the wishvel.

#### Wishvel

- **Definition**: The velocity vector that the player intends to have, calculated using wishdir and wishspeed.
- **Significance**: Combines both direction (wishdir) and speed (wishspeed) to form the complete desired velocity vector, used to adjust the player's current velocity.

#### Air Max Wishspeed

- **Definition**: A cap on the wishspeed when the player is in the air, set to limit the maximum air acceleration. By default, it's set to 30 units and cannot be changed without modifying the game code or using external tools like server plugins.
- **Significance**: Limits the maximum amount of velocity that can be added through air acceleration per tick, preventing indefinite acceleration in the air.

#### Veer

- **Definition**: The component of the player's current velocity in the direction of the desired movement (wishdir).
- **Calculation**: Determined by the dot product of the current velocity and wishdir.
- **Significance**: Indicates how much the player is already moving in the desired direction, guiding how much acceleration is needed.

#### Gravity

- **Definition**: A constant downward force applied to the player, simulated by continuously decreasing the player's Z velocity each tick.
- **Significance**: Crucial for surfing as it provides the constant downward acceleration that, when combined with ramp angles, results in forward acceleration.

#### Surface Normal

- **Definition**: A vector perpendicular to a surface, used to determine how the player's velocity should be adjusted upon collision.
- **Significance**: For a surf ramp, this represents the angle of the ramp's slope and is crucial in determining how the player's velocity is affected when surfing.

#### Origin

- **Definition**: The player's current position in 3D space, specified by X, Y, and Z coordinates in a vector.

#### Backoff

- **Definition**: In the context of `ClipVelocity`, the amount of velocity directed into a surface that needs to be removed to prevent penetration.
- **Calculation**: Determined by the dot product of the incoming velocity and the surface normal, scaled by the overbounce factor.

#### Overbounce

- **Definition**: A factor in `ClipVelocity` that determines how much of the velocity component directed into a surface is negated.
- **Significance**: Affects how "bouncy" surfaces are. For normal surfaces, including surf ramps, it's usually set to 1.0 to prevent bouncing.

---

### Game Settings and Functions

#### sv_airaccelerate

- **Definition**: A game setting (console variable) that determines the rate at which a player can accelerate towards wishdir while in the air.
- **Significance**: Higher values allow for quicker direction and speed changes in the air, enhancing maneuverability.

#### Frametime

- **Definition**: The time interval between server ticks, typically represented in seconds (e.g., 0.015 for a 66 tick rate).
- **Significance**: Used in various calculations to ensure consistent physics. In multiplayer games like CS:S, movement calculations are tied to the server's tick rate to maintain consistent physics across different frame rates.

#### ClipVelocity

- **Definition**: A function that adjusts the player's velocity when colliding with a surface by removing the component of velocity into the surface.
- **Significance**: Critical for surfing, as it allows the player to maintain momentum along surfaces by preserving the component of velocity parallel to the surface.

#### TryPlayerMove

- **Definition**: A function that handles player movement, including collision detection and response.
- **Significance**: Coordinates the overall movement process, including calling `ClipVelocity` when necessary.

#### AirMove

- **Definition**: A function that handles player movement while in the air.
- **Significance**: Manages air control and calls `AirAccelerate` to apply air acceleration.

#### AirAccelerate

- **Definition**: A function that applies acceleration to the player while in the air based on player input.
- **Significance**: Allows players to gain speed and maneuver in the air, essential for advanced movement techniques like surfing and bunny-hopping.

#### StartGravity and FinishGravity

- **Definition**: Functions that apply gravity to the player's velocity at different points in the movement calculation, splitting gravity application into two halves each frame.
- **Significance**: Provide the constant downward acceleration necessary for surfing mechanics and improve numerical stability in physics calculations.

---

### CGameMovement::TryPlayerMove

`TryPlayerMove` is a core function in the Source engine's movement system. It handles collision detection and response, allowing the player to move through the game world while interacting with surfaces and obstacles.

#### 1. Initialization

```
int bumpcount, numbumps;
Vector dir;
float d;
int numplanes;
Vector planes[MAX_CLIP_PLANES];
Vector primal_velocity, original_velocity;
Vector new_velocity;
int i, j;
trace_t pm;
Vector end;
float time_left, allFraction;
int blocked;

numbumps = 4;  // Bump up to four times

blocked = 0;   // Assume not blocked
numplanes = 0; // and not sliding along any planes

VectorCopy(mv->m_vecVelocity, original_velocity);  // Store original velocity
VectorCopy(mv->m_vecVelocity, primal_velocity);

allFraction = 0;
time_left = gpGlobals->frametime;  // Total time for this movement operation.

new_velocity.Init();
```

- **Significance of `numbumps`**:
  - The `numbumps` variable limits the number of collision checks per frame to prevent excessive computation, allowing up to four collisions to be resolved each frame. This ensures performance while handling multiple collisions.

- Sets up variables for collision handling, including `bumpcount`, `numbumps`, `numplanes`, and `blocked`.
- Stores the original velocity and calculates the time left for the movement.

#### 2. Main Movement Loop

```
for (bumpcount = 0; bumpcount < numbumps; bumpcount++)
{
    if (mv->m_vecVelocity.Length() == 0.0)
        break;

    // Calculate end position
    VectorMA(mv->GetAbsOrigin(), time_left, mv->m_vecVelocity, end);

    // Perform trace
    TracePlayerBBox(mv->GetAbsOrigin(), end, PlayerSolidMask(), COLLISION_GROUP_PLAYER_MOVEMENT, pm);

    allFraction += pm.fraction;

    // Check for being stuck in a solid object
    if (pm.allsolid)
    {
        VectorCopy(vec3_origin, mv->m_vecVelocity);
        return 4;
    }

    // Move if we're not blocked
    if (pm.fraction > 0)
    {
        // Check for precision issues
        if (numbumps > 0 && pm.fraction == 1)
        {
            trace_t stuck;
            TracePlayerBBox(pm.endpos, pm.endpos, PlayerSolidMask(), COLLISION_GROUP_PLAYER_MOVEMENT, stuck);
            if (stuck.startsolid || stuck.fraction != 1.0f)
            {
                VectorCopy(vec3_origin, mv->m_vecVelocity);
                break;
            }
        }

        // Update position and reset plane count
        mv->SetAbsOrigin(pm.endpos);
        VectorCopy(mv->m_vecVelocity, original_velocity);
        numplanes = 0;
    }

    // Check if we've moved the entire distance
    if (pm.fraction == 1)
        break;

    // Add entity we collided with to touched list
    MoveHelper()->AddToTouched(pm, mv->m_vecVelocity);

    // Check if we hit a floor or wall
    if (pm.plane.normal[2] > 0.7)
        blocked |= 1;  // floor
    if (!pm.plane.normal[2])
        blocked |= 2;  // step / wall

    // Reduce remaining time
    time_left -= time_left * pm.fraction;

    // Store the impact plane
    if (numplanes >= MAX_CLIP_PLANES)
    {
        VectorCopy(vec3_origin, mv->m_vecVelocity);
        break;
    }
    VectorCopy(pm.plane.normal, planes[numplanes]);
    numplanes++;

    // Handle velocity modification
    if (numplanes == 1 && player->GetMoveType() == MOVETYPE_WALK && player->GetGroundEntity() == NULL)
    {
        // Reflect velocity for first impact when airborne
        for (i = 0; i < numplanes; i++)
        {
            if (planes[i][2] > 0.7)
            {
                // floor or slope
                ClipVelocity(original_velocity, planes[i], new_velocity, 1);
                VectorCopy(new_velocity, original_velocity);
            }
            else
            {
                // wall
                ClipVelocity(original_velocity, planes[i], new_velocity, 1.0 + sv_bounce.GetFloat() * (1 - player->m_surfaceFriction));
            }
        }
        VectorCopy(new_velocity, mv->m_vecVelocity);
        VectorCopy(new_velocity, original_velocity);
    }
    else
    {
        // Handle multiple plane collisions
        for (i = 0; i < numplanes; i++)
        {
            ClipVelocity(original_velocity, planes[i], mv->m_vecVelocity, 1);
            for (j = 0; j < numplanes; j++)
                if (j != i && mv->m_vecVelocity.Dot(planes[j]) < 0)
                    break;
            if (j == numplanes)
                break;
        }

        if (i == numplanes)
        {
            if (numplanes != 2)
            {
                VectorCopy(vec3_origin, mv->m_vecVelocity);
                break;
            }
            CrossProduct(planes[0], planes[1], dir);
            dir.NormalizeInPlace();
            d = dir.Dot(mv->m_vecVelocity);
            VectorScale(dir, d, mv->m_vecVelocity);
        }

        if (mv->m_vecVelocity.Dot(primal_velocity) <= 0)
        {
            VectorCopy(vec3_origin, mv->m_vecVelocity);
            break;
        }
    }
}
```

This main loop handles the core collision detection and response:

- Iterates up to `numbumps` times (default 4) to handle multiple collisions per frame.
- For each iteration:
  - Calculates the end position based on current velocity and time left.
  - Performs a trace to detect collisions.
  - Updates the player's position if not blocked.
  - Adjusts the player's velocity upon collision using `ClipVelocity`.
- **Multiple Collision Handling**:
  - `ClipVelocity` can be called multiple times per frame, especially when colliding with multiple planes (e.g., edges between ramps). This allows for complex collision resolution, essential when navigating transitions between different ramp surfaces in surfing maps.
- **Special Handling for Airborne Collisions**:
  - When the player is airborne (`GetGroundEntity() == NULL`), special handling allows for the characteristic behavior of "sticking" to surf ramps without falling off.

#### 3. Final Checks and Adjustments

```
if (allFraction == 0)
{
    VectorCopy(vec3_origin, mv->m_vecVelocity);
}

// Check for slamming into a wall
float fSlamVol = 0.0f;
float fLateralStoppingAmount = primal_velocity.Length2D() - mv->m_vecVelocity.Length2D();
if (fLateralStoppingAmount > PLAYER_MAX_SAFE_FALL_SPEED * 2.0f)
    fSlamVol = 1.0f;
else if (fLateralStoppingAmount > PLAYER_MAX_SAFE_FALL_SPEED)
    fSlamVol = 0.85f;

PlayerRoughLandingEffects(fSlamVol);

return blocked;
```

- If no movement occurred (`allFraction == 0`), velocity is set to zero.
- Checks for sudden stops (slamming into walls) and applies appropriate effects.
- Returns the `blocked` status, indicating what type of collision occurred (floor or wall/step).

#### Key Behavior for Surfing

- **Sliding Along Surfaces**: The function allows the player to slide along surfaces when colliding, essential for maintaining momentum on surf ramps.
- **Multiple Collision Handling**: Can handle multiple collisions per frame, allowing for complex interactions with surf ramps and other surfaces.
- **Velocity Adjustment**: The `ClipVelocity` calls adjust the player's velocity upon collision, preserving motion parallel to surfaces.
- **Air Movement**: Special handling for collisions while airborne allows players to "stick" to ramps, a key aspect of surfing.
- **Precision Checks**: Includes checks to prevent getting stuck in surfaces, ensuring smooth movement.

---

### CGameMovement::ClipVelocity

The `ClipVelocity` function is a critical component in the Source engine's movement system. It's responsible for adjusting the player's velocity when colliding with surfaces. This function is especially important for surfing mechanics, as it determines how the player's velocity changes when interacting with surf ramps.

#### Parameters

- `Vector& in`: The incoming velocity vector.
- `Vector& normal`: The normal vector of the surface being collided with.
- `Vector& out`: The resulting velocity vector after the collision.
- `float overbounce`: A factor that determines how much of the velocity should "bounce" off the surface.

#### 1. Surface Type Determination

```
float angle;
int blocked;

angle = normal[2];
blocked = 0x00;         // Assume unblocked.
if (angle > 0)          // If the plane that is blocking us has a positive Z component, then assume it's a floor.
    blocked |= 0x01;
if (!angle)             // If the plane has no Z, it is vertical (wall/step)
    blocked |= 0x02;
```

- Determines the type of surface based on the Z component of the normal vector.
- A positive Z component indicates a floor-like surface.
- A zero Z component indicates a vertical surface (wall or step).

#### 2. Velocity Adjustment Calculation

```
float backoff;
float change;
int i;

// Determine how far along plane to slide based on incoming direction.
backoff = DotProduct(in, normal) * overbounce;
for (i = 0; i < 3; i++)
{
    change = normal[i] * backoff;
    out[i] = in[i] - change;
}
```

- Calculates the amount of velocity to remove (`backoff`) based on how much the incoming velocity is going into the surface.
- The `overbounce` factor affects how much of this velocity is removed.
- Adjusts each component of the velocity vector to remove the calculated amount.
- **Importance of Uniform Application**:
  - The `backoff` value is uniformly applied to each X, Y, and Z component of velocity.
  - This means that any component of velocity directed into the surface normal is reduced, which is crucial for accurate collision response.

#### 3. Prevent Penetration

```
// Iterate once to make sure we aren't still moving through the plane
float adjust = DotProduct(out, normal);
if (adjust < 0.0f)
{
    out -= (normal * adjust);
}
```

- Checks if the resulting velocity is still moving into the surface.
- If so, it further adjusts the velocity to prevent penetration.
- **Note on `adjust` Variable**:
  - While it might seem minor, this step is important for collision accuracy, ensuring the player doesn't penetrate the surface after velocity adjustments.

#### 4. Return Result

```
// Return blocking flags.
return blocked;
```

- Returns the `blocked` value, indicating what type of surface was hit, if at all.

#### Key Behavior for Surfing

- **Velocity Preservation**:
  - By removing only the component of velocity going into the surface, `ClipVelocity` allows the player to maintain their speed along the surface of a ramp.
  - This preservation of parallel velocity is essential for smooth surfing.

- **Angle Sensitivity**:
  - The amount of velocity adjustment depends on the angle of collision.
  - The negative Z velocity from gravity is always going to be subtracted every single frame on the ramp since it is never going to be parallel with the ramp's normal.
  - This leaves the X and Y components (horizontal movement), which are more aligned with the ramp due to player input via air acceleration.

- **Overbounce Factor**:
  - The `overbounce` parameter is typically 1.0 for normal surfaces, including surf ramps.
  - **Effects of Different Overbounce Values**:
    - **Overbounce > 1.0**: Surfaces become "bouncy," causing the player to bounce off upon collision, which is undesirable for surfing.
    - **Overbounce < 1.0**: Surfaces absorb more velocity, causing the player to "sink" into the surface, also undesirable.
  - **For Surfing**: Keeping `overbounce` at 1.0 ensures that players smoothly glide along ramps without bouncing or losing speed.

- **Multiple Plane Collisions**:
  - `ClipVelocity` can be called multiple times, once per plane.
  - This is important when colliding in the middle of two surf ramps or other complex geometry.

---

### Air Movement in Source Engine: Comprehensive Breakdown

Air movement in the Source engine is primarily handled by two functions: `AirMove` and `AirAccelerate`. This article provides a detailed analysis of these functions, explaining their role in creating the characteristic feel of air control in Source games.

---

#### CGameMovement::AirMove

The `AirMove` function is responsible for handling player movement while in the air. It calculates the desired movement direction and speed based on player input, applies air acceleration, and attempts to move the player accordingly.

#### Function Breakdown

```
void CGameMovement::AirMove(void)
{
    int i;
    Vector wishvel;
    float fmove, smove;
    Vector wishdir;
    float wishspeed;
    Vector forward, right, up;

    AngleVectors(mv->m_vecViewAngles, &forward, &right, &up);  // Determine movement angles

    // Copy movement amounts
    fmove = mv->m_flForwardMove;
    smove = mv->m_flSideMove;

    // Zero out z components of movement vectors
    forward[2] = 0;
    right[2] = 0;
    VectorNormalize(forward);  // Normalize remainder of vectors
    VectorNormalize(right);

    for (i = 0; i < 2; i++)       // Determine x and y parts of velocity
        wishvel[i] = forward[i] * fmove + right[i] * smove;
    wishvel[2] = 0;             // Zero out z part of velocity

    VectorCopy(wishvel, wishdir);   // Determine magnitude of speed of move
    wishspeed = VectorNormalize(wishdir);

    // Clamp to server-defined max speed
    if (wishspeed != 0 && (wishspeed > mv->m_flMaxSpeed))
    {
        VectorScale(wishvel, mv->m_flMaxSpeed / wishspeed, wishvel);
        wishspeed = mv->m_flMaxSpeed;
    }

    AirAccelerate(wishdir, wishspeed, sv_airaccelerate.GetFloat());

    // Add in any base velocity to the current velocity.
    VectorAdd(mv->m_vecVelocity, player->GetBaseVelocity(), mv->m_vecVelocity);

    TryPlayerMove();

    // Now pull the base velocity back out.
    VectorSubtract(mv->m_vecVelocity, player->GetBaseVelocity(), mv->m_vecVelocity);
}
```

#### Detailed Analysis

- **Movement Angle Calculation**:
  - `AngleVectors` converts the player's view angles into forward, right, and up vectors.
  - This allows the function to determine the direction the player is facing.

- **Input Processing**:
  - `fmove` and `smove` store the forward and sideways movement inputs respectively.
  - These values represent the player's intended movement direction.

- **Zeroing Out Z Components**:
  - By setting `forward[2]` and `right[2]` to zero, the game ensures that player input does not directly influence vertical (Z-axis) velocity while in the air.
  - This constrains movement input to the horizontal plane, important for realistic air movement.

- **Wishvel Calculation**:
  - Calculates `wishvel`, the desired velocity vector based on movement inputs and orientation.
  - Combines the normalized forward and right vectors with the movement inputs.

- **Wishspeed Normalization**:
  - `wishdir` is set to the normalized `wishvel`.
  - `wishspeed` stores the magnitude of the desired velocity.

- **Speed Clamping**:
  - If `wishspeed` exceeds `mv->m_flMaxSpeed`, it's clamped to this maximum value.
  - Prevents players from exceeding the maximum allowed velocity.

- **Air Acceleration**:
  - Calls `AirAccelerate` with the calculated `wishdir`, `wishspeed`, and the value of `sv_airaccelerate` cvar.

- **Base Velocity Handling**:
  - Adds the player's base velocity (from moving platforms, etc.) to the current velocity.
  - After movement calculation, it subtracts the base velocity again.
  - This allows for correct interaction with moving objects in the game world.

- **Movement Execution**:
  - Calls `TryPlayerMove` to actually move the player, handling collisions and other physics interactions.

---

#### CGameMovement::AirAccelerate

The `AirAccelerate` function is crucial for changing the player's velocity while in the air. It's key to allowing skilled players to gain speed during air movement.

#### Function Breakdown

```
void CGameMovement::AirAccelerate(Vector& wishdir, float wishspeed, float accel)
{
    int i;
    float addspeed, accelspeed, currentspeed;
    float wishspd;

    wishspd = wishspeed;

    if (player->pl.deadflag)
        return;

    if (player->m_flWaterJumpTime)
        return;

    // Cap speed
    if (wishspd > GetAirSpeedCap())
        wishspd = GetAirSpeedCap();

    // Determine veer amount
    currentspeed = mv->m_vecVelocity.Dot(wishdir);

    // See how much to add
    addspeed = wishspd - currentspeed;

    // If not adding any, done.
    if (addspeed <= 0)
        return;

    // Determine acceleration speed after acceleration
    accelspeed = accel * wishspeed * gpGlobals->frametime * player->m_surfaceFriction;

    // Cap it
    if (accelspeed > addspeed)
        accelspeed = addspeed;

    // Adjust velocity
    for (i = 0; i < 3; i++)
    {
        mv->m_vecVelocity[i] += accelspeed * wishdir[i];
        mv->m_outWishVel[i] += accelspeed * wishdir[i];
    }
}
```

#### Detailed Analysis

- **Initial Checks**:
  - Returns immediately if the player is dead or water jumping.
  - Prevents unnecessary calculations in these states.

- **Speed Capping**:
  - Caps `wishspd` to `GetAirSpeedCap()`, which is almost always 30 units.
  - This limit prevents excessive acceleration, maintaining game balance.

- **Current Speed Calculation**:
  - `currentspeed` is calculated using the dot product of the current velocity and `wishdir`.
  - This measures how much of the player's current velocity is in the desired direction.
  - **Significance of `currentspeed`**:
    - Indicates alignment with `wishdir`.
    - A higher value means the player is already moving in the desired direction, requiring less acceleration.

- **Additional Speed Calculation**:
  - `addspeed` is the difference between desired speed (`wishspd`) and current speed (`currentspeed`).
  - If `addspeed` is less than or equal to zero, no acceleration is needed.

- **Acceleration Calculation**:
  - Calculates `accelspeed` based on `accel` (`sv_airaccelerate`), `wishspeed`, frametime, and surface friction.
  - Determines how much speed will be added this frame toward `wishdir`.

- **Acceleration Capping**:
  - Ensures `accelspeed` doesn't exceed `addspeed`.
  - Prevents overshooting the desired speed.

- **Velocity Adjustment**:
  - Adds the calculated acceleration to the player's velocity in the `wishdir` direction.
  - Updates `mv->m_outWishVel` for other systems to reference.

#### Key Implications for Air Movement and Surfing

- **Speed Control**:
  - The speed cap and acceleration calculations allow for fine-tuned control over player movement.
  - Players cannot gain infinite speed in the air due to the `Air Max Wishspeed` limit.

- **Directional Acceleration**:
  - By using `wishdir`, players can gain speed by turning into their movement direction.
  - Strafing on a ramp is similar to strafing midair; air acceleration contributes to maintaining or gaining speed while surfing.

- **Precision Movement**:
  - The careful capping and adjustment of acceleration allow for precise control and simulation in the air.
  - Air strafing techniques leverage this system to maximize speed and control.

- **Interaction with Surf Ramps**:
  - When `AirMove` calls `TryPlayerMove`, it initiates collision detection and updates the player's velocity and location each tick.
  - As the player moves through the air, `TryPlayerMove` detects collisions with ramp surfaces.
  - Upon collision, it calls `ClipVelocity` to adjust the player's velocity relative to the ramp angle.
  - The air acceleration applied by `AirAccelerate` enables players to control their movement along ramps and not fall off.

---

### Gravity Application in Source Engine

The Source engine applies gravity through two main functions: `StartGravity` and `FinishGravity`. These functions work together to simulate the continuous effect of gravity on the player throughout a frame.

#### Why Gravity is Applied in Two Halves

- Applying gravity in two halves (at the start and end of the frame) is a simple numerical integration technique known as the leapfrog method (or symplectic Euler method).
- This approach provides better stability and accuracy in physics simulations compared to applying it all at once.
- It ensures that velocity and position updates are synchronized in a way that accurately reflects continuous acceleration due to gravity.

---

#### CGameMovement::StartGravity

This function applies the initial half of the gravity effect at the start of a movement frame.

```
void CGameMovement::StartGravity(void)
{
    float ent_gravity;

    if (player->GetGravity())
        ent_gravity = player->GetGravity();
    else
        ent_gravity = 1.0;

    // Add gravity so they'll be in the correct position during movement
    mv->m_vecVelocity[2] -= (ent_gravity * GetCurrentGravity() * 0.5 * gpGlobals->frametime);
    mv->m_vecVelocity[2] += player->GetBaseVelocity()[2] * gpGlobals->frametime;

    Vector temp = player->GetBaseVelocity();
    temp[2] = 0;
    player->SetBaseVelocity(temp);

    CheckVelocity();
}
```

Key points:

- Applies half of the frame's gravity to the player's vertical velocity.
- Adds any vertical base velocity (e.g., from moving platforms).
- Resets the vertical component of base velocity to zero.
- Uses the player's custom gravity if set; otherwise, uses default gravity (1.0).
- Calls `CheckVelocity()` to ensure the velocity is within acceptable limits.

---

#### CGameMovement::FinishGravity

This function applies the remaining half of the gravity effect at the end of a movement frame.

```
void CGameMovement::FinishGravity(void)
{
    float ent_gravity;

    if (player->m_flWaterJumpTime)
        return;

    if (player->GetGravity())
        ent_gravity = player->GetGravity();
    else
        ent_gravity = 1.0;

    // Apply the remaining gravity
    mv->m_vecVelocity[2] -= (ent_gravity * GetCurrentGravity() * gpGlobals->frametime * 0.5);

    CheckVelocity();
}
```

Key points:

- Applies the remaining half of the frame's gravity to the player's vertical velocity.
- Skips gravity application if the player is water jumping.
- Does not add base velocity or reset it, as this was done in `StartGravity`.
- Uses the player's custom gravity if set; otherwise, uses default gravity (1.0).
- Calls `CheckVelocity()` to ensure the velocity is within acceptable limits.

#### Importance for Surfing

- **Continuous Acceleration**:
  - By applying gravity throughout the frame, these functions create a constant downward acceleration.
  - This is essential for simulating realistic physics and movement.

- **Interaction with Ramps**:
  - The downward velocity from gravity, when combined with `ClipVelocity` on angled surfaces, results in forward acceleration.
  - **How It Works**:
    - Gravity adds a downward component to the player's velocity vector.
    - When this velocity vector intersects with an angled ramp surface, `ClipVelocity` removes the component perpendicular to the surface.
    - The remaining velocity is parallel to the ramp surface, which includes both downward and forward components.
    - As gravity continues to apply, it adds more downward velocity, which again gets partially converted to forward velocity along the ramp.

---

### Example Flow

**Scenario**: A player is in the air, has some initial velocity, and is approaching a downward-sloping surf ramp.

#### Execution Flow

1. **Frame Start**:
   - **Function**: `CGameMovement::StartGravity()`
   - **Role**: Applies the initial half of gravity for this frame.
   - **Effect**: Increases the player's downward velocity slightly.

2. **Air Movement Calculation**:
   - **Function**: `CGameMovement::AirMove()`
   - **Role**: Calculates the player's intended movement based on input.
   - **Effect**: Determines the player's `wishdir` and `wishspeed` based on their input.

3. **Air Acceleration**:
   - **Function**: `CGameMovement::AirAccelerate()`
   - **Role**: Applies acceleration to the player's velocity based on their input.
   - **Effect**: Adjusts the player's velocity in the direction they're aiming.

4. **Movement Attempt**:
   - **Function**: `CGameMovement::TryPlayerMove()`
   - **Role**: Attempts to move the player and handles collisions.
   - **Effect**: Detects that the player is about to collide with the surf ramp.

5. **Collision Handling**:
   - **Function**: `CGameMovement::ClipVelocity()`
   - **Role**: Adjusts the player's velocity based on the collision with the ramp.
   - **Effect**:
     - Removes the component of velocity going into the ramp (negative Z from gravity).
     - Preserves the component of velocity along the ramp (X and Y from air acceleration).

6. **Continued Movement**:
   - **Function**: `CGameMovement::TryPlayerMove()` (continued)
   - **Role**: Continues moving the player along the adjusted trajectory.

7. **Frame End Gravity**:
   - **Function**: `CGameMovement::FinishGravity()`
   - **Role**: Applies the second half of gravity for this frame.
   - **Effect**: Further increases downward velocity, which gets partially converted to forward velocity along the ramp in the next frame.

**Throughout this process**:

- `CheckVelocity()` is called regularly to ensure the player's speed doesn't exceed engine limits.
- The player's position is updated each frame based on their velocity.
- Gravity is continually applied, affecting the player's trajectory.
- Air acceleration allows the player to control their movement along the ramp.

---

### Notes

**Version Information**:

- **Draft #3**
- **Dated 11/11/2024**

**Note**: This document may be updated periodically to reflect new insights or changes in the game's codebase.

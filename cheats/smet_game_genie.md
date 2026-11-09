# Super Metroid (SNES) Game Genie codes

These codes were made from the Japan/USA release (the only NTSC version).
SNES Game Genie codes have no compare byte; the original byte in each raw
form is documentation only. All effects come from the disassembly. They
have not been tested in play unless an entry says so.

## Samus jumps much higher, even without the Hi-Jump Boots

**Code:** `D5B8-C107`
**Raw form:** `909EB9:07` (orig `$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:15:19

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the Super Metroid version of the Metroid "jump
much higher" code. `Make_Samus_Jump` (`bank_90.asm`) sets Samus's first
Y speed from a table, picked by whether she is in air, water or
lava/acid:

```
    LDA.W EquippedItems
    BIT.W #$0100                 ; Hi-Jump Boots
    BNE .hiJumpEquipped
    LDA.W SamusPhysicsConstants_InitialYSubSpeeds_Jumping,X
    STA.W SamusYSubSpeed
    LDA.W SamusPhysicsConstants_InitialYSpeeds_Jumping,X
    STA.W SamusYSpeed
```

`SamusPhysicsConstants_InitialYSpeeds_Jumping` is `4, 1, 2` (air, water,
lava/acid) and the subspeeds are `$E0, $C0, $C0`, so a jump in air
starts at 4.875 pixels per frame. The Hi-Jump Boots table starts at 6.0.
The code changes the air speed to 7, so a jump starts at 7.875 pixels
per frame. With the air gravity (`$1C00` subpixels per frame per frame),
a full jump goes about 2.6 times as high: about 18 tiles instead of
about 7 (the Hi-Jump Boots give about 10).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `04 00 01 00 02 00` at `$90:9EB9` (PC `$081EB9`)
and `00 E0 00 C0 00 C0` at `$90:9EBF`, the subspeed table. Only
`Make_Samus_Jump` reads `SamusPhysicsConstants_InitialYSpeeds_Jumping`.
The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Once Samus has the Hi-Jump Boots, the game uses the Hi-Jump table and
  this code has no effect. To keep the high jump, unequip the boots on
  the status screen.
- Only jumps in air change. Wall jumps use their own table.
- Releasing jump still stops the rise, so short hops still work.
- The heights are worked out from the physics constants, so they are
  rough. Samus can go above the top of the screen in rooms one screen
  tall, and may hit ceilings that a normal jump doesn't reach.

## Samus runs much faster

**Code:** `D9B1-3907`
**Raw form:** `909F65:05` (orig `$02`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:16:40

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the Super Metroid version of the Metroid "run
faster" code. `SamusXSpeedTable_Normal` (`bank_90.asm`) has one row per
Samus movement type: X acceleration, max X speed and X deceleration, each
a 32-bit speed.pixel fraction value:

```
    dd wordSwap($0030*!SPF*$100),wordSwap($02C0*!SPF*$100),wordSwap($0080*!SPF*$100) ; 1: Running
```

`CalculateSamusXBaseSpeed_DecelerationDisallowed` adds the acceleration
to Samus's X base speed each frame and caps it at the max X speed. The
running max is `$02.C0`, 2.75 pixels per frame. The code changes the
whole-pixel part to 5, so the cap is 5.75 pixels per frame. The extra run
speed from holding the dash button is added on top of this, as before.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `00 00 00 30 02 00 00 C0 00 00 00 80` at `$90:9F61`
(PC `$081F61`), the running row. The max X speed is the second group of
four bytes: the speed word `02 00`, then the subspeed word `00 C0`. The
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only running on land, out of liquid, changes. Water and lava/acid use
  `SamusXSpeedTable_InWater` and `SamusXSpeedTable_InLavaAcid`.
- The acceleration is the same, so Samus takes about half a second to
  reach full speed, twice as long as before.
- Jumps have their own max X speed, so Samus slows down to the normal
  jump speed when she leaves the ground.
- The Speed Booster may charge at a different time, since its extra run
  speed builds on top of the base speed. Not checked.

## Samus can use the Speed Booster before she has it

**Codes:** `DD6D-196F` + `6D69-156F` + `DDBB-30DF` (use all three)
**Optional, for higher running jumps too:** `DDB4-10D4` + `DDBC-1504`
**Raw form:** `908506:00` (orig `$10`); `90855E:80` (orig `$D0`);
`909790:00` (orig `$36`); optional `909920:00` (orig `$19`);
`9099AD:00` (orig `$19`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:17:46

**Mechanism:** The game checks the Speed Booster bit (`$2000`) of
`EquippedItems` in six places, each with `LDA.W EquippedItems : BIT.W
#$2000` and a branch. The codes make the branches act as if the bit is
set (`bank_90.asm`):

- `Handle_Samus_XExtraRunSpeed`, `BEQ .noSpeedBooster` gets offset 0.
  While Samus runs with the dash button held, her extra run speed then
  builds up to the Speed Booster's max (7 pixels per frame out of
  liquid) instead of 2, and the boost timer starts.
- `Handle_SpeedBooster_AnimationDelay`, `BNE .speedBoostTimer` becomes
  `BRA`. This counts down the boost timer and raises
  `SamusBoostCounter`. At 4, the speed echo sound plays, and
  `Handle_Samus_XExtraRunSpeed` sets `ContactDamageIndex` to 1, the
  speed boosting contact damage that breaks blocks and kills enemies.
- The running animation code at `$90:84FF`, `BEQ .noSpeedBooster` gets
  offset 0, so the running animation speeds up with
  `SamusBoostCounter`.

The optional codes give the same offset 0 to `BEQ +` in
`Make_Samus_Jump` and `Make_Samus_WallJump`. These add half of the extra
run speed to the first Y speed of a jump, as the real Speed Booster
does.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD A2 09 89 00 20` and then the branch at
`$90:84FF` (`F0 10`), `$90:8558` (`D0 1D`), `$90:9789` (`F0 36`),
`$90:9919` (`F0 19`) and `$90:99A6` (`F0 19`) (PC `$0804FF`, `$080558`,
`$081789`, `$081919`, `$0819A6`). The sixth check is in
`UpdateSamusPoseDueToChangeOfEquipment` (`bank_91.asm`) at `$91:E644`;
it isn't changed (see below). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Pausing the game during a run stops the boost. On unpause,
  `UpdateSamusPoseDueToChangeOfEquipment` still sees no Speed Booster and
  clears the run momentum. A code for it would be a sixth code, more than a real Game
  Genie holds.
- The shinespark, the blue suit and speed blocks were not checked
  separately. They look at `SamusBoostCounter` and `ContactDamageIndex`,
  not at the item, so they should work.
- The boost needs a long enough run, as with the real item.
- Don't use with "Samus runs much faster". Not checked together.

## The Speed Booster charges almost at once

**Codes:** `D06B-19DF` + `DFBD-3007` (use both)
**Raw form:** `908594:04` (orig `$01`); `909F01:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:18:38

**Mechanism:** This is the Super Metroid version of the SMB3 "P-meter
fills at once" code. The Speed Booster has two parts that build up while
Samus runs with the dash button held (`bank_90.asm`):

- The boost stage. `Handle_SpeedBooster_AnimationDelay` runs once per
  running animation cycle. When the boost timer runs out (it is reloaded
  with 1 from `SpeedBoostTimerResetValues`, so every cycle), it adds 1 to
  `SamusBoostCounter`, in the high byte of a 16-bit add:

  ```
  LDA.W SamusBoostCounter-1
  BIT.W #$0400
  BNE +
  CLC
  ADC.W #$0100
  STA.W SamusBoostCounter-1
  BIT.W #$0400
  BEQ +                        ; speed echo sound at stage 4
  ```

  At stage 4, Samus glows and `Handle_Samus_XExtraRunSpeed` sets
  `ContactDamageIndex` to 1, the speed boosting damage. The first code
  changes `ADC.W #$0100` to `ADC.W #$0400`, so the counter goes from 0 to
  4 at the end of the first animation cycle instead of the fourth.
- The extra run speed. `Handle_Samus_XExtraRunSpeed` adds
  `SamusPhysicsConstants_XAccelerations_DashHeld` (pixels) and
  `XSubAccelerations_DashHeld` (subpixels) to it each frame, up to the
  max (7 pixels per frame with the Speed Booster). Out of liquid that is
  `$0000.$1000`, 1/16 pixel per frame, so it takes about 112 frames. The
  second code changes the whole-pixel part to 1, so it takes 7 frames.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 3E 0B 89 00 04 D0 19 18 69 00 01 8D 3E 0B` at
`$90:8589` (PC `$080589`); `$01` at `$90:8594` is the high byte of
`ADC.W #$0100`. It has `00 00 00 00 00 00` at `$90:9F01` (the
`XAccelerations_DashHeld` table: out of liquid, water, lava/acid) and
`00 10 00 04 00 04` at `$90:9F07` (the subaccelerations). The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus still needs the Speed Booster, or "Samus can use the Speed
  Booster before she has it". With both, use both codes of this entry
  and the three main codes of that one (five codes).
- The boost starts at the end of the first running animation cycle,
  about half a second, not on the very first frame.
- Samus still needs room to run. She reaches full speed in 7 frames,
  about 3 tiles.
- The second code also applies without the Speed Booster: the dash then
  reaches its normal max (2 pixels per frame) at once.
- Only running out of liquid is faster. Water and lava/acid use their own
  accelerations.

## Samus can morph into a ball without the Morphing Ball

**Code:** `DDE2-3909`
**Raw form:** `91F7D5:00` (orig `$16`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:19:21

**Mechanism:** This is the Super Metroid version of the Metroid "Maru
Mari" code. Pressing down while crouching picks the morphing transition
pose (`37h` facing right, `38h` facing left) from the pose transition
tables, whether or not Samus has the item. The pose's init routine,
`InitializeSamusPose_MorphingTransition` (`bank_91.asm`), then checks
for it:

```
    LDA.W EquippedItems
    BIT.W #$0004                 ; Morphing Ball
    BEQ .noMorphBall
    ...
  .noMorphBall:
    LDA.W PreviousPose
    STA.W Pose                   ; cancel the pose change
    SEC
    RTS
```

The code sets the `BEQ` offset to `$00`, so the branch always falls
through and the morph goes ahead.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD A2 09 89 04 00 F0 16` at `$91:F7CE` (PC
`$08F7CE`). This is the only Morphing Ball check in Samus's movement
code. The other one, in `Change_Pose_due_to_Equipment_Change_MovementTypes_7_9`
(`bank_82.asm`), is for unused movement types. The code decodes back to
its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Bombs, the Spring Ball and the Boost Ball still need their items.
- Samus can roll into morph-ball passages long before the game expects
  her to, which may skip parts of the game or get her stuck where she has
  no way out yet.

## Samus walks through walls; get stuck if you jump

**Code:** `05B3-499F`
**Optional, for shootable and grapple blocks too:** `05B3-459F` + `05BE-409F`
**Raw form:** `9494E5:47` (orig `$49`); optional `9494ED:47` (orig
`$49`); `9494F1:47` (orig `$49`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:20:02

**Mechanism:** This is the Super Metroid version of the Metroid, Zelda 1
and ALttP "walk through walls" codes. When Samus moves sideways,
`SamusBlockCollisionDetection_Horizontal` (`bank_94.asm`) calls
`SamusBlockCollisionReaction_Horizontal` for each block in her path. It
jumps through `SamusBlockCollisionReactionPointers_Horizontal` by block
type:

```
    dw CLCRTS_948F47                                       ; 0: Air
    ...
    dw SamusBlockCollisionReaction_Horizontal_SolidShootableGrapple ; 8: Solid block
    ...
    dw SamusBlockCollisionReaction_Horizontal_SolidShootableGrapple ; Ch: Shootable block
    dw BlockShotBombedGrappledCollisionInsideReaction_VerticalExt   ; Dh: Vertical extension
    dw SamusBlockCollisionReaction_Horizontal_SolidShootableGrapple ; Eh: Grapple block
```

The solid routine (`$94:8F49`) sets the carry ("collision") and the
distance to the wall. `CLCRTS_948F47` (`$94:8F47`) is the air reaction:
it clears the carry ("no collision"). Both are in page `$8F`, so the
code changes only the low byte of the solid block entry, from `$49` to
`$47`. Samus then moves sideways through solid blocks as if they were
air. The source notes the same idea next to the solid routine: "RTS this
routine to enable walk through walls, works surprisingly effectively".
The optional codes do the same for shootable blocks (type `Ch`) and
grapple blocks (type `Eh`).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has the table at `$94:94D5` (PC `$0A14D5`): `47 8F BB
8F 18 90 6F 90 47 8F 11 94 47 8F F9 92 49 8F ...`, with `49 8F` at
`$94:94E5`, `$94:94ED` and `$94:94F1`. It has `18 60` (`CLC : RTS`) at
`$94:8F47` and `64 14` (`STZ.B DP_Temp14`) at `$94:8F49`. Only
`SamusBlockCollisionReaction_Horizontal` reads the table. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only sideways movement changes. Floors and ceilings are still solid,
  so Samus can't fall or jump through them. Inside a wall she can stand
  on the blocks under her and hit her head on the blocks above, so she
  may get stuck in the wall until she walks out.
- Doors, slopes, spike blocks, bomb blocks and special blocks have their
  own reactions and work as before. Without the optional codes,
  shootable and grapple blocks are still solid.
- Enemies, shots and other objects still collide with walls as usual.
- Walking out of the room's edge where there's no door may put Samus out
  of bounds, where the game can act oddly.

## Low gravity: Samus jumps twice as high and falls slowly

**Code:** `D3BC-C067`
**Raw form:** `909EA2:0E` (orig `$1C`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:20:34

**Mechanism:** This is the Super Metroid version of the SMB3 and SMW low
gravity codes. `Determine_Samus_YAcceleration` (`bank_90.asm`) sets
Samus's gravity from `SamusPhysicsConstants`. Out of liquid it uses:

```
  .notSubmerged:
    LDA.W SamusPhysicsConstants_YSubAccelerationInAir   ; $1C00
    STA.W SamusYSubAcceleration
    LDA.W SamusPhysicsConstants_YAccelerationInAir      ; $0000
    STA.W SamusYAcceleration
```

`Samus_Y_Movement_WithSpeedCalculations` takes this off her Y speed
each frame while she rises, and adds it while she falls, until she
reaches the max falling speed of 5 pixels per frame. The code changes
the high byte of `$1C00` to `$0E`, so gravity is half as strong. Jumps
start at the same speed, so Samus rises twice as high, and she takes
about 1.4 times as long to fall the same height.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `00 1C 00 08 00 09` at `$90:9EA1` (PC `$081EA1`),
the air, water and lava/acid subaccelerations. The only other reader is
the game-start setup at `$91:E111`, which loads the same value. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- A full jump goes about 14 tiles high, about the height of the screen.
  Samus can reach ledges and ceilings early.
- Water and lava/acid without the Gravity Suit keep their own gravity.
  With the Gravity Suit, Samus uses the air gravity everywhere, so it is
  low gravity in liquid too.
- The Space Jump needs a falling speed in a set window, so its timing
  feels different.
- Bomb jumps, morph ball bounces and knockback also use this gravity.

## Water, lava and acid never slow Samus down

**Code:** `1039-49D7`
**Raw form:** `90EC54:64` (orig `$85`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:21:57

**Mechanism:** This is a reversed version of the SMW and SMB3 swim codes:
Samus moves in liquid as if she wore the Gravity Suit. About ten
routines in `bank_90.asm` pick Samus's physics with the same test. If
she doesn't have the Gravity Suit, they call `Get_Samus_Bottom_Boundary`
and compare her bottom edge (`$12`) with the liquid surface:

```
    JSL.L Get_Samus_Bottom_Boundary
    LDA.W FX_YPosition           ; water surface
    BMI .negativeYPosition
    CMP.B DP_Temp12
    BMI .submergedInWater        ; surface above her feet
    ...
  .negativeYPosition:
    LDA.W FX_LavaAcidYPosition   ; lava/acid surface
    BMI .notSubmerged
    CMP.B DP_Temp12
    BMI .submergedInAcidLava
```

The callers are `Determine_Samus_YAcceleration` (gravity),
`Determine_SamusXSpeedTable_EntryPointer` (run speed),
`Handle_Samus_XExtraRunSpeed` (dash), `Make_Samus_Jump`,
`Make_Samus_WallJump`, `Make_Samus_BombJump`, `SetSamusYSpeedForKnockback`
and `DetermineGrappleSwing_SamusXSpeedTable_EntryPointer`. `Get_Samus_Bottom_Boundary` ends with `STA.B
DP_Temp12` (`85 12`). The code changes it to `STZ.B DP_Temp12` (`64
12`), so it always returns 0, the top of the room. A liquid surface is
never above 0, so none of these routines see Samus as in liquid.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `08 C2 30 AD 1C 0A 0A 0A 0A AA BF 2F B6 91 29 FF
00 18 6D FA 0A 3A 85 12 28 6B` at `$90:EC3E` (PC `$086C3E`); `85 12` is
at `$90:EC54`. `$64` is `STZ` direct page. All callers of
`Get_Samus_Bottom_Boundary` were checked: the physics routines above,
`InitializeSamusPose_WallJumping`, and the footstep and landing graphics
routines. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Lava and acid still hurt Samus, and heated rooms still drain energy.
  Only the movement changes.
- Some code uses `Get_Samus_BottomTop_Boundary` instead, which isn't
  changed. So the Space Jump still needs the Gravity Suit in liquid, and
  Samus's animation and splash sounds still act as in liquid.
- The landing dust, footstep splashes and wall-jump dust use `$12` as
  their Y position, so they show at the top of the room instead of at
  Samus's feet. They also show in liquid, where they normally don't.

## Samus slides as if the ground were ice

**Code:** `FDB1-35D7`
**Raw form:** `909F6C:10` (orig `$80`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:23:32

**Mechanism:** This is the Super Metroid version of the SMB2, SMB3 and
ALttP ice codes. When Samus is running and the d-pad is let go,
`Set_ProspectivePoseChangeCommand` (`bank_91.asm`) keeps the running
pose and picks command 1, `ProspectivePoseCmd_1_Decelerate`. That adds
the extra run speed into her X base speed and sets
`SamusXAccelerationMode`. From then on,
`CalculateSamusXBaseSpeed_DecelerationAllowed` (`bank_90.asm`) takes the
X deceleration of her movement type's row in `SamusXSpeedTable_Normal`
off her speed each frame, until it reaches 0:

```
    dd wordSwap($0030*!SPF*$100),wordSwap($02C0*!SPF*$100),wordSwap($0080*!SPF*$100) ; 1: Running
```

The running deceleration is `$0000.$8000`, half a pixel per frame per
frame, so Samus stops in a few frames. The code changes the subspeed's
high byte to `$10`, 1/16 pixel per frame per frame. A normal run then
slides about 4 tiles before stopping, a dash run about 11 tiles, and a
Speed Booster run much farther.

The game also has an unused ice block: `Setup_IcePhysics` (`bank_84.asm`)
sets `SamusXDecelerationMultiplier` to `$10`, which scales the
deceleration's high byte the same way. `SamusBlockInsideHandling`
(`bank_94.asm`) clears the multiplier every frame, so this code changes
the table instead.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `00 00 00 30 02 00 00 C0 00 00 00 80` at `$90:9F61`
(PC `$081F61`, row 1). The deceleration is the last four bytes: the
speed word `00 00`, then the subspeed word `00 80`. The code decodes back
to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Tested by the user in Mesen2.
- Only stopping from a run slides. Samus still starts running at the
  normal rate, and the morph ball, jumps and liquid use
  their own rows or tables.
- During the slide Samus stays in her running pose.
- It's easy to slide off ledges and into enemies, spikes and lava.

## Skip the "1994 Nintendo presents" intro and go straight to file select

**Codes:** `94B5-3F43` + `BEB5-3F13` (use both)
**Raw form:** `8B9B74:52` (orig `$47`), `8B9B75:9F` (orig `$9A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:25:42

Tags: `curated`

**Mechanism:** After the Nintendo boot logo, game state 1 runs the title
sequence through the pointer in `CinematicFunction`. Its first function,
`CinematicFunction_LoadTitleSequence` (`bank_8B.asm`), loads the title
graphics, queues the title music, spawns the "1994" scrolling text, and
then sets the next function to `RTS_8B9A47`, so nothing happens until the
text's instruction list starts the "Nintendo presents" scenes:

```
    LDA.W #RTS_8B9A47                                                    ;8B9B73;
    STA.W CinematicFunction                                              ;8B9B76;
```

The codes change the operand from `$9A47` to `$9F52`,
`CinematicFunction_TransitionToFileSelectMenu`. That is the function the
title screen picks when Start is pressed. It fades the screen out, clears
the title sequence's RAM (the cinematic sprite objects too, so the
"1994" text can't start the scenes later), and sets `GameState` to 4,
the file select menus. The title music keeps playing, as it does when
you press Start on the title screen.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 47 9A 8D 51 1F` at `$8B:9B73` (PC `$059B73`).
The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Use both codes. With only one, the game jumps to `$9A52` or `$9F47`,
  which are not the start of a function, and will likely crash.
- The Nintendo boot logo still shows; only the intro after it is
  skipped.
- The title screen and the attract demos never appear. Leaving the file
  select menu for the title screen comes straight back to file select,
  as does a soft reset.
- There may be a short dark flash while the screen fades out.

## Enemies move only while a button is held

**Codes:** `68B5-1AB4` + `BAB5-1A24` + `93B5-12F4` + `F6B5-1294` +
`2DB5-12B4` (use all five)
**Raw form:** `B49972:8B` (orig `$91`) + `B49973:9C` (orig `$89`) +
`B49974:5E` (orig `$20`) + `B49975:18` (orig `$00`) + `B49976:D0` (orig
`$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:29:54

**Mechanism:** This is the Super Metroid version of the "time moves only
while a button is held" codes for Metroid, SMB3, SMW and ALttP. It uses
debug code that was left in the released game. Each frame of gameplay,
`GameState_8_MainGameplay` (`bank_82.asm`) calls `DebugHandler`
(`bank_B4.asm`). With no debug mode on, that runs
`DebugHandler_0_Default`, which toggles `DebugTimeIsFrozenForEnemies` when
L is newly pressed on controller 2:

```
  .checkL:
    LDA.B DP_Controller2New                                              ;B49971;
    BIT.W #$0020                                                         ;B49973;
    BEQ .notNewlyPressedStartL                                           ;B49976;
    LDA.W DebugTimeIsFrozenForEnemies                                    ;B49978;
    EOR.W #$0001                                                         ;B4997B;
    STA.W DebugTimeIsFrozenForEnemies                                    ;B4997E;

  .notNewlyPressedStartL:
```

The game only reads controller 2 in debug mode, so normally this never
runs. The codes change it to:

```
    LDA.B DP_Controller1Input           ; buttons held on controller 1
    STZ.W DebugTimeIsFrozenForEnemies   ; unfreeze
    BNE .notNewlyPressedStartL          ; any button held: done
    LDA.W DebugTimeIsFrozenForEnemies   ; else 0 ...
    EOR.W #$0001                        ; ... becomes 1 ...
    STA.W DebugTimeIsFrozenForEnemies   ; ... so freeze
```

`STZ` doesn't change the flags, so the `BNE` still tests the `LDA`. When
the flag is set, `Main_Enemy_Routine` (`bank_A0.asm`) treats enemies as if
time were frozen, as the X-Ray Scope does: it skips their collisions,
runs only their time-frozen AI (most enemies have none), and stops their
animations and flash timers. `HandleSpriteObjects` (`bank_B4.asm`) also
stops sprite objects such as explosions.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 91 89 20 00 F0 09 AD 5E 18 49 01 00 8D 5E 18` at
`$B4:9971` (PC `$1A1971`), so the debug code is in the released ROM. The
call to it at `$82:8B4B` is `22 09 98 B4`. `$8B` is `DP_Controller1Input`,
`$91` is `DP_Controller2New` and `$185E` is `DebugTimeIsFrozenForEnemies`
(`memory.asm`). `$9C` is `STZ` absolute and `$D0` is `BNE`. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only enemies freeze. Samus, her shots, enemy shots, doors, and other
  room objects keep moving. Freezing everything is possible (the same
  routine can tell the game to skip the frame), but then Samus and the
  enemies aren't drawn, so they vanish whenever no button is held.
- Only controller 1 counts. Any button, including the d-pad, counts.
- While enemies are frozen they can't touch Samus and her shots and
  bombs pass through them. A shot fired just before letting go of the
  buttons misses.
- Some bosses may still act while frozen if they have a time-frozen AI.

## The game runs at half speed

**Codes:** `A965-1FAC` + `6465-14DC` + `6D65-140C` (use all three)
**Raw form:** `828977:C5` (orig `$B0`) + `828978:82` (orig `$8A`) +
`828979:80` (orig `$82`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:32:59

Tags: `demo-ready` `play-start`

**Mechanism:** This is the Super Metroid version of the Metroid, Zelda 1
and ALttP half-speed codes. `MainGameLoop` (`bank_82.asm`) runs one frame
of the game, then waits for the NMI to send it to the screen:

```
    JSL.L Finalise_OAM                                                   ;828972;
if !DEBUG
    JSL.L ShowSpareCPUDebug_UpdatePrevCtrl1Input                         ;828976;
endif
    JSL.L WaitForNMI                                                     ;82897A;
```

The codes change the middle call to `WaitUntilTheEndOfAVBlank`
(`$80:82C5`, `bank_80.asm`), which waits for the next v-blank to start
and then end. The NMI at the start of that v-blank finds `NMI_Request`
clear, so it does nothing but count a lag frame. `WaitForNMI` then waits
for the NMI after that. So each frame of the game now takes two frames.
Controllers are read only by the NMI that runs a frame, so no button
presses are lost.

`ShowSpareCPUDebug_UpdatePrevCtrl1Input` is debug code left in the
released game. It shows spare CPU time when debug mode is on, and copies
`DP_Controller1Input` to `PreviousController1InputDrawing`. Samus's own
code also keeps that variable up to date (`$90:EACF`), so skipping the
call should be harmless.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 6E 89 80 22 B0 8A 82 22 38 83 80` at `$82:8972`
(PC `$010972`), the three calls above. `WaitUntilTheEndOfAVBlank` at
`$80:82C5` (PC `$0002C5`) is `48 08 E2 20 AD 12 42 10 FB AD 12 42 30 FB 28
68 6B`. It saves and restores A and the flags, and ends with `RTL`. The
codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Use all three codes. With only some, the call goes to the wrong
  address and the game will likely crash.
- Music and sound effects play at normal speed. The sound chip runs on
  its own.
- The Nintendo boot logo and some loading loops call `WaitForNMI`
  themselves, outside the main loop, so they still run at full speed.
- The in-game time counts game frames, so it runs at half speed too.

## No music

**Code:** `4A64-3FA7`
**Raw form:** `808F27:2C` (orig `$8D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:34:58

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the Super Metroid version of the Metroid, Zelda 1,
SMB3 and ALttP "no music" codes. Game code queues songs, and
`HandleMusicQueue` (`bank_80.asm`) sends each new track to the sound
chip when its timer runs out:

```
    AND.B #$7F                                                           ;808F1F;
    STA.W MusicTrackIndex                                                ;808F21;
    STZ.W MusicTrackIndex+1                                              ;808F24;
    STA.W $2140                                                          ;808F27;
    STA.W APU_CurrentMusicTrack                                          ;808F2A;
```

The code changes the `STA` (`$8D`) that writes `$2140` to `BIT` (`$2C`).
`BIT` only reads `$2140`, so the sound chip never gets the track number
and plays nothing. `MusicTrackIndex` still gets it, so the game's own
checks of the current track (for example, whether a room's music has to
change) work as before. Sound effects go through `$2141`..`$2143` and
are not affected.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `E2 20 29 7F 8D F5 07 9C F6 07 8D 40 21 8D 4C 06` at
`$80:8F1D` (PC `$000F1D`); the `8D 40 21` is at `$80:8F27`. This is the
only `STA.W $2140` in the source; the other writes to `$2140` are in the
code that uploads data to the sound chip. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Jingles that are music tracks, such as the item fanfare, are silent
  too.
- Uploading new music data to the sound chip still pauses the game
  briefly at the same times as before.

## No low-energy beep

**Code:** `DD36-C0A4`
**Raw form:** `90EA83:00` (orig `$1F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:35:37

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the Super Metroid version of the Metroid, Zelda 1
and ALttP "no low-energy beep" codes. `LowEnergyCheck` (`bank_90.asm`)
runs every frame. When Samus has less than 31 energy, it starts the
beep (library 3 sound 2) and sets `CriticalEnergyWarning`:

```
    LDA.W Energy                                                         ;90EA7F;
    CMP.W #$001F                                                         ;90EA82;
    BMI .lowEnergy                                                       ;90EA85;
```

The code changes the compare to `CMP.W #$0000`. Energy is never
negative, so `.lowEnergy` never runs and the beep never starts.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD C2 09 C9 1F 00 30 11` at `$90:EA7F` (PC
`$086A7F`). No other code in the source queues library 3 sound 2, so
this is the only place that starts the beep. The code decodes back to
its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Without the beep, it's easy to miss that energy is low. Watch the
  energy counter.

## Message boxes close almost at once

**Codes:** `6D66-47B9` + `6D6C-4799` (use both)
**Raw form:** `85848E:80` (orig `$F0`), `8584AD:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:36:25

**Mechanism:** This is the Super Metroid version of the Zelda 1 text
speed and SMW message box codes. Item pickups, map stations, recharge
stations and saving show a message box through `MessageBox_Routine`
(`bank_85.asm`). The game stops until the box closes.
`Handle_MessageBox_Interaction` waits 10 frames for the station messages
(`14h`, `15h`, `16h`, `18h`) and 360 frames (6 seconds) for the rest, and
then waits until a button is pressed:

```
    LDX.W #$000A                                                         ;85847A;
    ...
    CMP.B #$18                                                           ;85848C;
    BEQ .lagLoop                                                         ;85848E;
    LDX.W #$0168                                                         ;858490;

  .lagLoop:
    ...                         ; wait X frames
  .loopInput:
    LDA.W $4212                                                          ;8584A3;
    BIT.B #$01                                                           ;8584A6;
    BNE .loopInput                                                       ;8584A8;
    LDA.W $4218                                                          ;8584AA;
    BNE .busyReturn                                                      ;8584AD;
    LDA.W $4219                                                          ;8584AF;
    BEQ .loopInput                                                       ;8584B2;
```

The first code changes `BEQ .lagLoop` (`$F0`) to `BRA` (`$80`), so every
message waits only 10 frames. The second changes `BNE .busyReturn`
(`$D0`) to `BRA`, so the box closes without waiting for a button. The box
still opens and closes with its usual animation.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C9 18 F0 03 A2 68 01` at `$85:848C` (PC
`$02848C`) and `AD 18 42 D0 05 AD 19 42 F0 EF 60` at `$85:84AA`. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The "Would you like to save?" boxes (`17h`, `1Ch`) use their own code
  and still wait for a choice, so saving works as before.
- With only the first code, boxes still wait for a button. With only the
  second, item boxes still last 6 seconds.
- Skipping message boxes completely isn't done here: the save prompt
  returns the player's choice, and without the box saving would always
  answer "no".

## Screen fade-ins are instant

**Code:** `CB61-1F64`
**Optional, for door transitions too:** `DF21-1D6C`
**Raw form:** `808966:A9` (orig `$29`); optional `82D962:01` (orig `$0C`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:38:28

**Mechanism:** This is the Super Metroid version of the SMW fade-in code.
Most fade-ins (game start, the pause menu, file select, the intro and
ending) call `HandleFadingIn` (`bank_80.asm`) once per frame. It adds 1 to
the screen brightness (`$51`) every few frames until it reaches `Fh`:

```
    LDA.B DP_Brightness                                                  ;808963;
    INC                                                                  ;808965;
    AND.B #$0F                                                           ;808966; If brightness is not max:
    BEQ .return                                                          ;808968;
    STA.B DP_Brightness                                                  ;80896A; Increment brightness (disable forced blank)
```

The code changes `AND.B #$0F` (`29 0F`) to `LDA.B #$0F` (`A9 0F`), so
the first step sets full brightness. The callers check for brightness
`Fh` to know the fade is done, so they move on at once.

Door transitions don't use the brightness. They fade the palettes with
`Advance_GradualColorChange_ofAllPalettes_Denominator_C` (`bank_82.asm`),
which takes 13 steps (denominator `Ch`, plus 1):

```
    LDA.W #$000C                                                         ;82D961;
    STA.L PaletteChangeDenominator                                       ;82D964;
```

The optional code changes the denominator to 1, so the door fade takes 2
steps.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 51 1A 29 0F F0 02 85 51` at `$80:8963` (PC
`$000963`) and `A9 0C 00` at `$82:D961` (PC `$015961`). The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The first brightness step still waits for the fade delay
  (`ScreenFadeDelay`), so a fade-in may take a frame or two.
- The optional code makes the door fade-out fast too, since both door
  fades call the same routine.
- Fades that use other palette routines (Kraid's background, for
  example) are not changed.

## Samus can jump again in mid-air from a normal jump too

**Codes:** `6DC5-49DF` + `78C5-490F` + `71C0-35AD` (use all three)
**Optional, for falling off ledges too:** `71C9-39AD` + `C0C9-31DD`
**Raw form:** `90A474:80` (orig `$D0`), `90A475:3B` (orig `$12`),
`90A34F:36` (orig `$2E`); optional `90A357:36` (orig `$8D`), `90A358:A4`
(orig `$A5`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:41:19

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the Super Metroid version of the Metroid and SMB3
"jump again in mid-air" codes. While Samus spin jumps,
`SamusMovement_SpinJumping` (`bank_90.asm`) handles the Space Jump. Out
of liquid, it checks for the item, and then for a falling speed in a
small window, before it reads the jump button:

```
    LDA.W EquippedItems
    BIT.W #$0200
    BNE .spaceJumpEligible
    ...
  .spaceJumpEligible:
    LDA.W SamusYDirection
    CMP.W #$0002                 ; falling?
    BNE .screwAttackEligible
    ...                          ; Y speed must be $280..4FF
+   LDA.W neverRead0DFA
    ...
    LDA.B DP_Controller1New
    AND.W JumpBinding
    BEQ .screwAttackEligible
    JSL.L Make_Samus_Jump
```

The first two codes change `BNE .spaceJumpEligible` (`D0 12`) to `BRA +`
(`80 3B`). This skips both the item check and the speed window, so every
press of jump during a spin jump calls `Make_Samus_Jump` and starts a
new jump. Samus can jump while still rising, not only in the falling
window that the Space Jump needs.

Super Metroid has no "is Samus on the ground?" check to turn off. A jump
starts only when the pose transition table for Samus's current pose
(`TransitionTable`, `bank_91.asm`) has an entry for a new jump press.
The standing and running tables have one. The jumping and falling
tables don't, so a jump press in the air does nothing. The only mid-air
jump is the Space Jump code in `SamusMovement_SpinJumping`
(`bank_90.asm`), which the first two codes open up to every spin jump.

Each movement type runs its movement routine from a table in
`bank_90.asm`:

```
    dw SamusMovement_Jumping                                             ;90A34F; 2: Normal jumping
    dw SamusMovement_SpinJumping                                         ;90A351; 3: Spin jumping
    ...
    dw SamusMovement_Falling                                             ;90A357; 6: Falling
```

The third code changes the normal jumping entry from `$A42E` to `$A436`
(`SamusMovement_SpinJumping`), so normal jumps also run the Space Jump
code. The optional codes do the same for falling (`$A58D` to `$A436`).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `2E A4 36 A4` at `$90:A34F` (PC `$08234F`), `8D A5`
at `$90:A357`, `AD A2 09 89 00 02 D0 12` at `$90:A46E`, and `AD FA 0D`
(`LDA.W neverRead0DFA`, the `+` label) at `$90:A4B1`.
`$A476 + $3B = $A4B1`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Normal jumps and falls now also use the spin jump's horizontal
  movement (`Samus_SpinJumping_Movement`), so moving in the air may feel
  a little different. The first frames of a normal jump also lose their
  special handling in `Samus_Jumping_Movement`.
- With a fully charged beam, a normal jump does damage on contact, as a
  spin jump does (`ContactDamageIndex` = 4).
- When falling, the optional codes skip the falling pose's animation
  fix in `SamusMovement_Falling`, so the falling animation may look
  wrong.
- Normal jumps keep the normal jump pose after the new jump.
- In water, lava or acid without the Gravity Suit, the code is skipped,
  so Samus can't jump again there.
- All five codes fill every slot on a SNES Game Genie.

## Samus speed boosts whenever she runs, without the dash button or the Speed Booster

**Codes:** `DD6D-196F` + `6D69-156F` + `DDBB-30DF` + `6D60-190F` +
`6DB6-39DF` (use all five)
**Raw form:** `908506:00` (orig `$10`); `90855E:80` (orig `$D0`);
`909790:00` (orig `$36`); `908545:80` (orig `$D0`); `909784:80` (orig
`$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:42:16

**Mechanism:** The first three codes are the three main codes of "Samus
can use the Speed Booster before she has it". They make the game act as
if Samus has the Speed Booster.

The last two codes remove the dash button check. Two routines in
`bank_90.asm` build up the boost only while the dash button is held, and
only while Samus runs (movement type 1):

```
Handle_SpeedBooster_AnimationDelay:
    ...
  .checkPressingRun:
    LDA.B DP_Controller1Input                                            ;908540;
    BIT.W DashBinding                                                    ;908542;
    BNE .checkMovementType                                               ;908545;
    JMP.W .finish                                                        ;908547;

Handle_Samus_XExtraRunSpeed:
    ...
  .checkDash:
    LDA.B DP_Controller1Input                                            ;90977F;
    AND.W DashBinding                                                    ;909781;
    BNE .running                                                         ;909784;
    JMP.W .notRunning                                                    ;909786;
```

The codes change both `BNE` (`$D0`) to `BRA` (`$80`), so the game acts
as if the dash button is always held. The running check after it is
not changed, so the boost still builds up only while Samus runs on the
ground.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 8B 2C B6 09 D0 03 4C DA 85` at `$90:8540` (PC
`$080540`) and `A5 8B 2D B6 09 D0 03 4C 08 98` at `$90:977F` (PC
`$08177F`). The bytes for the first three codes are listed in the
Speed Booster entry. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus always runs at dash speed. There is no way to walk slowly.
- The boost still needs a long enough run. "The Speed Booster charges
  almost at once" would add two more codes, more than a Game Genie
  holds.
- The pause caveat of "Samus can use the Speed Booster before she has
  it" also applies here.

## New games skip Ceres Station and start in Crateria

**Codes:** `DD60-4D66` + `6D64-14A6` (use both)
**Raw form:** `828042:00` (orig `$06`); `82812B:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-20 22:46:33

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** After the intro, `CinematicFunction_FlyToCeres_StartGameAtCeres`
(`bank_8B.asm`) sets game state `1Fh` (set up new game) and stores `1Fh`
in `SRAMMirror_LoadingGameState`. Game state `1Fh` runs
`GameState_6_1F_28_LoadingGameData_SetupNewGame_LoadDemoData`
(`bank_82.asm`), which picks the start place from that value:

```
  .ceres:
    LDA.W #$0006                                                         ;828041;
    STA.W AreaIndex                                                      ;828044;
    STZ.W LoadStationIndex                                               ;828047;
    JSL.L ClearTimerRAM                                                  ;82804A;
```

The first code changes area 6 (Ceres) to area 0 (Crateria). Load station
0 of Crateria is the Landing Site, at the gunship save station.

Later the same routine sets Samus up for the start area:

```
    LDA.L SRAMMirror_LoadingGameState                                    ;828124;
    CMP.W #$001F                                                         ;828128;
    BNE .setupSamusOnZebes                                               ;82812B;
    ...                                     ; Samus command 8: set up Samus for Ceres start
  .setupSamusOnZebes:
    LDA.W #$0009                                                         ;82813D;
    JSL.L Run_Samus_Command                                              ;828140;
```

Samus command 8 (`SamusCommand_8_SetupSamusForCeresStart`, `bank_90.asm`)
locks Samus and spawns the Ceres elevator pad. The second code changes
`BNE` (`$D0`) to `BRA` (`$80`), so Samus command 9 (set up Samus for
Zebes start) always runs, as when a save file is loaded.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 06 00 8D 9F 07 9C 8B 07` at `$82:8041` (PC
`$010041`) and `AF 14 D9 7E C9 1F 00 D0 10` at `$82:8124` (PC `$010124`).
The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The intro and the flight to Ceres still play. The game starts after
  them.
- Samus starts at the gunship as if loading a save there. There is no
  gunship landing scene, and the Ceres explosion and the flight to Zebes
  don't play.
- Until Samus saves, the save file still says "set up new game"
  (`1Fh`). Continuing after a game over starts at the gunship again.

## Start a new game with every item

**Codes:** `4E83-C400` + `E783-C460` + `628E-C460` + `628E-C700` +
`2D8D-3460` (use all five)

**Raw form:** `81B2E9:2F` (orig `$00`), `81B2EA:F3` (orig `$00`),
`81B2FA:8D` (orig `$9C`), `81B2FD:8D` (orig `$9C`), `81B30A:D0` (orig
`$D8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:56:45

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** `NewSaveFile` (`$81:B2CB`, `bank_81.asm`) sets up Samus's
equipment for a new file:

```
    LDA.W #$0000                                                         ;81B2E8;
    STA.W MaxPowerBombs                                                  ;81B2EB;
    STA.W PowerBombs                                                     ;81B2EE;
    STZ.W SelectedHUDItem                                                ;81B2F1;
    STZ.W CollectedBeams                                                 ;81B2F4;
    STZ.W EquippedBeams                                                  ;81B2F7;
    STZ.W CollectedItems                                                 ;81B2FA;
    STZ.W EquippedItems                                                  ;81B2FD;
    STZ.W ReserveTankMode                                                ;81B300;
    STZ.W MaxReserveEnergy                                               ;81B303;
    STZ.W ReserveEnergy                                                  ;81B306;
    STZ.W ReserveMissiles                                                ;81B309;
```

- The first two codes change `LDA #$0000` to `LDA #$F32F`. `$F32F` has a
  bit set for every item: Varia Suit (1), Spring Ball (2), Morphing Ball
  (4), Screw Attack (8), Gravity Suit (`$20`), Hi-Jump Boots (`$100`),
  Space Jump (`$200`), Bombs (`$1000`), Speed Booster (`$2000`), Grapple
  Beam (`$4000`) and X-Ray Scope (`$8000`).
- The third and fourth codes change `STZ` (`$9C`) to `STA` (`$8D`), so
  `CollectedItems` and `EquippedItems` get `$F32F` instead of 0.
- The two `STA`s before them also put `$F32F` in `MaxPowerBombs` and
  `PowerBombs`. The fifth code changes the operand of the last `STZ` from
  `$09D8` (`ReserveMissiles`) to `$09D0` (`MaxPowerBombs`), so the maximum
  is cleared again. The HUD draws the Power Bomb icon only if
  `MaxPowerBombs` isn't 0, so Samus starts with no Power Bombs shown.

`ReserveMissiles` is used only by `$91:DF80` (adding missiles past the
maximum). It is cleared again by `$8B:80B4`.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 00 00 8D D0 09 8D CE 09 9C D2 09 9C A8 09 9C A6
09 9C A4 09 9C A2 09 9C C0 09 9C D4 09 9C D6 09 9C D8 09` at `$81:B2E8`
(PC `$00B2E8`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Beams aren't included. Samus starts with only the Power Beam. Missiles,
  Super Missiles, Power Bombs and Energy Tanks aren't included either.
- `PowerBombs` (the current count) is left at `$F32F`. It isn't shown
  while the maximum is 0. After Samus picks up a Power Bomb tank, the
  count may show a junk value.
- `ReserveMissiles` isn't cleared for the new file, so a value left from
  an earlier game in the same session may stay.
- Ceres Station is played with every item too.
- Loading an existing save file isn't affected.

## Start a new game with the Morphing Ball

**Codes:** `D083-C400` + `628E-C460` + `628E-C700` + `2D8D-3460` (use all
four)

**Raw form:** `81B2E9:04` (orig `$00`), `81B2FA:8D` (orig `$9C`), `81B2FD:8D`
(orig `$9C`), `81B30A:D0` (orig `$D8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:57:12

**Mechanism:** The same as "Start a new game with every item", with a
different item value. In `NewSaveFile` (`$81:B2CB`, `bank_81.asm`):

- The first code changes the low byte of `LDA.W #$0000` at `$81:B2E8`,
  so it becomes `LDA.W #$0004`. `$0004` is the item bit for the Morphing Ball.
- The second and third codes change `STZ CollectedItems` and `STZ
  EquippedItems` (`$9C`) to `STA` (`$8D`), so both get that value.
- The two `STA`s after the `LDA` also put the value in `MaxPowerBombs`
  and `PowerBombs`. The fourth code changes the operand of `STZ
  ReserveMissiles` (`$09D8`) at `$81:B309` to `$09D0` (`MaxPowerBombs`),
  so the maximum is cleared again and the HUD shows no Power Bombs.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 00 00 8D D0 09 8D CE 09` at `$81:B2E8` (PC
`$00B2E8`), `9C A4 09 9C A2 09` at `$81:B2FA`, and `9C D8 09` at
`$81:B309`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- `PowerBombs` (the current count) is left at the item value. It isn't
  shown while the maximum is 0. After Samus picks up a Power Bomb tank,
  the count may be higher than the maximum.
- `ReserveMissiles` isn't cleared for the new file, so a value left from
  an earlier game in the same session may stay.
- The last three codes are shared with "Start a new game with every
  item". Change only the first code to pick a different item.
- Loading an existing save file isn't affected.

## Start a new game with Bombs

**Codes:** `FD83-C460` + `628E-C460` + `628E-C700` + `2D8D-3460` (use all
four)

**Raw form:** `81B2EA:10` (orig `$00`), `81B2FA:8D` (orig `$9C`), `81B2FD:8D`
(orig `$9C`), `81B30A:D0` (orig `$D8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:57:12

**Mechanism:** The same as "Start a new game with every item", with a
different item value. In `NewSaveFile` (`$81:B2CB`, `bank_81.asm`):

- The first code changes the high byte of `LDA.W #$0000` at `$81:B2E8`,
  so it becomes `LDA.W #$1000`. `$1000` is the item bit for Bombs.
- The second and third codes change `STZ CollectedItems` and `STZ
  EquippedItems` (`$9C`) to `STA` (`$8D`), so both get that value.
- The two `STA`s after the `LDA` also put the value in `MaxPowerBombs`
  and `PowerBombs`. The fourth code changes the operand of `STZ
  ReserveMissiles` (`$09D8`) at `$81:B309` to `$09D0` (`MaxPowerBombs`),
  so the maximum is cleared again and the HUD shows no Power Bombs.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 00 00 8D D0 09 8D CE 09` at `$81:B2E8` (PC
`$00B2E8`), `9C A4 09 9C A2 09` at `$81:B2FA`, and `9C D8 09` at
`$81:B309`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Bombs can only be laid in morph ball form, so they can't be used until
  Samus finds the Morphing Ball.
- `PowerBombs` (the current count) is left at the item value. It isn't
  shown while the maximum is 0. After Samus picks up a Power Bomb tank,
  the count may be higher than the maximum.
- `ReserveMissiles` isn't cleared for the new file, so a value left from
  an earlier game in the same session may stay.
- The last three codes are shared with "Start a new game with every
  item". Change only the first code to pick a different item.
- Loading an existing save file isn't affected.

## Start a new game with the Speed Booster

**Codes:** `4D83-C460` + `628E-C460` + `628E-C700` + `2D8D-3460` (use all
four)

**Raw form:** `81B2EA:20` (orig `$00`), `81B2FA:8D` (orig `$9C`), `81B2FD:8D`
(orig `$9C`), `81B30A:D0` (orig `$D8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:57:12

**Mechanism:** The same as "Start a new game with every item", with a
different item value. In `NewSaveFile` (`$81:B2CB`, `bank_81.asm`):

- The first code changes the high byte of `LDA.W #$0000` at `$81:B2E8`,
  so it becomes `LDA.W #$2000`. `$2000` is the item bit for the Speed Booster.
- The second and third codes change `STZ CollectedItems` and `STZ
  EquippedItems` (`$9C`) to `STA` (`$8D`), so both get that value.
- The two `STA`s after the `LDA` also put the value in `MaxPowerBombs`
  and `PowerBombs`. The fourth code changes the operand of `STZ
  ReserveMissiles` (`$09D8`) at `$81:B309` to `$09D0` (`MaxPowerBombs`),
  so the maximum is cleared again and the HUD shows no Power Bombs.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 00 00 8D D0 09 8D CE 09` at `$81:B2E8` (PC
`$00B2E8`), `9C A4 09 9C A2 09` at `$81:B2FA`, and `9C D8 09` at
`$81:B309`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- `PowerBombs` (the current count) is left at the item value. It isn't
  shown while the maximum is 0. After Samus picks up a Power Bomb tank,
  the count may be higher than the maximum.
- `ReserveMissiles` isn't cleared for the new file, so a value left from
  an earlier game in the same session may stay.
- The last three codes are shared with "Start a new game with every
  item". Change only the first code to pick a different item.
- Loading an existing save file isn't affected.

## Start a new game with the Screw Attack

**Codes:** `D683-C400` + `628E-C460` + `628E-C700` + `2D8D-3460` (use all
four)

**Raw form:** `81B2E9:08` (orig `$00`), `81B2FA:8D` (orig `$9C`), `81B2FD:8D`
(orig `$9C`), `81B30A:D0` (orig `$D8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:57:12

**Mechanism:** The same as "Start a new game with every item", with a
different item value. In `NewSaveFile` (`$81:B2CB`, `bank_81.asm`):

- The first code changes the low byte of `LDA.W #$0000` at `$81:B2E8`,
  so it becomes `LDA.W #$0008`. `$0008` is the item bit for the Screw Attack.
- The second and third codes change `STZ CollectedItems` and `STZ
  EquippedItems` (`$9C`) to `STA` (`$8D`), so both get that value.
- The two `STA`s after the `LDA` also put the value in `MaxPowerBombs`
  and `PowerBombs`. The fourth code changes the operand of `STZ
  ReserveMissiles` (`$09D8`) at `$81:B309` to `$09D0` (`MaxPowerBombs`),
  so the maximum is cleared again and the HUD shows no Power Bombs.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 00 00 8D D0 09 8D CE 09` at `$81:B2E8` (PC
`$00B2E8`), `9C A4 09 9C A2 09` at `$81:B2FA`, and `9C D8 09` at
`$81:B309`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The Screw Attack works only in a spin jump.
- `PowerBombs` (the current count) is left at the item value. It isn't
  shown while the maximum is 0. After Samus picks up a Power Bomb tank,
  the count may be higher than the maximum.
- `ReserveMissiles` isn't cleared for the new file, so a value left from
  an earlier game in the same session may stay.
- The last three codes are shared with "Start a new game with every
  item". Change only the first code to pick a different item.
- Loading an existing save file isn't affected.

## Firing a missile doesn't use one up

**Code:** `C288-C5A7`

**Optional, for Super Missiles too:** `C28A-C9D7`

**Raw form:** `90BEBF:AD` (orig `$CE`); optional `90BEC4:AD` (orig `$CE`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:57:41

**Mechanism:** `HUDSelectionHandler_Missiles_SuperMissiles` (`$90:BE62`,
`bank_90.asm`) fires a missile and then takes one from the count:

```
    LDA.W SelectedHUDItem                                                ;90BEB7;
    CMP.W #$0002                                                         ;90BEBA;
    BEQ .decSuperCount                                                   ;90BEBD;
    DEC.W Missiles                                                       ;90BEBF;
    BRA +                                                                ;90BEC2;

  .decSuperCount:
    DEC.W SuperMissiles                                                  ;90BEC4;

+   LDX.B DP_Temp14                                                      ;90BEC7;
    LDA.W #$0004                                                         ;90BEC9;
```

The code changes `DEC Missiles` (`$CE`) to `LDA Missiles` (`$AD`), which
only reads the count. The optional code does the same for `DEC
SuperMissiles`. A is loaded again two instructions later, so the read
value isn't used.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `CE C6 09 80 03 CE CA 09` at `$90:BEBF` (PC
`$083EBF`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus still needs at least one missile to fire.
- Power Bombs are still used up. The Crystal Flash still uses up ammo.

## Missile pickups fill Samus up to her maximum

**Code:** `EEE3-4496`

**Optional, for Super Missile and Power Bomb pickups too:** `EEEE-44F6` + `EEE2-44B6`

**Raw form:** `86F0E9:FF` (orig `$02`); optional `86F0F8:FF` (orig
`$01`), `86F0DA:FF` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:58:16

Tags: `curated`

**Mechanism:** When Samus touches a pickup dropped by an enemy, the
pickup's function adds a fixed amount (`bank_86.asm`):

```
;;; $F0E8: Apply pickup to Samus - missiles ;;;
Function_EnemyProjectile_Pickup_ApplyMissiles:
    LDA.W #$0002                                                         ;86F0E8;
    JSL.L Restore_A_Missiles_ToSamus                                     ;86F0EB;
```

The code changes the 2 to `FFh` (255). `Restore_A_Missiles_ToSamus`
(`$91:DF80`) adds A to `Missiles` and caps the result at `MaxMissiles`.
The most missiles Samus can carry is 230, so one pickup always fills her
up.

The optional codes do the same for `$86:F0F7` (Super Missiles, `LDA
#$0001`) and `$86:F0D9` (Power Bombs, `LDA #$0001`).
`Restore_A_SuperMissiles_ToSamus` and `Restore_A_PowerBombs_ToSamus` also
cap at the maximum.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 02 00 22 80 DF 91` at `$86:F0E8` (PC `$0370E8`),
`A9 01 00 22 D3 DF 91` at `$86:F0F7` and `A9 01 00 22 F0 DF 91` at
`$86:F0D9`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only pickups dropped by enemies are changed. Missile Expansions and
  missile refill stations work as before.
- The two optional codes are independent. Either can be used alone.
- When `MaxMissiles` is 99 or more, the overflow is added to
  `ReserveMissiles` (capped at 99). Nothing else uses that value.

## Energy pickups refill all of Samus's energy

**Codes:** `DEE8-4796` + `DEEA-47F6` (use both)

**Raw form:** `86F0BD:0F` (orig `$00`), `86F0CC:0F` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:59:04

Tags: `curated`

**Mechanism:** When Samus touches an energy pickup dropped by an enemy,
the pickup's function adds a fixed amount (`bank_86.asm`):

```
;;; $F0BB: Apply pickup to Samus - small health ;;;
Function_EnemyProjectile_Pickup_ApplySmallEnergy:
    LDA.W #$0005                                                         ;86F0BB;
    JSL.L Restore_A_Energy_ToSamus                                       ;86F0BE;
...
;;; $F0CA: Apply pickup to Samus - big health ;;;
Function_EnemyProjectile_Pickup_ApplyBigEnergy:
    LDA.W #$0014                                                         ;86F0CA;
    JSL.L Restore_A_Energy_ToSamus                                       ;86F0CD;
```

The codes change the high byte of each operand from `$00` to `$0F`, so
the small pickup gives `$0F05` (3845) energy and the big pickup gives
`$0F14` (3860). The first code is for the small pickup, the second for the
big one.

`Restore_A_Energy_ToSamus` (`$91:DF12`) adds A to `Energy`. If the result
is `MaxEnergy` or more, it puts the extra into `ReserveEnergy` (capped at
`MaxReserveEnergy`) and sets `Energy` to `MaxEnergy`. The most energy
Samus can have is 1499, so the sum stays below `$8000` and the signed
compare (`BMI`) still works.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 05 00 22 12 DF 91` at `$86:F0BB` (PC `$0370BB`)
and `A9 14 00 22 12 DF 91` at `$86:F0CA` (PC `$0370CA`). The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The extra energy also fills the Reserve Tanks, since the game already
  puts overflow there. As in the original game, this also turns reserve
  mode on (auto) if it was off.
- Only pickups dropped by enemies are changed. Energy Tanks and refill
  stations work as before.
- Each code works alone, for one pickup size.

## Every killed enemy drops an item

**Code:** `D0E1-CD96`

**Raw form:** `86F261:04` (orig `$06`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 22:59:58

Tags: `curated`

**Mechanism:** `Random_Drop_Routine` (`$86:F106`, `bank_86.asm`) picks a
drop from the enemy's drop chances: small energy, big energy, missiles,
nothing, Super Missiles or Power Bombs. It returns the drop type from this
table:

```
  .drops:                                                                ;86F25E;
    db $01 ; Small health
    db $02 ; Big health
    db $04 ; Missiles
    db $06 ; Nothing
    db $05 ; Super missiles
    db $03 ; Power bombs
```

The code changes the "nothing" entry from 6 to 4 (missiles). Both
callers (`Instruction_EnemyProjectile_EnemyDeathExplosion_BecomePickup`
at `$86:EEAF`, for normal enemy deaths, and `InitAI_EnemyProjectile_Pickup`
at `$86:EF29`, used by `Spawn_Enemy_Drops`) treat type 6 or more as no
drop. Type 4 makes a missile pickup instead.

The "nothing" entry is also used by `.returnNothing`, when the enemy has
no drop chances at all (`EnemyHeaders_dropChances` is 0), so those
enemies drop missiles too.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `01 02 04 06 05 03` at `$86:F25E` (PC `$03725E`). The
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The extra drops are missiles. For small energy use `DFE1-CD96`
  (`86F261:01`). For big energy use `D4E1-CD96` (`86F261:02`).
- A missile pickup gives 2 missiles, capped at Samus's maximum
  (`Restore_A_Missiles_ToSamus`). Before Samus has a Missile Tank, the
  pickups give nothing.
- When the random roll picks "nothing", the drop is missiles even if
  Samus is full of them.
- Enemies still drop their normal items when the random roll picks one.
- An original bug is kept: after `Random_Drop_Routine`, the `BEQ` tests the
  enemy projectile index, so a death explosion in enemy projectile slot 0
  never becomes a pickup.
- Enemies whose death doesn't go through these routines still drop
  nothing.

## No pop-up text when Samus picks up a special item

**Codes:** `CE6F-1DB4` + `CE63-4424` + `CE67-14B4` + `CE61-1D94` (use all
four)

**Optional, for Energy, Reserve, Missile, Super Missile and Power Bomb tanks too:** `CE65-1724` + `CE6C-1DB4` + `CE6A-1424` + `CE6E-1FF4` + `CE6F-C794`

**Raw form:** `848912:AF`, `8488EB:AF`, `84893A:AF`, `848961:AF` (orig
`$22` each); optional `84897F:AF`, `8489A2:AF`, `8489CB:AF`, `8489F4:AF`,
`848A1D:AF` (orig `$22` each)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:00:54

**Mechanism:** Each item's PLM instruction list runs an instruction in
`bank_84.asm` that gives Samus the item, starts the item fanfare, and
then shows the message box:

```
;;; $88F3: Instruction - pick up equipment [[Y]] and display message box [[Y] + 2] ;;;
Instruction_PLM_PickUpEquipment_DisplayMessageBox:
    ...
    LDA.W #$0168                                                         ;848905;
    JSL.L Play_Room_Music_Track_After_A_Frames                           ;848908;
    LDA.W $0002,Y                                                        ;84890C;
    AND.W #$00FF                                                         ;84890F;
    JSL.L MessageBox_Routine                                             ;848912;
    INY                                                                  ;848916;
    INY                                                                  ;848917;
    INY                                                                  ;848918;
    RTS                                                                  ;848919;
```

Each code changes `JSL MessageBox_Routine` (`22 80 80 85`) to `LDA
$85:8080` (`AF 80 80 85`). That is also a 4-byte instruction, so it only
loads A, which the instruction doesn't use again. The item is still
given.

The four main codes are for these instructions:

- `$84:8912`: `Instruction_PLM_PickUpEquipment_DisplayMessageBox` (suits,
  boots, Morphing Ball, Bombs, Spring Ball, Screw Attack, Space Jump,
  Speed Booster)
- `$84:88EB`: `Instruction_PLM_PickUpBeam_DisplayMessageBox` (Charge, Ice,
  Wave, Spazer, Plasma)
- `$84:893A`: `Inst_PLM_PickUpEquipment_AddGrappleHUD_DisplayMessageBox`
- `$84:8961`: `Inst_PLM_PickUpEquipment_AddXrayToHUD_DisplayMessageBox`

The optional codes do the same in `Instruction_PLM_CollectHealth_EnergyTank`
(`$84:897F`), `Instruction_PLM_CollectHealth_ReserveTank` (`$84:89A2`),
`Instruction_PLM_CollectAmmo_MissileTank` (`$84:89CB`),
`Instruction_PLM_CollectAmmo_SuperMissileTank` (`$84:89F4`) and
`Instruction_PLM_CollectAmmo_PowerBombTank` (`$84:8A1D`).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `22 80 80 85` at each of the nine addresses (for
example `$84:8912`, PC `$020912`). The codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The SNES Game Genie holds 5 codes, so all nine can't be used at once.
  Each code works alone, for its own item group.
- The item fanfare still plays. The game no longer stops for it, and the
  room music comes back after 360 frames (6 seconds) as before.
- Map, recharge and save station messages still show.

## Pickups never vanish

**Code:** `DD33-3498`

**Raw form:** `86EFE9:00` (orig `$41`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:01:22

Tags: `curated`

**Mechanism:** A pickup dropped by an enemy is an enemy projectile. When
it appears, `EnemyProjectile_Var1` is set to `$0190` (400 frames).
`PreInstruction_EnemyProjectile_Pickup` (`$86:EFE0`, `bank_86.asm`) counts
it down every frame and removes the pickup when it reaches 0:

```
    DEC.W EnemyProjectile_Var1,X                                         ;86EFE2;
    LDA.W EnemyProjectile_Var1,X                                         ;86EFE5;
    BEQ .timerExpired                                                    ;86EFE8;
    LDA.W #$000D                                                         ;86EFEA;
```

The code changes the branch offset from `$41` to `$00`. `BEQ` then goes
to the next instruction whether it is taken or not, so `.timerExpired` is
never reached. The timer keeps counting down past 0 (to `$FFFF` and on).
The only other use of the timer is the grapple check (`CMP #$0180` /
`BPL`, so a pickup can't be grappled in its first 16 frames). Negative
values pass it.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `DE 23 1B BD 23 1B F0 41 A9 0D` at `$86:EFE2` (PC
`$036FE2`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Pickups still go away when Samus leaves the room.
- Each pickup holds an enemy projectile slot until it is collected. With
  many pickups left in a room, enemies may not be able to fire.
- An enemy that respawns comes back only after its pickup goes away
  (`InstList_EnemyProjectile_Pickup_HandleRespawningEnemy`), so it now
  comes back only after Samus collects the pickup.
- About 9 minutes (32768 frames) after the timer passes 0, it wraps from
  `$8000` to `$7FFF`. The Grapple Beam then can't collect the pickup for
  about 9 more minutes, until the timer drops below `$0180`. Touching it
  still works.

## Bombs go off almost at once

**Code:** `FD8B-31A7`

**Raw form:** `90BF9B:10` (orig `$3C`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:02:03

**Mechanism:** When Samus lays a bomb or a Power Bomb, its timer is set
from one word (`bank_90.asm`):

```
;;; $BF9B: Bomb timer reset value ;;;
BombTimerResetValue:
    dw regional($003C, $0034)                                            ;90BF9B;
```

On NTSC this is `3Ch` (60 frames, one second). `HandleBomb` (`$90:C128`)
and `HandlePowerBomb` (`$90:C157`) count the timer down once a frame. At
`0Fh` they switch the bomb to its fast-flashing animation, and at 0 the
bomb explodes. The code changes the value to `10h` (16 frames), so the
bomb flashes fast after 1 frame and explodes about a quarter of a second
after it is laid.

The value is read at `$90:BFFB` (bombs) and `$90:C06D` (Power Bombs).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `3C 00` at `$90:BF9B` (PC `$083F9B`). The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.
On PAL the original value is `34h`; the code still sets `10h`.

**Caveats:**

- Not tested.
- Power Bombs go off sooner too.
- Bomb jumps happen sooner, so bomb jump timing (for example the
  infinite bomb jump) changes.
- The bomb spread from a charged shot in morph ball form uses its own
  timers (`BombSpreadData_timers`) and isn't changed.

## Enemies can't move

**Code:** `DDB1-460D`

**Raw form:** `A09061:00` (orig `$0E`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:02:47

**Mechanism:** `Main_Enemy_Routine` (`$A0:8FD4`, `bank_A0.asm`) runs each
active enemy's AI once a frame. When time is frozen (as when the reserve
tanks refill Samus), it runs the enemy's "time is frozen" AI instead,
from enemy header offset `20h`, or no AI at all if that is 0:

```
  .interactEnd:
    STZ.W DisableDrawingOfEnemies                                        ;A09057;
    LDA.W TimeIsFrozenFlag                                               ;A0905A;
    ORA.W DebugTimeIsFrozenForEnemies                                    ;A0905D;
    BEQ .timeNotFrozen                                                   ;A09060;
    LDA.W Enemy.ID,X                                                     ;A09062;
    TAY                                                                  ;A09065;
    LDA.W $0020,Y                                                        ;A09066;
    BEQ .processAIEnd                                                    ;A09069;
    STA.W EnemyAIPointer                                                 ;A0906B;
    BRA .determineAIEnd                                                  ;A0906E;

  .timeNotFrozen:
```

The code changes the branch offset of `BEQ .timeNotFrozen` from `$0E` to
`$00`, so it goes to the next instruction either way. Every enemy then
takes the time-frozen path. Most enemies have no time-frozen AI, so their
AI doesn't run and they stay where they are.

Collision handling (`EnemyCollisionHandling`, at `$A0:903E`) checks
`TimeIsFrozenFlag` itself and isn't changed. Enemies can still be shot,
and touching them still hurts Samus.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 78 0A 0D 5E 18 F0 0E BD 78 0F A8 B9 20 00 F0` at
`$A0:905A` (PC `$10105A`). The code decodes back to its raw form with
`snes_gg_encode.py`. The `ORA DebugTimeIsFrozenForEnemies` is in the ROM
(the vanilla build has `!DEBUG` on).

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Enemy instructions aren't processed either, so enemies don't animate.
  Enemies with a time-frozen AI still run it.
- Bosses don't move or attack. Boss fights that need the boss to act
  (Ceres Ridley flying off, boss death sequences, Mother Brain) may
  never finish. Turn the code off for boss fights.
- Enemy projectiles that are already flying still move.

## Enemies with shared movement code move twice as fast

**Codes:** `D1AC-C80F` + `41AC-C8AF` + `6DA8-C60F` (use all three)

**Optional, for vertical movement too:** `D1A6-3CDF` + `41A6-3C6F` + `6DA6-38DF`

**Raw form:** `A0C6AD:06` (orig `$A5`), `A0C6AF:26` (orig `$05`),
`A0C6B1:80` (orig `$D0`); optional `A0C788:06` (orig `$A5`),
`A0C78A:26` (orig `$05`), `A0C78C:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:04:28

Tags: `challenge`

**Mechanism:** Most enemies move with two shared routines in
`bank_A0.asm`. They move the enemy by the signed distance `$14.$12`
(pixels.subpixels) and check for walls:
`MoveEnemyRightBy_14_12_Common` (`$A0:C6AD`, reached from
`MoveEnemyRightBy_14_12_IgnoreSlopes` and its two siblings, 87 calls) and
`MoveEnemyDownBy_14_12_BranchEntry` (`$A0:C788`, reached from
`MoveEnemyDownBy_14_12`, 94 calls). Both start the same way:

```
    LDA.B DP_Temp12                                                      ;A0C6AD;
    ORA.B DP_Temp14                                                      ;A0C6AF;
    BNE .notZero                                                         ;A0C6B1;
    CLC                                                                  ;A0C6B3;
    RTL                                                                  ;A0C6B4;

  .notZero:
```

The codes change these 6 bytes from `A5 12 05 14 D0 02` to `06 12 26 14
80 02`:

```
    ASL.B DP_Temp12
    ROL.B DP_Temp14
    BRA .notZero
```

`ASL`/`ROL` double the 32-bit distance (the sign is kept). The rest of the
routine, both the wall check and the position update, reads `$12` and `$14`
again, so the enemy checks for walls at the new distance and moves twice
as far. The `BRA` is needed because `ROL` leaves Z set for distances under
1 pixel; with `BNE` slow enemies would stop. A distance of 0 now goes
through the wall check too, which adds 0 and returns carry clear, as
before.

The main codes are for horizontal movement, the optional codes for
vertical movement (falling, jumping, flying up and down).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A5 12 05 14 D0 02` at `$A0:C6AD` (PC `$1046AD`) and
at `$A0:C788` (PC `$104788`). The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- All six codes together are more than the SNES Game Genie's 5. The
  three horizontal codes work alone, and so do the three vertical ones.
- Enemies with their own movement code (many bosses) aren't changed.
  Zoomers and other crawlers use these routines too, so their crawling
  along walls changes and may act strangely.
- `$12` and `$14` are doubled in place. An enemy that uses them again
  after the call (for example to move down by the same amount) moves even
  farther.
- An enemy moving more than 16 pixels in one frame can pass through a
  wall one block thick.
- Enemies that fall or jump with gravity land at twice the speed, but
  their speed builds up as before.

## Zoomers move much faster

**Code:** `E33B-CBA2`

**Optional, for the Wrecked Ship orange Zoomer too:** `E331-46DA`

**Raw form:** `A3E697:FE` (orig `$F0`); optional `A3E060:FE` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:06:02

**Mechanism:** Crawlers (Zoomers, Geemers (`zeela`), Sovas, Violas,
Scisers and Zeros) get their crawling speed once, when they are set up,
from `CrawlersSpeedTable` (`$A3:E5F0`, `bank_A3.asm`). The index is the
enemy's first speed parameter from the room's enemy list:

```
;;; $E5F0: Creepy crawly speed table ;;;
CrawlersSpeedTable:
; Speed. Unit is 1/100h px/frame. Indexed by [enemy parameter 1] * 2
    dw $0040, $0080, $00C0, $0100, $0140, $0180, $01C0, $0200 ; (NTSC values)
    dw $0240, $0280, ...
```

`InitAI_Crawlers_Common` (`$A3:E67A`) reads it:

```
    ASL                                                                  ;A3E694;
    TAY                                                                  ;A3E695;
    LDA.W CrawlersSpeedTable,Y                                           ;A3E696;
    STA.W Crawler.XVelocity,X                                            ;A3E699;
    STA.W Crawler.YVelocity,X                                            ;A3E69C;
```

The code changes the low byte of the table address from `$F0` to `$FE`,
so the `LDA` reads 7 entries later in the table. Zoomers use parameter 1
or 2, so their speed goes from `$80` (½ pixel a frame) to `$0240` (2¼
pixels), and from `$C0` to `$0280`. That is about 3½ to 4½ times as
fast.

The optional code makes the same change in `InitAI_HZoomer` (`$A3:E043`),
the Wrecked Ship orange Zoomer, which reads the table itself
(`LDA.W CrawlersSpeedTable,Y` at `$A3:E05F`).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `0A A8 B9 F0 E5 9D` at `$A3:E694` (PC `$11E694`),
`0F 0A A8 B9 F0 E5` at `$A3:E05C`, and `40 00 80 00 C0 00 00 01` at
`$A3:E5F0`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The other crawlers (Geemers, Sovas, Violas, Scisers, Zeros) speed up
  too. Stone Zoomers (`MZoomer`) share the Zoomer setup and speed up as
  well.
- Crawlers set up with parameter `FFh` (some Geemers) keep their speed.
- At these speeds the crawling code may miss corners, and crawlers may
  fall off walls.
- A parameter of 25 or more would read past the end of the table. The
  vanilla rooms use 0..6, apart from one Sova with `5000h`, which already
  reads outside the table.

## Zoomers move much slower

**Codes:** `4D3E-1662` + `7D3E-1BD2` (use both)

**Raw form:** `A3E5F2:20` (orig `$80`), `A3E5F4:30` (orig `$C0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:06:02

**Mechanism:** This is the Super Metroid version of the slow Brinstar
Zoomers in Metroid. Crawlers get their crawling speed from
`CrawlersSpeedTable` (`$A3:E5F0`, `bank_A3.asm`), indexed by the enemy's
first speed parameter from the room's enemy list. Unit is 1/100h pixel a
frame:

```
;;; $E5F0: Creepy crawly speed table ;;;
CrawlersSpeedTable:
    dw $0040, $0080, $00C0, $0100, ...   ; (NTSC values)
```

Every Zoomer in the game uses parameter 1 or 2. The first code changes
entry 1 from `$0080` (½ pixel a frame) to `$0020` (⅛ pixel). The second
changes entry 2 from `$00C0` to `$0030`. Both are a quarter of the
original speed.

The table is read by `InitAI_Crawlers_Common` (`$A3:E696`) and
`InitAI_HZoomer` (`$A3:E05F`), so the Wrecked Ship orange Zoomer slows
down too.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `40 00 80 00 C0 00 00 01` at `$A3:E5F0` (PC
`$11E5F0`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Other crawlers with parameter 1 or 2 slow down too: most Scisers,
  Sovas, Violas and Geemers (`zeela`).
- Each code works alone, for one parameter.

## Enemies never hurt Samus

**Code:** `10C9-486F`

**Raw form:** `A0A45E:64` (orig `$85`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:06:56

**Mechanism:** Damage from enemies and enemy projectiles goes through
`Suit_Damage_Division` (`$A0:A45E`, `bank_A0.asm`), which halves it for
the Varia Suit and quarters it for the Gravity Suit:

```
Suit_Damage_Division:
    STA.B DP_Temp12                                                      ;A0A45E;
    LDA.W EquippedItems                                                  ;A0A460;
    BIT.W #$0020                                                         ;A0A463;
    BNE .quarterDamage                                                   ;A0A466;
    LSR                                                                  ;A0A468;
    BCC .fullDamage                                                      ;A0A469;
    LSR.B DP_Temp12                                                      ;A0A46B;

  .fullDamage:
    LDA.B DP_Temp12                                                      ;A0A46D;
    RTL                                                                  ;A0A46F;

  .quarterDamage:
    LSR.B DP_Temp12                                                      ;A0A470;
    LSR.B DP_Temp12                                                      ;A0A472;
    LDA.B DP_Temp12                                                      ;A0A474;
    RTL                                                                  ;A0A476;
```

The code changes `STA $12` (`$85`) to `STZ $12` (`$64`). Every path
returns `$12`, so the damage is always 0.

Callers: enemy touch (`NormalEnemyTouchAI_NoDeathCheck`, `$A0:A56D`),
`Samus / projectile interaction handling` (`$A0:985C`), enemy projectile
collision (`$A0:9951`), `Hurt Samus` (`$86:C35C`), and boss code for
Draygon (`$A5:9BAA`), Ridley (`$A6:D45E`, `$A6:DFF5`) and Mother Brain
(`$A9:BD48`, `$A9:BD64`).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `85 12 AD A2 09 89 20 00 D0 08` at `$A0:A45E` (PC
`$10245E`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus is still knocked back when hit.
- Lava, acid, spikes and other room hazards still hurt. So does damage
  from code that doesn't use this routine.
- Mother Brain's second phase attacks until Samus is low on energy
  (`Function_MBBody_Phase2_FinishSamusOff_GetSamusToLowEnergy`). With no
  damage this may never happen and the fight may never end. Turn the code
  off for that fight.

## Enemies barely hurt Samus

**Code:** `6DC1-4B6F`

**Raw form:** `A0A466:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:06:56

**Mechanism:** Damage from enemies and enemy projectiles goes through
`Suit_Damage_Division` (`$A0:A45E`, `bank_A0.asm`):

```
    STA.B DP_Temp12                                                      ;A0A45E;
    LDA.W EquippedItems                                                  ;A0A460;
    BIT.W #$0020                                                         ;A0A463;
    BNE .quarterDamage                                                   ;A0A466;
```

Bit `20h` is the Gravity Suit. The code changes `BNE` (`$D0`) to `BRA`
(`$80`), so damage is always cut to a quarter, as if Samus had the
Gravity Suit. The callers are listed in "Enemies never hurt Samus".

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `85 12 AD A2 09 89 20 00 D0 08` at `$A0:A45E` (PC
`$10245E`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only the damage is changed. The Gravity Suit's other effects (water and
  lava movement, lava protection) aren't given.
- With the Gravity Suit, damage is a quarter as before.
- Lava, acid, spikes and other room hazards aren't changed.

## Samus passes through enemies without being hurt or knocked back

**Codes:** `C2B5-386F` + `C2B5-366F` (use both)

**Raw form:** `A0977E:AD` (orig `$20`), `A09772:AD` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:07:40

Tags: `demo-ready` `curated`

**Mechanism:** `Main_Enemy_Routine` calls `EnemyCollisionHandling`
(`$A0:9758`, `bank_A0.asm`) for each enemy. It checks the enemy against
Samus's shots, her bombs, and Samus herself:

```
EnemyCollisionHandling:
    ...
    BEQ .notExtendedSpritemap                                            ;A0976A;
    JSR.W Enemy_vs_Projectile_CollisionHandling_ExtendedSpritemap        ;A0976C;
    JSR.W Enemy_vs_Bomb_CollisionHandling_ExtendedSpritemap              ;A0976F;
    JSR.W Enemy_vs_Samus_CollisionHandling_ExtendedSpritemap             ;A09772;
    PLB                                                                  ;A09775;
    PLP                                                                  ;A09776;
    RTS                                                                  ;A09777;

  .notExtendedSpritemap:
    JSR.W Enemy_vs_Projectile_CollisionHandling                          ;A09778;
    JSR.W Enemy_vs_Bomb_CollisionHandling                                ;A0977B;
    JSR.W Enemy_vs_Samus_CollisionHandling                               ;A0977E;
```

The Samus checks call the enemy's touch AI (for most enemies
`NormalEnemyTouchAI`, which takes energy and sets the invincibility and
knockback timers). Each code changes a `JSR` (`$20`) to `LDA` absolute
(`$AD`), which is also 3 bytes. The `LDA` only loads A, and the routine
ends right after it, so the Samus check is skipped. The first code is for
normal enemies, the second for enemies with extended spritemaps (mostly
bosses).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 7F 9B 20 23 9D 20 5A 9A` at `$A0:976C` (PC
`$10176C`) and `20 43 A1 20 36 A2 20 7A A0` at `$A0:9778`. These are the
only calls to the two routines. The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Enemy projectiles still hurt Samus. So do attacks that bosses check in
  their own AI (for example Ridley's tail and Draygon's tail whip).
- The Screw Attack, Speed Booster and Shinespark no longer hurt enemies
  on contact, since that is also done by the touch AI.
- Enemies that do something when Samus touches them (for example
  Metroids latching on) no longer do it.
- Samus still stands on and bumps into solid enemies, which use other
  code.

## Lava and acid don't hurt Samus

**Codes:** `DDB6-C5D7` + `DDBB-C0D7` + `DDBB-C007` (use all three)

**Raw form:** `909E8C:00` (orig `$80`), `909E90:00` (orig `$80`),
`909E91:00` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:08:24

**Mechanism:** While Samus is in lava or acid,
`AnimateSamus_FX_Lava` (`$90:81C0`) and `AnimateSamus_FX_Acid`
(`$90:8219`) add a fixed amount to `PeriodicDamage`.`PeriodicSubDamage`
every frame. `HandlePeriodicDamageToSamus` (`$90:E9CE`) then takes it from
Samus's energy. The amounts are in `SamusPhysicsConstants` (`$90:9E8B`,
`bank_90.asm`):

```
  .LavaSubDamagePerFrame:
    dw $80*!SPF*$100                                                     ;909E8B;  ($8000 on NTSC)
  .LavaDamagePerFrame:
    dw $0000                                                             ;909E8D;
  .AcidSubDamagePerFrame:
    dw $180*!SPF*$100                                                    ;909E8F;  ($8000 on NTSC)
  .AcidDamagePerFrame:
    dw $0001                                                             ;909E91;
```

Lava does ½ energy a frame, acid 1½. The first code zeroes the high byte
of the lava subdamage. The second and third codes zero the acid
subdamage high byte and the acid damage. All four words are then 0, so
nothing is added.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `00 80 00 00 00 80 01 00` at `$90:9E8B` (PC
`$081E8B`). These words are read only at `$90:81FB`, `$90:8204`,
`$90:823D` and `$90:8246`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The first code works alone for lava. The other two together work for
  acid.
- Lava and acid still slow Samus down. The hurt sound (`2Dh`, played
  every 8 frames while Samus has 47h energy or more) still plays.
- Heat damage in hot rooms, spikes and other hazards still hurt.

## Samus stays immune much longer after a hit

**Codes:** `DFC5-1BAF` + `DFB4-1B04` (use both)

**Optional, for hits from reflected shots too:** `DFB1-4B64`

**Raw form:** `A0A577:01` (orig `$00`), `A09925:01` (orig `$00`); optional
`A09866:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:09:06

**Mechanism:** After a hit, Samus flashes and can't be hurt while
`SamusInvincibilityTimer` is not 0. `DecrementSamusHurtTimers_ClearActiveEnemyIndicesLists`
(`$A0:9169`) counts it down once a frame. The hit code sets it to `$0060`
(96 frames, about 1.6 seconds):

```
;;; $A4A1: Normal enemy touch AI - no death check ;;;
    ...
    JSL.L Suit_Damage_Division                                           ;A0A56D;
    JSL.L Deal_A_Damage_to_Samus                                         ;A0A571;
    LDA.W #$0060                                                         ;A0A575;
    STA.W SamusInvincibilityTimer                                        ;A0A578;
```

```
;;; $9923: Handle enemy projectile collision with Samus ;;;
HandleEnemyProjectileCollisionWithSamus:
    LDA.W #$0060                                                         ;A09923;
    STA.W SamusInvincibilityTimer                                        ;A09926;
```

The codes change the high byte of each `LDA #$0060` to `$01`, so the
timer starts at `$0160` (352 frames, about 5.9 seconds). The first code is
for touching an enemy, the second for enemy projectiles. The optional
code does the same at `$A0:9864` in `Samus_Projectiles_Interaction_Handling`,
for Samus's own shots reflected back at her.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 60 00 8D A8 18` at `$A0:A575` (PC `$102575`),
`$A0:9923` and `$A0:9864`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Each code works alone, for its kind of hit.
- Some bosses (Ridley, Mother Brain) and a few hazards set the timer
  themselves and aren't changed.
- Samus flashes for the whole time.

## Samus is knocked back much farther from hits

**Codes:** `F0C5-18DF` + `F0B4-1C64` (use both)

**Optional, for hits from reflected shots too:** `F0B1-4CA4`

**Raw form:** `A0A57C:14` (orig `$05`), `A0992A:14` (orig `$05`); optional
`A0986B:14` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:10:38

**Mechanism:** After a hit, Samus is in knockback while
`SamusKnockbackTimer` is not 0. `DecrementSamusHurtTimers_ClearActiveEnemyIndicesLists`
(`$A0:9169`) counts it down once a frame, and `SamusIsHit_Interruption`
(`$90:DDE9`) ends the knockback when it reaches 0. While it runs,
`KnockbackMoement_Up`/`_Down` (`$90:DF53`/`DF64`) push Samus sideways,
speeding up by 1.8 px a frame to at most 5 px a frame (row Ah of the X
speed tables). The hit code sets the timer to 5 frames:

```
    LDA.W #$0060                                                         ;A0A575;
    STA.W SamusInvincibilityTimer                                        ;A0A578;
    LDA.W #$0005                                                         ;A0A57B;
    STA.W SamusKnockbackTimer                                            ;A0A57E;
```

The codes change the `#$0005` to `#$0014` (20 frames), so Samus is pushed
for four times as long, about 90 pixels instead of about 20. The first code
is for touching an enemy (`$A0:A4A1` normal enemy touch AI), the second for
enemy projectiles (`HandleEnemyProjectileCollisionWithSamus`, `$A0:9923`).
The optional code is at `$A0:986A` in `Samus_Projectiles_Interaction_Handling`,
for Samus's own shots reflected back at her.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 05 00 8D AA 18` at `$A0:A57B` (PC `$10257B`),
`$A0:9929` (PC `$101929`) and `$A0:986A` (PC `$10186A`). The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Each code works alone, for its kind of hit.
- Knockback also ends early if Samus lands on the ground or hits a ceiling.
- Spikes and other hazards (bank `$94`, 10 frames) and some bosses set the
  timer themselves and aren't changed.

## Touching an enemy hurts it, as with the Screw Attack

**Codes:** `CBCC-460F` + `D7CC-466F` + `DDCC-46AF` + `6DCA-4CDF` (use all)

**Raw form:** `A0A4A1:A9` (orig `$AD`), `A0A4A2:03` (orig `$6E`),
`A0A4A3:00` (orig `$0A`), `A0A4C8:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:11:54

Tags: `curated`

**Mechanism:** When Samus touches an enemy, most enemies run
`NormalEnemyTouchAI_NoDeathCheck` (`$A0:A4A1`). It reads
`ContactDamageIndex`: 0 for normal Samus, 1 speed boosting, 2 shinesparking,
3 screw attack, 4 pseudo screw attack. If it is 0, the enemy hurts Samus.
If not, Samus hurts the enemy and isn't hurt herself:

```
    LDA.W ContactDamageIndex                                             ;A0A4A1;
    BNE .notNormal                                                       ;A0A4A4;
    JMP.W .normalSamus                                                   ;A0A4A6;

  .notNormal:
    CLC                                                                  ;A0A4A9;
    ADC.W #$000F                                                         ;A0A4AA;
    STA.B DP_Temp14                                                      ;A0A4AD;
    ...
    LDY.W #$07D0                                                         ;A0A4C2;
    CMP.W #$0003                                                         ;A0A4C5;
    BEQ .damageCalculated                                                ;A0A4C8;
```

`DP_Temp14` picks the column of the enemy's vulnerability table (12h is the
screw attack column) and `Y` is the damage (`$07D0` = 2000 for the screw
attack).

The first three codes change `LDA.W ContactDamageIndex` to `LDA.W #$0003`,
so the routine always takes the `.notNormal` path with column 12h (screw
attack). The fourth code changes the `BEQ .damageCalculated` at `$A0:A4C8`
to `BRA`. Without it, the second read of `ContactDamageIndex` (still 0)
would fall through to the default damage (200). With it, normal Samus does
the full screw attack damage of 2000.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 6E 0A D0` at `$A0:A4A1` (PC `$1024A1`) and
`A0 D0 07 C9 03 00 F0 16` at `$A0:A4C2` (PC `$1024C2`). The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Use all four codes. The first three alone stop enemies from hurting Samus
  on touch, but do only 200 damage.
- Enemies that resist the screw attack (for example the space pirates)
  still aren't hurt, but they don't hurt Samus either.
- Bosses and other enemies with their own touch code aren't changed.
- Speed boosting and shinesparking now use the screw attack column of the
  vulnerability table instead of their own. The pseudo screw attack does
  full screw attack damage and no longer ends the beam charge.

## Every enemy dies in one shot

**Code:** `DDC1-48D4`

**Raw form:** `A0A86C:00` (orig `$4A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:12:59

Tags: `demo-ready` `curated`

**Mechanism:** When one of Samus's shots hits an enemy,
`NormalEnemyShotAI_NoDeathCheck_NoEnemyShotGraphic` (`$A0:A6DE`) works out
the damage and takes it off the enemy's health. If health is left, it
branches to `.return` and stores the new health. If not, `.checkKill`
either freezes the enemy (ice beam) or sets its health to 0:

```
  .notPlasma:
    LDA.W Enemy.health,X                                                 ;A0A862;
    SEC                                                                  ;A0A865;
    SBC.W EnemySpritemapEntryXPositionDuringCollision                    ;A0A866;
    BEQ .checkKill                                                       ;A0A869;
    BCS .return                                                          ;A0A86B;

  .checkKill:
    ...
  .enemyHealthZero:
    LDA.W #$0000                                                         ;A0A8B4;

  .return:
    STA.W Enemy.health,X                                                 ;A0A8B7;
```

The code changes the `BCS .return` offset to `$00`, so the branch goes to
the next instruction whether or not it is taken. Every shot that does any
damage then goes to `.checkKill` and sets the enemy's health to 0.
`NormalEnemyShotAI` (`$A0:A63D`) then sees health 0 and kills the enemy.

Most bosses have their own shot code, but it calls the same damage
routine (through its `_External` entry at `$A0:A6A7` or
`NormalEnemyShotAI_NoDeathCheck` at `$A0:A6B4`) and then checks whether
the boss's health is 0, so bosses die in one hit too:

- Spore Spawn: `$A5:ED5A`
- Kraid: `$A7:AFAA` (Kraid's mouth)
- Phantoon: `$A7:DD9B`
- Botwoon: `$B3:A016`
- Draygon: `$A5:95F0`
- Ridley: `$A6:DF8A`
- Bomb Torizo and Golden Torizo: `$AA:C97C`, `$AA:D6A6`
- Mother Brain's brain: `$A9:B507`

For example, Phantoon:

```
    LDA.W Enemy.health,X                                                 ;A7DDA9;
    PHA                                                                  ;A7DDAC;
    JSL.L NormalEnemyShotAI_NoDeathCheck_NoEnemyShotGraphic_External     ;A7DDAD;
    PLA                                                                  ;A7DDB1;
    STA.B DP_Temp12                                                      ;A7DDB2;
    LDA.W Enemy.health,X                                                 ;A7DDB4;
    BNE .alive                                                           ;A7DDB7;
```

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `BD 8C 0F 38 ED 7A 18 F0 02 B0 4A` at `$A0:A862`
(PC `$102862`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Shots an enemy is immune to (no damage) still do nothing.
- The ice beam freezes enemies that can be frozen instead of killing them,
  as it does now on the killing shot. Shoot a frozen enemy again to kill
  it.
- A boss must be hit where it can take damage at that moment (for
  example Kraid only in the mouth, Spore Spawn only when it is open).
- Mother Brain has several phases. Each one may end after one hit, and the
  scripted parts of the fight still play out. Not checked.
- Crocomire isn't killed by damage, so it isn't changed.
- Enemies with their own shot code that doesn't call this routine aren't
  changed. Some enemies that use this routine but handle death themselves
  may react differently.

## Shots hit every enemy on the screen at once

**Codes:** `6DC8-1BAD` + `6DCA-18AD` + `6DCD-CB0D` + `6DBA-4867` (use all)

**Raw form:** `A0A1B7:80` (orig `$90`), `A0A1CF:80` (orig `$90`),
`A0A205:80` (orig `$D0`), `A09CCE:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:14:52

Tags: `demo-ready` `curated`

**Mechanism:** Each frame, `Enemy_vs_Projectile_CollisionHandling`
(`$A0:A143`) runs for each enemy being processed (the ones on or near the
screen). It checks each of Samus's shots against the enemy's hitbox. If
the X distance is too big, or the Y distance is too big, the shot misses:

```
+   SEC                                                                  ;A0A1B3;
    SBC.W SamusProjectile_XRadii,Y                                       ;A0A1B4;
    BCC .Y                                                               ;A0A1B7;
    CMP.W Enemy.XHitboxRadius,X                                          ;A0A1B9;
    BCS .nextProjectile                                                  ;A0A1BC;
    ...
+   SEC                                                                  ;A0A1CB;
    SBC.W SamusProjectile_YRadii,Y                                       ;A0A1CC;
    BCC .checkSuper                                                      ;A0A1CF;
    CMP.W Enemy.YHitboxRadius,X                                          ;A0A1D1;
    BCC .checkSuper                                                      ;A0A1D4;
    BRA .nextProjectile                                                  ;A0A1D6;
```

On a hit, the shot is marked for deletion unless it is a plasma shot:

```
    LDA.W SamusProjectile_Types,Y                                        ;A0A1FF;
    BIT.W #$0008                                                         ;A0A202;
    BNE .noDelete                                                        ;A0A205;

  .delete:
    LDA.W SamusProjectile_Directions,Y                                   ;A0A207;
    ORA.W #$0010                                                         ;A0A20A;
    STA.W SamusProjectile_Directions,Y                                   ;A0A20D;

  .noDelete:
    JSL.L .executeEnemyShot                                              ;A0A210;
```

The first two codes change both `BCC` to `BRA`, so the X and Y checks
always pass and every shot hits every processed enemy, wherever it is. The
last two codes change `BNE .noDelete` to `BRA` here and in the same check
in `Enemy_vs_Projectile_CollisionHandling_ExtendedSpritemap` (`$A0:9CCE`),
so every shot passes through enemies like a plasma shot. Without them, a
shot would hit only the first enemy in the list and then vanish.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `90 05` at `$A0:A1B7` (PC `$1021B7`), `90 07` at
`$A0:A1CF`, `D0 09` at `$A0:A205` and `D0 09` at `$A0:9CCE`. The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- A shot hits every enemy again on each frame it is on screen (only plasma
  shots give the enemy a short invincibility time), so each shot does many
  times its normal damage.
- Super missiles shake the screen on every hit.
- Enemies with extended spritemaps (most bosses) have their own hitbox
  check, which isn't changed. They are hit only when a shot really
  touches them.
- Some enemies always stop shots (enemy properties bit `$1000`), and shots
  that do no damage are still stopped.
- The last two codes alone are "Shots keep flying after they hit an
  enemy".

## Shots keep flying after they hit an enemy

**Codes:** `6DCD-CB0D` + `6DBA-4867` (use both)

**Raw form:** `A0A205:80` (orig `$D0`), `A09CCE:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:14:52

**Mechanism:** When one of Samus's shots hits an enemy,
`Enemy_vs_Projectile_CollisionHandling` (`$A0:A143`) marks it for deletion
(bit `$10` of its direction) unless it is a plasma shot (type bit `$0008`):

```
    LDA.W SamusProjectile_Types,Y                                        ;A0A1FF;
    BIT.W #$0008                                                         ;A0A202;
    BNE .noDelete                                                        ;A0A205;

  .delete:
    LDA.W SamusProjectile_Directions,Y                                   ;A0A207;
    ORA.W #$0010                                                         ;A0A20A;
    STA.W SamusProjectile_Directions,Y                                   ;A0A20D;

  .noDelete:
    JSL.L .executeEnemyShot                                              ;A0A210;
```

The codes change `BNE .noDelete` to `BRA` here and in the same check in
`Enemy_vs_Projectile_CollisionHandling_ExtendedSpritemap` (`$A0:9CCE`, for
enemies with extended spritemaps, such as most bosses). Every beam shot
and missile then goes through enemies the way a plasma shot does.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `D0 09` at `$A0:A205` (PC `$102205`) and `D0 09` at
`$A0:9CCE` (PC `$101CCE`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only plasma shots give the enemy a short invincibility time after a hit.
  Other shots hit again on each frame they overlap the enemy, so they do
  more damage.
- Missiles go through enemies instead of exploding on them.
- Some enemies always stop shots (enemy properties bit `$1000`). Shots that
  do no damage (immune enemies) are still stopped.

## The Power Beam does much more damage

**Code:** `A667-4002`

**Optional, for the charged Power Beam too:** `D467-1162`

**Raw form:** `938431:C8` (orig `$14`); optional `93853A:02` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)

**Recorded:** 2026-10-20 23:15:22

**Mechanism:** Each kind of shot has a projectile data table in bank `$93`
whose first word is its damage. When a shot is fired,
`InitializeProjectile` (`$93:8000`) reads it into `SamusProjectile_Damages`:

```
  .merge:
    LDA.W $0000,Y                                                        ;93803C;
    STA.W SamusProjectile_Damages,X                                      ;93803F;
    BPL .dontCrash                                                       ;938042;
```

The Power Beam's tables:

```
ProjectileDataTable_Uncharged_Power:
    dw $0014                                                             ;938431; Damage
...
ProjectileDataTable_Charged_Power:
    dw $003C                                                             ;938539; Damage
```

The code changes the uncharged damage from `$0014` (20) to `$00C8` (200),
ten times as much. The optional code changes the high byte of the charged
damage, from `$003C` (60) to `$023C` (572). The enemy's vulnerability
multiplier is applied after this, as usual.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `14 00` at `$93:8431` (PC `$098431`) and `3C 00` at
`$93:8539` (PC `$098539`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only the plain Power Beam is changed. Once Samus has the Wave, Ice,
  Spazer or Plasma Beam equipped, her shots use other tables.
- Damage must stay below `$8000`, or the game crashes (`BPL .dontCrash`).
  Both values here are far below that.

## Sidehoppers, Puyos, Beetoms and Oums float down slowly after a long fall

**Code:** `DF6B-160F`
**Raw form:** `A08591:01` (orig `$08`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:35:40

**Mechanism:** Many enemies take their falling speed from
`CommonEnemySpeeds_QuadraticallyIncreasing` (`bank_A0.asm`, P.JBoy: "I.e.
gravity"). Each row is 8 bytes: subspeed, speed, negated subspeed,
negated speed. An enemy keeps a table index, adds to it each frame, and
moves down by that row's speed. Most falling code caps the index at row
`$40`, so that row is their top falling speed:

```
HopperMovement_Falling_UpsideUp:
    ...
    LDA.W CommonEnemySpeeds_QuadraticallyIncreasing+2,Y
    STA.B DP_Temp14
    LDA.W CommonEnemySpeeds_QuadraticallyIncreasing,Y
    STA.B DP_Temp12
    JSL.L MoveEnemyDownBy_14_12
    ...
    CMP.W #$0040
    BMI +
    LDA.W #$0040
```

Row `$40` (`$A0:858F`) is subspeed `$2120`, speed `$0008`, so the top
speed is about 8.1 pixels per frame. The code changes the speed to
`$0001`, so an enemy that falls far enough to reach row `$40` drops to
about 1.1 pixels per frame and floats down the rest of the way.

These cap at row `$40`: sidehoppers, desgeegas and their super versions
(`HopperMovement_Falling_*`, `bank_A3.asm`), Beetoms (`bank_A8.asm`, also
when Samus shakes one off), Puyos (`PuyoMovement`, `bank_A2.asm`, cap
`$4000` in 8.8 form) and Oums (`Function_Oum_Falling`, `bank_A2.asm`).
Boulders cap at row `$50` and only pass through row `$40` for a frame.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 21 08 00 E0 DE F7 FF` at `$A0:858F` (PC
`$10058F`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only long falls are changed. From a standstill, an enemy that steps
  one row per frame reaches row `$40` after about 170 pixels (an Oum
  steps 1.5 rows per frame, so about 115). A normal hop that lands at
  the same height looks the same as before.
- The change is sudden: the enemy speeds up as usual, and then slows
  down at once when it reaches row `$40`.
- Only the falling half of the row is changed. Rising uses the negated
  words, so jumps start as before.
- An Oum that bounces after a long fall rises from row `$30`, so it
  bounces about as high as before even though it landed slowly.
- Changes the same byte as "Sidehoppers, Puyos, Beetoms and Oums hang
  in mid-air after a long fall". Don't use both.
- Enemy projectiles use their own copy of the table
  (`$A0:CBC7`) and are not changed.
- Other enemies that read the table without this cap, such as
  Botwoon's death fall, may slow down for a frame as they pass row
  `$40`. Not checked.

## Sidehoppers, Puyos, Beetoms and Oums hang in mid-air after a long fall

**Code:** `DD6B-160F`
**Raw form:** `A08591:00` (orig `$08`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:43:32

**Mechanism:** This is a stronger form of "Sidehoppers, Puyos, Beetoms
and Oums float down slowly after a long fall", and changes the same
byte. Those enemies take their falling speed from
`CommonEnemySpeeds_QuadraticallyIncreasing` (`bank_A0.asm`) and cap
their table index at row `$40`, so that row is their top falling speed:

```
HopperMovement_Falling_UpsideUp:
    ...
    LDA.W CommonEnemySpeeds_QuadraticallyIncreasing+2,Y
    STA.B DP_Temp14
    LDA.W CommonEnemySpeeds_QuadraticallyIncreasing,Y
    STA.B DP_Temp12
    JSL.L MoveEnemyDownBy_14_12
    ...
    CMP.W #$0040
    BMI +
    LDA.W #$0040
```

Row `$40` (`$A0:858F`) is subspeed `$2120`, speed `$0008`, about 8.1
pixels per frame. The code changes the speed to `$0000`, so only the
subspeed is left: about 0.13 pixels per frame. An enemy that falls far
enough to reach row `$40` almost stops, and takes about 2 seconds to
drop one block (16 pixels).

These cap at row `$40`: sidehoppers, desgeegas and their super versions
(`HopperMovement_Falling_*`, `bank_A3.asm`), Beetoms (`bank_A8.asm`, also
when Samus shakes one off), Puyos (`PuyoMovement`, `bank_A2.asm`, cap
`$4000` in 8.8 form) and Oums (`Function_Oum_Falling`, `bank_A2.asm`).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `20 21 08 00 E0 DE F7 FF` at `$A0:858F` (PC
`$10058F`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only long falls are changed. From a standstill, an enemy that steps
  one row per frame reaches row `$40` after about 170 pixels (an Oum
  steps 1.5 rows per frame, so about 115). A normal hop that lands at
  the same height looks the same as before.
- The change is sudden: the enemy speeds up as usual, and then almost
  stops at once when it reaches row `$40`.
- A long fall now takes a long time to finish. An enemy that falls into
  a deep pit may hang there for a long time.
- Only the falling half of the row is changed. Rising uses the negated
  words, so jumps start as before.
- Changes the same byte as "Sidehoppers, Puyos, Beetoms and Oums float
  down slowly after a long fall". Don't use both.
- Enemy projectiles use their own copy of the table (`$A0:CBC7`) and are
  not changed.

## Samus smashes blocks as if she were Speed Boosting

**Codes:** `6DAE-1D27` + `6DA6-C7F7` (use both)
**Raw form:** `84CDF3:80` (orig `$F0`); `84CE8C:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:56:42

**Mechanism:** When Samus runs, jumps or falls into a speed block or a
bomb block, the block collision code (`bank_94.asm`) spawns a PLM whose
setup decides whether the block breaks. Both setups (`bank_84.asm`)
first check for a full Speed Boost:

```
Setup_Collision_RespawningSpeedBoostBlock:
    LDA.W SamusBoostCounter-1
    AND.W #$0F00
    CMP.W #$0400
    BEQ .speed
    LDA.W Pose
    CMP.W #$00C9                ; shinespark poses C9h..CEh
    ...
```

`Setup_Collision_RespawningBombBlock` is the same, with the screw attack
poses (81h/82h) added and the branch going to `.screwOrSpeed`. If no
check passes, the PLM is deleted and the block stays solid. The codes
change both `BEQ` instructions (`$F0`) to `BRA` (`$80`), so every touch
breaks the block, as if `SamusBoostCounter` were 4.

These two setups cover all of the collision PLMs for speed blocks (PLMs
`$D030..D040`, respawning or not) and bomb blocks (`$D098..D0B4`, all
sizes, respawning or not).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 3E 0B 29 00 0F C9 00 04 F0 29` at `$84:CDEA`
(PC `$024DEA`) and `AD 3E 0B 29 00 0F C9 00 04 F0 33` at `$84:CE83`
(PC `$024E83`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Collision from below and above also counts, so Samus falls through a
  floor of speed or bomb blocks as soon as she stands on it, and breaks
  them overhead when she jumps into them. Some rooms may become hard to
  cross, or a way back may be lost until the blocks respawn.
- Crumble blocks, shot blocks, super missile and power bomb blocks are
  not changed.
- Only the blocks change. Samus doesn't glow or hurt enemies; see
  "Touching an enemy hurts it, as if Samus were Speed Boosting".

## Touching an enemy hurts it, as if Samus were Speed Boosting

**Codes:** `CBCC-460F` + `DFCC-466F` + `DDCC-46AF` + `6DC8-4CDF` (use all)
**Raw form:** `A0A4A1:A9` (orig `$AD`); `A0A4A2:01` (orig `$6E`);
`A0A4A3:00` (orig `$0A`); `A0A4B8:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:57:05

**Mechanism:** This is the Speed Booster form of "Touching an enemy
hurts it, as with the Screw Attack". When Samus touches an enemy, most
enemies run `NormalEnemyTouchAI_NoDeathCheck` (`bank_A0.asm`). It reads
`ContactDamageIndex`: 0 for normal Samus, 1 speed boosting, 2
shinesparking, 3 screw attack, 4 pseudo screw attack. If it is 0, the
enemy hurts Samus. If not, Samus hurts the enemy:

```
    LDA.W ContactDamageIndex                                             ;A0A4A1;
    BNE .notNormal                                                       ;A0A4A4;
    JMP.W .normalSamus                                                   ;A0A4A6;

  .notNormal:
    CLC                                                                  ;A0A4A9;
    ADC.W #$000F                                                         ;A0A4AA;
    STA.B DP_Temp14                                                      ;A0A4AD;
    LDY.W #$01F4                                                         ;A0A4AF;
    LDA.W ContactDamageIndex                                             ;A0A4B2;
    CMP.W #$0001                                                         ;A0A4B5;
    BEQ .damageCalculated                                                ;A0A4B8;
```

`DP_Temp14` picks the column of the enemy's vulnerability table (10h is
the Speed Booster column) and `Y` is the damage (`$01F4` = 500 for speed
boosting).

The first three codes change `LDA.W ContactDamageIndex` to `LDA.W
#$0001`, so the routine always takes the `.notNormal` path with column
10h. The fourth code changes the `BEQ .damageCalculated` at `$A0:A4B8`
to `BRA`. Without it, the second read of `ContactDamageIndex` (still 0)
would fall through to the default damage (200). With it, normal Samus
does the full Speed Booster damage of 500, times the enemy's multiplier
(1x for most enemies).

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 6E 0A D0` at `$A0:A4A1` (PC `$1024A1`) and
`AD 6E 0A C9 01 00 F0 26` at `$A0:A4B2` (PC `$1024B2`). The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Use all four codes. The first three alone stop enemies from hurting
  Samus on touch, but do only 200 damage.
- Enemies that resist the Speed Booster still aren't hurt, but they don't
  hurt Samus either.
- Bosses and other enemies with their own touch code aren't changed.
  Enemy shots still hurt Samus.
- Shinesparking, the Screw Attack and the pseudo screw attack now use the
  Speed Booster column and damage (500) instead of their own. The pseudo
  screw attack no longer ends the beam charge.
- Changes the same bytes as "Touching an enemy hurts it, as with the
  Screw Attack". Don't use both.

## Crouching always charges a Shinespark

**Code:** `DDE8-3109`
**Raw form:** `91F7B9:00` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:57:36

**Mechanism:** In the real game, Samus stores a Shinespark by crouching
while she is fully Speed Boosting. When she goes into a crouch (poses
35h/36h), `InitializeSamusPose_CrouchingTransition` (`bank_91.asm`)
checks the boost counter:

```
InitializeSamusPose_CrouchingTransition:
; Gives Samus shinespark
    LDA.W SamusBoostCounter-1
    AND.W #$FF00
    CMP.W #$0400
    BMI .returnCarryClear
    LDA.W #regional($00B4, $0090)
    STA.W SpecialPaletteTimer
    LDA.W #$0001
    STA.W SpecialSamusPaletteType
    STZ.W SpecialSamusPaletteFrame
```

`SpecialPaletteTimer` is the stored Shinespark: Samus flashes while it
counts down from `$B4` (180 frames, 3 seconds). If Samus does a normal
straight-up jump (poses 4Dh/4Eh, 15h/16h, 69h/6Ah) while it is not 0,
`InitializeSamusPose_NormalJumping` starts the Shinespark windup
instead. The code sets the `BMI` offset to 0, so every crouch stores a
Shinespark, as if Samus had been fully Speed Boosting.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 3E 0B 29 00 FF C9 00 04 30 0F` at `$91:F7B0`
(PC `$08F7B0`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus doesn't need the Speed Booster.
- To Shinespark: crouch (press down while standing), stand up, and jump
  straight up within 3 seconds.
- Samus flashes after every crouch, and a straight-up jump within 3
  seconds of a crouch is always a Shinespark. Wait for the flashing to
  stop to make a normal jump.
- Running jumps and spin jumps are not changed.

## Samus smashes blocks, hurts enemies and can Shinespark as if she were Speed Boosting

**Codes:** `6DAE-1D27` + `6DA6-C7F7` + `CBCC-460F` + `DFCC-466F` +
`DDCC-46AF` + `6DC8-4CDF` + `DDE8-3109` (use all)
**Raw form:** `84CDF3:80` (orig `$F0`); `84CE8C:80` (orig `$F0`);
`A0A4A1:A9` (orig `$AD`); `A0A4A2:01` (orig `$6E`); `A0A4A3:00` (orig
`$0A`); `A0A4B8:80` (orig `$F0`); `91F7B9:00` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:57:52

**Mechanism:** This puts together the codes of three other entries:
"Samus smashes blocks as if she were Speed Boosting" (first two codes),
"Touching an enemy hurts it, as if Samus were Speed Boosting" (next four)
and "Crouching always charges a Shinespark" (last code). See those
entries for how each part works. In short:

- Speed blocks and bomb blocks break when Samus runs, jumps or falls into
  them, as if `SamusBoostCounter` were 4 (fully charged).
- Enemies with the normal touch code take the Speed Booster's contact
  damage (500, scaled by the enemy's Speed Booster vulnerability) instead
  of hurting Samus.
- Every crouch stores a Shinespark for 3 seconds, as if Samus had
  crouched while Speed Boosting.

**Evidence:** See the three entries named above. `smet_original.smc`
(copier header, so file offset = PC offset + `$200`) has the original
bytes listed in the raw form. The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Seven codes, more than a real Game Genie holds (5). Use the three
  separate entries to pick the parts you want.
- Samus doesn't glow blue or run faster, and other things that check for
  a real Speed Boost (quicksand, air spikes, Draygon's goop, enemy
  shots) still act as normal.
- The caveats of the three separate entries apply.

## The Screw Attack hits every enemy on the screen

**Code:** `15CB-4C6D`
**Raw form:** `A0A09A:67` (orig `$1D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 13:58:53

Tags: `demo-ready` `curated`

**Mechanism:** Each frame, `Enemy_vs_Samus_CollisionHandling`
(`bank_A0.asm`) runs for each enemy being processed (the ones on or near
the screen). When `ContactDamageIndex` is not 0 (Samus is speed
boosting, shinesparking, screw attacking or pseudo screw attacking), it
clears Samus's invincibility and skips to `.tangible`, which checks the
enemy has a touch routine and then checks Samus's hitbox against the
enemy's:

```
    LDA.W ContactDamageIndex                                             ;A0A091;
    BEQ .normalContactDamage                                             ;A0A094;
    STZ.W SamusInvincibilityTimer                                        ;A0A096;
    BRA .tangible                                                        ;A0A099;
    ...
  .hasTouchAI:
    LDX.W EnemyIndex                                                     ;A0A0CD;
    LDA.W SamusXPosition                                                 ;A0A0D0;
    ...                                     ; X and Y distance checks
+   LDX.W EnemyIndex                                                     ;A0A102;
    ...
    JSL.L .executeEnemyTouch                                             ;A0A120;
```

The code changes the `BRA` offset from `$1D` (to `.tangible`) to `$67`
(to `$A0:A102`), so for a Screw Attacking Samus the touch-routine check
and both distance checks are skipped. Every processed enemy runs its
touch routine every frame, as if Samus were touching it. For most
enemies this is `NormalEnemyTouchAI`, which does the Screw Attack's
contact damage (2000, times the enemy's Screw Attack multiplier).

Skipping the touch-routine check is safe for the enemy headers in
`bank_A0.asm`: none has a touch routine of `RTS_xx804B`, the case that
check guards against. Enemies with no touch code use `RTL_xx804C`, which
just returns.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 6E 0A F0 05 9C A8 18 80 1D` at `$A0:A091` (PC
`$102091`). All enemy AI banks used by `RTL_xx804C` touch routines (`$A2`
to `$AA`, `$B2`, `$B3`) have `60 6B` (RTS, RTL) at `$xx:804B`. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The Speed Booster, Shinesparks and the pseudo screw attack (spin jump
  with a charged beam) also hit every enemy on the screen, with their own
  damage.
- Enemies that resist the Screw Attack still aren't hurt.
- Enemies with extended spritemaps (most bosses) have their own hitbox
  check, which isn't changed.
- Enemies with their own touch code act as if Samus touched them, from
  anywhere on the screen. Some of these do more than take damage (for
  example Metroids, Yapping Maws and Kraid's nails). Not checked.
- Frozen enemies are still skipped, as before.

## Wall jumps don't need timing

**Codes:** `DDB9-1107` + `DDB1-19D7` (use both)
**Raw form:** `909D59:00` (orig `$0B`); `909D64:00` (orig `$1B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 14:00:19

**Mechanism:** A wall jump normally takes two steps. While spin jumping
next to a wall, Samus presses away from it, and her animation switches
to the wall-jump-ready frame. Jump must then be pressed while that frame
lasts. `WallJump_Check` (`bank_90.asm`) runs each frame of a spin jump
or wall jump and picks the step by the animation frame:

```
  .spinJumping:
    LDA.W Pose
    CMP.W #$0081
    BEQ .screwAttack
    CMP.W #$0082
    BEQ .screwAttack
    LDA.W SamusAnimationFrame
    CMP.W #$000B
    BMI .spinningAnimation
    JMP.W .wallJumpEligibleAnimation

  .screwAttack:
    LDA.W SamusAnimationFrame
    CMP.W #$001B
    BMI .spinningAnimation
    JMP.W .wallJumpEligibleAnimation
```

In `.spinningAnimation`, pressing away from a wall within 8 pixels only
sets the animation frame to `$0A` (`$1A` with the Screw Attack), which
leads to the ready frame. In `.wallJumpEligibleAnimation`, pressing away
from the wall plus a new jump press starts the wall jump. The codes
change the frame numbers to 0, so the `BMI` is never taken and every
frame of a spin jump counts as ready. Pressing away from the wall and
jump on the same frame now starts a wall jump at once.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD 96 0A C9 0B 00 30 0E 4C DD 9D AD 96 0A C9 1B 00
30 03 4C` at `$90:9D55` (PC `$081D55`). The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus still has to be spin jumping (or wall jumping), within 8 pixels
  of the wall, pressing away from it, and press jump.
- The first code alone covers a normal spin jump. The second covers the
  Screw Attack spin.
- Samus no longer shows the wall-jump-ready frame.
- Wall jumps from solid enemies (for example frozen ones) are covered too.

## Samus can wall jump from twice as far from the wall

**Codes:** `FDBB-C5A7` + `FDB5-C1A7` + `FDB9-C167` (use all three)
**Raw form:** `909E9F:10` (orig `$08`); `909E7B:10` (orig `$08`);
`909E5A:10` (orig `$08`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 14:00:48

**Mechanism:** Each frame of a spin jump, the spin jump movement code
(`bank_90.asm`) calls `WallJump_Check` with the distance to look for a
wall:

```
    LDA.W SamusPhysicsConstants_MaximumDistanceFromWallForWallJump
    STA.B DP_Temp12
    STZ.B DP_Temp14
    JSR.W WallJump_Check
```

`SamusPhysicsConstants_MaximumDistanceFromWallForWallJump` is 8 pixels.
`WallJump_Check` looks that far away from the side Samus is pressing
away from, for a solid enemy (`Samus_vs_SolidEnemy_CollisionDetection`)
or a solid block (`WallJumpBlockCollisionDetection`). If one is found,
it checks the returned distance again before it starts the wall jump:

```
  .checkDistanceEnemy:
    LDA.B DP_Temp12
    CMP.W #$0008
    BPL .returnCarryClear
    ...
  .checkDistanceBlock:
    LDA.B DP_Temp12
    CMP.W #$0008
    BPL .returnCarryClear
```

The first code changes the distance constant to 16 pixels (one block).
The other two change both `CMP.W #$0008` to `CMP.W #$0010`, for blocks
and for solid enemies. Samus can now wall jump with a gap of up to 15
pixels to the wall instead of 7.

The block check looks only at the block column at the far end of the
distance. With 16 pixels, that is still the first column of the wall
for any gap from 0 to 15, so walls one block thick still work.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `08 00` at `$90:9E9F` (PC `$081E9F`), `A5 12 C9 08
00 10` at `$90:9E57` and `A5 12 C9 08 00 10` at `$90:9E78`. Only the
spin jump movement code reads the distance constant. The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Samus still has to press away from the wall and time the jump. Use
  with "Wall jumps don't need timing" (five codes in all) for the
  easiest wall jumps.
- The wall-jump-ready frame also starts from 16 pixels away.
- The first two codes alone cover walls. The third is for solid enemies.

## Bomb blasts bounce Samus much higher and farther

**Codes:** `D0BE-C907` + `D1B4-3107` (use both)
**Raw form:** `909EF5:04` (orig `$02`); `909F29:06` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 14:01:42

**Mechanism:** When a bomb blast hits Samus, `SetupBombJump` starts a
bomb jump, and `Make_Samus_BombJump` (`bank_90.asm`) sets her first Y
speed from a table, picked by whether she is in air, water or
lava/acid:

```
  .merge:
    LDA.W SamusPhysicsConstants_InitialYSubSpeeds_BombJump,X
    STA.W SamusYSubSpeed
    LDA.W SamusPhysicsConstants_InitialYSpeeds_BombJump,X
    STA.W SamusYSpeed
```

`SamusPhysicsConstants_InitialYSpeeds_BombJump` is `2, 0, 0` and the
subspeeds are `$C000, $1000, $1000`, so in air Samus starts up at 2.75
pixels per frame. The first code changes the air speed to 4, so she
starts at 4.75 pixels per frame. With the air gravity (`$1C00`
subpixels per frame per frame), she goes about 3 times as high: about
6 blocks instead of about 2.

When the blast is to one side, `MoveSamus_Horizontally_KnockbackBombJump`
moves Samus sideways while she rises, with the speeds in
`SamusPhysicsConstants_XAccelSpeeds_DiagonalBombJump`: acceleration
`$0000.$3000`, max speed `$0003.$0000`, deceleration `$0000.$0800`. The
second code changes the max speed to 6 pixels per frame. With the longer
rise, a sideways bounce goes much farther.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `02 00 00 00 00 00 00 C0 00 10 00 10` at `$90:9EF5`
(PC `$081EF5`) and `00 00 00 30 03 00 00 00 00 00 00 08` at `$90:9F25`
(PC `$081F25`). Only `Make_Samus_BombJump` reads the Y tables, and only
`MoveSamus_Horizontally_KnockbackBombJump` reads the X table (knockback
uses its own speeds there). The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only bomb jumps in air are changed. In water and lava/acid they are
  the same as before.
- The heights are worked out from the physics constants, so they are
  rough. Chained Morph Ball bomb jumps likely need different timing.
- Samus can reach places early that a normal bomb jump can't.
- The first code alone gives higher bounces with the normal sideways
  speed.

## Enemies drop big energy instead of small energy

**Code:** `D4E9-C7B6`
**Raw form:** `86F25E:02` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 14:02:22

**Mechanism:** `Random_Drop_Routine` (`bank_86.asm`) picks a drop from
the enemy's drop chances, in the order small energy, big energy,
missiles, nothing, Super Missiles, Power Bombs. It turns the index it
picked into a drop type with this table:

```
  .drops:                                                                ;86F25E;
    db $01 ; Small health
    db $02 ; Big health
    db $04 ; Missiles
    db $06 ; Nothing
    db $05 ; Super missiles
    db $03 ; Power bombs
```

The code changes the small health entry from 1 to 2 (big health). When
the roll picks small energy, the enemy drops big energy instead: 20
energy (`Apply pickup to Samus - big health`, `$86:F0CA`) instead of 5
(`$86:F0BB`). The chances are not changed, so energy drops come as often
as before. Missile, Super Missile and Power Bomb drops are not changed.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `01 02 04 06 05 03` at `$86:F25E` (PC `$03725E`).
Both callers of `Random_Drop_Routine` (`$86:EEB1`, enemy death
explosions, and `$86:EF42`, `InitAI_EnemyProjectile_Pickup`) use this
table. The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Changes a different byte of the same table as "Every killed enemy
  drops an item". The two work together. With that code's big energy
  option (`D4E1-CD96`), every drop that isn't a missile, Super Missile
  or Power Bomb is big energy.

## Samus has the Varia Suit's protection without it

**Codes:** `6D35-37E0` + `DDC1-4C6F` + `DD33-1904` (use all three)
**Raw form:** `8DE37F:80` (orig `$D0`); `A0A46A:00` (orig `$02`);
`90E9E5:00` (orig `$2B`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 14:03:12

**Mechanism:** The Varia Suit is bit `$0001` of `EquippedItems`. It does
three things, and each code fakes one of them:

- No heat damage. In heated rooms (Norfair), the palette FX
  pre-instruction `PreInstruction_PaletteFXObject_SamusInHeat`
  (`bank_8D.asm`) adds 1/4 energy per frame of periodic damage unless
  Samus has the Varia or Gravity Suit:

  ```
      LDA.W EquippedItems                                                  ;8DE379;
      AND.W #$0021                                                         ;8DE37C;
      BNE +                                                                ;8DE37F;
  ```

  The first code changes the `BNE` to `BRA`, so the heat damage (and
  its warning sound) is always skipped.
- Half damage from enemies. `Suit_Damage_Division` (`bank_A0.asm`)
  shifts `EquippedItems` right, so the Varia bit goes into carry:

  ```
      LSR                                                                  ;A0A468;
      BCC .fullDamage                                                      ;A0A469;
      LSR.B DP_Temp12                                                      ;A0A46B;
  ```

  The second code sets the `BCC` offset to 0, so the damage is always
  halved. Enemy touches, enemy shots and some boss attacks use this
  routine.
- Half periodic damage (lava, acid, spikes and the like).
  `HandlePeriodicDamageToSamus` (`bank_90.asm`) skips the halving with
  `BIT.W #$0001 : BEQ .crashIfNegative` when the Varia bit is clear. The
  third code sets that `BEQ` offset to 0.

The Gravity Suit is still checked first in the last two routines, so
with it Samus still takes a quarter of the damage.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD A2 09 29 21 00 D0 2A` at `$8D:E379` (PC
`$06E379`), `AD A2 09 89 20 00 D0 08 4A 90 02 46` at `$A0:A460` (PC
`$102460`) and `AD A2 09 89 20 00 D0 1B 89 01 00 F0 2B` at `$90:E9D9`
(PC `$0869D9`). The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The first code alone lets Samus explore heated rooms without damage.
- Samus still looks like she is in the Power Suit, and heated rooms
  still flash her palette as if she had no Varia.
- Lava still hurts, at half the rate, as with the real Varia Suit.
- The item itself is not given. Anything else that checks for it is not
  changed.

## Samus takes double damage

**Codes:** `DDC1-4C6F` + `D1C1-4CAF` + `DD33-1904` + `DC33-1104` (use all)
**Raw form:** `A0A46A:00` (orig `$02`); `A0A46B:06` (orig `$46`);
`90E9E5:00` (orig `$2B`); `90E9E9:0A` (orig `$4A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-21 14:03:55

Tags: `demo-ready` `curated` `challenge`

**Mechanism:** This turns the Varia Suit's half damage into double
damage, and gives it to Samus all the time. There are two damage paths:

- Enemy damage. `Suit_Damage_Division` (`bank_A0.asm`) halves the damage
  when the Varia bit of `EquippedItems` is set:

  ```
      LSR                                                                  ;A0A468;
      BCC .fullDamage                                                      ;A0A469;
      LSR.B DP_Temp12                                                      ;A0A46B;
  ```

  The first code sets the `BCC` offset to 0, so the next instruction
  always runs. The second changes `LSR.B DP_Temp12` (`$46`) to
  `ASL.B DP_Temp12` (`$06`), so the damage is doubled instead of halved.
  Enemy touches, enemy shots and some boss attacks use this routine.
- Periodic damage (heat, lava, acid, spikes and the like).
  `HandlePeriodicDamageToSamus` (`bank_90.asm`) halves the frame's
  damage for the Varia Suit:

  ```
      BIT.W #$0001                                                         ;90E9E1;
      BEQ .crashIfNegative                                                 ;90E9E4;
      LDA.W PeriodicSubDamage+1                                            ;90E9E6;
      LSR                                                                  ;90E9E9;
  ```

  `PeriodicSubDamage+1` is the damage in 8.8 form (high byte of the
  subdamage and low byte of `PeriodicDamage`). The third code sets the
  `BEQ` offset to 0, and the fourth changes `LSR` to `ASL`, so it is
  doubled instead of halved.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `4A 90 02 46 12` at `$A0:A468` (PC `$102468`) and
`F0 2B AD 4F 0A 4A` at `$90:E9E4` (PC `$0869E4`). The codes decode back
to their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The first two codes alone double enemy damage only. The last two
  alone double periodic damage only.
- With the Gravity Suit, Samus takes a quarter of the damage as before.
  The Gravity Suit is checked first.
- With the Varia Suit, Samus also takes double damage, not half.
- Damage that uses neither routine is not changed. Which attacks those
  are was not checked; `HandlePeriodicDamageToSamus` notes that Metroids,
  Beetoms, the rainbow beam and Shinesparks don't use it.
- Shares two codes with "Samus has the Varia Suit's protection without
  it" and changes the same routines. Don't use both.

## New games skip the opening cinematic

**Codes:** `4E33-CDD8` + `8533-CD08` + `DD81-3F42` + `AF81-3F12` + `4A81-3412` (use all)
**Raw form:** `82EEE0:2F` (orig `$95`); `82EEE1:B7` (orig `$A3`);
`8BB764:00` (orig `$A0`); `8BB765:C1` (orig `$BC`); `8BB769:2C`
(orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-22 00:16:08

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** When a new file is started, `GameOptionsMenu_4_StartGame`
(`bank_82.asm`) sets game state `1Eh` (intro) and the first cinematic
function:

```
  .intro:
    LDA.W #$001E                                                         ;82EED9;
    STA.W GameState                                                      ;82EEDC;
    LDA.W #CinematicFunction_Intro_Initial                               ;82EEDF;
    STA.W CinematicFunction                                              ;82EEE2;
```

The intro (the "last Metroid is in captivity" story) ends with
`CinematicFunction_Intro_Finish` (`bank_8B.asm`). Once the screen is
faded out, it clears the cinematic object RAM, sets up the flight to
Ceres, puts back the button settings that the intro saved, and clears
the intro's 900 missiles:

```
    LDA.W #CinematicFunction_FlyToCeres_Initial                          ;8BB763;
    STA.W CinematicFunction                                              ;8BB766;
    JSR.W Recover_Previous_Button_Assignments                            ;8BB769;
    STZ.W MaxMissiles                                                    ;8BB76C;
    STZ.W Missiles                                                       ;8BB76F;
```

After the flight, `CinematicFunction_FlyToCeres_StartGameAtCeres` sets
game state `1Fh` (set up new game) and the loading game state to `1Fh`,
picks Ceres, and saves the game.

- The first two codes change the first cinematic function from
  `$A395` (`CinematicFunction_Intro_Initial`) to `$B72F`
  (`CinematicFunction_Intro_Finish`). The options menu has already
  faded the screen out, so the finish function goes on at once.
- The third and fourth codes change its next function from `$BCA0`
  (`CinematicFunction_FlyToCeres_Initial`) to `$C100`
  (`CinematicFunction_FlyToCeres_StartGameAtCeres`), so the flight is
  skipped too.
- The last code changes `JSR` (`$20`) to `BIT` (`$2C`), so
  `Recover_Previous_Button_Assignments` isn't called. The intro never
  ran, so there is nothing to put back, and the button settings from the
  options menu are kept.

The game then starts at Ceres as before, with the file saved.

Going straight to `CinematicFunction_FlyToCeres_StartGameAtCeres` from
the menu (two codes) wasn't used. The intro's object handlers run after
the cinematic function on the same frame. The menus keep the selection
missile's position in `Mode7Object_InstListPointers` ($19A1), so
without the clear in `CinematicFunction_Intro_Finish`, the Mode 7
object handler would run junk as an instruction list and likely crash.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `A9 95 A3 8D 51 1F` at `$82:EEDF` (PC `$016EDF`),
`A9 A0 BC 8D 51 1F 20 75 BC 9C C8 09` at `$8B:B763` (PC `$05B763`),
`22 24 89 80` (`JSL.L HandleFadingOut`) at `$8B:B72F` and `A9 1F 00 8F
14 D9` at `$8B:C100`. The `LDA` at `$82:EEDF` is the only reference to
`CinematicFunction_Intro_Initial`, and the code at `$8B:B763` is the
only reference to `CinematicFunction_FlyToCeres_Initial` and the only
call of `Recover_Previous_Button_Assignments`, so the codes change
nothing else. `HandleFadingOut` returns at once with brightness 0, and
the options menu leaves `DP_Brightness` at `$80`, which is what
`CinematicFunction_Intro_Finish` waits for. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Use all five codes. With only one of each pair, the game jumps into
  the middle of a function and will likely crash. Without the last code,
  the button settings are set from RAM that the intro never filled.
- Only new files change. Loading a save works as before. The Ceres
  explosion and the flight to Zebes still play after Ceres.
- Five codes, the most a real Game Genie holds.
- Works with "New games skip Ceres Station and start in Crateria"
  (different addresses). With both, a new game goes straight from the
  options menu to the gunship on Zebes. That needs seven codes.

## Enemies that shots can't hurt take normal damage

**Codes:** `D7C9-38DF` + `D7C0-3B6F` (use both)
**Raw form:** `A0A75C:03` (orig `$4B`); `A0A746:03` (orig `$61`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-22 01:44:44

**Mechanism:** Each enemy has a vulnerability table with one byte per
weapon (`bank_B4.asm`). The low bits are the damage multiplier times 2,
so `$02` is normal damage and `$00` is none. Rippers, Boyons, Squeepts,
Gold Space Pirates, Kamers and many others have `$00` for some or all
weapons. `NormalEnemyShotAI_NoDeathCheck_NoEnemyShotGraphic`
(`bank_A0.asm`) stores the projectile's damage, then multiplies it by
the enemy's multiplier:

```
  .calculateDamage:
    LDA.W EnemySpritemapEntryXPositionDuringCollision                    ;A0A74A;
    LSR                                                                  ;A0A74D;
    STA.B DP_Temp26                                                      ;A0A74E;
    LDA.W Temp_DamageMultiplier                                          ;A0A750;
    STA.B DP_Temp28                                                      ;A0A753;
    JSL.L Multiplication_32bit_A0B6FF                                    ;A0A755;
    LDA.B DP_Temp2A                                                      ;A0A759;
    BEQ .noDamage                                                        ;A0A75B;
    STA.W EnemySpritemapEntryXPositionDuringCollision                    ;A0A75D;
    JMP.W .damage                                                        ;A0A760;
```

A result of 0 goes to `.noDamage`, which makes the dud shot and its
"tink" sound. The first code changes the `BEQ` offset from `$4B` to
`$03`, so a result of 0 goes to `JMP.W .damage` and skips the `STA`. The
damage is then still the projectile's own damage, as with a multiplier
of `$02`.

Charged beams use the charged beam entry instead, and skip the multiply
when it is 0:

```
    AND.W #$000F                                                         ;A0A742;
    BEQ .noDamage                                                        ;A0A745;
    STA.W Temp_DamageMultiplier                                          ;A0A747;

  .calculateDamage:
```

The second code changes that `BEQ` offset from `$61` to `$03`, so it
goes to `.calculateDamage` with the uncharged beam multiplier. If that
is also 0, the first code gives normal damage. 24 tables have `$00` as
their charged beam entry; none has `$FF`.

The same routine handles beams, missiles, Super Missiles, bombs and
Power Bombs. Most enemies use it through `Common_NormalEnemyShotAI`, and
about 120 other calls in banks `$A0`..`$B3` reach it from enemy and boss
shot routines.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `C9 FF 00 F0 66 29 0F 00 F0 61 8D 32 0E` at
`$A0:A73D` (PC `$10273D`) and `AD 32 0E 85 28 22 FF B6 A0 A5 2A F0 4B
8D 7A 18 4C 08 A8` at `$A0:A750` (PC `$102750`). `$A0:A75D` + 3 is
`$A0:A760` (`4C 08 A8`, `JMP.W .damage`), and `$A0:A747` + 3 is
`$A0:A74A` (`.calculateDamage`). The projectile damage is stored at
`$A0:A6E7` and isn't changed before `.damage`. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- The first code alone works for everything but charged beam shots on
  enemies whose charged beam entry is 0. Those still make a dud.
- `$FF` entries (freeze, no damage) aren't changed. Ice beams still
  freeze Rippers, Boyons, Metroids and the like without hurting them,
  as before. Other beams now hurt them.
- Enemies that take reduced damage still take the reduced amount; only
  a result of 0 changes. Rarely, a weak shot on an enemy that takes half
  damage rounds down to 0 and now does full damage instead.
- Enemies whose shot routine doesn't call this routine (for example the
  ones with `RTL` shot routines: Kzans, Hibashi, Puromi, Steam, Chozo
  statues) are not changed. Which custom shot routines call it was not
  checked one by one.
- Some "enemies" are part of the room: Kamers and Shutters (moving
  platforms and gates), Mama Turtle, lava rocks and Sbugs use this
  routine with an all-zero table. They can now be killed. Killing a
  platform or a gate may make a room hard or impossible to get through.
- Bosses whose shot code uses this routine may take damage from weapons
  that normally don't hurt them. Phantoon's and Golden Torizo's tables,
  for example, have `$80` (0 damage) for beams. Their shot code was not
  checked.
- Contact damage (Speed Booster, Screw Attack, Shinespark) uses other
  code and isn't changed.

## No delay between beam shots

**Code:** `DDC0-41D7`
**Raw form:** `90AC48:00` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 18:43:55

**Mechanism:** `Check_if_Samus_Can_Fire_Beam` (`bank_90.asm`) lets
Samus fire a beam only if fewer than 5 shots are out and the cooldown
timer is 0:

```
    LDA.W SamusProjectile_ProjectileCounter                              ;90AC39;
    CMP.W #$0005                                                         ;90AC3C;
    BPL .noFire                                                          ;90AC3F;
    LDA.W SamusProjectile_CooldownTimer                                  ;90AC41;
    AND.W #$00FF                                                         ;90AC44;
    BNE .noFire                                                          ;90AC47;
```

After a shot, `Fire_Uncharge_Beam` sets the timer from
`ProjectileCooldowns_Uncharged` (`$0F` frames, `$0C` for Plasma + Ice)
when the shot button was just pressed, or from `BeamAutoFireCooldowns`
(`$19` frames) when it is held. The code changes the `BNE` offset from
`$0F` to `$00`, so the timer is not checked. The 5-shot limit still
applies.

Without the Charge Beam, holding the shot button fires a beam every
frame until 5 are on the screen, and then a new one each time one goes
away. With the Charge Beam, each tap fires a shot, with no wait between
taps.

**Evidence:** `smet_original.smc` (copier header, so file offset = PC
offset + `$200`) has `AD CE 0C C9 05 00 10 17 AD CC 0C 29 FF 00 D0 0F`
at `$90:AC39` (PC `$082C39`). The `$0F` is at `$90:AC48` (PC
`$082C48`). The missile check (`$90:AC5A`) has its own cooldown test,
so this code does not change missiles. The code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** Japan/USA only (the only NTSC version); PAL not checked.

**Caveats:**

- Not tested.
- Only beams. Missiles, Super Missiles and bombs keep their delay.
- At most 5 beam shots can be out at a time, as before.
- Each shot plays the beam sound, so the sound repeats every frame
  while the button is held.

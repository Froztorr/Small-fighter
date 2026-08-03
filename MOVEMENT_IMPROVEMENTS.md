# Small Fighter movement improvements

This note captures the next implementation targets for making movement feel more natural and more like a 1v1 fighting game.

## High-priority movement changes

1. Add acceleration/deceleration instead of directly setting `vx`.
2. Add true movement transition states:
   - `prejump`
   - `landing`
   - `dashForward`
   - `dashBack`
3. Add throw input and close-range throw logic.
4. Add meter spend helpers for EX/super mechanics.
5. Add debug overlay for state, frame data, hitboxes, and hurtboxes.

## Suggested helper functions

```js
approachVelocity(target, accel) {
    if (this.vx < target) this.vx = Math.min(target, this.vx + accel);
    else if (this.vx > target) this.vx = Math.max(target, this.vx - accel);
}

applyGroundFriction(amount = CFG.friction) {
    if (Math.abs(this.vx) <= CFG.minSpeed) {
        this.vx = 0;
        return;
    }
    this.vx -= sign(this.vx) * amount;
}

startJump(dirX) {
    this.state = "prejump";
    this.stateT = 0;
    this.jumpDirX = dirX;
}

land() {
    this.y = CFG.groundY;
    this.vy = 0;
    this.airborne = false;
    this.state = "landing";
    this.stateT = 0;
    fxDust(this.x, CFG.groundY, this.dir, 4);
}

startDash(dir) {
    this.state = dir > 0 ? "dashForward" : "dashBack";
    this.stateT = 0;
    this.vx = this.dir * dir * this.S.dashSpeed;
    fxDust(this.x, CFG.groundY, this.dir * -dir, 6);
}

canSpendMeter(cost) {
    return this.meter >= cost;
}

spendMeter(cost) {
    if (this.meter < cost) return false;
    this.meter -= cost;
    return true;
}
```

## Throw design

Suggested input: `LP + LK`.

```js
tryThrow(opp) {
    if (Math.abs(opp.x - this.x) > 72) return false;
    if (!this.grounded || !opp.grounded) return false;
    if (["launched", "down", "wakeup"].includes(opp.state)) return false;

    opp.hp = Math.max(0, opp.hp - 90);
    opp.vx = this.dir * 9;
    opp.vy = -7;
    opp.y -= 1;
    opp.state = "launched";
    opp.airborne = true;
    shake(8);
    fxHit(opp.skel.j.chest.x, opp.skel.j.chest.y, true);
    return true;
}
```

## Implementation order

1. Add helper functions to `Fighter`.
2. Replace direct walk/run velocity assignment with `approachVelocity`.
3. Route jump input through `startJump`.
4. Add `prejump` and `landing` cases to update and pose logic.
5. Replace backdash brake behavior with `startDash(-1)`.
6. Add `tryThrow` before normals/specials in neutral.
7. Add EX/super meter functions.
8. Add debug overlay after core feel is stable.

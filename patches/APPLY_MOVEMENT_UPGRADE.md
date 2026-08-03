# Apply movement upgrade

This patch implements the movement functions requested for `index.html`:

- `approachVelocity`
- `applyGroundFriction`
- `startJump` / `prejump`
- `land` / `landing`
- `startDash`
- `canSpendMeter` / `spendMeter`
- `tryThrow` with `LP + LK`
- updated walk/run acceleration and landing transitions

Apply from repo root:

```bash
git apply patches/movement-upgrade.patch
```

Then test in browser and commit.

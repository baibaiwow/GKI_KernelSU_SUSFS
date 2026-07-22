# Safemode false-positive fix

SukiSU counts `KEY_VOLUMEDOWN` for rescue safemode. On some GKI builds the
input kprobe can keep counting after boot, so opening the manager later may
show "Safe mode" while modules/root still work.

The build applies this automatically in `kernel_builder.apply_safemode_fix()`
right after `add_kernelsu()`:

1. Decide safemode **once** during early boot (`decided` flag).
2. Ignore further volume-down counting after modules mounted / boot completed.

No manual `git apply` is required; this file is documentation only.

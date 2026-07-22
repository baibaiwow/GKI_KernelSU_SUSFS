# Safemode false-positive fix

## Why the first fix failed

The GKI build runs `curl setup.sh | bash -s builtin`, which checks out
SukiSU's **builtin** branch. That tree uses:

- `kernel/runtime/ksud.c` (vol_detector input handler)
- NOT `kernel/runtime/ksud_integration.c` (main branch)

The first patch only looked for main-layout files, so CI skipped the fix
while the build still succeeded. Flashing that artifact left the bug intact.

## Builtin root cause

With `CONFIG_KSU_SUSFS` + `KSU_COMPAT_USE_STATIC_KEY`, `on_post_fs_data()`
only disables a static key and prints `ksu_input_hook is disabled`.
It does **not** call `stop_input_hook()` → `vol_detector_exit()`, so volume
key presses after boot keep setting `safe_mode_flag`. Opening the manager
later then reports Safe mode while modules stay loaded.

## What the build applies now

`kernel_builder.apply_safemode_fix()` patches builtin `runtime/ksud.c`:

1. Always `stop_input_hook()` from `on_post_fs_data` (unregister vol_detector).
2. Latch `ksu_is_safe_mode()` decision once (`decided` / `result`).
3. Ignore volume events after hook stopped / modules mounted / boot completed.

Look for `BAIBAI_SAFEMODE_ONCE_FIX` and log line `已写入 safemode 修复` in CI.

# Ploopyco Firmware History

Notable changes to the Ploopyco keyboards made on top of upstream QMK.

## 2026-10-03 — Host-aware drag scroll sensitivity (A+)

**Symptom:** On macOS/iOS the A+ drag scroll was far more sensitive than the
Ploopy Adept — roughly 8x more horizontal and 27x more vertical scroll for the
same ball movement.

**Cause:** The A+ advertises a high-resolution scroll wheel
(`WHEEL_EXTENDED_REPORT` + `POINTING_DEVICE_HIRES_SCROLL_ENABLE`) and sets
drag-scroll divisors of `1.0`/`0.3`, tuned for the Windows/Linux Resolution
Multiplier path. macOS/iOS ignore that multiplier, so the same divisors produce
much larger scroll amounts there. The Adept uses the `8.0`/`8.0` defaults.

**Change:**

* Drag-scroll sensitivity is exposed through the weak runtime hooks
  `ploopy_dragscroll_divisor_h()` / `ploopy_dragscroll_divisor_v()`, which
  default to the existing `PLOOPY_DRAGSCROLL_DIVISOR_*` macros. Every other
  Ploopy device is unaffected.
* The A+ keymap overrides these hooks to keep the compact divisors on
  Windows/Linux, and to fall back to `PLOOPY_DRAGSCROLL_DIVISOR_FALLBACK_H` /
  `PLOOPY_DRAGSCROLL_DIVISOR_FALLBACK_V` (default `8.0`, matching the Adept) on
  hosts that do not use the high-resolution scroll multiplier.

**Files changed:**

* `keyboards/ploopyco/ploopyco.c`, `keyboards/ploopyco/ploopyco.h` — weak
  drag-scroll divisor hooks used by `pointing_device_task_kb()`.
* `keyboards/ploopyco/aplus/config.h` — added
  `PLOOPY_DRAGSCROLL_DIVISOR_FALLBACK_H` / `PLOOPY_DRAGSCROLL_DIVISOR_FALLBACK_V`.
* `keyboards/ploopyco/aplus/keymaps/default/keymap.c` — host-aware divisor
  overrides using `detected_host_os()`.
* `keyboards/ploopyco/readme.md` — documented the fallback defines.

**Status:** prototype on branch `aplus-dragscroll-fix`, verified on hardware.
Builds for `ploopyco/aplus/rev2_001:default`.

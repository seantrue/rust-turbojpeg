# AI Contributions — rust-turbojpeg (seantrue fork)

*About AI slop.* Fair enough: the one change this fork carries was written by a machine. Call it
instant ramen: fast, cheap, genuinely edible, and nobody should pretend it simmered on a stove all
afternoon. The honest answer is the record below: what the maintainer asked for, what the model
wrote, and what went wrong, including defects in AI-written code that building and running the
dependent apps on a real Windows machine caught rather than luck. Bon appétit, but mind the sodium
content.

Tracks what AI helpers contributed to this fork versus the human maintainer.

## Human maintainer

rust-turbojpeg is Jan Špaček's project (114 commits from 2021-03-20 up to the turbojpeg-sys 1.2.0
release, 93 of them his, with other contributors), none of it AI work. This fork exists only to
carry an MSVC C run-time fix for the maintainer's projects (impressionist-rs, virtimp_com_app),
which consume `turbojpeg-sys` through `[patch.crates-io]`. The maintainer chose the fork-and-patch
approach over waiting for the same fix upstream (honzasp/rust-turbojpeg#43, a third party's).

## Contributions

### 2026-11-09 — turbojpeg-sys: libjpeg-turbo built with Rust's C run-time on MSVC (Claude Opus 5.5)

**What the maintainer asked for.** "Address the other open issues", one of which was the linker
warning `LNK4098: defaultlib 'LIBCMT' conflicts with use of other libs` in virtimp_com_app's
desktop app on Windows 11 ARM64; then, of the options offered, "Fork + patch".

**Cause.** libjpeg-turbo's CMake build uses the static C run-time (`/MT`, `LIBCMT`) on MSVC unless
`WITH_CRT_DLL` is on, and `turbojpeg-sys` never turned it on. Rust links the DLL run-time (`/MD`,
`MSVCRT`) unless the target has `crt-static`, and so did every other C library in the build
(freetype, libpng, zlib, stacker; read with `dumpbin /directives`). Two run-times in one binary
work until memory or handles cross between them.

**What the AI did.** `turbojpeg-sys/build.rs` (~11 LOC), on branch `msvc-crt-dll` from `05da705`
(the commit `turbojpeg-sys` 1.2.0 was published from, per its `.cargo_vcs_info.json`), so the
patched crate is that release plus this change: on MSVC, `WITH_CRT_DLL=ON` and
`CMAKE_MSVC_RUNTIME_LIBRARY` set to `MultiThreadedDLL`, or `MultiThreaded` under `crt-static`.
The same change as #43.

**Verified.** On the ARM64 VM (MSVC 14.44, HostARM64), virtimp_com_app's desktop app built with
this `build.rs` as a local `[patch.crates-io]`: no `LNK4098`; `dumpbin /directives` on
`jpeg-static.lib` and `turbojpeg-static.lib` now shows `MSVCRT` where it showed `LIBCMT`; and
`virtimp-desktop render` rendered Cartoon #1 on rustcpu. Not built with `crt-static`, nor on
x86_64 Windows; non-MSVC targets skip the new block.

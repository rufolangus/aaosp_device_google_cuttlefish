# AAOSP overlay — device/google/cuttlefish

This repository is an **orphan-root snapshot** of the AAOSP overlay on AOSP
`device/google/cuttlefish`. Corresponds to the `device/google/cuttlefish`
project in the [AAOSP umbrella](https://github.com/rufolangus/AAOSP)
repo-manifest.

## Base

AOSP, upstream commit `c8b2053b7b8e21904d5437bbd9c3d0d1f7cd9806`.

## AAOSP change

**One line removed** from `vsoc_x86_64/phone/aosp_cf.mk`:

```makefile
-PRODUCT_ENFORCE_ARTIFACT_PATH_REQUIREMENTS := relaxed
```

Removing that line (rather than leaving `:= relaxed` in place) lets
the AAOSP system-image build pass the artifact-path requirements
check for `system/lib64/libllm_jni.so`. Confirmed load-bearing by
v0.5.1 testing — restoring upstream (`:= relaxed`) caused `m -j32`
to fail at the Make-setup phase:

```
build/make/core/artifact_path_requirements.mk:31:
  warning: device/google/cuttlefish/vsoc_x86_64/phone/aosp_cf.mk
  produces files inside build/make/target/product/generic_system.mk
  artifact path requirement.
Offending entries: system/lib64/libllm_jni.so
```

The pre-v0.5.1 AAOSP commit also touched `shared/device.mk` with a
`PRODUCT_COPY_FILES` line pushing the 0.5B GGUF to
`/data/local/llm/`; that was vestigial (v0.5 bakes the 3B GGUF into
`/product/etc/llm/` via `aaosp_platform_build`) and has been
dropped. Net AAOSP delta is one line.

## Why orphan-root

Build VM uses `repo sync`, which creates shallow clones. Pushing to
GitHub with ancestry preserved would require unshallowing against
AOSP `googlesource.com` (hundreds of MB of upstream history) for a
one-line delta — not worth it. The snapshot is the smallest honest
artifact.

To diff against upstream: clone AOSP at the base commit and compare
`vsoc_x86_64/phone/aosp_cf.mk`.

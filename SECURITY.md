# Security Policy

`Aladdin-R-D/lwext4` is a maintained fork of [gkostka/lwext4](https://github.com/gkostka/lwext4),
an ext2/ext3/ext4 filesystem library intended for embedded and microcontroller targets.

---

## Supported versions

Only the latest release is supported. Security fixes are merged into `master`
and published as a new patch release; there is no backport branch and older
tags do not receive fixes.

| Version | Supported |
| --- | --- |
| [latest release](https://github.com/Aladdin-R-D/lwext4/releases) | ✅ Yes |
| `master` | ✅ Yes - fixes land here first, before the next release |
| Any earlier tag | ❌ No - upgrade to the latest release |
| Upstream `gkostka/lwext4` | ❌ Not maintained here (see *Upstream coordination*) |
| Other forks / vendored copies | ❌ No |

If you ship this library inside a product, pin a release tag or a commit hash
and track new releases for security fixes.

## Threat model and scope

### In scope

The primary trust boundary is **untrusted filesystem data**: an ext2/3/4 image,
partition, SD/MMC card or USB mass-storage device supplied by an attacker and
mounted, scanned or repaired by an application built on this library.

Issues reachable through that boundary are in scope, including:

* memory corruption while parsing the superblock, block group descriptors, inodes,
  extent trees, directory entries or the HTree directory index;
* out-of-bounds reads/writes, use-after-free, double free, type confusion and
  integer overflows in `src/` and `include/`;
* defects in extended-attribute (`ext4_xattr.c`) and extent (`ext4_extents.c`) handling;
* journal replay and recovery issues, including crafted or inconsistent journals;
* unbounded allocations or recursion driven by image-controlled values
  (block size, depth, counts), where they lead to memory corruption or a
  reliable crash of the host application;
* infinite loops or deadlocks triggered by crafted metadata;
* defects in the block-device layer (`blockdev/`) and in the `lwext4-generic` /
  `lwext4-mkfs` tools under `fs_test/`, when driven by untrusted input;
* build-configuration-dependent issues (any combination of `CONFIG_*` options in
  `include/ext4_config.h`) - please state the configuration you used.

### Out of scope

* Integrator error: calling the public API with invalid arguments, unsynchronised
  access from multiple threads where the configuration does not enable locking,
  or passing a block device that violates the documented contract.
* Attacks that require the attacker to already have arbitrary code execution or
  write access to the library's own memory.
* Findings produced solely by an automated scanner or an LLM, with no root-cause
  analysis and no reproducer.
* Issues that only reproduce in a fork, a patched tree or a vendored copy that is
  not this repository.
* Purely theoretical weaknesses with no demonstrated impact, and missing
  compiler hardening flags in downstream build systems.

## How to report

**Do not open a public issue, pull request or discussion for a security defect,
and do not disclose it publicly before the coordination process below has run.**

Reporting channel:

1. **GitHub private vulnerability reporting** (preferred) -
   [open a private report](https://github.com/Aladdin-R-D/lwext4/security/advisories/new).
   This keeps the report, the draft advisory, the patch review and the CVE request
   in one place and gives you an audit trail.

## What to include in a report

A report that can be reproduced in one attempt is triaged and fixed far faster.
Please include as much of the following as you have:

* the release tag or the commit hash of `master` you tested, and the exact build configuration
  (CMake toolchain, `CONFIG_*` flags, sanitizers, target architecture, endianness);
* a minimised reproducer - the ext2/3/4 image or metadata blob, compressed,
  plus the exact command line (for example `lwext4-generic -i <image>`);
* the harness, if the issue was found by fuzzing (AFL++, libFuzzer, honggfuzz),
  and whether it was built with ASan/UBSan/MSan;
* the sanitizer or debugger output: the full ASan/UBSan report or the backtrace,
  with the faulting function and the source line;
* your assessment of the defect class (CWE) and of the impact - read vs. write,
  attacker control over the offset and the written value, reachability from a
  typical integration;
* any proposed patch, if you have one - patches are welcome but not required;
* whether you have already reported the issue elsewhere (upstream, oss-security,
  a CNA, a distribution), and any embargo dates you are already bound by.


## Coordinated disclosure

* The default embargo is **90 calendar days** from the acknowledgement of the
  report, or until a fix is published — whichever comes first.
* We ask reporters not to publish details, reproducers or harnesses before the
  embargo ends. We will not ask for an indefinite extension; if we need more time,
  we will propose a specific new date and a reason.
* The embargo may be shortened by mutual agreement, and will be shortened if the
  issue is being exploited in the wild or is independently disclosed by a third party.
* Publication happens as a GitHub Security Advisory (GHSA) on this repository,
  together with the fixing commit and the release that contains it. Release notes
  reference the advisory, so that integrators can tell a security release from a
  functional one.
* **CVE:** we will request a CVE identifier for any issue with a security impact on
  the library. If you have already obtained a CVE for the issue, tell us the ID
  and we will reference it instead of requesting a duplicate.
* **Credit:** reporters are credited by name or handle in the advisory, with a
  link of their choice, unless they ask to remain anonymous. Tell us how you want
  to be named. This project does not operate a bug bounty and offers no monetary
  reward.

## Upstream coordination

This repository is a fork. Most of the code is shared with
[gkostka/lwext4](https://github.com/gkostka/lwext4) and with downstream consumers
of it, so a defect found here usually affects them too.

* If the issue reproduces upstream, we will notify the upstream maintainer and
  offer the patch, and we will say so in the advisory.
* Please do not open a public upstream issue for the same defect while the embargo
  is in effect; if you would rather drive the upstream report yourself, tell us and
  we will coordinate timing with you rather than duplicating the work.
* Where the fix is a divergence specific to this fork rather than an upstream bug,
  the advisory will state that clearly.

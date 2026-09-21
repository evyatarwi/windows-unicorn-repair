# Windows Unicorn Repair 🦄

A Windows 11 repair-media research project exploring how Windows Update repair content is staged, captured, analyzed, and transformed into validated Windows 11 Pro installation media for controlled recovery testing.

> **Status:** ISO build and structural validation completed. Runtime boot, clean-install, and in-place repair testing are still pending.

---

## Why "Windows Unicorn"?

Windows can offer a **repair version** through Windows Update that reinstalls the current version of Windows while preserving applications, files, and settings.

That workflow is extremely useful for IT recovery — but the downloaded repair content is not exposed as a reusable ISO.

This project started with a simple question:

> If Windows Update downloads a repair version, can we preserve it, understand how it works, and turn that knowledge into reusable repair media?

The answer was much more complicated than simply finding an ISO in the Windows Update cache.

---

## Project Result

The investigation produced Windows 11 Pro installation media with the following target image:

| Property | Result |
|---|---|
| Edition | Windows 11 Pro |
| Architecture | x64 |
| Language | en-US |
| Release | Windows 11 25H2 |
| Build | 26200.9457 |
| Integrated update | KB5129195 |
| Image format | install.wim |
| ISO packaging | Microsoft Oscdimg |
| Boot entries | BIOS + UEFI |

The resulting image was validated using:

- DISM image metadata
- Offline registry inspection
- Package-state verification
- SHA256 comparison of key ISO contents
- Microsoft Oscdimg exit-code validation
- Embedded `install.wim` verification

Runtime validation has **not yet been completed**.

---

## What We Captured

During an active Windows Update repair operation, available repair files were preserved before Windows could clean them up.

The final capture contained approximately:

**664,474 files / ~37.36 GiB**

The captured environment included:

- ESD files
- CAB packages
- Component WIM files
- CompDB metadata
- Setup staging files
- Windows Update packages
- Expanded servicing content

The important discovery was that the repair cache was **not simply a hidden Windows ISO**.

It contained servicing and staging material used by Windows Setup and Windows Update, but not a ready-made generic installation-media structure.

---

## Experimental Reconstruction

An initial attempt was made to reconstruct installation media directly from the captured repair payload using the third-party `uup-converter-wimlib` toolchain.

During processing, Microsoft Defender detected:

```text
Trojan:Win32/Commando.A!ml
```

The detection occurred during dynamic PowerShell execution associated with PSF decompression.

The build was stopped immediately.

No Defender exclusion was created, and no artifact from that interrupted build was accepted as trusted installation media.

The project then moved to a workflow based on official Windows installation media and Microsoft servicing tools.

---

## Successful Build Method

The successful workflow used:

1. Official Windows 11 consumer installation media
2. Windows 11 Pro exported from the multi-edition `install.wim`
3. Microsoft DISM for offline servicing
4. Official Microsoft KB5129195 update package
5. Microsoft Windows ADK Deployment Tools
6. Microsoft Oscdimg for ISO creation
7. Post-build image, registry, package, and hash verification

The resulting operating-system image reports:

```text
Edition:      Windows 11 Pro
Architecture: x64
Language:     en-US
Release:      25H2
Version:      10.0.26200.9457
```

---

## High-Level Workflow

```text
Windows Update repair appears
            │
            ▼
Capture live repair files
            │
            ▼
Analyze ESD / CAB / WIM / CompDB content
            │
            ▼
Attempt direct reconstruction
            │
            ├── Defender detection
            │       │
            │       ▼
            │      STOP
            │
            ▼
Switch to official Windows installation media
            │
            ▼
Export Windows 11 Pro
            │
            ▼
Integrate KB5129195 with DISM
            │
            ▼
Validate offline image
            │
            ▼
Package with Microsoft Oscdimg
            │
            ▼
Validate ISO contents and hashes
            │
            ▼
Controlled runtime testing
```

---

## Validation Status

| Validation | Status |
|---|---|
| Source ISO identity | ✅ |
| Microsoft update signature | ✅ |
| Windows 11 Pro export | ✅ |
| KB5129195 integration | ✅ |
| DISM image verification | ✅ |
| Offline registry verification | ✅ |
| Package-state verification | ✅ |
| BIOS boot files present | ✅ |
| UEFI boot files present | ✅ |
| Oscdimg exit code | ✅ |
| ISO content SHA256 comparison | ✅ |
| Final ISO SHA256 recorded | ✅ |
| VM boot test | ⏳ Pending |
| Clean installation | ⏳ Pending |
| In-place repair | ⏳ Pending |

---

## Repository Structure

```text
windows-unicorn-repair/
│
├── README.md
├── CHANGELOG.md
├── .gitignore
│
├── docs/
│   ├── PROJECT-HISTORY.md
│   ├── BUILD-GUIDE.md
│   ├── VALIDATION.md
│   ├── TROUBLESHOOTING.md
│   └── screenshots/
│
└── scripts/
```

Detailed technical documentation will be maintained under [`docs`](docs/).

---

## Safety and Scope

This repository does **not** distribute Microsoft Windows installation media.

It is intended to contain:

- Documentation
- PowerShell automation
- Build procedures
- Validation logic
- Research findings
- Screenshots and non-proprietary evidence

It does **not** contain:

- Windows ISO files
- WIM or ESD images
- Windows Update payloads
- Microsoft binaries
- Product keys
- Activation data
- Captured user data

Windows installation media and updates must be obtained from legitimate Microsoft sources.

---

## Important Limitations

This project does **not** claim to reproduce Microsoft's Windows Update repair mechanism exactly.

The successful artifact is customized official Windows installation media created for the same broad repair objective.

For the documented build:

- `install.wim` was updated to build 26200.9457
- Windows Setup remained based on the official source media
- `boot.wim` remained based on the official source media
- WinRE remained at its source-media servicing level
- Captured .NET and Dynamic Update packages were not independently integrated

Matching the Windows build does not guarantee that this media will resolve every issue that Microsoft's own Windows Update repair workflow could address.

---

## Intended Use

The resulting ISO is intended for **controlled repair testing** on compatible Windows 11 Pro systems.

For an in-place repair, Windows Setup should be launched from inside the running Windows installation.

Before continuing, verify compatibility of:

- Edition
- Architecture
- Installation language
- Windows build

The Setup option:

**Keep personal files and apps**

must be available before proceeding with an in-place repair.

---

## Next Milestones

- Test UEFI boot in a disposable VM
- Perform a clean Windows 11 Pro installation
- Verify installed edition and build
- Test Windows Update after installation
- Test on approved physical hardware
- Perform a backed-up in-place repair
- Convert historical scripts into reusable public scripts
- Add screenshots and validation evidence
- Publish the first tested release

---

## Project Philosophy

This project prioritizes **validation over simply producing an ISO**.

A file ending in `.iso` is not automatically trusted installation media.

The validation chain is:

```text
Source
  ↓
Servicing
  ↓
Image identity
  ↓
Package state
  ↓
Offline registry
  ↓
ISO packaging
  ↓
Embedded-content verification
  ↓
Runtime testing
```

Runtime testing is the next milestone.

---

## Disclaimer

Windows, Windows 11, DISM, Windows Update, Windows ADK, and related Microsoft technologies are products or trademarks of Microsoft Corporation.

This is an independent research and IT engineering project.

It is not affiliated with, endorsed by, or certified by Microsoft.

Use customized installation or repair media only on systems you are authorized to administer, and maintain an appropriate backup before operating-system servicing.

---

**Windows Unicorn Repair** 🦄  
*Researching the repair path Windows normally keeps hidden.*
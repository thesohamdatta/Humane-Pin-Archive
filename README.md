# Let's Revive the Humane AI Pin

The company is gone, but the hardware is still worth understanding.

The Humane AI Pin was a screenless wearable computer built around a Qualcomm Snapdragon 720G, a laser-scanning projector, a camera, a depth sensor, cellular connectivity, and a small internal battery. Much of its assistant functionality depended on network services rather than local model inference.

I started this archive to keep the useful technical knowledge in one place. The details are scattered across teardowns, FCC filings, patents, Humane documentation, firmware work, and community reverse engineering. This README keeps the parts that can be tied to evidence and leaves uncertain details marked as such.

## What it is

The Pin is a two-piece wearable.

The outward-facing Pin contains the computer, sensors, projector, audio hardware, radios, and an internal battery. A magnetically attached Battery Booster sits behind clothing and sends power to the Pin wirelessly. The internal battery bridges brief Booster swaps.

The main unit is about **47.5 × 44.5 × 15 mm** and **34.2 g**. The Battery Booster is about **47.2 × 45.2 × 8.3 mm** and **20.5 g**.

The device has no conventional display. Its main visual output is a **720p Laser Ink Display** built around laser beam scanning and MEMS optics. Its input methods include a capacitive touch surface, voice, camera input, and depth sensing.

## Hardware

Only the components that are useful for understanding the system are listed here.

| Part | Identifier | Role |
|---|---|---|
| Qualcomm SoC | **SM7125-100-AB** | Main application processor / platform |
| Memory + storage | **Kingston 32EM32-M4DTX29** | 4 GB LPDDR4X + 32 GB flash |
| Main PMIC | **PM6250-102** | Primary system power management |
| Secondary PMIC | **PM6150A-102** | Companion power management |
| Wireless power receiver | **STWLC86** | Receives power from the Battery Booster |
| System MCU | **STM32F410TB** | Low-level control / peripheral management |
| Touch MCU | **CY8C4045FNI-S412T** | Capacitive touch sensing |
| Wi-Fi / Bluetooth | **WCN3988-000** | Wireless connectivity |
| Cellular RF transceiver | **SDR675-005** | Cellular RF path |
| Cellular PA | **SKY77643-61** | Cellular transmit amplification |
| Cellular diversity module | **SKY53735-61** | Receive-side RF path |
| RF switch | **SKY13351-378LF** | Antenna-path switching |
| Audio amplifier | **WSA8810-0VV** | Speaker amplification |
| LED driver | **LED1202** | Trust Light / status LEDs |
| Magnetometer | **MMC5633NJL** | Magnetic sensing |
| Projector boost converter | **MP3438GTL** | Projector power conversion |
| Projector current monitor | **INA280A1QDCKRQ1** | Projector current monitoring |

The iFixit chip-identification teardown explicitly identifies the Snapdragon SM7125-100-AB and Kingston 32EM32-M4DTX29, along with additional board-level components. PenumbraOS documentation provides the broader hardware picture.

The Pin also contains a 6-axis STMicroelectronics accelerometer/gyroscope, an ambient-light sensor, a 13 MP ultrawide camera, and an indirect-ToF depth sensor. Public hardware documentation lists the camera at **13 MP / 120° FOV / f/2.4** and the depth sensor at **640 × 480 / 125° FOV**.

The exact camera sensor, ToF die, and IMU part numbers are not established by the public documentation used here.

### Projector

The projector is a separate optical/electronic subsystem rather than a normal Android display panel.

PenumbraOS documents a **720p, 498 nm green laser projector** using MEMS scanning. Its teardown documentation describes a separate projector board with dedicated control electronics, including a Lattice FPGA and an STM32F412-class MCU.

Some research reports give additional exact optical measurements. I am not treating those numbers as settled unless they can be tied directly to primary or teardown evidence.

## Inside the Pin

The physical design is a dense stack of boards and modules.

- The main logic board carries the Snapdragon platform, memory, power-management parts, RF hardware, audio electronics, and sensors.
- The projector sits on a separate board connected by flex.
- The Battery Booster carries its own battery and wireless-power transmitter.
- The Pin receives power through the STWLC86 wireless receiver.
- The internal battery is integrated into the device rather than being a conventional user-swappable pack.

FCC filings provide the regulatory record and internal photographs for the HU0123 Pin.

## How it works

A simplified view of the original system is:

**Touch / voice / camera / depth**
→ local sensor and signal processing
→ cellular or Wi-Fi
→ Humane services and external AI services
→ response
→ speaker / laser projector / LEDs

The Snapdragon platform handled the local operating system, media and sensor plumbing, networking, and other device work. Heavy assistant processing was largely remote.

The Pin's interaction model was deliberately different from a phone. The user could interact through touch and voice, use the camera for visual context, and use the projected interface rather than looking at an LCD. The depth sensor was part of the spatial interaction system.

## Software

Humane called the operating system **CosmOS**. Public reverse-engineering evidence identifies it as an Android-based system running on Qualcomm's **atoll** platform.

A PenumbraOS reverse-engineering artifact records an **Android 12** build with a **Linux 4.14.190-perf** kernel. The complete vendor/framework split, boot-chain behavior, security policy, and proprietary service layout are not documented publicly well enough to present every detail as settled fact.

The important architectural point is that CosmOS was not a normal phone UI with a conventional app launcher. Humane built a system around assistant services, hardware interfaces, connectivity, and cloud-backed interaction.

### What ran where

**On the Pin**

- touch and sensor handling
- camera and depth acquisition
- audio capture and low-level processing
- networking
- projector and speaker output
- Android/CosmOS system software

**In the cloud**

- major speech and language processing
- assistant request handling
- external information services
- much of the vision and contextual processing

The public reverse-engineering record supports a cloud-heavy design. It does **not** support the stronger claim that every useful computation was remote or that the Snapdragon hardware had no local processing beyond basic I/O.

## After Humane

The original service-dependent product stopped being a normal supported device after Humane shut down its backend.

The Pin was designed around account services, network access, and Humane-controlled software infrastructure. When those services disappeared, the original interaction loop could no longer operate normally.

That is why preservation work is not just about recovering an Android device. It is about separating the physical machine from the services it was built to depend on.

## Revival

The community has done substantial reverse engineering, but the revival state should not be oversold.

### PenumbraOS

PenumbraOS is the main open-source restoration effort. Its current organization describes the project as a jailbreak restoration of the Humane Ai Pin and says the restoration stack uses runtime patches plus a clone of parts of the Humane gRPC backend.

`pinitd` provides a custom rootless init system and uses **CVE-2024-31317** to obtain privileged Android app-level identities and SELinux domains. Its own documentation calls the project experimental.

`system-injector` uses **CVE-2024-34740** to install packages with system/platform-level characteristics. Its documentation describes the exploit path and notes limitations around SELinux and app launch behavior.

`humane-system-hook` patches Humane applications and redirects their API traffic to a local server implementation. This is the key bridge between the original software and community-hosted services.

`mabl` is the current PenumbraOS launcher/orchestrator project. Its README describes pluggable LLM, STT, and TTS providers, tool calling, conversation persistence, and experimental laser-ink UI rendering. It is explicitly marked experimental.

### Hardware access

Community work has also documented an interposer for the Pin's rear USB test pads. The documented pad order is **VCC, D-, D+, GND** in the orientation specified by the project. The original interposer documentation notes that bare USB access initially produced an unauthorized ADB state, so hardware access and ADB authorization are separate problems.

PenumbraOS also maintains a newer interposer project and a custom ADB client for devices where normal ADB authentication is a problem.

The useful distinction is simple:

**hardware access exists → software access can be established → the original cloud services still need replacement or interception.**

That is the core of the revival work.

## Patents

Humane's patents are useful for understanding the ideas behind its wearable computing system, but they should not be read as a production parts list.

**US10924651B2**, *Wearable multimedia device and cloud computing platform with application ecosystem*, describes a wearable multimedia device coupled to a cloud platform for processing captured context data.

**US12230029B2**, *Wearable multimedia device and cloud computing platform with laser projection system*, describes a wearable system involving sensing, gesture/context data, and laser projection. The patent record lists Imran A. Chaudhri and Bethany Bongiorno among the inventors and Humane as the original assignee.

These patents explain design concepts and intended system behavior. They are not proof that every described implementation shipped unchanged in the final hardware.

## What we still do not know

Several details remain genuinely uncertain in public sources:

- the exact camera sensor part number
- the exact ToF sensor part number
- the exact STMicroelectronics IMU part number
- the complete projector optical bill of materials
- the full Battery Booster cell specification
- the complete boot/security chain as implemented on every production firmware revision
- the exact division of work between Qualcomm DSPs, Android services, and cloud services
- the full proprietary CosmOS service architecture
- which revival features are stable across all firmware revisions

Unknown is more useful than an invented number.

## Technical references

- [iFixit: Humane AI Pin Chip ID](https://www.ifixit.com/Guide/Humane%2BAI%2BPin%2BChip%2BID/172518)
- [iFixit: Humane AI Pin](https://www.ifixit.com/Device/Humane_AI_Pin)
- [FCC ID 2BAFM-HU123](https://fccid.io/2BAFM-HU123)
- [PenumbraOS hardware documentation](https://penumbraos.com/about/hardware/)
- [PenumbraOS teardown](https://penumbraos.com/about/hardware/teardown/)
- [PenumbraOS](https://github.com/PenumbraOS)
- [pinitd](https://github.com/PenumbraOS/pinitd)
- [system-injector](https://github.com/PenumbraOS/system-injector)
- [humane-system-hook](https://github.com/PenumbraOS/humane-system-hook)
- [mabl](https://github.com/PenumbraOS/mabl)
- [AI Pin interposer](https://github.com/agg23/ai-pin-interposer)
- [US10924651B2](https://patents.google.com/patent/US10924651B2/en)
- [US12230029B2](https://patents.google.com/patent/US12230029B2/en)

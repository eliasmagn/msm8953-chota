# Project Concept

This repository tracks mainline enablement work for Qualcomm MSM8953-based
platforms. The goal is to upstream or stage kernel changes that improve power
management, peripheral support, and day-to-day usability on these devices.

The latest change introduces a safe "charge-through" mode for the SMB charger
when the device acts as a USB OTG host. Instead of forcing the PMIC to source
VBUS when an external supply is already present, the driver now allows the PMIC
to sink current and recharge conservatively. This keeps host peripherals alive
while preventing power conflicts.

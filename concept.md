# Project Concept

This repository tracks mainline enablement work for Qualcomm MSM8953-based
platforms. The goal is to upstream or stage kernel changes that improve power
management, peripheral support, and day-to-day usability on these devices.

The latest change introduces a safe "charge-through" mode for the SMB charger
when the device acts as a USB OTG host. Instead of forcing the PMIC to source
VBUS when an external supply is already present, the driver now allows the PMIC
to sink current and recharge conservatively. The driver also tracks and restores
the previous USB path and current limit settings so that normal charging resumes
cleanly once the external supply or OTG session ends. Input current limits can
be tuned dynamically while the policy is active so hardware validation can dial
in the exact draw that keeps peripherals stable. An extcon notifier now feeds
role and VBUS changes back into the policy automatically, and a dedicated mutex
guards the update path so sysfs writes and notifiers cannot race. Device-tree
bindings document the new opt-in properties for downstream integrators.

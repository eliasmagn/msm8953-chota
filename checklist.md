# Implementation Checklist

- [x] Add sysfs controls to toggle charge-through OTG behavior and adjust the
      input current limit with live register updates.
- [x] Detect simultaneous OTG host mode and external VBUS to avoid sourcing
      power while allowing conservative charging.
- [x] Track and restore the previous USB sink state and input current limit
      after exiting charge-through mode, reacting to extcon notifier updates.
- [x] Guard charge-through policy updates with a dedicated mutex so sysfs and
      notifier contexts cannot race the hardware sequencing.
- [ ] Validate the new policy on real hardware under various host peripherals.
- [x] Document the device-tree bindings for the new DT properties upstream.

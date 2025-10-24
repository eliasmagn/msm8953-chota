# Implementation Checklist

- [x] Add sysfs controls to toggle charge-through OTG behavior and adjust the
      input current limit.
- [x] Detect simultaneous OTG host mode and external VBUS to avoid sourcing
      power while allowing conservative charging.
- [ ] Validate the new policy on real hardware under various host peripherals.
- [ ] Document the device-tree bindings for the new DT properties upstream.

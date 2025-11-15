# Implementation Checklist

- [x] Add sysfs controls to toggle charge-through OTG behavior and adjust the
      input current limit with live register updates.
- [x] Replace per-event extcon toggles with a debounced combined decision
      table that selects between host-only, sink-only, charge-through, or idle
      behaviour on every notification.
- [x] Track and restore the previous USB sink state and input current limit
      after exiting charge-through mode so normal charging resumes cleanly.
- [x] Guard charge-through policy updates with a dedicated mutex so sysfs and
      notifier contexts cannot race the hardware sequencing.
- [x] Disable the opposing power path before enabling the requested role to
      prevent brownouts when switching between sink and source states.
- [x] Re-evaluate the combined policy when the OTG regulator is disabled so
      charge-through sessions remain in sink mode while VBUS is present.
- [x] Clamp regulator disable handling to sink/idle outcomes so consumers cannot
      inadvertently re-enable OTG sourcing when requesting a shutdown.
- [x] Emit human-readable OTG policy transition logs to simplify hardware bring-up.
- [x] Initialise the policy bookkeeping at probe so charge-through starts from a
      known "no cable" state.
- [x] Route USB source and ID IRQ notifications through the debounced worker to
      avoid mixed timing paths.
- [x] Drive all delayed policy evaluations through a shared debounce constant so
      future tweaks stay consistent across notification sources.
- [x] Notify the USB power-supply instance on sink and charge-through transitions
      to keep user space aligned with policy changes.
- [x] Trigger power-supply notifications after releasing the policy mutex and
      rely on the debounced worker for USB-source IRQ updates to avoid
      duplicate events and lock inversion risks.
- [ ] Validate the new policy on real hardware under various host peripherals.
- [x] Document the device-tree bindings for the new DT properties upstream.

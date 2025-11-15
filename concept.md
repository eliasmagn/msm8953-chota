# Project Concept

This repository tracks mainline enablement work for Qualcomm MSM8953-based
platforms. The goal is to upstream or stage kernel changes that improve power
management, peripheral support, and day-to-day usability on these devices.

The latest change replaces the ad-hoc OTG charge-through hooks with a unified
state machine inside the SMB charger driver. Whenever extcon reports a host or
VBUS transition, a short debounce window coalesces the events and the driver
computes the combined state (host-only, sink-only, charge-through, or idle) in a
single critical section. The selected policy disables the opposing power path
before enabling the required one and, in charge-through mode, restores the
previous USB sink state and current limit once the session ends. Input current
limits remain tunable at runtime, allowing validation teams to balance draw
against hub stability while observing the new behaviour. Device-tree bindings
still describe the opt-in properties so downstream integrators can ship sensible
defaults without losing the runtime controls.

The latest refinements keep the policy decision central: even manual OTG regulator disable calls now rerun the combined evaluation so an attached charger stays active, and every transition is logged to dmesg so bring-up teams can verify the charge-through mode chosen in the field. The chip state now starts from an explicit "no cable" baseline at probe time, and both the extcon notifier and hardware IRQ paths funnel through the same debounced worker, preventing ordering races while keeping the logs authoritative. The regulator disable hook now constrains the recomputed policy to sink-or-idle outcomes so consumers cannot inadvertently re-enable VBUS sourcing while trying to turn the supply off, the shared debounce constant keeps every policy update on the same timing, and each mode transition proactively notifies the USB power-supply instance so user space immediately sees sink and charge-through transitions.

Notifications now fire only after the policy mutex has been released, and the USB source detect IRQ relies solely on the shared worker, eliminating lock inversion risks and duplicate change events while preserving user space visibility. The policy helper now surfaces hardware write failures back to the regulator enable/disable paths, ensuring OTG consumers see proper errors if sourcing or sinking transitions cannot be applied. Regulator status queries mirror that behaviour by propagating regmap read failures to callers instead of silently claiming the rail is off, so diagnostics can distinguish a missing cable from a bus access fault.

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

The latest refinements keep the policy decision central: even manual OTG regulator disable calls now rerun the combined evaluation so an attached charger stays active, and the resulting `OTG policy: old -> new` logs are ratelimited so noisy cables cannot flood dmesg while still giving bring-up teams authoritative breadcrumbs. The chip state now starts from an explicit "no cable" baseline at probe time, and both the extcon notifier and hardware IRQ paths funnel through the same debounced worker, preventing ordering races while keeping the logs authoritative. The regulator disable hook now constrains the recomputed policy to sink-or-idle outcomes so consumers cannot inadvertently re-enable VBUS sourcing while trying to turn the supply off, the shared debounce window keeps every policy update on the same timing, and each mode transition proactively notifies the USB power-supply instance so user space immediately sees sink and charge-through transitions. Boards can also tune that debounce interval at runtime via the new `otg_policy_debounce_ms` sysfs attribute or set a static default with the `qcom,otg-policy-debounce-ms` device-tree property (clamped to 50–500 ms) so slower extcon pairings remain stable without kernel rebuilds.

Notifications now fire only after the policy mutex has been released, and the USB source detect IRQ relies solely on the shared worker, eliminating lock inversion risks and duplicate change events while preserving user space visibility. When a policy transition fails part-way through, the driver now attempts a best-effort rollback to the previous mode so neither power path is left half-switched, and the policy helper surfaces hardware write failures back to the regulator enable/disable paths so OTG consumers see proper errors if sourcing or sinking transitions cannot be applied. Regulator status queries mirror that behaviour by propagating regmap read failures to callers instead of silently claiming the rail is off, and the battery info cache acquired from `power_supply_get_battery_info()` is devm-managed so probe failures and remove paths automatically release the allocation. The extcon notifier registration now uses the devm helpers too, ensuring probe errors unwind cleanly without leaking callbacks, while every delayed-work reschedule snapshots the runtime debounce window with `READ_ONCE()` so IRQ and notifier contexts all apply the latest knob value without locking. Hot-path OTG regulator and charge-through warnings are ratelimited to keep noisy hubs from spamming dmesg during stress tests.

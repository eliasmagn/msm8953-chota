# MSM8953 Mainline Staging Kernel

This tree contains staging work to support Qualcomm MSM8953 devices on recent
mainline Linux kernels. Each change focuses on improving hardware enablement or
adding quality-of-life features needed for day-to-day use.

## Charge-through OTG Safe Mode

The Qualcomm SMB charger driver (`drivers/power/supply/qcom-smbchg.c`) now
drives a debounced state machine that evaluates the combined extcon host/VBUS
signals whenever anything changes. Rather than reacting independently to each
cable notification, the driver waits briefly for the pair of events, computes a
single mode (host-only, sink-only, charge-through, or idle), and then applies it
by disabling the opposing power path before enabling the requested one. When the
device is acting as an OTG host while external VBUS is present and the
charge-through policy is allowed, the state machine selects the sink path and
sets a conservative current limit so the battery can recharge without disrupting
the connected hub.

Manual OTG regulator disable requests now rerun the same policy evaluation
instead of forcing the charger idle, so an attached power supply keeps feeding
the phone whenever VBUS remains present. The disable handler constrains that
re-evaluation to sink-or-idle results, ensuring a host-only scenario cannot
silently re-enable VBUS sourcing after a consumer requests the regulator be
turned off. Transitions are logged via a ratelimited
`OTG policy: <old> -> <new>` message to make
validation runs easier to audit, and the USB source detect / ID change IRQ
paths now feed into the same debounced worker as the extcon notifier so every
state change follows a single timing model driven by a shared debounce
constant. When the mode toggles between sink, host, and charge-through, the
driver also triggers `power_supply_changed()` on the USB power-supply instance
so user space immediately sees the updated charging posture. Those
notifications are deferred until after the policy mutex is released, and the
USB source detect IRQ relies exclusively on the debounced worker to avoid
duplicate user-space updates while keeping the timing model consistent.

### Runtime controls

Three sysfs attributes are exposed under the SMB charger platform device:

- `allow_charge_while_otg` – enable or disable the policy (default disabled).
- `otg_charge_icl_ua` – configure the input current limit used while sinking in
  this mode (defaults to 500 mA when unset).
- `otg_policy_debounce_ms` – adjust the shared debounce window (clamped between
  50 ms and 500 ms, defaults to 150 ms) without rebuilding the kernel.

Each attribute takes effect immediately when updated, even while an OTG session
is underway, so the sink current and debounce window can be tuned in real time.

The driver explicitly initialises its policy bookkeeping to "no cable" during
probe so early boot messages reflect the real configuration before any
notifications fire. When the policy is active, the driver records the previous
USB sink state and input current limit before switching into charge-through
mode. The debounced extcon worker keeps the state machine in sync with
role/VBUS changes, and once the external VBUS source disappears or OTG host
mode ends, those settings are restored so the charger returns to its prior
behavior without manual intervention. If a policy transition fails part-way
through, the driver now attempts a best-effort rollback to the previous mode so
the hardware is not left half-switched, and a dedicated mutex guards these
transitions so concurrent sysfs writes and notifier callbacks cannot
desynchronize the sequencing. If any of the underlying register writes fail
during these transitions, the regulator enable/disable callbacks now return the
hardware error so OTG consumers immediately know that sourcing or sinking could
not be applied. The OTG regulator status callback mirrors that behaviour by
propagating regmap read failures to its callers, which helps higher-level
consumers and diagnostics distinguish between a deliberately disabled rail and
a bus access problem.

Every delayed policy evaluation now snapshots the runtime-configurable debounce
window with `READ_ONCE()` before scheduling work so IRQs and notifiers all apply
the latest knob value without taking the mutex. The extcon notifier is
devm-managed to guarantee probe errors unwind without leaking callbacks, and
noisy regulator warnings (for OTG enable toggles and charge-through current
updates) use the ratelimited logging helpers so bouncing hubs cannot flood
dmesg during stress runs.

### Device tree properties

Boards can opt-in by setting the following properties on the SMB charger node:

```dts
qcom,allow-charge-while-otg;
qcom,otg-charge-icl-ua = <500000>; /* microamps */
qcom,otg-policy-debounce-ms = <150>; /* optional 50-500 ms */
```

If the properties are absent, the runtime sysfs controls remain available for
manual testing. Boards can either set the device-tree property up front or use
the `otg_policy_debounce_ms` sysfs knob to pick any value between 50 ms and
500 ms for slower extcon pairings without rebuilding the kernel.
The binding updates are documented in
`Documentation/devicetree/bindings/power/supply/qcom,smbchg.yaml` for downstream
integrators and upstream review.

## Contributing

Please test changes on real hardware whenever possible and document any device
specific quirks in follow-up patches.

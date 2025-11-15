# Roadmap

## Near Term
- Validate the debounced charge-through state machine on supported devices,
  covering every plug order (charger first, hub first, hot replug) while tuning
  the default current limit via the live sysfs path.
- Stress-test the delayed extcon worker so combined host/VBUS updates always
  land in the expected mode (source, sink, or charge-through) without regressions
  in OTG sourcing behaviour.
- Exercise the USB source detect and ID change IRQ paths to verify they follow
  the same debounced worker timing and never regress into per-event toggling.
- Capture the new `OTG policy: <old> -> <new>` logs during bring-up to confirm
  regulator disable requests now fall back to the correct sink/idle state in mixed
  host/charger scenarios and never re-enable sourcing when only a host cable
  remains.
- Check early boot logs to ensure the initial `OTG policy` line reports `none -> ...`
  only after real notifications arrive, confirming the explicit policy reset.
- Prepare upstream submission for the documented
  `qcom,allow-charge-while-otg` and `qcom,otg-charge-icl-ua` bindings alongside
  the driver changes.

## Mid Term
- Integrate board-specific defaults for the charge-through policy based on
  measured peripheral power requirements.
- Expand automated tests around OTG role switching and charger status updates.

## Long Term
- Continue reducing downstream patches by upstreaming MSM8953-specific power and
  peripheral drivers.
- Achieve feature parity with downstream vendor kernels for daily driver use.

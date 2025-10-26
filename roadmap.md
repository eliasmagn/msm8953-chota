# Roadmap

## Near Term
- Validate the charge-through OTG policy on supported devices and tune the
  default current limit as needed, exercising the live sysfs adjustment path.
- Stress-test the extcon notifier path so role/VBUS changes always trigger the
  mutex-guarded policy update without regressions in OTG sourcing.
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

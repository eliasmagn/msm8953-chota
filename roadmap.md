# Roadmap

## Near Term
- Validate the charge-through OTG policy on supported devices and tune the
  default current limit as needed.
- Upstream documentation for the new `qcom,allow-charge-while-otg` and
  `qcom,otg-charge-icl-ua` device-tree properties.

## Mid Term
- Integrate board-specific defaults for the charge-through policy based on
  measured peripheral power requirements.
- Expand automated tests around OTG role switching and charger status updates.

## Long Term
- Continue reducing downstream patches by upstreaming MSM8953-specific power and
  peripheral drivers.
- Achieve feature parity with downstream vendor kernels for daily driver use.

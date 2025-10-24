# MSM8953 Mainline Staging Kernel

This tree contains staging work to support Qualcomm MSM8953 devices on recent
mainline Linux kernels. Each change focuses on improving hardware enablement or
adding quality-of-life features needed for day-to-day use.

## Charge-through OTG Safe Mode

The Qualcomm SMB charger driver (`drivers/power/supply/qcom-smbchg.c`) now
supports a "charge-through" policy when the device is acting as a USB OTG host
while external VBUS is present. Instead of sourcing power onto the bus, the
charger switches to a conservative sink mode so the battery can recharge from
the external supply without disrupting connected peripherals.

### Runtime controls

Two new sysfs attributes are exposed under the SMB charger platform device:

- `allow_charge_while_otg` – enable or disable the policy (default disabled).
- `otg_charge_icl_ua` – configure the input current limit used while sinking in
  this mode (defaults to 500 mA when unset).

Both attributes take effect immediately when updated.

### Device tree properties

Boards can opt-in by setting the following properties on the SMB charger node:

```dts
qcom,allow-charge-while-otg;
qcom,otg-charge-icl-ua = <500000>; /* microamps */
```

If the properties are absent, the runtime sysfs controls remain available for
manual testing.

## Contributing

Please test changes on real hardware whenever possible and document any device
specific quirks in follow-up patches.

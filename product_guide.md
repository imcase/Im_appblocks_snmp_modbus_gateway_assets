> **Firmware:** `SNMP_Modbus_Gateway` v1.0.2  
> **Platform:** Tibbo TPP2W(G2) in a TPB2L(G2) enclosure

<br><br>

---

## 1. Overview

The SNMP–Modbus Gateway lets **SNMP monitoring software read** values from
**Modbus equipment** over **Ethernet**. It is used when equipment such as meters, sensors, 
controllers, or other industrial devices communicates with Modbus, while the
customer's monitoring system uses SNMP.

The gateway regularly reads the Modbus values selected by the user and makes
them available as SNMP values.
A monitoring system can then display the values, record them, or use them to raise alarms.

Where specifically configured and authorized, the gateway can also accept an
**SNMP write** command and send the corresponding Modbus write to the field device.
This **remote-control function** should be enabled only for approved control points.

![Modbus field device to SNMP monitoring-system gateway flow](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_flow.png)

The user configures which Modbus values to read, and how each value appears to SNMP software.
This means the same gateway can be used with different types of
Modbus equipment instead of being limited to a single sensor or meter.

<br><br>

---

## 2. Hardware

| Item | Detail |
| --- | --- |
| Controller | Tibbo TPP2W(G2) Tibbo Project PCB |
| Enclosure | TPB2L(G2), DIN-rail mounting kit, vibration-proof kit |
| Network | On-board 10/100 Base-T Ethernet with RJ45 |
| Field interface | Tibbit #05 RS-485 port for Modbus RTU |
| Field connector | Tibbit #21 (4-terminal block) or Tibbit #20 (9-terminal block) |
| Power | Tibbit #10 power supply module in socket 11 |
| Power adapter | APR-P0008 adapter |
| Storage | 1 MB AT45DB081E dataflash, LittleFS file system |
| Status LEDs | On-board red/green LEDs used for the setup-error indication |
| MD button | On-board setup (MD) button. Hold for 5 s and release to reset all settings to their defaults |
| RST button | On-board restart (RST) button |

<br><br>

---

## 3. Network configuration and services

| Item | Setting |
| --- | --- |
| Ethernet | static IP/netmask/gateway, or DHCP  |
| SNMP | SNMPv2c and SNMPv3 agent, port 161, configurable read/write communities and SNMPv3 security credentials |
| Time sync | SNTP against 216.239.35.0 (time.google.com), applied with the configured time zone |
| Device Console | HTTP, port 80, 3 concurrent connections |

<br>

> **NOTE:**
> - Out of the box, the device uses the **default** static IP address
> `192.168.1.40`. This placeholder makes the device reachable on first
> power-up; change it to match your network.
> - A **network connection** is required for communication between the device
> and its configured SNTP servers, as well as authorized SNMP managers.

<br><br>

---

## 4. Device Console

> The browser-based configuration interface is called the **Device Console**. It
> may also be referred to as the Web Console in AppBlocks documentation.

<br>

### 4.1 Database
For every value you want to monitor, configure the gateway with the relevant Modbus device and register.
The gateway reads that register at the **configured polling interval** and makes the latest value available to
your SNMP monitoring system.

| Table | Storage | Entries | User-defined | After reboot | Purpose |
| --- | --- | --- | --- | --- | --- |
| `TABLE_DEVICES_INPUT` | Flash | Max. 30 | Yes | Retained | Modbus-to-SNMP mappings |
| `TABLE_DEVICES_OUTPUT` | RAM | Max. 30 | No | Rebuilt | Latest polling results |
| `TABLE_SETTINGS` | RAM | 22 predefined | No | Recreated | Device Console settings |

<br>

> **NOTE:**
> - Use `TABLE_DEVICES_INPUT` to define which Modbus values the gateway monitors.
> - `TABLE_DEVICES_OUTPUT` shows the latest polling results. It is not saved and is rebuilt after each restart.
> - `TABLE_SETTINGS` contains the settings used by the Device Console.
> - **Restart the device after changing `TABLE_DEVICES_INPUT` or a Setting**. Changes are saved immediately but take
>   effect only after restart.

<br>

### Column reference

The following table explains the columns shown in the Device Console database
tables.

| Column | Used by table | Meaning |
| --- | --- | --- |
| `NAME` | Input, Output, Settings | A descriptive name for the mapping or setting. |
| `DEVICE_ID` | Input, Output | The Modbus address of the field device. |
| `REG_ADDR` | Input, Output | The Modbus register address to read or write. |
| `FUNC_CODE` | Input | The Modbus function code used to access the register. |
| `OID_COLUMN` | Input, Output, Settings | The identifier used to associate the entry with its SNMP object. |
| `HASH_ID` | Output | An internal unique identifier for the polling result. |
| `VALUE` | Output | The latest value read from the Modbus device. |
| `INFO` | Settings | Additional information describing the Device Console setting. |

<br>

> **NOTE:**
> - **Register addressing and value interpretation:**
> Enter the register address (REG_ADDR) exactly as shown in the connected Modbus device's
> register map.
> - **Warning:** If the register address (REG_ADDR) is incorrect, the gateway may not receive a result
> when polling the Modbus device.

<br>

### 4.2 Settings

Use the Settings area to review device information and configure polling,
network, and SNMP access. 

> **Reminder:**
> - Restart the device after changing a setting.
> - The new value takes effect after restart.

#### 4.2.1 Info

The `Info` group displays the device's restart counters and the current and maximum number of `TABLE_DEVICES_INPUT` records.

![Device Console Settings, Info group](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_info.png)

#### 4.2.2 Config

Use the `Config` group to set the **Polling Interval** and **Debugprint**.

> Adjust the `Polling Interval` to suit your application requirements.

![Device Console Settings, Config group](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_config.png)

#### 4.2.3 Network

The Network group lets you enable DHCP or configure a static Ethernet IP
address, netmask, and gateway.

> **NOTE:**
> - When DHCP is enabled, the static Ethernet address fields are **Unavailable**.
> - DHCP may assign a different IP address and interrupt your Device Console connection.
> - To keep the device reachable at a known address, a **static** IP address is recommended.

![Device Console Settings, Network group](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_network.png)

#### 4.2.4 SNMP

Use the `SNMP` group to configure how your monitoring system connects to the gateway.
You can set the SNMP port, read and write community strings, device description, system object ID, and SNMPv3 user and security settings.

![Device Console Settings, SNMP basic settings](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_snmp_basic_settings.png)

> **NOTE:**
> - **Read:** Use SNMP monitoring software to read gateway values with `snmpget` or `snmpwalk`.
> - **Write:** Authorized SNMP clients can configure **ALL** gateway SNMP objects with `snmpset`.

<br>

![Device Console Settings, SNMPv3 security settings](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_snmp_security_settings.png)

#### 4.2.4.1 Scalar Instance Object-ID

Each mapped gateway value is available through a **full SNMP OID**.
Its composition is:

```text
<SysObjectID>.1.<OID_COLUMN>.0
```

> **NOTE:**
> - By default, `sysObjectID` is set to `1.3.6.1.4.1.20738`.
> - The `sysObjectID` may be extended with a child OID, such as `45.46`, to create `1.3.6.1.4.1.20738.45.46`.
> - The final `.0` is required when querying an individual value.
> - Each `OID_COLUMN` value must be **UNIQUE**.
> - To minimize mistakes, use the full OID returned by `snmpwalk`.

<br>

### 4.3 System

The `System` area lets you restart the device or restore its default settings.

#### 4.3.1 Reboot Device

Click **Reboot Device** to restart the device.

> Restart the device after changing `TABLE_DEVICES_INPUT` or Device Console
> settings to apply the new configuration.

#### 4.3.2 Initialize Settings

Click **Initialize Settings** to restore the device's default configuration.

> **MD-button:**
>
> You can also restore the default settings using the on-board **MD button**:
>
> - Press and hold the button for at least **5 seconds**, then release it.
> - Releasing the button *before* 5 seconds have elapsed has **no effect**.

<br>

![Device Console, System area](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_system.png)


<br><br>

---

## 5. Deployment Example

This example shows how to connect a Modbus device to the gateway, configure a
Modbus value for polling, and read it through an SNMP monitoring system using
`snmpwalk`.

It also demonstrates how an authorized SNMP client can use `snmpset` to write
a Modbus value and change a device setting.

<br>

### 5.1 Connect the Modbus BP#02 to the Gateway

The deployment steps use a Tibbo Modbus BP#02, a sensor for ambient temperature
and relative humidity. The BP#02 is only used as an example; you can connect
another compatible Modbus device instead.

> **IMPORTANT:** Before replacing a Modbus sensor or changing its wiring,
> disconnect power from the device. Reconnect power only after the new sensor
> and all wiring have been connected correctly.

Connect the BP#02 to the TPP2W(G2) as shown below. Connect each wire to the
terminal with the matching label: `VCC` to `VCC`, `GND` to `GND`, `Tx` to
`Tx`, and `Rx` to `Rx`. When using a different Modbus device, follow its
documentation for the required power and communication connections.

![Connecting the BP#02 sensor to the TPP2W(G2)](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/tpp2wg2_modbus.png)

Should you be interested, more Tibbo Modbus models are available [here](https://tibbo.com/store/rs485-modbus-sensors.html).

<br>

### 5.2 Connect the Network Cable and Power Adapter

Connect an Ethernet cable from the device's RJ45 port to your local network.
Then connect the supplied power adapter to the device. The device starts when
power is connected. Allow up to 20 seconds for the device to fully boot and
become ready for use.

![Connecting the Ethernet cable and power adapter to the TPP2W(G2)](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/tpp2wg2_ethernet_power.png)

### 5.3 Connect to the Device Console

Using your preferred web browser, navigate to [http://192.168.1.40](http://192.168.1.40). The **Home** page should be displayed.

![Device Console, Home area](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_home.png)

<br>

### 5.4 Configure Settings

Open **Settings** in the Device Console. Review the network, SNMP, and polling
settings required for your installation. Restart the device after making any
changes so that the new settings are applied.

> For an overview of the available settings, see [Section 4.2, Settings](#42-settings).

#### 5.4.1 Polling Interval

Set the **Polling Interval** in the **Config** group to a value that suits your
application. This setting determines how often the gateway reads values from
the connected Modbus device.

#### 5.4.2 Network

Choose the network method required by your installation in the **Network**
group:

- Enable **DHCP** if your network automatically assigns IP addresses.
- Leave **DHCP** disabled to use a static IP address, then enter the IP address,
  netmask, and gateway supplied by your network administrator.

> This guide uses `192.168.1.40` throughout the example.


After changing the network settings and restarting the device, reconnect to
the Device Console using its new IP address.

#### 5.4.3 SNMP

Configure the SNMP port and access credentials required by your monitoring
system in the **SNMP** group. Configure either SNMPv2c community strings or
SNMPv3 user and security settings, as required by your organization.

Enable write access only when it is required for approved control points. Use
credentials and security settings approved by your organization.

> This guide uses the default SNMP settings throughout the example.

<br>

### 5.5 Configure Modbus Mapping in `TABLE_DEVICES_INPUT`

Open **Database** in the Device Console. In `TABLE_DEVICES_INPUT`, add a
mapping for each Modbus value that you want to make available through SNMP.

> **NOTE:**
> - Enter the `DEVICE_ID`, `REG_ADDR`, and `FUNC_CODE` exactly as specified in
>   the connected field device's Modbus register map or datasheet.
> - Each `OID_COLUMN` value must be **UNIQUE**. For details, see
>   [Section 4.2.4.1, Scalar Instance Object-ID](#4241-scalar-instance-object-id).
> - `OID_COLUMN = 207` is used later in this guide to demonstrate **Write Data
>   to Modbus**.

<br>

![Device Console Database, TABLE_DEVICES_INPUT](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devices_input.png)



<br>

### 5.6 Restart Device

Restart the device so that the new configuration is applied. Allow up to 20
seconds for the device to fully boot and become ready for use.


<br>

### 5.7 Verify Modbus Polled Data in `TABLE_DEVICES_OUTPUT`

In **Database**, select `TABLE_DEVICES_OUTPUT`. Confirm that the mapping
configured in `TABLE_DEVICES_INPUT` appears in this table and that its `VALUE`
field shows the expected value read from the Modbus device.

The `TABLE_DEVICES_OUTPUT` is a live view of the gateway's polling
results, updated at the configured polling interval.

> **NOTE:**
> - If the expected mapping or value is **not** shown, check the Modbus wiring.
> - Also check the `DEVICE_ID`, `REG_ADDR`, `FUNC_CODE`, and `OID_COLUMN` configured in
>   `TABLE_DEVICES_INPUT`.
> - You can ignore the `HASH_ID`, since it is an internal identifier used by
>   the gateway.

![Device Console Database, TABLE_DEVICES_OUTPUT](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devices_output.png)



<br>

### 5.8 Verify Modbus Polled Data via SNMP

This example uses a Linux computer and SNMPv2c to read the Modbus value
published by the gateway. The SNMP settings used below are configured in
[Section 4.2.4, SNMP](#424-snmp).

Use `snmpwalk` to view **ALL** SNMP values available under the gateway's
`SysObjectID`.

```text
snmpwalk -v2c -c <read-community> <device-ip> <SysObjectID>
```

For this example, use the configured read community, device IP address, and
`SysObjectID`:

```bash
snmpwalk -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46
```

![Linux terminal, SNMP walk example](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpwalk_example.png)

The command displays the available SNMP values and their full OIDs. Locate the
value with `OID_COLUMN` `401` in the command output.

Use `snmpget` to read one specific value. The full scalar OID has the following
format:

```text
<SysObjectID>.1.<OID_COLUMN>.0
```

Use the following command to read the value with `OID_COLUMN` `401`:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.401.0
```

![Linux terminal, SNMP get example](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_example.png)

> **NOTE:**
> - Replace `public`, `192.168.1.40`, and `1.3.6.1.4.1.20738.45.46` when
>   the corresponding settings of your device are different.
> - SNMPv3 can also be used. Use the username, authentication settings, and
>   privacy settings configured in the Device Console. The general command
>   format is:
>
>   ```text
>   snmpwalk -v3 -l authPriv -u <username> -a <auth-protocol> \
>     -A <authentication-password> -x <privacy-protocol> \
>     -X <privacy-password> <device-ip> <SysObjectID>
>   ```
>
>   ```text
>   snmpget -v3 -l authPriv -u <username> -a <auth-protocol> \
>     -A <authentication-password> -x <privacy-protocol> \
>     -X <privacy-password> <device-ip> <SysObjectID>.1.<OID_COLUMN>.0
>   ```
>
> - Other SNMP tools can be used instead of the Linux command line. For
>   example, Windows users can use Paessler SNMP Tester.



<br>

### 5.9 Write Data to Modbus

This section shows how an authorized SNMP client can write a value to a Modbus
register through the gateway. In this example, `OID_COLUMN 207` identifies the
configured Modbus write mapping. The SNMP settings used below are configured in
[Section 4.2.4, SNMP](#424-snmp).

> **NOTE:**
> - The SNMP write command sends the requested value to the connected Modbus
>   device.
> - Use this function only for approved control points and only when it is safe
>   to change the value on the field device.

#### 5.9.1 Read Current Value

Before writing a new value, use `snmpget` to read the current value of the
mapping with `OID_COLUMN` set to `207`.

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.207.0
```

![Linux terminal, SNMP get OID 207 before write](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_oid_207_before_write_example.png)

> The current value will be used later to compare with the newly written value.

#### 5.9.2 Write New Value

Use the following command to write the value `4546` to `OID_COLUMN` `207`:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.207.0 i 4546
```

![Linux terminal, SNMP set OID 207](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_oid_207_write_example.png)

> The command sends the value to the connected Modbus device.

#### 5.9.3 Verify SNMP Write Response

The gateway also provides a confirmation of the most recent SNMP write through
`OID_COLUMN` `65534`. Run the following command immediately after the write:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

![Linux terminal, SNMP get OID 65534 after write](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_oid_65534_after_write_example.png)

Confirm that the returned response shows `SUCCESS`.

> **NOTE:**
> - The SNMP write response shows whether the most recent `snmpset` command
>   succeeded (`SUCCESS`) or failed.
> - The response remains available for 30 seconds, then resets to `IDLE`.
> - If the most recent `snmpset` command **failed**, follow the hint shown in the
>   response and check the Modbus wiring and configuration before trying again.

#### 5.9.4 Verify Written Data via SNMPGET

Use `snmpget` to confirm the written value through SNMP:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.207.0
```

![Linux terminal, SNMP get OID 207 after write](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_oid_207_after_write_example.png)

Confirm that the returned value is `4546`.

#### 5.9.5 Verify Written Data in the Device Console

Wait for at least one polling interval after sending the `snmpset` command.
Then open **Database** in the Device Console and select
`TABLE_DEVICES_OUTPUT`. Confirm that the entry with `OID_COLUMN` `207` shows
the new `VALUE` of `4546`.

![Device Console Database, TABLE_DEVICES_OUTPUT OID 207](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devices_output_oid_207.png)

> **NOTE:**
> - `TABLE_DEVICES_OUTPUT` is READ-ONLY and does not update automatically.
>   Click **Refresh** to update the displayed data.
> - Modbus writes take effect immediately. Restarting the device is therefore
>   NOT necessary.

<br>

### 5.10 Change Settings via SNMP

Authorized SNMP clients can change device settings. Each setting is identified
in `TABLE_SETTINGS` by its `OID_COLUMN`.

> **NOTE:**
> - Changing network or SNMP settings **can interrupt your connection** to the device.
> - A changed device setting is **NOT** applied until the device restarts.

#### 5.10.1 Review the Setting in `TABLE_SETTINGS`

In **Database**, select `TABLE_SETTINGS`. Locate the setting you want to change
and note its `OID_COLUMN`.

> **NOTE:**
> - `TABLE_SETTINGS` is read-only.
> - Device settings can be changed through the Device Console or SNMP.


#### 5.10.2 Write Network Settings Using SNMP

Most installations use a static IP address that matches the local network. For
example, configure the device with the following settings:

- `NET_DHCP` to `disabled`
- `NET_IP` to `192.168.1.121`
- `NET_MASK` to `255.255.255.0`
- `NET_GATEWAY` to `192.168.1.1`

The marked rows below identify these settings and their `OID_COLUMN` values.

![Device Console Database, TABLE_SETTINGS network settings](https://raw.githubusercontent.com/imcase/Im_snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_settings_network.png)

Send the following commands one at a time. The commands use the device's
current IP address, `192.168.1.40`.

> After each `snmpset` command, run the response command within 30 seconds to confirm that the write succeeded.

##### 5.10.2.1 Disable DHCP

Disable DHCP:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65503.0 s disabled
```

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

Confirm that the returned response shows `SUCCESS`.

<!-- TODO: Show the `snmpset` command and its `OID_COLUMN` `65534` response,
including a screenshot. -->

##### 5.10.2.2 Set the IP Address

Set the device IP address:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65505.0 s 192.168.1.121
```

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

Confirm that the returned response shows `SUCCESS`.

<!-- TODO: Show the `snmpset` command and its `OID_COLUMN` `65534` response,
including a screenshot. -->

##### 5.10.2.3 Set the Subnet Mask

Set the subnet mask:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65506.0 s 255.255.255.0
```

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

Confirm that the returned response shows `SUCCESS`.

<!-- TODO: Show the `snmpset` command and its `OID_COLUMN` `65534` response,
including a screenshot. -->

##### 5.10.2.4 Set the Gateway Address

Set the gateway address:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65504.0 s 192.168.1.1
```

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

Confirm that the returned response shows `SUCCESS`.

<!-- TODO: Show the `snmpset` command and its `OID_COLUMN` `65534` response,
including a screenshot. -->

#### 5.10.3 Verify Written Settings via SNMPGET

<!-- TODO: Show the `snmpget` commands used to verify the written settings and
their results, including a screenshot. -->

#### 5.10.4 Verify Written Settings in the Device Console

<!-- TODO: Show the written settings in the Device Console, including a
screenshot. -->

#### 5.10.5 Reboot Device via SNMP

<!-- TODO: Show the SNMP command used to reboot the device and its result,
including screenshots. -->



<br><br>

--- 

## 6. Commissioning checklist

1. Record the field device's Modbus register map, communication parameters,
   data representation, and expected engineering values.
2. Install and verify the RS-485 hardware and field wiring.
3. Configure Ethernet addressing for the local network and limit management
   access to authorized hosts.
4. Create the Modbus-to-SNMP mappings and restart the device if required.
5. Prove the southbound path first: confirm that the gateway successfully
   polls the expected raw Modbus value.
6. From an SNMP manager, walk the private enterprise branch and use the exact
   returned scalar OID for an SNMP GET.
7. Change the physical input or simulated register, wait for a successful
   Modbus poll, and verify that the SNMP value changes accordingly.
8. Where a write mapping is deliberately enabled, test it in a safe bench
   setup and verify the resulting field-device register separately.

<br><br>

---

## 7. Troubleshooting

| Symptom | Checks |
| --- | --- |
| SNMP responds but value is stale or zero | Verify the Modbus poll result, unit ID, function code, register addressing, serial settings, scaling, and byte/word order. An SNMP response alone does not prove the field link works. |
| SNMP GET returns `No Such Object` or `No Such Instance` | Walk the configured enterprise branch and query the full scalar OID, including its `.0` instance suffix. |
| No Modbus response | Check RS-485 polarity, termination/biasing, serial parameters, selected Tibbit/socket, unit ID, and whether the target actually supports the requested function/register. |
| SNMP SET fails | Confirm that the object is configured read/write, the caller is authorized, the write mapping exists, and the field device accepts the function/value. |
| Value has the wrong magnitude | Check signedness, scale factor, register count, and byte/word order. |


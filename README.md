# Quantum Fiber Q1000K Bypass with X-ONU-SFPP and UDM-Pro

A working configuration using an ADTRAN OLT and WAN VLAN 201.

## Results

I replaced my Quantum Fiber Q1000K with an X-ONU-SFPP XGS-PON ONU stick installed directly in my UDM-Pro’s SFP+ WAN port.

- Internet works with VLAN 201 enabled on the UDM-Pro.
- The stick reports `O5.1, Associated state`.
- The Ethernet link between the stick and UDM-Pro reports `10,000 Mbps`.

This documents my working connection. It does not establish official Quantum support or compatibility with every Quantum installation.

## Equipment and Firmware

| Component | Value |
| --- | --- |
| Provider | Quantum Fiber |
| Connection type | XGS-PON |
| Original SmartNID | Q1000K |
| Provider OLT | ADTRAN |
| Replacement ONU | X-ONU-SFPP |
| Router | Ubiquiti UDM-Pro |
| Community firmware mod | 8311 Community Firmware MOD by djGrr, basic v2.8.3 |
| Underlying firmware | LEDE PTXG_CX_V0.03 |
| Reported Ethernet link | 10,000 Mbps |

This guide starts with the 8311 mod installed and its GUI accessible. It does not cover firmware installation.

## 1. Identify Your OLT Vendor

While your Q1000K is still connected to the fiber:

1. Log into the Q1000K at <https://192.168.0.1>.
2. In the same browser, open:

   ```text
   https://192.168.0.1/cgi/cgi_get?Object=Device.Optical.Interface.1
   ```

3. Find the `MODEL` field in the response.
4. A value of `ADTN` identifies **ADTRAN**.

My successful bypass was on an ADTRAN OLT. This guide does not confirm compatibility with other OLT vendors.

Confirm that your connection uses **XGS-PON**. The Q1000K supports both GPON and XGS-PON, so its model alone does not establish which service you have.

## 2. Prepare Your Equipment

- Keep your Q1000K available for rollback.
- Record its FSAN ID, hardware version, and software version.
- Back up your stick’s existing configuration.
- Disconnect the fiber from the stick while editing its settings.
- Keep fiber connector ends clean and capped when disconnected.
- Do not publish your complete FSAN ID, passwords, or configuration backup.

## 3. Configure the PON Identity

Open **8311 Configuration → PON**.

| Field | Setting |
| --- | --- |
| PON Serial Number / ONT ID | Your Q1000K’s complete FSAN ID, entered exactly |
| Vendor ID | The four-letter prefix of your FSAN ID |
| Hardware Version | Your Q1000K’s hardware version |
| PON Mode | `XGS-PON` |

My FSAN prefix was `AXON`, and my Q1000K hardware version was `1.1`.

### Use the FSAN ID, Not the Device Serial

The Q1000K label has separate entries for:

- **Serial no.**
- **FSAN ID**

Enter the **FSAN ID** in the stick’s PON Serial Number field. Do not use the device serial beginning with `Q1000K`.

Preserve uppercase letters. Use your own device’s identity.

My initial configuration used the wrong PON identity and encountered `O7, Emergency Stop`. After correcting the identity to match my Q1000K’s FSAN ID, the stick reached `O5.1` and internet worked.

## 4. Review the Remaining PON Settings

These settings were present during my successful test:

| Field | Setting |
| --- | --- |
| Sync Circuit Pack Version | Checked |
| Registration ID | Blank |
| Logical ONU ID | Blank |
| Logical Password | Blank |
| MIB File | `/etc/mibs/prx300_1U.ini` |
| PON Slot | Blank |
| Firmware Version Match | Blank |
| Active firmware bank override | Unset |
| Committed firmware bank override | Unset |
| OMCC Version | Default |
| OMCI Interoperability Mask | Default |

### Equipment ID and Software Version Overrides

My configuration also contained Equipment ID and Software Version overrides. I have not verified whether those values were accurate or necessary.

If you configure these fields, use values verified from your own Q1000K. Do not assume another subscriber’s software version is required for your connection.

## 5. Review ISP Fixes

Open **8311 Configuration → ISP Fixes**.

| Field | Setting during my successful test |
| --- | --- |
| Fix VLANs | Enabled |
| Internet VLAN | Left at the displayed default |
| Services VLAN | Left unchanged |

The Internet VLAN field displayed a default of `0`. My working connection still requires **VLAN 201 on the UDM-Pro WAN**.

Do not assume that the displayed default means you should remove the UDM-Pro’s WAN VLAN tag.

Save your configuration and follow any reboot prompt.

## 6. Configure the UDM-Pro WAN

- Assign the SFP+ port containing the stick as your WAN.
- Enable WAN VLAN tagging.
- Set the WAN VLAN ID to **201**.
- Verify that the port links at **10 Gbps**.
- Keep your existing working Quantum WAN address settings.

VLAN 201 is part of the confirmed working configuration documented here.

## 7. Connect and Test

1. Move the fiber from the Q1000K to the stick.
2. Allow the stick to register.
3. Open the stick’s PON Status page.
4. Check for `O5.1, Associated state`.
5. Confirm that the UDM-Pro receives a WAN IP address.
6. Test internet access from a device on your LAN.

`O5.1` alone does not prove internet traffic is passing. Confirm actual connectivity.

## Troubleshooting

### O1.1, Off-Sync

If the stick reports:

```text
PON PLOAM Status: O1.1, Off-sync state
RX Power: -inf dBm
```

It reports no received optical signal. This is expected when the fiber is disconnected.

If the fiber is connected, investigate the optical connection and confirm XGS-PON compatibility before changing authentication or VLAN settings.

### O7, Emergency Stop

Disconnect the stick’s fiber and investigate its PON identity settings and alarms.

- Verify that the PON serial matches your Q1000K’s FSAN ID.
- Verify that the Vendor ID matches the FSAN prefix.
- Do not enable an ignore-O7 option.
- Do not change laser calibration values.

O7 alone does not identify the cause.

### O5.1 but No Internet

- Check whether the UDM-Pro has a WAN IP address.
- Verify that WAN VLAN tagging is enabled with VLAN ID `201`.
- Inspect the stick’s VLAN Tables page before changing additional settings.

### Restore the Q1000K

If you need to roll back:

1. Disconnect the fiber from the stick.
2. Reconnect the fiber to the Q1000K.
3. Restore the original Ethernet connection.
4. Restore your previous UDM-Pro WAN settings if you changed them.

## After It Works

- Set a root password on the stick.
- Export a configuration backup.
- Record your working settings and firmware version.
- Keep the Q1000K available for rollback and provider troubleshooting.

## Confirmed Working Combination

- ADTRAN OLT, identified by `ADTN` in the Q1000K’s `MODEL` field.
- Q1000K removed from the active connection.
- X-ONU-SFPP installed directly in the UDM-Pro.
- PON status `O5.1, Associated`.
- 10G Ethernet link between the stick and UDM-Pro.
- Working internet with WAN VLAN `201` enabled.

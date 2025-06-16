# Parkside Robot Lawnmower Integration with LocalTuya for Home Assistant

This guide explains how to integrate the Parkside robot lawnmower into Home Assistant using the LocalTuya integration. It covers extracting device credentials, configuring entities, and example scripts for controlling the mower.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Extract Tuya Device Credentials](#extract-tuya-device-credentials)
3. [Install LocalTuya Integration](#install-localtuya-integration)
4. [Configure LocalTuya Entities](#configure-localtuya-entities)
5. [Verify Entities in Home Assistant](#verify-entities-in-home-assistant)
6. [Example Automations and Scripts](#example-automations-and-scripts)
7. [Troubleshooting Tips](#troubleshooting-tips)
8. [License](#license)

---

## Prerequisites

* **Home Assistant** (Core 2023.7+ recommended)
* **HACS** installed
* Parkside robot mower already added to Tuya IoT Platform or Tuya Smart App
* Access to your local network (to retrieve IP address)

---

## Extract Tuya Device Credentials

1. **Log in** to [Tuya IoT Platform](https://iot.tuya.com) or use `tuya-cli`.
2. **Locate** your Parkside mower under **Cloud Development** > **Devices**.
3. **Copy** the following details:

   * **Device ID**
   * **Local Key**
   * **Local IP Address**

Note these values for the LocalTuya configuration.

---

## Install LocalTuya Integration

1. Open **Home Assistant** UI.
2. Go to **HACS** → **Integrations** → **Explore & Add repositories**.
3. Search for **LocalTuya** and install.
4. Restart Home Assistant.

---

## Configure LocalTuya Entities

Add the following to your `configuration.yaml` or use the **Configure** UI in the LocalTuya integration:

```yaml
localtuya:
  - host: YOUR_MOWER_IP
    device_id: "YOUR_DEVICE_ID"
    local_key: "YOUR_LOCAL_KEY"
    friendly_name: "Parkside Mower"
    protocol_version: "3.3"
    entities:
      # Start/Stop Switch (DPS 2 writes to MachineControlCmd)
      - platform: switch
        friendly_name: "Mower Start/Stop"
        id: 2
        write_function: 115

      # Enum Select: Mode (DPS 3)
      - platform: select
        friendly_name: "Mower Mode"
        id: 3
        options:
          - "auto"
          - "manual"
          - "standby"

      # Enum Select: Control Command (DPS 115)
      - platform: select
        friendly_name: "Mower Control Command"
        id: 115
        options:
          - "PauseWork: Pause"
          - "CancelWork: Cancel"
          - "ContinueWork: Continue"
          - "StartMowing: Start"
          - "StartFixedMowing: Spot"
          - "StartReturnStation: Dock"

      # Battery Level (DPS 13)
      - platform: sensor
        friendly_name: "Mower Battery Level"
        id: 13
        unit_of_measurement: "%"
        device_class: battery
        state_class: measurement

      # Rain Mode Switch (DPS 104)
      - platform: switch
        friendly_name: "Rain Mode"
        id: 104

      # Status Sensor (DPS 5)
      - platform: sensor
        friendly_name: "Mower Status"
        id: 5
```

> **Note:** The `write_function: 115` forces writes on the `switch` to use DPS 115 (MachineControlCmd) instead of DPS 2.

If you configure via the UI, fill in the **options** field for each select entity as shown above (one `device_value: friendly name` per line).

---

## Verify Entities in Home Assistant

1. Go to **Developer Tools** → **States**.
2. Confirm each entity (`switch.mower_start_stop`, `select.mower_mode`, etc.) appears and reports correct values.
3. Test toggling **Rain Mode** or changing **Mower Mode**.
4. For **Start/Stop**, use the `"Start"` option in the **Mower Control Command** dropdown.

---

## Example Automations and Scripts

### Script: Start Mower

```yaml
alias: "Script: Start Mower"
sequence:
  - alias: "Select Start option"
    service: select.select_option
    target:
      entity_id: select.mower_control_command
    data:
      option: "Start"
mode: single
```

### Script: Return Mower To Dock

```yaml
alias: "Script: Return Mower To Dock"
description: |
  Pause mower, cancel task, then send to dock once in standby.
sequence:
  - alias: "Step 1: Pause mower"
    service: select.select_option
    data:
      entity_id: select.mower_control_command
      option: "Pause"

  - alias: "Step 2: Wait for mower to pause"
    wait_for_trigger:
      - platform: state
        entity_id: sensor.mower_status
        to: "PAUSED"
    timeout: "00:00:02"

  - alias: "Step 3: Cancel current task"
    service: select.select_option
    data:
      entity_id: select.mower_control_command
      option: "Cancel"

  - alias: "Step 4: Wait for standby"
    wait_for_trigger:
      - platform: state
        entity_id: sensor.mower_status
        to: "STANDBY"
    timeout: "00:00:02"

  - alias: "Step 5: Send to dock"
    service: select.select_option
    data:
      entity_id: select.mower_control_command
      option: "Dock"
mode: single
```

### Automation: Low Battery Notification

```yaml
alias: "Mower Battery Low Alert"
trigger:
  - platform: numeric_state
    entity_id: sensor.mower_battery_level
    below: 20
action:
  - service: notify.notify_telegram
    data:
      message: "⚠️ Mower battery low: {{ states('sensor.mower_battery_level') }}%."
mode: single
```

---

## Troubleshooting Tips

* **No dropdown options?** Ensure `options:` are explicitly defined in YAML or UI.
* **Switch doesn’t start mower?** Verify `write_function: 115` is set for DPS 2.
* **Mower stuck in error?** Check `sensor.machine_error` and `sensor.machine_warning` for clues.
* **Entities not updating?** Restart Home Assistant and mower.

---

## License

This project is licensed under the MIT License. Feel free to copy, modify, and share.

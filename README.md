# Home Assistant: IKEA E1766 (TRÅDFRI On/Off Switch) via DIRIGERA Matter — Use as an Automation Trigger

This repo explains a reliable workaround to use an **IKEA TRÅDFRI E1766 switch** (paired to **IKEA DIRIGERA**) as an **automation trigger** in **Home Assistant** when connected **via Matter**.

---

## Problem

When IKEA TRÅDFRI E1766 (on/off switch) is paired to **DIRIGERA** and bridged to **Home Assistant via Matter**, Home Assistant often does **not** expose a “button pressed” **Device Trigger** in the Automation UI. The only device trigger shown may be **battery level changes**.

However, Home Assistant still receives button presses as **`event.*` entities**. These entities often keep the same state (e.g. always `pressed`), so a normal `platform: state` trigger may not fire reliably.

---

## Solution

Listen to Home Assistant’s internal **`state_changed`** event (Automation trigger type: **Manual event**) and filter by the `event.*` entity_id.

---

## Prerequisites

- Home Assistant with the IKEA switch paired to **DIRIGERA**, and DIRIGERA bridged into HA via **Matter**
- You can see one or more `event.*` entities for the switch (enabled)

---

## Step 1 — Locate the button Event Entity IDs

1. In Home Assistant, go to **Settings → Devices & services → Devices**.
2. Select your IKEA E1766 **TRADFRI open/close remote** device (paired via DIRIGERA/Matter).
3. On the device page, find the **Events** section. You should see **two entries named “Button”** (one per physical button).
4. Click the first **Button** entry to open it.
5. In the top-right corner, click the **gear icon** (Settings).
6. Copy the **Entity ID** (it will look like `event.something_button`).
7. Go back and repeat steps 4–6 for the second **Button** entry to get the second `event.*` entity ID.

You’ll use these two `event.*` entity IDs in the automation later.

---

## Step 2 — Quick test

1. On the device page, find the **Events** section where you see the two **Button** entries.
2. Press the physical **ON** and **OFF** buttons on the E1766.
3. Confirm that the “last triggered / last updated” time shown next to each **Button** entry updates when you press it.

If those timestamps update, Home Assistant is receiving both button presses and you can proceed to the automation step.

---

## Step 3 — Create the automation

### Option A: YAML

Paste this into a **single automation** in Home Assistant (Automation UI → overflow menu → “Edit in YAML”).  
Replace entity IDs and the actions you want to run:

```yaml
alias: "IKEA E1766 (DIRIGERA Matter) -> Automation trigger"
mode: restart
trigger:
  - platform: event
    event_type: state_changed
    event_data:
      entity_id: event.YOUR_Entity_ID_button
    id: on
  - platform: event
    event_type: state_changed
    event_data:
      entity_id: event.YOUR_Entity_ID_button_2
    id: off
action:
  - choose:
      - conditions:
          - condition: trigger
            id: on
        sequence:
          # Example action: open blinds fully
          - service: script.blinds_100
      - conditions:
          - condition: trigger
            id: off
        sequence:
          # Example action: close blinds
          - service: script.blinds_0
```
***
### Option B: UI-only
Automation → Add trigger → **Manual event**

For the “on” entity:
- **Event type:** `state_changed`
- **Event data:**  
Replace entity IDs you want to use
```
entity_id":"event.YOUR_Entity_ID_button
```

Add a second trigger for the “off” entity:
```
entity_id":"event.YOUR_Entity_ID_button_2
```

Then use a “Choose” action, or create two separate automations (one per entity).

---

## Notes / Troubleshooting

### “Why not use a normal state trigger?”
For many Matter-bridged IKEA remotes/switches, the entity state can remain `pressed` forever; only timestamps/attributes update. A state trigger may not fire because the state value doesn’t change. Listening to `state_changed` is more reliable.

### I only have one `event.*` entity (not two)
If you only see one `event.*` entity, you’ll likely need to inspect `new_state.attributes` in the `state_changed` event to distinguish on/off. Use Developer Tools → Events to inspect the payload and branch based on attributes.

### No events show up in Developer Tools
- Ensure the `event.*` entity is **enabled**
- Confirm the device is online in DIRIGERA
- Re-pair the switch or re-add the Matter bridge if needed

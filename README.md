blueprint:
  name: Motion-activated Light with Time-based Color
  description: Turn on a light when motion is detected and change its color based on the time of day.
  domain: automation
  input:
    motion_entity:
      name: Motion Sensor
      selector:
        entity:
          filter:
            device_class: motion
            domain: binary_sensor
    light_target:
      name: Light
      selector:
        target:
          entity:
            domain: light
    no_motion_wait:
      name: Wait time
      description: Time to leave the light on after last motion is detected.
      default: 120
      selector:
        number:
          min: 0
          max: 3600
          unit_of_measurement: seconds

mode: restart
max_exceeded: silent

trigger:
  platform: state
  entity_id: !input motion_entity
  from: "off"
  to: "on"

action:
  - alias: "Turn on the light with appropriate color"
    choose:
      - conditions:
          condition: time
          after: "08:00:00"
          before: "22:59:59"
        sequence:
          - service: light.turn_on
            target: !input light_target
            data:
              color_name: "white"
      - conditions:
          condition: time
          after: "23:00:00"
          before: "07:59:59"
        sequence:
          - service: light.turn_on
            target: !input light_target
            data:
              color_name: "blue"
  - alias: "Wait until there is no motion from device"
    wait_for_trigger:
      platform: state
      entity_id: !input motion_entity
      from: "on"
      to: "off"
  - alias: "Wait the number of seconds that has been set"
    delay: !input no_motion_wait
  - alias: "Turn off the light"
    service: light.turn_off
    target: !input light_target

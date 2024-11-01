# Home Assistant
## With Homematic (Legacy) & HmIP for Floor Heating with Smart Thermostat (PID)

### Situation
- Old House from 1904, heavily modified by previous owners
- Some rooms have radiators only, some have floor heating (water) only, some have both
- Some rooms use Homematic Legacy (the old version), some have been upgraded to Homematic IP
- Some rooms have a single floor circuit, some have two floor heating circuits per room for floor heating
- Floor heating for 9 rooms shall be controlled by Home Assistant using PWM (Pulse Width Management) using a PID Controller
- Selected integration is https://github.com/ScratMan/HASmartThermostat until money is found to upgrade to the Homematic IP floor heating controller
- ON/OFF valves are controlled by Homematic HmIP-DRSI4 and HM-LC-Sw4-DR-2 switches (230V NO-Valves screwed on the floor distribution units). **Note** this is just ON or OFF and no values in between.
  
### Notes
- Homematic: each room has a Wall Thermostat (HM-TC-IT-WM-W-EU or HmIP-WTH-1) with week programs and open window detection by optical sensors
- I prefer floor heating over radiator heating - MEASURED_TEMPERATURES are manipulated to be 0.3°C lower than measured for floor heating, so floor heating continues to warm the floor while the radiators are already off
- Open Task: synchronize Smart Thermostat PID with heating status (either pump or flame status)

## Installation
### Step 1: Install Smart Thermostat from HACS
Well, you know HACS. If not, google it ;)

### Step 2: Configuration in HA configuration.yaml
Use your preferred method to add definitions:

````
climate:
  # Smart Thermostat https://github.com/ScratMan/HASmartThermostat
  # Flur EG Treppe: Homematic IP - nur Fussboden
  - platform: smart_thermostat
    unique_id: smart_thermostat_flureg_treppe
    name: Fussbodenregelung FlurEG Treppe
    heater: 
      - switch.fbheizung_treppe
      - switch.hm_flur_eg_fbheizung_esszimmerbereich_rx
    target_sensor: sensor.read_flur_eg_treppe_hmip_thermostat_fb_smart_thermostat
    min_temp: 7
    max_temp: 35
    ac_mode: False
    kp: 50
    ki: 0.01
    kd: 2000
    target_temp: 20
    keep_alive:
      seconds: 60
    pwm: 00:20:00

  # Flur EG Eingang: Homematic IP - nur Fussboden
  - platform: smart_thermostat
    unique_id: smart_thermostat_flureg_eingang
    name: Fussbodenregelung FlurEG Eingang
    heater: switch.hm_flur_eg_fbheizung_eingangsbereich_rx
    target_sensor: sensor.read_flur_eg_eingang_hmip_thermostat_fb_smart_thermostat 
    min_temp: 7
    max_temp: 35            
    ac_mode: False          
    kp: 50                  
    ki: 0.01                
    kd: 2000                
    target_temp: 20         
    keep_alive:             
     seconds: 60            
    pwm: 00:20:00

  # Küche: Homematic Legacy - Fussboden + Radiatoren
  - platform: smart_thermostat
    unique_id: smart_thermostat_kueche
    name: Fussbodenregelung Küche
    heater: switch.hm_flur_eg_fbheizung_kueche_rx
    target_sensor: sensor.hm_kueche_wandthermostat_clima_tx_temperatur
    min_temp: 7             
    max_temp: 25            
    ac_mode: False          
    kp: 50                  
    ki: 0.01                
    kd: 2000                
    target_temp: 20         
    keep_alive:             
      seconds: 60           
    pwm: 00:20:00           
                            
  # Wintergarten: Homematic Legacy - Fussboden + Radiatoren
  - platform: smart_thermostat
    unique_id: smart_thermostat_wintergarten
    name: Fussbodenregelung Wintergarten
    heater: switch.hm_hwr_heizung_fbwintergarten_switch_rx
    target_sensor: sensor.hm_wintergarten_wandthermostat_clima_tx_temperatur
    min_temp: 7             
    max_temp: 35            
    ac_mode: False          
    kp: 50                  
    ki: 0.01                
    kd: 2000                
    target_temp: 20         
    keep_alive:             
      seconds: 60           
    pwm: 00:20:00

  # ... and so on per room ...
````

### Configure helpers
**Homematic IP** wall thermostats require a Helper to measure the ACTUAL_TEMPERATURE. **Homematic Legacy** exposes sensors already which can be used without helper.
In HA navigate to Settings >> Devices & Services >> Helpers. Add a **sensor helper** for each **Homematic IP** wall thermostat.

My naming convention is "READ **room** HmIP Thermostat -> FB Smart Thermostat" which will create sensors:

![Bildschirmfoto 2024-10-22 um 10 01 04](https://github.com/user-attachments/assets/4c9b0b36-a546-4421-bea7-7ba802809353)

![Bildschirmfoto 2024-10-22 um 10 01 34](https://github.com/user-attachments/assets/9f0b9882-38e8-41e0-92f1-9d436145616f)

![Bildschirmfoto 2024-10-22 um 10 02 17](https://github.com/user-attachments/assets/3c094467-dd4a-46eb-88a5-304ec00be0c0)

The ID refers to the wall thermostat and needs to be adjusted of course for each room and each helper.
You use these helpers in the configuration.yaml of Smart Thermostat as **target_sensor:**.

If you want floor heating to continue over radiator heating just subtract 0.3 from the read value before passing it to Smart Thermostat.

Now this should start working fine, but DESIRED_TEMPERATURES are not synced yet from wall thermostats to Smart Thermostat instances.

### Configure one Automatisation per Wall Thermostats to all Smart Thermostats
*Do not try* to combine all single automatisations into one big one. It seems that two triggers at the same time from different entities cannot be handled - at least that's my observation.

#### Homematic IP Sync example
````
alias: SYNC Wandthermostat Flur EG Eingang -> SmartThermostat
description: HmIP
triggers:
  - trigger: state
    entity_id:
      - climate.hmip_sth_000e60c9a481e4
    attribute: temperature
conditions: []
actions:
  - target:
      entity_id: climate.fussbodenregelung_flureg_eingang_2
    data:
      temperature: "{{ state_attr('climate.hmip_sth_000e60c9a481e4','temperature') }}"
    action: climate.set_temperature
mode: single
````

#### Homematic Legacy Sync example
````
alias: SYNC Wandthermostat Küche -> SmartThermostat
description: Hm Legacy
triggers:
  - trigger: state
    entity_id:
      - climate.hm_kuche_heizung_int0000005_2
    attribute: temperature
conditions: []
actions:
  - target:
      entity_id: climate.fussbodenregelung_kuche_2
    data:
      temperature: "{{ state_attr('climate.hm_kuche_heizung_int0000005_2','temperature') }}"
    action: climate.set_temperature
mode: single
````

Add one automatisation per wall thermostat, adjust trigger entity, target entity and state_attr entity (e.g. 3 values) accordingly

Now the Smart Thermostats will follow the active week profile of the Homematic Wall Thermostats. Succes.

### Visualisation
Navigate to HA Overview. Search for your Smart Thermostats instances and one by one click on the 3-dot menu, select the history icon, select "Show More", again the 3-dot menu on the upper right corner, select "Add current view to a card".
I use a "Floor Heating" Tab to show all floor heatings in one view.

![Bildschirmfoto 2024-10-22 um 10 09 02](https://github.com/user-attachments/assets/3cf34d30-0108-4e2b-93c3-8bf215ea4071)

### Traps
If target temperature in one room is stuck at one specific value (mine showed 20°C for like three days) double check that the wall thermostat is not in **heating** mode but in **auto** mode. You can switch modes in the Overview tab in Home Assistant. Only in **auto** mode the thermostat will follow the stored profile and the SYNC automatisations will trigger.

If you have added visualisations and they behave oddly after you tinkered with the Smart Thermostat configuration, delete them from the Dashboard and re-add them.

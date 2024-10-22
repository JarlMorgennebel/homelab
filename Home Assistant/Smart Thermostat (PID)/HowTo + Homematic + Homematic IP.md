# Home Assistant
## With Homematic (Legacy) & HmIP for Floor Heating with Smart Thermostat (PID)

### Situation
- Old House from 1904, heavily modified by previous owners
- Some rooms have radiators only, some have floor heating (water) only, some have both
- Some rooms use Homematic Legacy (the old version), some have been upgraded to Homematic IP
- Some rooms have a single pipe, some have two pipes per room for floor heating
- Floor heating for 9 rooms shall be controlled by Home Assistant
- Selected integration is https://github.com/ScratMan/HASmartThermostat until money is found to upgrade to the Homematic IP floor heating controller
- ON/OFF valves are controlled by Homematic HmIP-DRSI4 and HM-LC-Sw4-DR-2 (230V NO-Valves screwed on the floor distribution units)

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

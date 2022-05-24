# Northbound southbound edge devices
Setting up Energy manager on Industrial Edge Device
[Official documentation](https://github.com/industrial-edge/energy-manager-getting-started)

## index

* [a](#a)
    * [b](#b)

## Overview
![Overview](files/architecture.JPG)


### Setup

- Plant level
  - Connected to PORT 2 on edge device.

- OT Level edge device
  - Uses Port 1 for PC connection
  - Uses Port 2 for PLC connection
  - Applications
    - Dataservice - Saves data from southbound device in database    
    - IE Databus - Is used as data channel - MQTT broker      
    - SIMATIC S7 Connector - used for data retrieve from plc
    - Energy manager - Dashboarding of energy data  
    - Flow Creator - Debugging of dataflow
      
- OT Level PLC
  - [Uses Tia Tank sample application](https://github.com/industrial-edge/miscellaneous#tank-application)

# Get started

## Networks
  - PLANT-USER network 192.168.1.x/24 range
  - PLANT-OT network 192.168.0.x/24 range  
  - Devices:
    - PLC: 
      - 192.168.0.10
    - Edge Device: 
      - port 1: 192.168.1.10
      - port 2: 192.168.0.11
    - MYPC :
      - port: 192.168.1.11     

## OT - Level
  Run Tia tank project on PLC SIM Advanced, or use a real PLC - Use a 1500 plc [Link Tia Portal Project](https://github.com/industrial-edge/miscellaneous#tank-application)  Or use your own project.  
  Give this PLC ip adress in range of the OT-South network, for example 192.168.0.10

## PLANT - Level
Install the required apps on edge device
- Simatic s7 Connector 
- IE Databus 
- Energy Manager
- Data Service
- flow creator
 
Setup the network settings  on edge device
  - Give the Edge-Device Port 2 ip adress in range of the OT-South network, for example 192.168.0.11
  - Give the Edge-Device Port 1 ip adress in range of the South-North network, for example 192.168.1.10

### Simatic s7 Connector
1. Open the Industrial Edge Management - Go to Data Connections - Select the Simatic S7 connector
![s7connector1](files/edgedevice-s7-connector-1.JPG)
2. Launch on the Southbound device - select S7 or OPCUA (we use opcua) - add data Source 
![s7connector2](files/edgedevice-s7-connector-2-add-opcua.JPG)
3. Fill in the ip adress and port 192.168.0.10 port 4840 and save
4. Set the settings - use username: edge and password: edge, then press save.
![s7connector3](files/edgedevice-s7-connector-3-settings.JPG)
5. A new row should be available in the list, press browse tags, all the tags should be read from the datasource. add all from the database GBD.
![s7connector3](files/edgedevice-s7-connector-4-browse.JPG)
6. Deploy and start project, wait until done.

### IE Databus
1. Open the Industrial Edge Management - Go to Data Connections - Select the IE Databus
2. Launch on the Southbound device 
3. Add user + 
![iedatabus1](files/edgedevice-ie-databus-1.JPG)
4. Topic: ie/#, username: edge, password: edge, permission: publish and subscribe, click on add.
5. Deploy, wait until its done

### Flow Creator
1. open flow creator - on edge device, login with edge credentials 
2. add mqtt in node
3. add server: 
    - server: ie-databus
    - port: 1883
    - security - user: edge
    - security - password: edge
    - click on save
    - ![flowcreator1](files/edgedevice-flow-creator-1.JPG)
4. set topic:
    - ie/#
    - click on done.
5. add message node and connect, then deploy.
6. check if data is flowing in debug window.
![flowcreator2](files/edgedevice-flow-creator-2.JPG)


### Edgedevice - Dataservice 
1. Open Dataservice - on edge device.
2. Go set the adapter, connect the s7 adapter
![Dataservice](files/edgedevice-dataservice-1.JPG)
3. Fill in fields:
   - name: s7 connector (default)
   - url: tcp://ie-databus:1883
   - name: edge
   - password: edge
   - metadata: ie/d/j/simatic/v1/s7c1/dp/   (default)
4. Click save
5. and enable the new adapter, check if it is connected.
6. Add child asset to edge : "Plant" 
7. Add child asset to plant : "Energy" 
![Dataservice](files/edgedevice-dataservice-2.JPG)
8. Click on Energy Asset, and add multiple variables
9. Type in search: GDB.signals.energy
10. Select all the 4 tags. and save. > accept.


### Edgedevice - Energy Manager
1. Open Energy manager - on edge device
2. Create KPIs - add new KPI

![EnergyManager](files/energymanager-1.JPG)

    - Create KPI: name KPI_Total_Energy
        - Unit: Wh
        - Fill in Formula: operand EnergyFillingTank + Operand EnergyHeatingTank + Operand EnergyFIllingBottles

![EnergyManager2](files/energymanager-2.JPG)

    - Create KPI: name KPI_Costs_energy
        - Unit: $ or euro
        - Formula: Operand TotalEnergy * constant 0.35

![EnergyManager3](files/energymanager-3.JPG)

    - Create KPI: name KPI_Costs_Water
        - Unit: $ or euro
        - Formula: Operand TotalEnergy * constant 0.21

![EnergyManager4](files/energymanager-4.JPG)


3. Create KPI Instance
   - Select My plant
   - go to the Energy - Asset
   - click on the dropdown and click on parameter

![EnergyManager4](files/energymanager-7.JPG)

   - add new KPI instance
   - name: INSTANCE_Total_Energy
   - KPI basis:  On KPI type
   - KPI Type: KPI_Total_Energy
   - Connect EnergyFillingTank -> WaterConsumptionFillingTank
   - Connect EnergyHeatingTank -> ConsumptionHeatingTank
   - Connect EnergyFillingBottles -> ConsumptionFillingBottles

   ![EnergyManager4](files/energymanager-8.JPG)

3. Open menu > My Plant
    - open tab > Plant > Energy> create dashboard
    - give it a name.

![EnergyManager4](files/energymanager-5.JPG)

4. Heatmap Option 1
    - select the tab "My Plant"
    - go to the newly created dashboard
    - select "Create first widget" to start the wizard
        - choose the Heatmap widget type > Continue
        - enter a widget name and set the KPI calculation period to 1 Minute > Continue
        - click "Select parameter" and choose the KPI instance for 'Total energy' > Continue
        - no need to change anything in the general display options > Continue
        - no need to change anything in the Heatmap display options > Finish

![EnergyManager4](files/energymanager-6.JPG)


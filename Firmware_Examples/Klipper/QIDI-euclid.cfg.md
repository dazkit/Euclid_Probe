## euclid.cfg
# 0,0 is center of bed
# Movement Locations to work out
# Pre-flight position X0 Y-70
# Dock Adjacent  position X-125 Y-74
# Probe pikcup over dock X-150 Y-74
# Dock exit Position X-150 Y-30
# Probe Ready Position X0 Y-75

[probe]
##    Euclid Probe
pin: ^P1.24 # ZMax on SKR 1.1
x_offset: -22
y_offset: -2.0
z_offset: 11.57
speed: 2
samples: 1
samples_result: median # average or median
sample_retract_dist: 13.0
samples_tolerance: 0.050
samples_tolerance_retries: 1
lift_speed: 75

[bed_mesh]
speed: 125
horizontal_move_z: 15
mesh_min: -105,-60
mesh_max: 85,70
probe_count: 5,5


[gcode_macro preflight]
gcode:
  G90
  G0 X0 Y-70 Z15     ; Step 1, Pre-flight location X100 Y20

[gcode_macro move_to_dockside]
# Location adjacent to dock X-125 Y-74
gcode:
  G90                       ; absolute coordinate system
  G0 X-125 Y-74 F4800       ; move to Dock Adjacent
  G4 P500                   ; wait 1/2 second

[gcode_macro move_to_dock]
# Dock position at X-150 Y74 Z0
gcode:
  G90                       ; absolute coordinate system
  G0 X-150 Y-74 F1200       ; STEP 2, move to Dock
  G4 P500                   ; wait 1/2 second                  

[gcode_macro move_outof_dock]
# Location adjacent to dock for exit X+40 
gcode:
  G90                       ; absolute coordinate system
  G0 X-150 Y0 F4800         ; move to X93 Y350 Z3.7

[gcode_macro swipe_off]
# Location adjacent to dock X-125 Y-74
gcode:
  G90                       ; absolute coordinate system
  G0 X0 Y-70 F7200          ; Step 1, Pre-flight
  G4 P500                   ; wait 1/2 second

[homing_override]
gcode:
    G28 X
    G28 Y
    G28 Z
    M400
    G1 Z15


# Macro to Deploy Bed Probe
[gcode_macro M401]
gcode:
    G90
     {action_respond_info("Entering M401")}
    error_if_probe_deployed
    _M401
    # {action_respond_info("Exiting M401")}


[gcode_macro error_if_probe_deployed]
gcode:
    G4 P300
    QUERY_PROBE
    do_error_if_probe_deployed

[gcode_macro do_error_if_probe_deployed]
gcode:
    {% if not printer.probe.last_query %}
      {action_raise_error("Euclid Probe is already deployed - Remove and Return it to the dock")}
    {% endif %}


# Macro to deploy Bed Probe
[gcode_macro _M401]
gcode:
    G0 Z15 F3000          ;  set approach elevation of Z10
    preflight
    move_to_dockside      ;  
    move_to_dock          ;  translate over probe pickup location
    M400
    G4 P250               ;  pause for firmware detection
    move_outof_dock       ;  translate to side to exit dock   
    G0 Z15 F1200          ;  raise to elevation of Z20
    M400                  ; wait for moves to finish
    error_if_probe_not_deployed
    {action_respond_info("Exiting M401")}

[gcode_macro error_if_probe_not_deployed]
gcode:
    G4 P300
    QUERY_PROBE
    do_error_if_probe_not_deployed

[gcode_macro do_error_if_probe_not_deployed]
gcode:
    {% if printer.probe.last_query %}
      {action_raise_error("Euclid Probe Unsuccessfully Deployed!")}
    {% endif %}

# Macro to retract Bed Probe
[gcode_macro M402]
gcode:
    G90
    {action_respond_info("Entering M402")}
    error_if_probe_not_deployed
    _M402
    # {action_respond_info("Exiting M402")}

# Macro to Deploy Bed Probe
[gcode_macro _M402]
gcode:
    G90
    G1 X0 Y0 F4800
    move_outof_dock       ;  translate to front of the dock
    move_to_dock          ;  move into the dock
    M400                  ; wait for moves to finish
    G4 P250               ;  pause for firmware detection
    swipe_off             ;  translate over probe pickup location
    G0 Z15 F1200          ;  raise to elevation of Z20
    M400                  ; wait for moves to finish
    error_if_probe_deployed
    # {% endif %}
    {action_respond_info("Exiting M402")}

# Macro to perform a modified bed mesh calibration
[gcode_macro BED_MESH_CALIBRATE]
rename_existing:    BED_MESH_CALIBRATE_ORIGINAL
gcode:
  M401                           ; deploy Euclid Probe if needed
  BED_MESH_CALIBRATE_ORIGINAL    ; check bed level
  M402                           ; dock Euclid Probe
  

[gcode_macro G29]
gcode:
    M401
    BED_MESH_CALIBRATE_ORIGINAL
    M402

[gcode_macro GO_HOME]
gcode:    
    G91
    G1 Z15
    G90
    G1 X115 Y80 f4800
    G91
    G1 Z-15
    G90
    
[gcode_macro GO_CENTER]
gcode:    
    G91
    G1 Z15
    G90
    G1 X0 Y0 F4800
    G91
    G1 Z-15
    G90
    
[gcode_macro GO_PARK]
gcode:    
    G91
    G1 Z15
    G90
    G1 X150 Y-75 F4800 ; park at limits
    G91
    G1 Z-15
    G90
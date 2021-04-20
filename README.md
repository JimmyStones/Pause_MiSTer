# Pause_MiSTer [![Twitter](https://img.shields.io/twitter/url/https/twitter.com/mrjimmystones.svg?style=social&label=Follow%20%40mrjimmystones)](https://twitter.com/mrjimmystones) <span class="badge-buymeacoffee"><a href="https://ko-fi.com/jimmystones" title="Buy Me a Coffee at ko-fi.com'"><img src="https://img.shields.io/badge/buy%20me%20a%20coffee-donate-yellow.svg" alt="Buy Me a Coffee at ko-fi.com'" /></a></span>

This is a generic module for MiSTer arcade cores which simplifies pausing the CPU based on user input, OSD status and signals from the high score module.

Created by Jim Gregory ([JimmyStones](https://github.com/jimmystones))

## Features
- Pause can be triggered by user input, hiscore module or OSD opening (optionally controlled by setting in OSD)
- When paused the RGB outputs will be halved after 10 seconds to reduce burn-in (optionally controlled by setting in OSD)
- Reset signal will cancel user triggered pause

### History
#### 0001 - 2021-04-17
- First marked release

## Implementation instructions

- Add pause.v to ```rtl/``` folder and files.qip
- Add the following code sections to the main core .sv file.

### Add Pause options inside the CONF_STR
```verilog
	"-;",
	"P1,Pause options;",
	"P1OP,Pause when OSD is open,On,Off;",
	"P1OQ,Dim video after 10s,On,Off;",
	"-;",
```
_P & Q are the default bits used by the module which equate to status[26:25], these can be changed here and in the module ports if there is a collision which existing settings_

### Add pause button to input defaults
```verilog
	"J1,Fire,Fast,Start P1,Coin,Start P2,Pause;",
	"jn,A,B,Start,R,Select,L;",
```

### Add pause input signal
```verilog
wire m_pause  = joy[9];
```
_Input signal names will vary between cores._

### Instantiate Pause module
```verilog
// PAUSE SYSTEM
wire				pause_cpu;
wire [11:0]		rgb_out;
pause #(4,4,4,12) pause (
	.*,
	.clk_sys(CLK_12M),
	.reset(iRST),
	.user_button(m_pause),
	.pause_request(hs_pause),
	.options(~status[26:25])
);
```
_The .* wildcard will connect all ports with matching names.  In the above example clk_sys and reset are specified as the core is using different names for those signals._

See [Module parameters](#Module-parameters) for further details.

### Core specific implementation

The pause_cpu signal should be passed through the the main core module and used to halt any relevant CPUs or other processes.

### Module parameters

| Parameter | Description 
| ----------| -----------
| RW        | Width of red channel
| GW        | Width of green channel
| BW        | Width of blue channel
| CLKSPD    | Main clock speed in MHz

### Module ports
| Port           | Direction | Description 
| -------------- | --------- | ----------- 
| clk_sys        | in        | Core system clock (should match HPS module)
| reset          | in        | CPU reset signal (active-high)
| user_button    | in        | User pause button signal (active-high)
| pause_request  | in        | Pause requested by other code (active-high)
| options        | in        | Pause options from OSD.  [0] = pause in OSD (active-high), [1] = dim video (active-high)
| OSD_STATUS     | in        | OSD is open (active-high)
| r              | in        | Red channel
| g              | in        | Green channel
| b              | in        | Blue channel
| pause_cpu      | out       | Pause signal to CPU (active-high)
| rgb_out        | out       | RGB output to arcade_video moduleaccess

## License

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

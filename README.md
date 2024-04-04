
The repo contains the KiCad files for manufacturing the printed circuit board and front panel for a 4-channel Eurorack active mixer, based on the one Hagiwo assembled by hand [here](https://note.com/solder_state/n/nffc1a33053be). This project is a simpler precursor to building a Eurorack module with a Daisy Seed, in order to hone my skills and go one-time through the PCB manufacture process.

# Still to pay attention to! (A.K.A. the TODO list) 

- do the connections on my pots go downwards onto the board?
- the thonkiconn screws should also how 
- component heights should be such that they all poke through the front panel at the same height (check with 3D model)
- Note: our screw holes on the panel template are not perfectly placed! complete the mounting holes section in the table below

# Aims / Design Constraints

For this design I am aiming to:
- use through-hole components only
- mount the pots and jacks directly onto the PCB
    - as opposed to just securing them to the panel and wiring each individually to the board (as in the original Hagiwo design)
- place all component on a single board (no stacking)
    - this will limit the number of inputs to 4 (instead of Higawo's 5)
- also design the module front panel as a PCB and have it manufactured at the same time

# Components / Bill of Materials (BOM)

- 4 x 10K potentiometers Alpha 9mm (inputs)
- 1 x 100K potentiometer Alpha 9mm (output)
- 6 x [Thonkiconn 3.5mm jack sockets](https://www.thonk.co.uk/shop/thonkiconn/)
- 1 x TL072 dual op-amp (integrated circuit in PDIP package)
- 1 x 2x5 Pin Shrouded Header (Eurorack power connector)
- 5 x 100K resistors
- 1 x 22K resistor
- 1 x 4.7K resistor
- 1 x 1K resistor
- 2 x 4.7uF electrolytic capacitors

# Workflow

1. [Design the schematic](#1-design-the-schematic)
2. [Simulate the circuit (TODO)](#2-simulate-the-circuit-todo)
3. [Design Eurorack board templates](#3-design-eurorack-board-templates)
4. [Assign footprints and place components](#4-assign-footprints-and-place-components)
5. [Route traces](#5-route-traces)
6. [Design the front panel](#6-design-the-front-panel)
7. [Produce gerber files](#7-produce-gerber-files)
8. [Send to manufacturer (TODO)](#8-send-to-manufacturer-todo)

# 1. Design the schematic

I am following [Hagiwo's](https://note.com/solder_state/n/n54e016f19c94) hand draw schematic, as referenced here:

![Higawo hand-drawn schematic](https://assets.st-note.com/production/uploads/images/57501531/picture_pc_83b4ace226875c162b7daab2bf8e85d0.jpg?width=800)

The [Doepfer schematic](https://www.doepfer.de/DIY/mixer.gif) detailed on [this page](https://www.doepfer.de/DIY/a100_diy.htm) is also useful for comparison, but includes some extra details and differences.

## How the mixer works

To understand the schematic, I found it helped to describe it briefly in terms of signal flow:
1. We attentuate the incoming signal from each jack input using a potentiometer
2. The signals are then mixed together and fed to the input of an operational amplifier
3. The op amp attentuates the signal by a fixed level (to compensate for the volume increase of the combined inputs)
4. A potentiometer across second op amp acts as a master volume, attentuating the mixed signals
5. The attentuated output goes to the jack output

## Jacks

I'm using `AudioJack2_SwitchT` as the jack symbol on the schematic. It has only tip and sleeve, where:
- the tip is the "hot" pin which should carry the audio signal to the potentiometer input
- the sleeve goes to ground

## Power

Designing the mixer helped me get to grips with how to supply power to a module from a Eurorack power supply.

Key points:
- Power comes in via a 10-pin shrouded power header
    - Shrouded means that the cable can only be inserted one way
    - The symbol for this can be found in the [eurocad library](https://github.com/nebs/eurocad) under the name `EURO_PWR_2x5`
    - From the 10 pins it is [standard practice](https://pichenettes.github.io/mutable-instruments-documentation/modules/plaits/downloads/plaits_v50.pdf) to short together:
        - the two +12v rails
        - the two -12v rails
        - the six ground pins
- The +-12V rails are used to power our TL072 dual op-amp chip
    - When placing the TL072 dual op-amp schematic symbol, we must click three times
        - twice for each individual op amp
        - once for the power part of schematic
- 2 x 4.7uF electrolytic capacitors are used between the +12V and -12V rails and ground
    - their purpose is to smooth out unexpected small fluctuations in voltage

- some useful principles of voltage control and power in Eurorack are also covered [here](https://doepfer.de/a100_man/a100t_e.htm)

# 2. Simulate the circuit (TODO)

The aim here is to check that the signals meet Eurorack output levels

Key points:
- we will use Spice simulation
- [advice](https://www.reddit.com/r/KiCad/comments/1961s6p/using_the_spice_simulator_in_kicad_on_a_guitar/) is to replace potentiometers with fixed value resistors and vary the values between simulation passes
    - So check for a given input that we get an expected output
    - But what inputs to use?
- the schematic needs to be reworked to be suitable for a PSPICE simulation (It isn't as simple as "hit run, get a waveform.")
- AC Sweep is probably the analysis method to use

# 3. Design Eurorack board templates

I found it important to first design superimposed templates for both the front panel and main PCB. This is because the holes in the front panel template will be derived directly from the centre hole positions of the jacks and pots on the main board.

My starting point was this repo of [Euroack KiCad project templates](https://github.com/clacktronics/Eurorack_KiCAD_templates). Unfortunately, there was no 6HP template, so I used the opportunity to design my own.

The width of Eurorack modules is measured in HP (Horizontal Pitch), where one unit of HP is 5.08mm wide (0.2 of a an inch). Fortunately our KiCad grid already has a setting for this.

I started by defining three templates, using the original [Doepfer specifications](https://www.doepfer.de/a100_man/a100m_e.htm). This [online converter](https://gcoulby.github.io/Eurorack-Unit-HP-Converter/) was also useful.

## 1. Front panel template
- this is the exact 6HP width and normal height i.e. 30.48mm x 128.5mm
- it is helpful to keep everything on a HP grid
- i used a dotted outline and placed it on the User Drawings layer, as it won't be used for manufacture

## 2. Front panel itself
- this is 0.24mm narrower either side and the same height i.e. 30.00mm x 128.50mm
- it also includes two oval screw mounting holes (enough for modules < 12HP):
    - the horizontal centre of which should be 1HP in from the edge
    - the vertical centre of which should be 3mm from the edge
    - diameter should be 3.2mm
    - width of the oval should be 5.2mm
- I placed it on the Edge Cuts layer, as it represents the milled board outline

## 3. Main board
- this is 0.5mm narrower still either side than the front panel and not as tall i.e. 29.00m x 106mm
- the reduction in height is necessary so that the board passes the mounting rails
- I placed it on the Edge Cuts layer, as it represents the milled board outline

In KiCad, rectangles are specified using x and y start and end points. Here are points for all the elements mentioned above:

| Item                | X Start    | X End       | Y Start   | Y End       |
|---------------------|------------|-------------|-----------|-------------|
| Front Panel Template| 0          | 30.48       | 0         | 128.5       |
| Front Panel         | 0.24       | 30.24       | 0         | 128.5       |
#| Mounting Hole 1     | 5.08 - 2.6 | 5.08 + 2.6  | 3.2 - 1.5 | 3.2 + 1.5   |
#| Mounting Hole 2     | 5.08 - 2.6 | 5.08 + 2.6  | 3.2 - 1.5 | 3.2 + 1.5   |
| Main Board          | 0.74       | 29.74       | 11.25     | 117.25      |

Once all three templates were complete, I grouped them all together to ensure they are not accidentally moved out of position relative to one another. It is also possible to snap different elements within the design to grid points, to aid relative placement of components.

# 4. Assign footprints and place components

## Assign Footprints

Before the components can be laid out on the PCB, a footprint for each must be given in the schematic. Below are the footprints I used for some key components, some of of which are taken from the [eurocad library](https://github.com/nebs/eurocad)

| Component Name         | Footprint Name                |
|------------------------|-------------------------------|
| 3.5mm Jack Sockets     | eurocad:PJ301M-12_T_S        |
| Potentiometers         | eurocad:Alpha9mmPot          |
| TL072 Dual Op Amp      | Package_DIP:DIP-8_W7.62mm    |
| Shrouded Power Header  | Connector_IDC:IDC-Header_2x05_P2.54mm_Vertical |

## Placing Components

There are tools available purely for doing front panel design, such as [Front Panel Designer](https://www.frontpanelexpress.com/front-panel-designer) and [Synth Panels Designer](https://cdm.link/2020/07/synth-panels-designer-makes-diy-panels-free/). These could help to define the ideal component placement in future,but this time round I just placed by hand using the main board height as the core constraint.

Key points:
- Placement was done within the bounds of the main board template
- The height of the board (106mm) limits the design to five rows of jack potentiometer pairs in total
- If the components were not mounted on the board directly
    - an extra row could be squeezed in
    - or the existing rows could be better spaced 
- I followed this simple design constraint:
    - For input rows: Jacks on the left, pots on the right
    - For output row: Jack on the right, pot on the left
- The jack and potentiometer pairs were placed at equal vertical distance from one another
    - The KiCad measurement tool came in handy to check this
- The remaining components were placed in the spaces between the pairs, and the power connector at the bottom

# 5. Route traces

Key points:
- We use only two layers (front and back)
- The PCB component holes are 'vias' which allow the traces to be routed between the two layers
- I followed a simple 'ground plane' convention to decide which traces to put on which of the layers, which states that all connections that go to ground should be on one side of the board (the back) and all other connections on the other side (the front)
- I used thicker traces of 0.5mm for the power traces leading from the power header to the capacitors and op amp. This I had seen recommended in other designs.
- For all other traces I used the default thickness of 0.25mm

# 6. Design the front panel

Designing the front panel was a simple matter of:
1. Creating mounting holes whose centre points match those of the circles on the jack and pot footprints
2. Moving the panel template, plus mounting holes to one side (still on the grid)
3. Adding text, row markers and logos as drawings to the silkscreen layer
4. Mirroring the design at the end, so that the silk screening is visible when we flip the panel over to place it on top of the pot/jack shafts

Key points:
- To make the mounting holes, I oriented the grid so that the centre of the existing components was directly on a grid boundary
- The diamater of the mounting holes was as follows
    - Alpha 9mm pots: 7mm
    - Thonkiconn jacks: 6mm
- To make the holes, use the "Add a footprint" icon and select searching for `MountingHole`
    - Unlike plain circles, this makes sure they actually get included in the Gerber drill files
    - Note: Mounting hole footprints are measured in radius, not diameter, so I used 3.5mm and 3mm variants
- For cooler visual effects, it is also possible to draw on the solder mask layer or experiment with surface finishes (e.g. ENIG vs HASL)

# 7. Produce Gerber files

Once all the above has been done, we can temporarily delete the front panel elements to produce the PCB Gerber files and vice versa.

Key points:
- Go to `File Menu -> Plot` to start the export process for each board
    - Click `Generate Drill Files` - this should create a set of .drl files
    - Then click `Plot` - this should create a .gbr file for each KiCad layer
- Both the drill and Gerber files can (and should!) be checked in a Gerber viewer to ensure they are correct. I used `gerbv` i.e.
    - `sudo apt install gerbv`
- You may need to copy out the first set of .gbr and .drl files before producing the second.

# 8. Send to manufacturer (TODO)

To be decided...


## Links
https://www.ti.com/product/TL072#design-development  

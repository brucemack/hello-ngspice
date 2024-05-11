Tracing Device Parameters for Sky130 NMOS 
=========================================

Inside of your SPICE netlist you will see (for example) the subcircuit 
called "sky130_fd_pr__nfet_01v8" referenced:

        XMn1 vdd vg vss vss sky130_fd_pr__nfet_01v8 
        + L=0.30 W=1.5 nf=1 
        + ad='int((nf+1)/2) * W/nf * 0.29' 
        + as='int((nf+2)/2) * W/nf * 0.29' 
        + pd='2*int((nf+1)/2) * (W/nf + 0.29)'
        + ps='2*int((nf+2)/2) * (W/nf + 0.29)' 
        + nrd='0.29 / W' 
        + nrs='0.29 / W' 
        + sa=0 sb=0 sd=0 mult=1 m=1

This subcircuit is defined externally.  The linkage to the subciruit library is created
by this line that appears at the top of the SPICE netlist:

        .lib /home/ttuser/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice tt

The "tt" in the .lib statement is important because it controls te part of the 
included .spice file that is used.  We see this in sky130.lib.spice:

        * Typical corner (tt)
        .lib tt
        .param mc_mm_switch=0
        .param mc_pr_switch=0
        * MOSFET
        .include "corners/tt.spice"
        * Resistor/Capacitor
        .include "r+c/res_typical__cap_typical.spice"
        .include "r+c/res_typical__cap_typical__lin.spice"
        * Special cells
        .include "corners/tt/specialized_cells.spice"
        .endl tt

Notice the ".lib tt" statement is selected by the "tt" parameter of the original .lib statement.

The .include "corners/tt.spice" causes a file to be included using a relative path
(i.e. relative to the file that included it).  So we look into this file:

        /home/ttuser/pdk/sky130A/libs.tech/ngspice/corners/tt.spice

Among other things, the tt.spice file includes this line:

        .include "../../../libs.ref/sky130_fd_pr/spice/sky130_fd_pr__nfet_01v8__tt.pm3.spice"

That causes us to pull in this file:

    /home/ttuser/pdk/sky130A/libs.tech/ngspice/corners/../../../libs.ref/sky130_fd_pr/spice/sky130_fd_pr__nfet_01v8__tt.pm3.spice

Which contains this section:

        .subckt  sky130_fd_pr__nfet_01v8 d g s b
        +
        .param  l = 1 w = 1 nf = 1.0 ad = 0 as = 0 pd = 0 ps = 0 nrd = 0 nrs = 0 sa = 0 sb = 0 sd = 0 mult = 1
        msky130_fd_pr__nfet_01v8 d g s b sky130_fd_pr__nfet_01v8__model l = {l} w = {w} nf = {nf} ad = {ad} as = {as} pd = {pd} ps = {ps} nrd = {nrd} nrs = {nrs} sa = {sa} sb = {sb} sd = {sd}
        .model sky130_fd_pr__nfet_01v8__model.0 nmos
        * Model Flag Parameters
        + lmin = 2.0e-05 lmax = 0.0001 wmin = 7.0e-06 wmax = 0.0001
        + level = 54.0
        + version = 4.5
        ...

This subcircuit makes use of a MOSFET component called "msky130_fd_pr__nfet_01v8" which references 
to a model called "sky130_fd_pr__nfet_01v8__model".

You will notice that there are actully multiple models in the .subckt definition.  Here we make use
of the "binning" feature of NGSpice to select the correct model **based on the geometry of the MOSFET.**
The lmin/lmax/wmin/wmax parameters of the model are compared to the l/w parameters passed to the 
component to decide which model to use.  This mechanism is described [in this document](https://ngspice.sourceforge.io/docs/ngspice-html-manual/manual.xhtml) in section 3.2.3, excerpted here:

> **3.2.3 Model Binning**: Binning is a kind of range partitioning for geometry dependent models like MOSFET's. The purpose is to cover larger geometry ranges (Width and Length) with higher accuracy than the model built-in geometry formulas. Each size range described by the additional model parameters LMIN, LMAX, WMIN and WMAX has its own model parameter set. These model cards are defined by a number extension, like `nch.1'. ngspice has an algorithm to choose the right model card by the requested W and L.
This is implemented for BSIM3 (11.2.10) and BSIM4 (11.2.11) models.

In this example, the geometry of the MOSFET is L=0.3um, W=1.5um.  Searching through the various models
available in the sub-circuit we find this one that mataches the L/W range of our component:

        .model sky130_fd_pr__nfet_01v8__model.51 nmos
        * Model Flag Parameters
        + lmin = 2.5e-07 lmax = 5.0e-07 wmin = 1.26e-06 wmax = 1.68e-6
        + level = 54.0
        + version = 4.5
        + binunit = 2.0
        + mobmod = 0.0
        + capmod = 2.0
        + igcmod = 0.0
        + igbmod = 0.0
        ...
        + toxm = 4.148e-9
        ...
        + vth0 = {5.989188996e-01+MC_MM_SWITCH*AGAUSS(0,1.0,1)*(sky130_fd_pr__nfet_01v8__vth0_slope/sqrt(l*w*mult))} 
        ...
        + u0 = 5.344024449e-02


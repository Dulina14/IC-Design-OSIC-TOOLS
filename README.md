# IC-Design-OSIC-TOOLS
## Digital IC Implementation of 'AXIS UART MATRIX VECTOR MULTIPLIER' using OSIC Tools and IHP SG13G2 PDK

**Consolidated Project Report**
**Date:** [Insert Date]
**Author:** [Your Name/Group]

---

**Table of Contents:**
1.  Introduction
2.  Background
3.  Tools and Environment Setup
4.  Design Flow and Execution
    4.1. RTL Preparation & Synthesis
    4.2. Floorplanning
    4.3. Placement (Global and Detailed)
    4.4. Clock Tree Synthesis (CTS)
    4.5. Routing (Global and Detailed)
    4.6. Post-Routing and Finishing
    4.7. GDSII Generation
5.  Results
    5.1. Layout Generation
    5.2. Timing Analysis
    5.3. Physical Verification (DRC, LVS-implied)
    5.4. Power Analysis
    5.5. Resource Utilization
    5.6. IR Drop Analysis
    5.7. Visual Layouts
6.  Conclusion and Future Work

---

### 1. Introduction

This report details the implementation of a digital design, referred to as "project," from Register Transfer Level (RTL) to a GDSII layout. The primary objective was to successfully execute a complete Application-Specific Integrated Circuit (ASIC) design flow using open-source Electronic Design Automation (EDA) tools provided within the OSIC (Open Source IC) ecosystem. Specifically, this project utilized the OpenROAD flow scripts (ORFS) targeting the IHP SG13G2 Process Design Kit (PDK). The design appears to be a system involving a Matrix Vector Multiplication (MVM) core and a UART interface, based on path names in the reports (e.g., `MVM_UART_SYSTEM`).

### 2. Background

The proliferation of open-source EDA tools has democratized IC design, enabling wider access to learning, research, and development. Tools like Yosys for synthesis and OpenROAD for physical design offer a complete RTL-to-GDSII flow. The IHP SG13G2 is a 130nm SiGe BiCMOS technology, and its open-source PDK allows for designing and potentially fabricating chips using these free tools.

A typical digital ASIC design flow involves:
1.  **RTL Design:** Describing hardware functionality in a Hardware Description Language (HDL) like Verilog or SystemVerilog.
2.  **Synthesis:** Converting the HDL code into a gate-level netlist using a standard cell library.
3.  **Floorplanning:** Defining the chip's overall area, aspect ratio, and placement of I/O pins and large macros.
4.  **Placement:** Placing standard cells onto the chip floor.
5.  **Clock Tree Synthesis (CTS):** Building a balanced clock distribution network.
6.  **Routing:** Connecting the placed cells and pins according to the netlist.
7.  **Post-Routing Optimization & Finishing:** Inserting filler cells, performing final timing checks, and metal fill.
8.  **Verification:** Ensuring the design meets functional, timing, and physical requirements (DRC, LVS).
9.  **GDSII Generation:** Creating the final layout file for fabrication.

### 3. Tools and Environment Setup

The project was executed within an environment leveraging OSIC tools, likely within a Docker container (e.g., `iic-osic-tools`). The setup involved:
*   **OSIC Tools Container:** Provides a pre-configured environment with necessary EDA tools.
*   **Repository Cloning:** The project files and ORFS flow scripts were cloned into the working directory.
*   **Key Tools Used:**
    *   **Yosys (v0.51):** For RTL synthesis.
    *   **OpenROAD (v2.0-19705-gbe6d1c688):** For the physical design (PnR) flow, including floorplanning, placement, CTS, routing, and final GDS generation steps. It internally uses OpenSTA for static timing analysis.
    *   **KLayout:** For GDSII viewing and merging standard cell GDS files with the routed DEF.
*   **PDK:** IHP SG13G2, utilizing the `sg13g2_stdcell_typ_1p20V_25C.lib` standard cell library and associated LEF/GDS files.

The flow was initiated using a shell script (`./run_all.sh`), which first cleaned previous runs (`./clean_all.sh`) and then executed the ORFS flow (`./run_orfs.sh`).

### 4. Design Flow and Execution

The `run_orfs.sh` script, driven by a Makefile, orchestrated the OpenROAD flow. An initial attempt during the execution log failed at the global placement stage due to excessively high utilization (224.428%). The flow was subsequently cleaned and re-run, this time successfully. The following describes the successful run:

**4.1. RTL Preparation & Synthesis (Yosys)**
*   The Verilog RTL file (`project.v`) was processed.
*   Initial Yosys canonicalization step (`synth_canonicalize.tcl`) was performed. Warnings related to continuous assignments to `reg` types (`s_packets`, `tx`, `s_ready`) and replacement of a memory (`\tree`) with registers were noted.
*   Full synthesis (`synth.tcl`) was executed, targeting the IHP SG13G2 standard cell library (`sg13g2_stdcell_typ_1p20V_25C.lib`).
*   The clock period was set to 125000 ps (125 ns, or 8 MHz).
*   The output was a gate-level netlist (`1_synth.v`) and an initial SDC file (`1_synth.sdc`).
*   The synthesis process reported **784 cells** in the initial netlist.

**4.2. Floorplanning (OpenROAD)**
*   LEF files (`sg13g2_tech.lef`, `sg13g2_stdcell.lef`) were loaded.
*   A warning "port 'clock_i' not found" was issued by STA, suggesting a potential mismatch or unhandled clock input at this stage.
*   STA reported 153 unclocked register/latch pins and 177 unconstrained endpoints.
*   Core area was initialized. For the successful run, a design area of **13519 µm²** with **61% utilization** was established after snapping to a 39-row core. (The prior failed run had snapped to 20 rows, leading to the high utilization error).
*   I/O pins were placed randomly (`place_pins -random`).
*   No macros were present, so macro placement was skipped.
*   Tap cells were inserted.
*   Power Distribution Network (PDN) was generated.
*   **Timing (Floorplan):** TNS: 0.00, WNS: 0.00, Worst Slack: INF.
*   **Power (Floorplan):** Total: 5.15e-06 W (Sequential: 90.7%, Combinational: 9.3%).

**4.3. Placement (Global and Detailed - OpenROAD)**
*   **Global Placement (Skip IO):** Initial global placement was run with a target density of 0.65.
*   **IO Placement:** I/O pins were placed again, this time considering netlist connectivity. HPWL for I/O nets was 1638.53 µm.
*   **Global Placement (Full):**
    *   Run with routability and timing-driven options.
    *   The timing-driven mode noted "no slacks found" and was disabled, likely due to the largely unconstrained nature of the design.
    *   Resizing steps (buffer insertion, gate sizing) were performed.
    *   **Timing (Global Place):** TNS: 0.00, WNS: 0.00, Worst Slack: INF.
    *   **Power (Global Place):** Total: 5.15e-06 W (Sequential: 90.6%, Combinational: 9.4%).
*   **Resizing:**
    *   Port buffering inserted 3 input and 1 output buffer.
    *   Design repair inserted 83 buffers and resized 90 instances to fix fanout violations.
    *   Tie cells (23 Lo, 38 Hi) were inserted.
    *   Instance count increased from 784 to **930**. Design area became **14776 µm² (67% utilization)**.
*   **Detailed Placement:**
    *   Legalized cell positions, improving HPWL by 10.4% (from 19512.8 µm to 17474.6 µm after initial legalization, then further minor improvement).
    *   Cell flipping and mirroring optimized HPWL.
    *   **Timing (Detailed Place):** TNS: 0.00, WNS: 0.00, Worst Slack: INF.
    *   **Physical Checks (Detailed Place):** No violations for max slew, max capacitance, or max fanout. Setup/Hold violation counts were 0.
    *   **Power (Detailed Place):** Total: 6.36e-06 W (Sequential: 72.3%, Combinational: 27.7%).

**4.4. Clock Tree Synthesis (CTS - OpenROAD)**
*   CTS was run with sink clustering and level balancing.
*   A critical warning "[WARNING CTS-0082] No valid clock nets in the design" was issued. This indicates that the defined clock `core_clock` (likely virtual at this point) was not actually driving any synthesizable clock tree, or no flip-flops were properly associated with it.
*   Design repair was run post-CTS.
*   **Timing (CTS):** TNS: 0.00, WNS: 0.00, Worst Slack: INF. Clock skew for `core_clock` reported "No launch/capture paths found."
*   **Physical Checks (CTS):** No violations for max slew, max capacitance, or max fanout. Setup/Hold violation counts were 0.
*   **Power (CTS):** Total: 6.36e-06 W (Sequential: 72.3%, Combinational: 27.7%). (Identical to detailed place power, as no clock tree was effectively built for power calculation).

**4.5. Routing (Global and Detailed - OpenROAD)**
*   **Global Routing:**
    *   Performed with congestion iterations. Final congestion report showed 0 overflow.
    *   Total wirelength estimated at 34444 µm.
    *   No antenna violations were found at this stage.
    *   Design repair (buffering/resizing) was performed again.
    *   **Timing (Global Route):** TNS: 0.00, WNS: 0.00, Worst Slack: INF.
    *   **Physical Checks (Global Route):** No violations for max slew, max capacitance, or max fanout. Setup/Hold violation counts were 0.
    *   **Power (Global Route):** Total: 6.70e-06 W (Sequential: 68.8%, Combinational: 31.2%).
*   **Detailed Routing (TritonRoute):**
    *   Iteratively routed the design, fixing violations.
    *   Final DRC check reported **0 violations**.
    *   Final wirelength: **20420 µm**.
    *   Total vias: **5419**.
    *   No antenna violations reported after detailed routing.

**4.6. Post-Routing and Finishing (OpenROAD)**
*   **Filler Cell Insertion:** 1014 filler instances (`sg13g2_fill_1`, `sg13g2_fill_2`, `sg13g2_decap_4`, `sg13g2_decap_8`) were placed.
*   **Density Fill (Metal Fill):** Metal fill was added to layers Metal1 through TopMetal2 to meet density rules.
*   **Parasitic Extraction (RCX):** Parasitics were extracted for final timing and power analysis. 5907 wires, 4757 RC segments, 4757 caps, 6267 coupling caps extracted.
*   **IR Drop Analysis (PDNSim):**
    *   VDD: Worst-case IR drop of 1.56e-06 V.
    *   VSS: Worst-case IR drop of 3.36e-06 V.
    *   Both negligible.
*   **Final Design Area:** 20824 µm², **94% utilization** (due to filler cells).
*   **Final Cell Count (including fillers):** 1944 instances.
    *   Detailed breakdown from final report:
        *   `sg13g2_a21o_1`: 9
        *   `sg13g2_a21oi_1`: 64
        *   `sg13g2_a221oi_1`: 4
        *   `sg13g2_a22oi_1`: 12
        *   `sg13g2_and2_1`: 8
        *   `sg13g2_and2_2`: 1
        *   `sg13g2_and3_1`: 5
        *   `sg13g2_and4_1`: 4
        *   `sg13g2_buf_1`: 67
        *   `sg13g2_buf_2`: 8
        *   `sg13g2_buf_4`: 2
        *   `sg13g2_buf_8`: 1
        *   `sg13g2_dfrbp_1`: 153 (Sequential elements)
        *   `sg13g2_inv_1`: 39
        *   `sg13g2_mux2_1`: 53
        *   `sg13g2_nand2_1`: 45
        *   `sg13g2_nand2_2`: 1
        *   `sg13g2_nand2b_1`: 13
        *   `sg13g2_nand3_1`: 17
        *   `sg13g2_nand3b_1`: 12
        *   `sg13g2_nand4_1`: 16
        *   `sg13g2_nor2_1`: 72
        *   `sg13g2_nor2_2`: 2
        *   `sg13g2_nor2b_1`: 22
        *   `sg13g2_nor3_1`: 10
        *   `sg13g2_nor4_1`: 1
        *   `sg13g2_nor4_2`: 1
        *   `sg13g2_o21ai_1`: 70
        *   `sg13g2_or2_1`: 7
        *   `sg13g2_or3_1`: 2
        *   `sg13g2_or4_1`: 2
        *   `sg13g2_tiehi`: 1 (initial, more added by resizer)
        *   `sg13g2_tielo`: 1 (initial, more added by resizer)
        *   `sg13g2_xnor2_1`: 44
        *   `sg13g2_xor2_1`: 15
        *   (Plus buffer cells inserted during PnR and filler cells)
*   **Timing (Finish):** TNS: 0.00, WNS: 0.00, Worst Slack: INF.
*   **Physical Checks (Finish):** No violations for max slew, max capacitance, or max fanout. Setup/Hold violation counts were 0.
*   **Power (Finish):** Total: **7.08e-06 W** (Sequential: 65.4%, Combinational: 29.4%, Clock: 5.2%, Leakage: 7.7% of total). The clock power is now non-zero, likely due to RC extraction attributing some leakage to clock nets/buffers even if not fully switching.

**4.7. GDSII Generation (KLayout)**
*   The final DEF file (`6_final.def`) was merged with the standard cell GDS (`sg13g2_stdcell.gds`) using KLayout's `def2stream.py` script.
*   The output was `6_final.gds`.

### 5. Results

The project successfully completed the RTL-to-GDSII flow.

**5.1. Layout Generation**
A GDSII file (`6_final.gds`) representing the physical layout of the "project" design was successfully generated.

**5.2. Timing Analysis**
*   **TNS (Total Negative Slack):** 0.00 ps across all stages.
*   **WNS (Worst Negative Slack):** 0.00 ps across all stages.
*   **Worst Slack:** INF (Infinity) across all stages. This indicates that while there are no violations on *constrained* paths, many paths in the design remain unconstrained.
*   **Unconstrained Paths:** Examples were reported throughout the flow.
    *   Floorplan Stage: `ui_in[0]` to `MVM_UART_SYSTEM.UART_RX.m_data[15]$_DFFE_PN0P_` (data arrival 25.09 ns).
    *   Finish Stage: `rst_n` to `MVM_UART_SYSTEM.AXIS_MVM.SKID.m_data[11]$_DFFE_PN0P_` (data arrival 25.91 ns).
    *   The consistent "No paths found" for `report_checks -path_delay min/max` and `reg to reg` checks further confirms the lack of comprehensive constraints.
*   **Clock Skew:** The `core_clock` reported "No launch/capture paths found" or "No valid clock nets" throughout CTS and later stages, implying it was not properly synthesized as a propagating clock tree for timing analysis of synchronous paths.

**5.3. Physical Verification (DRC, LVS-implied)**
*   **DRC (Design Rule Checks):**
    *   Max Slew Violation Count: 0
    *   Max Fanout Violation Count: 0
    *   Max Capacitance Violation Count: 0
    *   Setup Violation Count: 0
    *   Hold Violation Count: 0
    *   Detailed router reported 0 DRC violations in the final layout.
*   **LVS (Layout vs. Schematic):** While not explicitly run as a separate step in the provided logs, the OpenROAD flow's correctness and KLayout's GDS merge imply a structurally correct netlist was physically implemented. A formal LVS check would be a recommended next step.

**5.4. Power Analysis (Finish Stage)**
*   **Total Power:** 7.08e-06 Watts
    *   Internal Power: 5.16e-06 W (72.9%)
    *   Switching Power: 1.37e-06 W (19.4%)
    *   Leakage Power: 5.45e-07 W (7.7%)
*   **Breakdown by Component Type (Finish Stage):**
    *   Sequential: 4.63e-06 W (65.4%)
    *   Combinational: 2.08e-06 W (29.4%)
    *   Clock: 3.69e-07 W (5.2%)
    *   Macro/Pad: 0.00 W

**5.5. Resource Utilization**
*   **Total Instances (Final, including fillers):** 1944
    *   Standard Cells (excluding fillers, post-PnR): 930
    *   Sequential Cells (sg13g2_dfrbp_1): 153
*   **Design Area:** 20824 µm²
*   **Core Utilization (Final):** 94% (Initial floorplan was 61% for standard cells before fillers).
*   **Total Wirelength (Detailed Route):** 20420 µm
*   **Total Vias (Detailed Route):** 5419

**5.6. IR Drop Analysis**
*   **VDD Net:**
    *   Average IR Drop: 1.32e-06 V
    *   Worst-case IR Drop: 1.56e-06 V (Negligible)
*   **VSS Net:**
    *   Average IR Drop: 3.05e-06 V
    *   Worst-case IR Drop: 3.36e-06 V (Negligible)

**5.7. Visual Layouts**

*   **Overall Final Layout:**
    `[IMAGE: final_all.png - Placeholder for the complete GDSII layout view]`

*   **Placement View:**
    `[IMAGE: final_placement.png - Placeholder for the cell placement view, pre-routing]`

*   **Clock Network (Conceptual, as CTS had issues):**
    `[IMAGE: final_clocks.png - Placeholder for clock net highlighting, if available]`

*   **Routing Detail:**
    `[IMAGE: final_routing.png - Placeholder for a zoomed-in view showing routing tracks]`

*   **IR Drop Map:**
    `[IMAGE: final_ir_drop.png - Placeholder for VDD/VSS IR drop heatmaps]`

*   **Resizer Impact (Conceptual):**
    `[IMAGE: final_resizer.png - Placeholder, could show highlighted buffered nets or resized cells if identifiable]`

### 6. Conclusion and Future Work

This project successfully demonstrated the capability of the OSIC open-source tools, particularly the OpenROAD flow with the IHP SG13G2 PDK, to take a Verilog design ("project") from RTL to a GDSII layout. The flow completed without DRC violations, and the final design showed negligible IR drop.

**Key Achievements:**
*   Successful execution of the entire RTL-to-GDSII flow using open-source tools.
*   Generation of a DRC-clean layout.
*   Achieved 0 TNS and 0 WNS for the constrained parts of the design.
*   Detailed power and resource utilization metrics were obtained.

**Challenges and Observations:**
1.  **Unconstrained Design:** The most significant observation is that the design is largely unconstrained. This is evident from:
    *   `Worst Slack: INF` throughout the flow.
    *   Numerous unconstrained path reports.
    *   The `core_clock` not being properly propagated by CTS ("No valid clock nets").
    *   The initial STA warning about "port 'clock_i' not found."
    This implies that the SDC file lacked proper clock definitions (e.g., `create_clock` on an actual input port, not just a virtual clock), input/output delay constraints, and potentially timing exceptions.
2.  **Initial Utilization Error:** An early attempt (visible in the logs before the successful run) failed due to placement utilization exceeding 100% (reported as 224%). This was rectified in the successful run by adjusting floorplan parameters, leading to an initial 61% utilization for standard cells. This highlights the sensitivity of placement to initial floorplan density.
3.  **High Final Utilization:** The final utilization of 94% after filler cell insertion is quite high and could pose challenges for routability or future engineering change orders (ECOs) in more complex designs.
4.  **Synthesis Warnings:** Yosys reported warnings about continuous assignments to `reg` types in the RTL, which, while handled, could be improved by using `wire` or proper procedural blocks. The replacement of memory `\tree` with registers should also be noted as a synthesis decision.

**Future Work:**
1.  **Comprehensive SDC Constraints:** Develop a robust SDC file defining the `core_clock` on its actual input port, specify input/output delays for all primary I/Os, and add any necessary timing exceptions. This will enable meaningful timing analysis and optimization.
2.  **Clock Tree Implementation:** Ensure the `core_clock` is correctly defined and connected in the RTL/constraints so that CTS can build a functional clock tree.
3.  **Formal Verification:** Perform LVS (Layout vs. Schematic) and potentially functional verification (e.g., gate-level simulation with SDF back-annotation) on the final GDSII.
4.  **Design Exploration:** Experiment with different floorplan utilization targets and PnR tool settings to optimize for area, power, or performance once proper constraints are in place.
5.  **Address Synthesis Warnings:** Refactor RTL to address Yosys warnings for better design practice.

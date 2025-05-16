# Verilog/SystemVerilog FPGA Development Workflow

### Optimized for VS Code + Vivado

---

## 📌 Table of Contents

1. [Tool Setup](#-tool-setup)
2. [Project Structure](#-project-structure)
3. [Workflow Steps](#-workflow-steps)
4. [TCL Automation](#-tcl-automation)
5. [Example Project](#-example-project)
6. [FAQ](#-faq)

---

## 🛠️ Tool Setup

### **VS Code Extensions**

| Extension                 | Purpose                      | Install Link                                                                             |
| ------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------- |
| Verilog-HDL/SystemVerilog | Syntax highlighting, linting | [Marketplace](https://marketplace.visualstudio.com/items?itemName=mshr-h.veriloghdl)     |
| Todo Tree                 | Track TODOs/FIXMEs           | [Marketplace](https://marketplace.visualstudio.com/items?itemName=Gruntfuggly.todo-tree) |
| Verible                   | Linting (Google style)       | [Marketplace](https://marketplace.visualstudio.com/items?itemName=mshr-h.verible)        |

### **External Tools**

- **Icarus Verilog**: Quick RTL checks
  ```bash
  sudo apt install iverilog gtkwave  # Linux
  Vivado: Simulation/Synthesis
  ```

📂 Project Structure
plaintext
project/  
├── rtl/ # RTL code (.v/.sv)  
├── tb/ # Testbenches  
├── scripts/ # TCL/Python scripts  
├── waveforms/ # Simulation outputs  
└── vivado/ # Vivado project files  
🔄 Workflow Steps

1. Develop in VS Code
   Write RTL/testbench with linting support.

Use // TODO: tags (tracked by Todo Tree).

2. Pre-Simulate with Icarus
   bash
   iverilog -o sim.out -g2012 tb/testbench.sv rtl/design.sv
   vvp sim.out
   gtkwave waveforms/dump.vcd
3. Simulate in Vivado
   Refresh Vivado project after editing files in VS Code.

Run behavioral simulation.

4. Debug & Fix
   Check waveforms in Vivado.

Fix errors in VS Code using linting hints.

5. Synthesize (Optional)
   Run synthesis/implementation in Vivado.

⚙️ TCL Automation
Run TCL Scripts in Vivado
Method 1: Command Line
bash
vivado -source scripts/create_project.tcl
Method 2: Vivado GUI
Open Vivado → Tools → Run TCL Script.

Select your .tcl file.

Example: create_project.tcl
tcl
create_project my_proj vivado/my_proj -part xc7z020clg400-1
add_files {rtl/design.sv}
add_files -fileset sim_1 {tb/testbench.sv}
set_property top testbench [get_filesets sim_1]
launch_simulation # Optional
📦 Example Project
8-Bit Adder
RTL: rtl/adder_8bit.sv

verilog
module adder_8bit (input [7:0] a, b, output [7:0] sum, output carry);
assign {carry, sum} = a + b;
endmodule
Testbench: tb/tb_adder.sv

verilog
module tb_adder;
logic [7:0] a, b, sum;
logic carry;
adder_8bit dut (.\*);

    initial begin
        $dumpfile("waveforms/tb_adder.vcd");
        $dumpvars(0, tb_adder);
        a = 8'h01; b = 8'h02; #10;
        $finish;
    end

endmodule
❓ FAQ
Q: How to debug Vivado errors?
A:

Check the TCL Console for detailed logs.

Verify file paths in TCL scripts.

Q: Can I use Python with Vivado?
A: Yes! Use Python to:

Generate test vectors.

Parse simulation logs.
Example:

python

# scripts/gen_tests.py

import random
with open("tb/inputs.txt", "w") as f:
f.write(f"{random.randint(0, 255)}\n")
🚀 Pro Tips
Version Control: Use Git + .gitignore for Vivado-generated files.

CI/CD: Run linting/tests with GitHub Actions.

📄 License: MIT
🔗 Adapted from: [Your Name]

---

### How to Use This Document

1. Save as `README.md` in your project root.
2. Customize the example project for your needs.
3. Update FAQs as you encounter new issues.

Let me know if you'd like to add/remove sections!

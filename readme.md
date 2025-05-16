# Verilog/SystemVerilog FPGA Development Workflow

### Optimized for VS Code + Vivado

> A modern development workflow for FPGA design using VS Code and Vivado, emphasizing productivity and code quality.

---

## 📌 Table of Contents

1. [Tool Setup](#-tool-setup)
2. [Project Structure](#-project-structure)
3. [Workflow Steps](#-workflow-steps)
4. [TCL Automation](#-tcl-automation)
5. [Example Project](#-example-project)
6. [Dependencies](#-dependencies)
7. [FAQ](#-faq)
8. [Contributing](#-contributing)

---

## 🛠️ Tool Setup

### **VS Code Extensions**

| Extension                 | Purpose                      | Install Link                                                                             |
| ------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------- |
| Verilog-HDL/SystemVerilog | Syntax highlighting, linting | [Marketplace](https://marketplace.visualstudio.com/items?itemName=mshr-h.veriloghdl)     |
| Todo Tree                 | Track TODOs/FIXMEs           | [Marketplace](https://marketplace.visualstudio.com/items?itemName=Gruntfuggly.todo-tree) |
| Verible                   | Linting (Google style)       | [Marketplace](https://marketplace.visualstudio.com/items?itemName=mshr-h.verible)        |
| GitLens                   | Git integration & history    | [Marketplace](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)       |

### **External Tools**

- **Icarus Verilog**: Quick RTL verification
  ```bash
  # Linux
  sudo apt install iverilog gtkwave
  
  # Windows (using Package Manager)
  winget install icarus-verilog
  winget install gtkwave
  ```

- **Vivado**: Full FPGA development suite
  - Download from [Xilinx Website](https://www.xilinx.com/support/download.html)
  - Ensure it's added to system PATH

## 📂 Project Structure

```
project/
├── rtl/                  # RTL code (.v/.sv files)
│   ├── src/             # Source RTL modules
│   └── packages/        # Shared packages & interfaces
├── tb/                  # Testbenches
│   ├── unit/           # Unit test benches
│   └── integration/    # Integration tests
├── scripts/            # TCL/Python automation
│   ├── tcl/           # Vivado TCL scripts
│   └── python/        # Test generators & utilities
├── constraints/        # Timing & pin constraints
├── docs/              # Documentation
├── waveforms/         # Simulation outputs
└── vivado/            # Vivado project files
```

## 🔄 Workflow Steps

### 1. Development (VS Code)
- Write RTL/testbench with real-time linting
- Use organized TODO tags:
  ```verilog
  // TODO(feature): Add overflow detection
  // TODO(optimize): Pipeline this stage
  // TODO(bug): Fix timing violation
  ```

### 2. Pre-Synthesis Verification
```bash
# Run Icarus simulation
iverilog -o sim.out -g2012 tb/testbench.sv rtl/design.sv
vvp sim.out
gtkwave waveforms/dump.vcd
```

### 3. Vivado Integration
1. Refresh project in Vivado
2. Run behavioral simulation
3. Check timing constraints
4. Run synthesis & implementation

### 4. Debug & Verification
- Analyze waveforms in Vivado/GTKWave
- Check timing reports
- Verify against requirements

## ⚙️ TCL Automation
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
📦 Example Project: 8-Bit Adder

### RTL Implementation (`rtl/adder_8bit.sv`)
```verilog
module adder_8bit (
    input  logic [7:0] a,
    input  logic [7:0] b,
    output logic [7:0] sum,
    output logic       carry
);
    assign {carry, sum} = a + b;
endmodule
```

### Testbench (`tb/tb_adder.sv`)
```verilog
module tb_adder;
    logic [7:0] a, b, sum;
    logic carry;
    
    // DUT instantiation
    adder_8bit dut (.*);

    // Test stimulus
    initial begin
        $dumpfile("waveforms/tb_adder.vcd");
        $dumpvars(0, tb_adder);
        
        // Test cases
        a = 8'h01; b = 8'h02; #10;
        assert(sum == 8'h03) else $error("Test 1 failed");
        
        a = 8'hFF; b = 8'h01; #10;
        assert({carry, sum} == 9'h100) else $error("Test 2 failed");
        
        $finish;
    end
endmodule
```

## ❓ FAQ

### Q: How to debug Vivado errors?
- Check the TCL Console for detailed logs
- Verify file paths in TCL scripts
- Enable verbose logging with `set_msg_config -id {Common 17-55} -new_severity INFO`

### Q: Can I use Python with Vivado?
Yes! Python integration enables:
- Test vector generation
- Results analysis
- Automation scripts

Example test generator:
```python
# scripts/python/gen_tests.py
import random

def generate_test_vectors(count=100):
    with open("tb/vectors.txt", "w") as f:
        for _ in range(count):
            a = random.randint(0, 255)
            b = random.randint(0, 255)
            f.write(f"{a:08b} {b:08b}\n")

if __name__ == "__main__":
    generate_test_vectors()
```

## 🚀 Pro Tips

1. Version Control
   - Use `.gitignore` for Vivado files
   - Commit constraints & TCL scripts
   - Regular backups of project state

2. CI/CD Integration
   - Automate with GitHub Actions
   - Run linting & simulation tests
   - Generate documentation

3. Best Practices
   - Use SystemVerilog assertions
   - Implement error checking
   - Follow coding guidelines

## 📄 License
MIT License

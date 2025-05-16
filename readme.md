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
├── rtl/                # RTL code (.v/.sv files)
│   ├── src/            # Source RTL modules
│   └── packages/       # Shared packages & interfaces
├── tb/                 # Testbenches
│   ├── unit/           # Unit test benches
│   └── integration/    # Integration tests
├── scripts/            # TCL/Python automation
│   ├── tcl/            # Vivado TCL scripts
│   └── python/         # Test generators & utilities
├── constraints/        # Timing & pin constraints
├── docs/               # Documentation
├── waveforms/          # Simulation outputs
└── vivado/             # Vivado project files
```

### `Or`

```
project/
├── rtl/                # RTL code (.v/.sv files)
│   ├── project.v       # Source RTL modules   
├── tb/                 # Testbenches
│   ├── tb_project.v    # Unit test benches
├── scripts/            # TCL/Python automation
│   ├── tcl/            # Vivado TCL scripts
│   └── python/         # Test generators & utilities
├── constraints/        # Timing & pin constraints
├── docs/               # Documentation
├── waveforms/          # Simulation outputs
└── vivado/             # Vivado project files
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

### 2. Pre-Synthesis Verification with Icarus Verilog

```bash
# Navigate to project directory
cd adder_project
# Run simulation script
./scripts/run_iverilog.sh
```

- Performs quick syntax check and basic tests
- View waveforms in GTKWave for initial verification
- Fast iteration cycle for basic debugging

### 3. Vivado Integration & Simulation

#### Project Setup

1. Launch Vivado and create new project

   - Navigate to: `Create Project → Next`
   - Set project location to: `<your_project>/vivado/`
   - Select project type: `RTL Project`

2. Add Source Files
   - Add RTL files: `rtl/adder_8bit.sv`
   - Add testbench: `tb/tb_adder_8bit.sv`
   - Set simulation language to `SystemVerilog`
   - `Simulation language should be based on your code you used in the projects.` 

#### Simulation Flow

1. Run behavioral simulation

   - Click: `Flow Navigator → Simulation → Run Simulation → Behavioral Simulation`
   - Monitor console for compile/elaboration messages
   - Debug using Vivado's integrated waveform viewer

2. Analyze Results
   - Check waveforms against expected behavior
   - Verify timing relationships
   - Validate test assertions

### 4. Debug & Verification

1. Error Resolution

   - Return to VS Code for fixes if Vivado reports issues
   - Use linting suggestions to guide corrections
   - Re-run simulation to verify fixes

2. Waveform Analysis
   - Compare Vivado/GTKWave outputs
   - Verify signal transitions
   - Check timing requirements

### 5. Synthesis & Implementation (Optional)

For FPGA targeting:

1. Run synthesis
2. Perform implementation
3. Generate bitstream
4. Analyze timing reports

## ⚙️ TCL Automation

### Running TCL Scripts in Vivado

#### Method 1: Command Line

```bash
vivado -source scripts/create_project.tcl
```

#### Method 2: Vivado GUI

1. Open Vivado
2. Navigate to: `Tools → Run TCL Script`
3. Select your `.tcl` file

### Example TCL Script

Here's a sample `create_project.tcl`:

```tcl
# Create new project
create_project my_proj vivado/my_proj -part xc7z020clg400-1

# Add design files
add_files {rtl/design.sv}
add_files -fileset sim_1 {tb/testbench.sv}

# Set testbench as top for simulation
set_property top testbench [get_filesets sim_1]

# Optional: Launch simulation
launch_simulation
```

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

## 🎯 Key Workflow Benefits

### 💻 VS Code Advantages

- **Real-time Error Detection**
  - Immediate syntax highlighting
  - Linting feedback as you type
  - Quick error location and fixes
- **Version Control Integration**
  - Git status in editor
  - Commit/branch management
  - Change tracking
- **Task Management**
  - Todo Tree for task tracking
  - Organized feature requests
  - Bug tracking

### ⚡ Vivado Benefits

- **Comprehensive Simulation**
  - Full Xilinx IP support
  - Accurate timing simulation
  - Advanced debugging tools
- **FPGA Implementation**
  - Complete synthesis flow
  - Timing analysis
  - Resource utilization reports

### 🚀 Icarus Verilog Advantages

- **Rapid Development**
  - Quick syntax verification
  - Fast simulation cycles
  - Lightweight environment
- **Open Source Benefits**
  - Community support
  - Cross-platform compatibility
  - Free for all use cases

## 📄 License

MIT License

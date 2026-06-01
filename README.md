# DESIGN-AND-SIMULATION-OF-UVM-MONITOR-AND-AGENT-PG_UVM_LAB
## EXPERIMENT – 5 DESIGN AND SIMULATION OF UVM MONITOR AND AGENT
## AIM
To design and simulate UVM monitor and agent components using SystemVerilog and verify DUT transactions using Synopsys VCS.
## SOFTWARE REQUIRED
•	Synopsys VCS 
•	Verdi (Optional) 
•	Linux/Ubuntu Environment 
•	UVM Library Support 
## THEORY
Universal Verification Methodology (UVM) is a standardized verification methodology based on SystemVerilog.
UVM Agent
A UVM agent is a container that includes:
•	Driver 
•	Sequencer 
•	Monitor 
The agent controls communication between verification components.
UVM Monitor
•	Observes DUT interface signals 
•	Captures transactions 
•	Sends monitored data to scoreboard or coverage collector 
UVM Driver
•	Receives transactions from sequencer 
•	Drives DUT signals 
UVM Sequencer
•	Generates sequence items 
## Advantages:
•	Reusable verification environment 
•	Better scalability 
•	Modular verification architecture 
## PROGRAM
SystemVerilog Design and UVM Testbench (exp5.sv)
```
`include "uvm_macros.svh"
import uvm_pkg::*;


//---------------------------------------------------
// Interface
//---------------------------------------------------

interface adder_if;

  logic clk;
  logic [3:0] a;
  logic [3:0] b;
  logic [4:0] sum;

endinterface


//---------------------------------------------------
// DUT
//---------------------------------------------------

module adder(
  input  logic [3:0] a,
  input  logic [3:0] b,
  output logic [4:0] sum
);

  assign sum = a + b;

endmodule


//---------------------------------------------------
// Transaction Class
//---------------------------------------------------

class transaction extends uvm_sequence_item;

  rand bit [3:0] a;
  rand bit [3:0] b;

  bit [4:0] sum;

  `uvm_object_utils(transaction)

  function new(string name = "transaction");
    super.new(name);
  endfunction

endclass


//---------------------------------------------------
// Sequencer
//---------------------------------------------------

class my_sequencer extends uvm_sequencer #(transaction);

  `uvm_component_utils(my_sequencer)

  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction

endclass


//---------------------------------------------------
// Sequence
//---------------------------------------------------

class my_sequence extends uvm_sequence #(transaction);

  `uvm_object_utils(my_sequence)

  function new(string name = "my_sequence");
    super.new(name);
  endfunction

  task body();

    transaction tr;

    repeat(5) begin

      tr = transaction::type_id::create("tr");

      start_item(tr);

      assert(tr.randomize());

      finish_item(tr);

    end

  endtask

endclass


//---------------------------------------------------
// Driver
//---------------------------------------------------

class my_driver extends uvm_driver #(transaction);

  `uvm_component_utils(my_driver)

  virtual adder_if vif;

  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction

  function void build_phase(uvm_phase phase);

    super.build_phase(phase);

    if(!uvm_config_db #(virtual adder_if)::get(this,"","vif",vif))
      `uvm_fatal("DRV","Virtual Interface Not Found")

  endfunction

  task run_phase(uvm_phase phase);

    transaction tr;

    forever begin

      seq_item_port.get_next_item(tr);

      vif.a = tr.a;
      vif.b = tr.b;

      #10;

      `uvm_info("DRIVER",
      $sformatf("a=%0d b=%0d",tr.a,tr.b),
      UVM_NONE)

      seq_item_port.item_done();

    end

  endtask

endclass


//---------------------------------------------------
// Monitor
//---------------------------------------------------

class my_monitor extends uvm_monitor;

  `uvm_component_utils(my_monitor)

  virtual adder_if vif;

  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction

  function void build_phase(uvm_phase phase);

    super.build_phase(phase);

    if(!uvm_config_db #(virtual adder_if)::get(this,"","vif",vif))
      `uvm_fatal("MON","Virtual Interface Not Found")

  endfunction

  task run_phase(uvm_phase phase);

    transaction tr;

    forever begin

      tr = transaction::type_id::create("tr");

      #10;

      tr.a   = vif.a;
      tr.b   = vif.b;
      tr.sum = vif.sum;

      `uvm_info("MONITOR",
      $sformatf("a=%0d b=%0d sum=%0d",
      tr.a,tr.b,tr.sum),
      UVM_NONE)

    end

  endtask

endclass


//---------------------------------------------------
// Agent
//---------------------------------------------------

class my_agent extends uvm_agent;

  `uvm_component_utils(my_agent)

  my_driver drv;
  my_monitor mon;
  my_sequencer seqr;

  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction

  function void build_phase(uvm_phase phase);

    super.build_phase(phase);

    drv  = my_driver::type_id::create("drv",this);
    mon  = my_monitor::type_id::create("mon",this);
    seqr = my_sequencer::type_id::create("seqr",this);

  endfunction

  function void connect_phase(uvm_phase phase);

    drv.seq_item_port.connect(seqr.seq_item_export);

  endfunction

endclass


//---------------------------------------------------
// Environment
//---------------------------------------------------

class my_env extends uvm_env;

  `uvm_component_utils(my_env)

  my_agent agt;

  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction

  function void build_phase(uvm_phase phase);

    super.build_phase(phase);

    agt = my_agent::type_id::create("agt",this);

  endfunction

endclass


//---------------------------------------------------
// Test
//---------------------------------------------------

class my_test extends uvm_test;

  `uvm_component_utils(my_test)

  my_env env;
  my_sequence seq;

  function new(string name, uvm_component parent);
    super.new(name,parent);
  endfunction

  function void build_phase(uvm_phase phase);

    super.build_phase(phase);

    env = my_env::type_id::create("env",this);

  endfunction

  task run_phase(uvm_phase phase);

    phase.raise_objection(this);

    seq = my_sequence::type_id::create("seq");

    seq.start(env.agt.seqr);

    #50;

    phase.drop_objection(this);

  endtask

endclass


//---------------------------------------------------
// Top Testbench
//---------------------------------------------------

module top;

  bit clk;

  always #5 clk = ~clk;

  adder_if vif();

  adder dut(
    .a(vif.a),
    .b(vif.b),
    .sum(vif.sum)
  );

  initial begin

    uvm_config_db #(virtual adder_if)::set(null,"*","vif",vif);

    run_test("my_test");

  end

endmodule
```
## PROCEDURE
1.	Open Linux terminal. 
2.	Create source file: 
gedit exp5.sv
3.	Save the code. 
4.	Compile using Synopsys VCS: 
vcs -sverilog exp5.sv -ntb_opts uvm -o simv
5.	Run simulation: 
./simv
6.	Observe UVM monitor and agent outputs. 
## EXPECTED OUTPUT
UVM_INFO DRIVER a=5 b=3
UVM_INFO MONITOR a=5 b=3 sum=8

UVM_INFO DRIVER a=2 b=4
UVM_INFO MONITOR a=2 b=4 sum=6
Output values may vary because of randomization.
## RESULT
Thus, UVM monitor and agent were successfully designed and simulated using SystemVerilog and Synopsys VCS.
## VIVA QUESTIONS
1.	What is UVM? 
2.	What is the role of UVM agent? 
3.	What is a UVM monitor? 
4.	Difference between monitor and driver? 
5.	What is a virtual interface? 
6.	What is the use of uvm_config_db? 
7.	Why is run_phase used? 
8.	Which simulator is used in this experiment? 
## CONCLUSION
The experiment demonstrated the implementation of UVM monitor and agent using System Verilog. Transactions were generated, driven, monitored, and verified successfully using Synopsys VCS simulation tool.



module tb_ProcessorTop();

    reg clk, reset;
    reg [7:0] IN_PORT;
    wire [7:0] OUT_PORT;

    ProcessorTop DUT(
        .clk(clk), 
        .reset(reset), 
        .IN_PORT(IN_PORT), 
        .OUT_PORT(OUT_PORT)
    );

    // Clock Generation
    always #5 clk = ~clk;

    // Monitor internal signals
    initial begin
        $monitor("Time:%3d | PC_F:%h | Op_D:%h | Z_Flag:%b | Flush:%b | R0:%h | R3:%h", 
                 $time, DUT.PC_f, DUT.opcode_d, DUT.Z, DUT.ifid.flush, DUT.rf.regs[0], DUT.rf.regs[3]);
    end

    // Track Output Port
    reg [7:0] out_history [0:10];
    integer out_count;
    always @(posedge clk) begin
        if (OUT_PORT !== 0 && (out_count == 0 || OUT_PORT !== out_history[out_count-1])) begin
            out_history[out_count] = OUT_PORT;
            $display(">>> [Output Event] OUT_PORT updated to %h at time %0t", OUT_PORT, $time);
            out_count = out_count + 1;
        end
    end

    initial begin
        clk = 0; reset = 1; IN_PORT = 8'h00;
        #10 reset = 0; 
        
       $display(" === Loading Basic ALU Test (ADD/SUB) === ");
// 1. LDM R1, #10 (0x0A)
DUT. fetch. imem.mem[0] = 8'hC1; DUT.fetch. imem.mem[1] = 8'h0A;
// 2. LDM R2, #3 (0x05)
DUT.fetch.imem.mem[2] = 8'hC2; DUT.fetch.imem.mem[3] = 8'h05;
// 3. ADD R1, R2 (R1 = R1 + R2 = 10 + 5 = 15 -> 0xF)
// Op=2, ra=1, rb=2 -> 26
DUT.fetch.imem.mem[4] = 8'h26;
// 4. SUB R1, R2 (R1 = R1 - R2 = 15 - 5 = 10 -> 0xA)
// Op=3, ra=1, rb=2 -> 36
DUT.fetch. imem.mem[5] = 8'h36;
// 5. OUT R1 (Should output 0xA)
// Op=7, ra=2, rb=1 -> 79
DUT.fetch.imem.mem[6] = 8'h79;
// End
DUT.fetch.imem.mem[7] = 8'h00;

        #200;
        $finish;
    end

    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, tb_ProcessorTop);
    end

endmodule
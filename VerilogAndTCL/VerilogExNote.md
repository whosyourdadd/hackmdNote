# VerilogExample

```verilog!
wire a
wire bus[7:0];//繞線
wire busA[31:0];//較常用
wire busB[0:31];//不常用
reg clock

integer/real/time verilog支援但無法產生電路合成


time save_sim_time;
initial
    save_sim_time = $time;//$time is simulation time
    
//Array
    reg [4:0] port_id[0:7];
    integer matrix[4:0][4:0];//1995 verilog 不允許但現在ok    

// Silce
byte = data[j +: k];
//j  -> bit start position
//k -> Number of bits up from j’th position
foreach(byte[i]) byte[i] = data[8*i +: 8];
//Above code can be expanded as below,
byte[0] = data[8*0 +: 8]; //data[7:0]
byte[i] = data[8*1 +: 8]; //data[15:8]
byte[i] = data[8*2 +: 8]; //data[23:16]
byte[i] = data[8*3 +: 8]; //data[31:24]


wire mysignal0 = A & B;     // continuous assignment, AND gate
logic mysignal1 = A & B;    // not synthesizable, initializes mysignal1 to the value of A & B at time 0 and then makes no further changes to it.
logic mysignal2;
assign mysignal2 = A & B;   // Continuous assignment, AND gate

//used to specify the a_in,b_in,c_in as logic net type and 8 bit wide
logic [7:0] a_in,b_in,c_in;//8 bit signed representation with logic net type
logic signed [7:0] d_in;
assign e_in = 4'b1100;//assigning binary value 1100 to net
assign f_in = 4'hE// assigning hex value 1110 to net
assign g_in = 3;//assign decimal value 3 to net 
assign h_in = -2;//assign demial value -2 to net
assign x_in = e_in[2:1];//Part select from the 4-bit binary number that is x_in = 10
assign y_in = {e_in,f_in};

```


## Example

### Div
```verilog=
//parameter DIV_N每+1都是/2
module clk_dev #( parameter DIV_N = 'd4 )(
    clock_in,
    clock_out,
    rst_n_i
    );
    input wire rst_n_i;
    input wire clock_in;
    output wire clock_out;
    reg[DIV_N-1:0] div_cnt;
    
    always @(posedge clock_in)begin
        if(!rst_n_i)begin
            div_cnt<= 32'b0;
        end
        else div_cnt <= div_cnt + 1;
    end
    assign clock_out = div_cnt[DIV_N-1];
endmodule
```
### Edge_Detect

Ref https://www.cnblogs.com/oomusou/archive/2008/08/11/verilog_edge_detection_circuit.html
```verilog=
//合併模組，根據需求選擇要用pos_o/neg_o輸出
 module edge_det ( input rst_n,
                      input sig_i,            // Input signal for which positive edge has to be detected
                      input clk_i,            // Input signal for clock
                      output pos_o,
                      output neg_o
                      );           // Output signal that gives a pulse when a positive edge occurs
 
    reg sig_dly_in0;
    reg sig_dly_in1;
    assign pos_o = ~sig_dly_in0 & sig_dly_in1;
    assign neg_o = sig_dly_in0 & ~sig_dly_in1;
 
    always@(posedge clk_i, negedge rst_n) begin
        if (!rst_n) begin
            sig_dly_in0 <= 0;
            sig_dly_in1 <= 0;
        end
        else begin
            sig_dly_in0 <= sig_dly_in1;
            sig_dly_in1 <= sig_i;
        end
   end
endmodule 
```
```verilog=
`include "NewSPI-2.v"
`include "edge_det.v"
`define DEBUG 1 



module NewSPI2_tb #(
        parameter tb_WORD_SIZE  = 7'd64,//8'd128,//7'd64,
        //parameter DataBufSize  = 9'd256,//7'd64,
        parameter CmdBufSize  = 6'd40
    )
    ();
    localparam integer WORD_SIZE = tb_WORD_SIZE;
    localparam integer RECV_CNT = (WORD_SIZE/8);
    localparam integer RECV_SIZE = (WORD_SIZE/8)+3;
    localparam integer DIV_SIZE = 2;
    integer CSN_RAND_Delay = {$random} %300;
    integer CSP_RAND_Delay = {$random} %300;
    reg  sysClk, rst_n,PSCS_n_i, miso;
    reg  [tb_WORD_SIZE-1:0]temp_InBuf;
    reg  [RECV_CNT-1:0] PS_InSize;
    
    reg  [WORD_SIZE-1:0]rxdata;
    reg  [WORD_SIZE-1:0]T_GetMOSI;
    
    wire [WORD_SIZE-1:0]data_get;
    wire  cs_n,sclk, mosi;
   
    //wire finish;
    reg finish;
    //wire CLK_DIV_Out;
`ifdef DEBUG


    wire  [1:0]tmpbuf;
    //wire T_SHEN;
    //wire T_LDEN;
    wire [DIV_SIZE-1:0] T_DIV_CNT;//[6:0] T_counter;
    wire [RECV_SIZE-1:0] T_counter;//[6:0] T_counter;
    wire [WORD_SIZE-1: 0] T_regdata;
    wire T_PEdgeOut;
    wire T_FinishP_O;
    wire T_CLK_out;
    wire T_SCLK_out;
`endif
    

NewSPI_Top #( 
        .WORD_SIZE(tb_WORD_SIZE),
        .DIV_N(2) 
        )
    UUT (
    
`ifdef DEBUG
      //.T_CLK_o(T_CLK_o),
      .tempOutTop_o(tmpbuf),
      //.T_SHEN_o(T_SHEN),
      //.T_LDEN_o(T_LDEN),
      .T_DIV_CNT(T_DIV_CNT),
      .T_counter(T_counter),
      .T_regdata_i(T_regdata),
      .T_CLK_out(T_CLK_out),
      .T_SCLK_out(T_SCLK_out),
`endif
      
      //.tempCounterTop_o(tempCounter),
      //.en_i(tri_tx),
      .PSCS_n_i(PSCS_n_i),
      .clock_100M_i(sysClk),
      .rst_n_i(rst_n),
      .CS_o(cs_n),//.CS(cs_n),
      .mosi_o(mosi),
      .miso_i(miso),
      .dSize_i(PS_InSize),
      .din_i(temp_InBuf),
      .dout_o(data_get),
      .SCLK_o(sclk),
      .finish_o(finish)//JAY
      //JAY

      //.T_regdata_i(T_regdata_i)
    );
    
    edge_det testUT1(
        .rst_n(rst_n),
        .sig_i(finish),
        .clk_i(sysClk),
        .neg_o(T_FinishP_O)
    );
    edge_det testUT2(
        .rst_n(rst_n),
        .sig_i(cs_n),
        .clk_i(sysClk),
        .neg_o(T_PEdgeOut)
    );
/*
neg_edge_det testUT2(
    .rst_n(rst_n),
    .sig_i(PSCS_n_i),
    .clk_i(sysClk),
    .pe_o(T_PEdgeOut)

    );
*/    
    assign miso = rxdata[WORD_SIZE-1];
    //always@(negedge cs_n)begin
    //    i <= 16;
        
    //end
    //always@(posedge CLK_DIV_Out, negedge rst_n)begin
    //always@(negedge sclk ,negedge rst_n)begin
    always@(posedge sclk ,negedge rst_n,posedge T_FinishP_O)begin
        if(!rst_n | T_FinishP_O)begin
            T_GetMOSI <= 16'h00;
            
        end
        else if(T_counter > 0 & cs_n == 0)begin
             T_GetMOSI <= {T_GetMOSI[WORD_SIZE-2:0],mosi} ;//always shift and get mosi data
             //rxdata <= {rxdata[WORD_SIZE-2:0],1'b0};//always shift and get mosi data
        end
    end
    always@(negedge sclk ,negedge rst_n)begin
        if(T_counter > 0 & cs_n == 0)begin
             rxdata <= {rxdata[WORD_SIZE-2:0],1'b0};//always shift and get mosi data
        end
    end
        
    always #5 sysClk = ~sysClk;
    //assign cs_n    = cs_reg;
    initial begin
      sysClk  = 0;
      rst_n   = 1;
      PSCS_n_i = 1;
      #10 rst_n   = 0;
      #10 rst_n   = 1;
      temp_InBuf  = 64'h1234_5678_9abc_def0;//64'hbeef_1000_1234_56ab;
      rxdata   = 64'h1234_5678_1234_56ab; 
      PS_InSize = 5'h1;//h18 = d24
      $display("1. Delay: CSN,CSP (%d,%d)",CSN_RAND_Delay,CSP_RAND_Delay);
      repeat(6) begin
      
      
      #10 
      
      #CSN_RAND_Delay 
      
      PSCS_n_i = 0;
      #CSP_RAND_Delay 
      PSCS_n_i = 1;
      CSN_RAND_Delay = {$random} %300;
      CSP_RAND_Delay = CSN_RAND_Delay + {$random} %300;
      //temp_InBuf = {$random} & 64'hffff_ffff_0000_0000;
      temp_InBuf = {$random} << 32;
      PS_InSize = {$random} %3+1;//h18 = d24
      $display("Delay: CSN,CSP (%d,%d) , %x, send size %d",CSN_RAND_Delay,CSP_RAND_Delay,temp_InBuf,PS_InSize);
      end
      //$stop;
    end
endmodule

```
// 如果你有一個 header 檔，可以用 `include "define.v"
// 這裡直接定義以解決編譯錯誤
`define PRE_W_1 3'd1
`define PRE_H_1 3'd3
`define PRE_W_2 3'd2
`define PRE_H_2 3'd4

module DIV #(
    parameter DENT_WIDTH = 32,          
    parameter SOR_WIDTH  = 8,           
    parameter QUOT_WIDTH = DENT_WIDTH   
)(
    input  wire                   clk,
    input  wire                   rst,
    input  wire [DENT_WIDTH-1:0]  dividend,
    input  wire [SOR_WIDTH-1:0]   divisor,
    input  wire [2:0]             cs,        
    output reg  [QUOT_WIDTH-1:0]  quotient,
    output reg                    done
);

    //=======================================================
    // 定義與常數計算
    //=======================================================
    localparam REG_WIDTH = SOR_WIDTH + DENT_WIDTH;
    localparam STEPS     = DENT_WIDTH / 2; 
    localparam CNT_WIDTH = 6;              

    reg [REG_WIDTH-1:0] shift_reg;
    reg [SOR_WIDTH-1:0] den;
    reg [CNT_WIDTH-1:0] div_cnt;

    reg [SOR_WIDTH:0]   rem_step1; 
    reg [SOR_WIDTH:0]   rem_step2;
    reg                 q_bit_1;
    reg                 q_bit_2;

    //=======================================================
    // Main Sequential Logic
    //=======================================================
    always @(posedge clk or posedge rst) begin
        if (rst) begin
            div_cnt   <= {CNT_WIDTH{1'b0}};
            shift_reg <= {REG_WIDTH{1'b0}};
            den       <= {SOR_WIDTH{1'b0}};
            quotient  <= {QUOT_WIDTH{1'b0}};
            done      <= 1'b0;
        end
        // ------------------------------------
        // PREP: 初始化
        // ------------------------------------
        else if ((cs == `PRE_W_1) || (cs == `PRE_H_1)) begin 
            done <= 1'b0;
            if (divisor != {SOR_WIDTH{1'b0}}) begin
                den       <= divisor;
                shift_reg <= {{SOR_WIDTH{1'b0}}, dividend};
                div_cnt   <= STEPS[CNT_WIDTH-1:0]; 
            end else begin
                div_cnt   <= {CNT_WIDTH{1'b0}};
                quotient  <= {QUOT_WIDTH{1'b1}};
                done      <= 1'b1;
            end
        end
        // ------------------------------------
        // CALC: 運算
        // ------------------------------------
        else if ((cs == `PRE_W_2) || (cs == `PRE_H_2)) begin
            if (div_cnt != {CNT_WIDTH{1'b0}}) begin
                shift_reg <= {rem_step2[SOR_WIDTH-1:0], 
                              shift_reg[DENT_WIDTH-3:0], 
                              q_bit_1, 
                              q_bit_2};
                div_cnt   <= div_cnt - 1'b1;
            end else begin
                quotient <= shift_reg[QUOT_WIDTH-1:0];
                done     <= 1'b1;
            end
        end
    end

    //=======================================================
    // Combinational Logic
    //=======================================================
    always @(*) begin
        // Step 1
        if ({shift_reg[REG_WIDTH-1:DENT_WIDTH], shift_reg[DENT_WIDTH-1]} >= den) begin
            rem_step1 = {shift_reg[REG_WIDTH-1:DENT_WIDTH], shift_reg[DENT_WIDTH-1]} - den;
            q_bit_1   = 1'b1;
        end else begin
            rem_step1 = {shift_reg[REG_WIDTH-1:DENT_WIDTH], shift_reg[DENT_WIDTH-1]};
            q_bit_1   = 1'b0;
        end

        // Step 2
        if ({rem_step1[SOR_WIDTH-1:0], shift_reg[DENT_WIDTH-2]} >= den) begin
            rem_step2 = {rem_step1[SOR_WIDTH-1:0], shift_reg[DENT_WIDTH-2]} - den;
            q_bit_2   = 1'b1;
        end else begin
            rem_step2 = {rem_step1[SOR_WIDTH-1:0], shift_reg[DENT_WIDTH-2]};
            q_bit_2   = 1'b0;
        end
    end

endmodule

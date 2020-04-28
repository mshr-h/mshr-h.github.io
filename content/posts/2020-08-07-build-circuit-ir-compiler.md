---
title: "CIRCT(Circuit IR Compilers and Tools)をビルドする"
slug: "build-circuit-ir-compiler"
subtitle:    ""
description: ""
date:        "2020-08-07"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["CIRCT", "MLIR"]
categories:  ["Tech"]
draft:       false
---

[CIRCT(Circuit IR Compilers and Tools)](https://github.com/llvm/circt)はMLIRベースのRTL生成ツール。
READMEにしたがってビルド、サンプル回路をコンパイルした。

## 環境

- Ryzen 5 1600
- 32GBメモリ
- Ubuntu 18.04 on WSL2 on Windows 10
- 作業ディレクトリは`~/workspace`とする

## ビルド

作業ディレクトリ(`~/workspace`)にリポジトリをクローンする。

```bash
cd ~/workspace
git clone --recursive https://github.com/llvm/circt.git
```

CIRCUITはMLIRベースに開発されているため、まずはMLIRをビルドする。

```bash
cd circt
mkdir llvm/build
cd llvm/build
cmake -G Ninja ../llvm -DLLVM_ENABLE_PROJECTS="mlir" -DLLVM_TARGETS_TO_BUILD="X86"  -DLLVM_ENABLE_ASSERTIONS=ON -DCMAKE_BUILD_TYPE=Release
ninja
ninja check-mlir
```

続いてCIRCT本体をビルドする。

```bash
cd ~/workspace/circt
mkdir build
cd build
cmake -G Ninja .. -DMLIR_DIR=~/workspace/circt/llvm/build/lib/cmake/mlir -DLLVM_DIR=~/workspace/circt/llvm/build/lib/cmake/llvm -DLLVM_ENABLE_ASSERTIONS=ON -DCMAKE_BUILD_TYPE=Release
ninja
ninja check-circt
```

`build/bin/`に`circuit-translate`と`circuit-opt`が生成されるので、環境変数パスにディレクトリを追加する。

```bash
export PATH=~/workspace/circt/build/bin:$PATH
```

## サンプル回路生成

サンプル入力ファイルのあるディレクトリに移動する。

```bash
cd ~/workspace/circt/test/EmitVerilog
```

### FIRRTLからVerilogへ変換

FIRRTL→Verilogへ変換する。

```bash
circt-translate --parse-fir verilog-basic.fir | circt-translate -emit-verilog
```

出力は以下。

```verilog
// Standard header to adapt well known macros to our needs.\n";
`ifdef RANDOMIZE_GARBAGE_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_INVALID_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_REG_INIT
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_MEM_INIT
`define RANDOMIZE
`endif
`ifndef RANDOM
`define RANDOM $random
`endif
// Users can define 'PRINTF_COND' to add an extra gate to prints.
`ifdef PRINTF_COND
`define PRINTF_COND_ (`PRINTF_COND)
`else
`define PRINTF_COND_ 1
`endif
// Users can define 'STOP_COND' to add an extra gate to stop conditions.
`ifdef STOP_COND
`define STOP_COND_ (`STOP_COND)
`else
`define STOP_COND_ 1
`endif

// Users can define INIT_RANDOM as general code that gets injected into the
// initializer block for modules with registers.
`ifndef INIT_RANDOM
`define INIT_RANDOM
`endif

// If using random initialization, you can also define RANDOMIZE_DELAY to
// customize the delay used, otherwise 0.002 is used.
`ifndef RANDOMIZE_DELAY
`define RANDOMIZE_DELAY 0.002
`endif

// Define INIT_RANDOM_PROLOG_ for use in our modules below.
`ifdef RANDOMIZE
  `ifndef VERILATOR
    `define INIT_RANDOM_PROLOG_ `INIT_RANDOM #`RANDOMIZE_DELAY begin end
  `else
    `define INIT_RANDOM_PROLOG_ `INIT_RANDOM
  `endif
`else
  `define INIT_RANDOM_PROLOG_
`endif
module inputs_only(
  input a, b);

endmodule

module no_ports();
  wire [3:0] x; // <stdin>:8:12

endmodule

module Expressions(
  input  [3:0] in4,
  input        clock,
  output       out1,
  output [3:0] out4);

  wire [1:0] x1;        // <stdin>:30:13
  wire [1:0] x2;        // <stdin>:35:13
  wire [3:0] x3;        // <stdin>:37:13
  wire [3:0] x4;        // <stdin>:40:13
  wire [9:0] x5;        // <stdin>:46:13
  wire [8:0] x6;        // <stdin>:50:13
  wire [3:0] x7;        // <stdin>:57:13
  wire [3:0] x8;        // <stdin>:62:13
  wire [1:0] x9;        // <stdin>:66:13
  wire [3:0] x10;       // <stdin>:69:14
  wire [5:0] x11;       // <stdin>:79:14
  wire [3:0] x12;       // <stdin>:81:14

  assign out1 = ^in4;   // <stdin>:12:12, :13:7
  assign out1 = &in4;   // <stdin>:14:12, :15:7
  assign out1 = |in4;   // <stdin>:16:12, :17:7
  assign out1 = ~in4;   // <stdin>:18:12, :19:7
  assign out4 = in4 % 4'h1;     // <stdin>:20:17, :21:12, :22:7
  assign out1 = clock;  // <stdin>:23:12, :24:7
  assign out1 = 1'h1;   // <stdin>:25:17, :26:12, :27:12, :28:7
  assign x1 = in4[1:0]; // <stdin>:29:12, :30:13
  assign x2 = in4[3:2] | {in4[2], 1'h0};        // <stdin>:31:12, :32:13, :33:13, :34:13, :35:13
  assign x3 = in4 >>> in4;      // <stdin>:36:13, :37:13
  assign x4 = $signed(in4) >>> in4;     // <stdin>:38:13, :39:13, :40:13
  assign x5 = {in4, clock, clock, in4}; // <stdin>:45:13, :46:13
  assign x6 = {1'b0, in4, in4}; // <stdin>:49:13, :50:13
  assign x7 = clock ? (clock ? 4'h1 : 4'h2) : 4'h3;     // <stdin>:20:17, :51:13, :52:13, :53:17, :54:13, :55:17, :56:13, :57:13
  assign x8 = clock ? 4'h1 : clock ? 4'h2 : 4'h3;       // <stdin>:20:17, :53:17, :55:17, :58:13, :59:13, :60:13, :61:13, :62:13
  assign x9 = in4[3:2] | in4[1:0];      // <stdin>:63:13, :64:13, :65:13, :66:13
  assign x10 = in4;     // <stdin>:68:13, :69:14
  assign x11 = { {2'd0}, in4} ^ {{2{in4[3]}}, in4} ^ {6{clock} }; // <stdin>:70:13, :71:13, :72:13, :73:13, :74:13, :75:13, :76:13, :77:13, :78:13, :79:14
  assign x12 = in4;     // <stdin>:80:13, :81:14
endmodule

module Precedence(
  input  [3:0] a, b, c,
  output       out1,
  output [3:0] out);

  assign out = a + b + c;       // <stdin>:84:12, :85:12, :86:7
  assign out = a + b - c;       // <stdin>:87:12, :88:12, :89:7
  assign out = a - (b + c);     // <stdin>:90:12, :91:12, :92:7
  assign out = a + b * c;       // <stdin>:93:12, :94:12, :95:7
  assign out = a * b + c;       // <stdin>:96:12, :97:12, :98:7
  assign out = (a + b) * c;     // <stdin>:99:13, :100:13, :101:7
  assign out = a * (b + c);     // <stdin>:102:13, :103:13, :104:7
  assign out = (a + b) * (b + c);       // <stdin>:105:13, :106:13, :107:13, :108:7
  assign out1 = ^(b + c);       // <stdin>:109:13, :110:13, :111:7
  assign out1 = b < c | b > c;  // <stdin>:112:13, :113:13, :114:13, :115:7
  assign out1 = (b ^ c) & (out1 | out1);        // <stdin>:116:13, :117:13, :118:13, :119:7
  assign out1 = out[3:2];       // <stdin>:120:13, :121:7
  assign out1 = out < a;        // <stdin>:122:13, :123:7
endmodule

module Sign(
  input  [3:0] a, b, c, d,
  output       out);

  assign out = a < b;   // <stdin>:126:12, :127:7
  assign out = $signed(c) < $signed(d); // <stdin>:128:12, :129:7
  assign out = $signed(a) < $signed(b); // <stdin>:130:12, :131:12, :132:12, :133:7
  assign out = a == b;  // <stdin>:134:12, :135:12, :136:12, :137:7
endmodule

module MultiUseExpr(
  input  [3:0] a,
  output       b);

  wire _T = &^a;        // <stdin>:140:12, :141:12
  wire [4:0] _T_0 = a + a;      // <stdin>:142:12
  assign b = &_T;       // <stdin>:143:12, :144:7
  assign b = ^_T;       // <stdin>:145:12, :146:7
  assign b = &_T_0;     // <stdin>:147:12, :148:7
  assign b = ^_T_0;     // <stdin>:149:12, :150:7
  assign b = ^(4'sh3 | 4'sh3);  // <stdin>:151:17, :152:12, :153:12, :154:12, :155:7
  wire [3:0] _T_1 = ~a; // <stdin>:156:13
  assign b = _T_1[3:2]; // <stdin>:157:13, :158:7
endmodule

module UseInstances(
  input  [7:0] a_in,
  output       a_out);

  wire [7:0] xyz_in;    // <stdin>:163:14
  wire       xyz_out;   // <stdin>:163:14
  wire [7:0] xyz2_in;   // <stdin>:168:15
  wire       xyz2_out;  // <stdin>:168:15

  FooExtModule xyz (    // <stdin>:163:14
    .in(xyz_in),
    .out(xyz_out)
  );
  assign xyz_in = a_in; // <stdin>:164:12, :165:7
  assign a_out = xyz_out;       // <stdin>:166:12, :167:7
  MyParameterizedExtModule #(.DEFAULT(0), .DEPTH(3.500000e+00), .FORMAT("xyz_timeout=%d\n"), .WIDTH(32)) xyz2 ( // <stdin>:168:15
    .in(xyz2_in),
    .out(xyz2_out)
  );
  assign xyz2_in = a_in;        // <stdin>:169:12, :170:7
  assign a_out = xyz2_out;      // <stdin>:171:12, :172:7
endmodule

module Stop(
  input clock, reset);


  always @(posedge clock) begin
    `ifndef SYNTHESIS
      if (`STOP_COND_ && reset) $fatal; // <stdin>:175:7
    `endif
  end // always @(posedge)
endmodule

module Stop2(
  input clock, reset);


  always @(posedge clock) begin
    `ifndef SYNTHESIS
      if (`STOP_COND_ && reset) begin
        $fatal; // <stdin>:178:7
        $finish;        // <stdin>:179:7
      end
    `endif // !SYNTHESIS
  end // always @(posedge)
endmodule

module Stop3(
  input clock1, clock2, reset);


  always @(posedge clock1) begin
    `ifndef SYNTHESIS
      if (`STOP_COND_ && reset) $fatal; // <stdin>:182:7
    `endif
  end // always @(posedge)
  always @(posedge clock2) begin
    `ifndef SYNTHESIS
      if (`STOP_COND_ && reset) $finish;        // <stdin>:183:7
    `endif
  end // always @(posedge)
endmodule

module Print(
  input       clock, reset,
  input [3:0] a, b);


  always @(posedge clock) begin
    `ifndef SYNTHESIS
      if (`PRINTF_COND_ && reset) $fwrite(32'h80000002, "Hi %x %x\n", a + a, b);        // <stdin>:186:12, :187:7
    `endif
  end // always @(posedge)
endmodule

module UninitReg1(
  input       clock, reset, cond,
  input [1:0] value);

  reg  [1:0] count;     // <stdin>:191:16
  wire [1:0] x; // <stdin>:192:12

  assign x = count;     // <stdin>:192:12

  `ifndef SYNTHESIS
  initial begin
    `INIT_RANDOM_PROLOG_
    `ifdef RANDOMIZE_REG_INIT
      count = `RANDOM;  // <stdin>:191:16
    `endif
  end // initial
  `endif // SYNTHESIS

  always @(posedge clock) begin
    count <= reset ? 2'h0 : cond ? value : count;       // <stdin>:193:12, :194:17, :195:12, :196:7
  end // always @(posedge)
endmodule

module InitReg1(
  input         clock, reset,
  input  [31:0] io_d,
  output [31:0] io_q,
  input         io_en);

  reg [31:0] reg_0;     // <stdin>:201:14

  assign io_q = reg_0;  // <stdin>:202:7

  `ifndef SYNTHESIS
  initial begin
    `INIT_RANDOM_PROLOG_
    if (reset) reg_0 = 32'h0;   // <stdin>:199:12, :200:18, :201:14
    `ifdef RANDOMIZE_REG_INIT
      if (~reset) reg_0 = `RANDOM;      // <stdin>:199:12, :200:18, :201:14
    `endif
  end // initial
  `endif // SYNTHESIS

  always @(posedge clock or posedge reset) begin
    reg_0 <= io_en ? io_d : reg_0;      // <stdin>:199:12, :203:12, :204:7
  end // always @(posedge)
endmodule

module Analog(
  output [4:0] io_pins_asrcn3v3);

endmodule

module MemSimple(
  input         clock1, clock2, inpred,
  input  [41:0] indata,
  output [41:0] result);

  wire [2:0]  _M_read_addr;     // <stdin>:209:13
  wire        _M_read_en;       // <stdin>:209:13
  wire        _M_read_clk;      // <stdin>:209:13
  wire [41:0] _M_read_data;     // <stdin>:209:13
  wire [2:0]  _M_write_addr;    // <stdin>:209:13
  wire        _M_write_en;      // <stdin>:209:13
  wire        _M_write_clk;     // <stdin>:209:13
  wire [41:0] _M_write_data;    // <stdin>:209:13
  wire        _M_write_mask;    // <stdin>:209:13
  reg  [41:0] _M[7:0];

  assign _M_read_data = _M[_M_read_addr];       // <stdin>:209:13
  assign result = _M_read_data; // <stdin>:210:12, :211:12, :212:7
  assign _M_read_addr = 1'h0;   // <stdin>:213:12, :214:12, :215:17, :216:7
  assign _M_read_en = 1'h1;     // <stdin>:217:12, :218:12, :219:17, :220:7
  assign _M_read_clk = clock1;  // <stdin>:221:12, :222:12, :223:7
  assign _M_write_addr = 3'h0;  // <stdin>:224:12, :225:12, :226:17, :227:13, :228:7
  assign _M_write_en = inpred ? 1'h1 : 1'h0;    // <stdin>:215:17, :219:17, :229:13, :230:13, :231:13, :232:7
  assign _M_write_clk = clock2; // <stdin>:233:13, :234:13, :235:13, :236:7
  assign _M_write_data = indata;        // <stdin>:237:13, :238:13, :239:13, :240:7
  assign _M_write_mask = 1'h1;  // <stdin>:219:17, :241:13, :242:13, :243:13, :244:7

  `ifndef SYNTHESIS
  initial begin
    `INIT_RANDOM_PROLOG_
    `ifdef RANDOMIZE_MEM_INIT
      integer initvar;
      for (initvar = 0; initvar < 8; initvar = initvar+1)
        _M[initvar] = `RANDOM;  // <stdin>:209:13
    `endif // RANDOMIZE_MEM_INIT
  end // initial
  `endif // SYNTHESIS

  always @(posedge _M_write_clk) begin
    if (_M_write_en & _M_write_mask) _M[_M_write_addr] <= _M_write_data;        // <stdin>:209:13
  end // always @(posedge)
endmodule

module MemAggregate(
  input clock1, clock2);

  wire [4:0] _M_read_addr;      // <stdin>:247:13
  wire       _M_read_en;        // <stdin>:247:13
  wire       _M_read_clk;       // <stdin>:247:13
  wire [3:0] _M_read_data_id;   // <stdin>:247:13
  wire [7:0] _M_read_data_other;        // <stdin>:247:13
  wire [4:0] _M_write_addr;     // <stdin>:247:13
  wire       _M_write_en;       // <stdin>:247:13
  wire       _M_write_clk;      // <stdin>:247:13
  wire [3:0] _M_write_data_id;  // <stdin>:247:13
  wire [7:0] _M_write_data_other;       // <stdin>:247:13
  wire       _M_write_mask_id;  // <stdin>:247:13
  wire       _M_write_mask_other;       // <stdin>:247:13
  reg  [3:0] _M_id[19:0];
  reg  [7:0] _M_other[19:0];

  `ifndef RANDOMIZE_GARBAGE_ASSIGN
    assign _M_read_data_id = _M_id[_M_read_addr];       // <stdin>:247:13
    assign _M_read_data_other = _M_other[_M_read_addr]; // <stdin>:247:13
  `else
    assign _M_read_data_id = _M_read_addr < 20 ? _M_id[_M_read_addr] : `RANDOM; // <stdin>:247:13
    assign _M_read_data_other = _M_read_addr < 20 ? _M_other[_M_read_addr] : `RANDOM;   // <stdin>:247:13
  `endif // RANDOMIZE_GARBAGE_ASSIGN

  `ifndef SYNTHESIS
  initial begin
    `INIT_RANDOM_PROLOG_
    `ifdef RANDOMIZE_MEM_INIT
      integer initvar;
      for (initvar = 0; initvar < 20; initvar = initvar+1) begin
        _M_id[initvar] = `RANDOM;
        _M_other[initvar] = `RANDOM;
      end       // <stdin>:247:13
    `endif // RANDOMIZE_MEM_INIT
  end // initial
  `endif // SYNTHESIS

  always @(posedge _M_write_clk) begin
    if (_M_write_en & _M_write_mask) begin
      _M_id[_M_write_addr] <= _M_write_data_id; // <stdin>:247:13
      _M_other[_M_write_addr] <= _M_write_data_other;   // <stdin>:247:13
    end
  end // always @(posedge)
endmodule

module MemEmpty();


  `ifndef SYNTHESIS
  initial begin
    `INIT_RANDOM_PROLOG_
    `ifdef RANDOMIZE_MEM_INIT
      integer initvar;
    `endif
  end // initial
  `endif // SYNTHESIS
endmodule

module MemOne();
  wire       _M_read_addr;      // <stdin>:253:13
  wire       _M_read_en;        // <stdin>:253:13
  wire       _M_read_clk;       // <stdin>:253:13
  wire [3:0] _M_read_data_id;   // <stdin>:253:13
  wire [7:0] _M_read_data_other;        // <stdin>:253:13
  wire       _M_write_addr;     // <stdin>:253:13
  wire       _M_write_en;       // <stdin>:253:13
  wire       _M_write_clk;      // <stdin>:253:13
  wire [3:0] _M_write_data_id;  // <stdin>:253:13
  wire [7:0] _M_write_data_other;       // <stdin>:253:13
  wire       _M_write_mask_id;  // <stdin>:253:13
  wire       _M_write_mask_other;       // <stdin>:253:13
  reg  [3:0] _M_id[0:0];
  reg  [7:0] _M_other[0:0];

  assign _M_read_data_id = _M_id[_M_read_addr]; // <stdin>:253:13
  assign _M_read_data_other = _M_other[_M_read_addr];   // <stdin>:253:13

  `ifndef SYNTHESIS
  initial begin
    `INIT_RANDOM_PROLOG_
    `ifdef RANDOMIZE_MEM_INIT
      _M_id[0] = `RANDOM;       // <stdin>:253:13
      _M_other[0] = `RANDOM;    // <stdin>:253:13
    `endif // RANDOMIZE_MEM_INIT
  end // initial
  `endif // SYNTHESIS

  always @(posedge _M_write_clk) begin
    if (_M_write_en & _M_write_mask) begin
      _M_id[_M_write_addr] <= _M_write_data_id; // <stdin>:253:13
      _M_other[_M_write_addr] <= _M_write_data_other;   // <stdin>:253:13
    end
  end // always @(posedge)
endmodule

module Attach(
  input a, b, c);


  `ifndef SYNTHESIS
    alias a = b = c;    // <stdin>:256:7
  `endif
  `ifdef SYNTHESIS
    assign a = b;       // <stdin>:256:7
    assign a = c;       // <stdin>:256:7
    assign b = a;       // <stdin>:256:7
    assign b = c;       // <stdin>:256:7
    assign c = a;       // <stdin>:256:7
    assign c = b;       // <stdin>:256:7
  `endif // SYNTHESIS
endmodule

module IsInvalid(
  output a);

endmodule

module Locations(
  input  [3:0] a,
  output [3:0] b);

  assign b = a + a + 4'h3 ^ 4'h1;       // <stdin>:262:12, :263:17, :264:12, :265:17, :266:12, :267:7
  assign b = a * a * 4'h2 ^ 4'h0;       // <stdin>:268:12, :269:17, :270:12, :271:17, :272:12, :273:7
endmodule
```

### RTL DialectからVerilogへ変換

MLIRのDialectとして定義されたRTL DialectからVerilogへ変換する。

```bash
circt-translate verilog-rtl-dialect.mlir --emit-verilog
```

出力は以下。

```verilog
// Standard header to adapt well known macros to our needs.\n";
`ifdef RANDOMIZE_GARBAGE_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_INVALID_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_REG_INIT
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_MEM_INIT
`define RANDOMIZE
`endif
`ifndef RANDOM
`define RANDOM $random
`endif
// Users can define 'PRINTF_COND' to add an extra gate to prints.
`ifdef PRINTF_COND
`define PRINTF_COND_ (`PRINTF_COND)
`else
`define PRINTF_COND_ 1
`endif
// Users can define 'STOP_COND' to add an extra gate to stop conditions.
`ifdef STOP_COND
`define STOP_COND_ (`STOP_COND)
`else
`define STOP_COND_ 1
`endif

// Users can define INIT_RANDOM as general code that gets injected into the
// initializer block for modules with registers.
`ifndef INIT_RANDOM
`define INIT_RANDOM
`endif

// If using random initialization, you can also define RANDOMIZE_DELAY to
// customize the delay used, otherwise 0.002 is used.
`ifndef RANDOMIZE_DELAY
`define RANDOMIZE_DELAY 0.002
`endif

// Define INIT_RANDOM_PROLOG_ for use in our modules below.
`ifdef RANDOMIZE
  `ifndef VERILATOR
    `define INIT_RANDOM_PROLOG_ `INIT_RANDOM #`RANDOMIZE_DELAY begin end
  `else
    `define INIT_RANDOM_PROLOG_ `INIT_RANDOM
  `endif
`else
  `define INIT_RANDOM_PROLOG_
`endif
module M1(
  input  [7:0] x,
  output [7:0] y,
  input  [7:0] z);

  assign y = (z + 8'h2A) * 8'h5;        // verilog-rtl-dialect.mlir:7:12, :8:11, :9:10, :10:10, :11:10, :12:5
  wire [7:0] _T = z * z * z;    // verilog-rtl-dialect.mlir:14:10
  assign y = {_T % 8'h5, z, _T};        // verilog-rtl-dialect.mlir:8:11, :15:10, :16:10, :17:10, :18:5
endmodule

module M2(
  input [7:0] x, y, z);

  wire [7:0] foo;       // verilog-rtl-dialect.mlir:38:11

  assign x = 8'h2A;     // verilog-rtl-dialect.mlir:35:12, :36:5
  assign foo = y;       // verilog-rtl-dialect.mlir:39:5
  assign z = foo;       // verilog-rtl-dialect.mlir:40:5
endmodule

module M3(
  input  [7:0]  x,
  output [7:0]  y,
  input  [7:0]  z,
  input  [15:0] q);

  wire [7:0] _T = z + 8'h2A;    // verilog-rtl-dialect.mlir:56:12, :59:10
  wire [7:0] _T_0 = _T & 8'h2A & 8'h5;  // verilog-rtl-dialect.mlir:56:12, :57:11, :60:11
  assign y = _T_0 ^ (_T | _T_0) ^ 8'h2A ^ q[15:8];      // verilog-rtl-dialect.mlir:56:12, :58:14, :61:11, :62:11, :63:10, :64:5
endmodule
```

## `translate`と`opt`のヘルプ

```bash
$ circt-translate -h
OVERVIEW: CIRCT translation driver

USAGE: circt-translate [options] <input file>

OPTIONS:

Color Options:

  --color                                              - Use colors in output (default=autodetect)

General options:

  --mlir-disable-threading                             - Disabling multi-threading within MLIR
  --mlir-elide-elementsattrs-if-larger=<uint>          - Elide ElementsAttrs with "..." that have more elements than the given upper limit
  --mlir-pretty-debuginfo                              - Print pretty debug info in MLIR output
  --mlir-print-debuginfo                               - Print debug info in MLIR output
  --mlir-print-elementsattrs-with-hex-if-larger=<long> - Print DenseElementsAttrs with a hex string that have more elements than the given upper limit (use -1 to disable)
  --mlir-print-op-on-diagnostic                        - When a diagnostic is emitted on an operation, also print the operation as an attached note
  --mlir-print-stacktrace-on-diagnostic                - When a diagnostic is emitted, also print the stack trace as an attached note
  -o=<filename>                                        - Output filename
  Translation to perform
      --emit-verilog                                      - emit-verilog
      --llhd-to-verilog                                   - llhd-to-verilog
      --parse-fir                                         - parse-fir
  --split-input-file                                   - Split the input file into pieces and process each chunk independently
  --verify-diagnostics                                 - Check that emitted diagnostics match expected-* lines on the corresponding line

Generic Options:

  --help                                               - Display available options (--help-hidden for more)
  --help-list                                          - Display list of available options (--help-list-hidden for more)
  --version                                            - Display the version of this program
```

```bash
$ circt-opt -h
OVERVIEW: circt modular optimizer driver

USAGE: circt-opt [options] <input file>

OPTIONS:

Color Options:

  --color                                              - Use colors in output (default=autodetect)

General options:

  --allow-unregistered-dialect                         - Allow operation with no registered dialects
  --enable-name-compression                            - Enable name/filename string compression
  --mlir-disable-threading                             - Disabling multi-threading within MLIR
  --mlir-elide-elementsattrs-if-larger=<uint>          - Elide ElementsAttrs with "..." that have more elements than the given upper limit
  --mlir-pretty-debuginfo                              - Print pretty debug info in MLIR output
  --mlir-print-debuginfo                               - Print debug info in MLIR output
  --mlir-print-elementsattrs-with-hex-if-larger=<long> - Print DenseElementsAttrs with a hex string that have more elements than the given upper limit (use -1 to disable)
  --mlir-print-op-on-diagnostic                        - When a diagnostic is emitted on an operation, also print the operation as an attached note
  --mlir-print-stacktrace-on-diagnostic                - When a diagnostic is emitted, also print the stack trace as an attached note
  -o=<filename>                                        - Output filename
  --pass-pipeline-crash-reproducer=<string>            - Generate a .mlir reproducer file at the given output path if the pass manager crashes or fails
  --pass-pipeline-local-reproducer                     - When generating a crash reproducer, attempt to generated a reproducer with the smallest pipeline.
  --pass-statistics                                    - Display the statistics of each pass
  --pass-statistics-display=<value>                    - Display method for pass statistics
    =list                                              -   display the results in a merged list sorted by pass name
    =pipeline                                          -   display the results with a nested pipeline view
  --pass-timing                                        - Display the execution times of each pass
  --pass-timing-display=<value>                        - Display method for pass timing data
    =list                                              -   display the results in a list sorted by total time
    =pipeline                                          -   display the results with a nested pipeline view
  --print-ir-after                                     - Print IR after specified passes
    --pass-pipeline                                    -   A textual description of a pass pipeline to run
    Passes:
      --analyze-dataflow                               -   Print resource (operation) statistics
      --canonicalize                                   -   Canonicalize operations
      --canonicalize-dataflow                          -   Canonicalize handshake IR
      --convert-llhd-to-llvm                           -   Convert LLHD to LLVM
      --create-dataflow                                -   Convert standard MLIR into dataflow IR
      --create-pipeline                                -   Create StaticLogic pipeline operations.
      --cse                                            -   Eliminate common sub-expressions
      --handshake-insert-buffer                        -   Insert buffers to break graph cycles.
      --inline                                         -   Inline function calls
        --disable-simplify                             - Disable running simplifications during inlining
        ---iterations=<uint>                        - Maximum number of iterations when inlining within an SCC
      --llhmaxd-early-code-motion                         -   Move side-effect-free instructions and llhd.prb up in the CFG
      --llhd-function-elimination                      -   Deletes all functions.
      --llhd-process-lowering                          -   Lowers LLHD Processes to Entities.
      --lower-firrtl-to-rtl                            -   Lower FIRRTL to RTL
      --lower-handshake-to-firrtl                      -   Lowering to FIRRTL Dialect
      --remove-block-structure                         -   Remove block structure in handshake IR
  --print-ir-after-all                                 - Print IR after each pass
  --print-ir-after-change                              - When printing the IR after a pass, only print if the IR changed
  --print-ir-before                                    - Print IR before specified passes
    --pass-pipeline                                    -   A textual description of a pass pipeline to run
    Passes:
      --analyze-dataflow                               -   Print resource (operation) statistics
      --canonicalize                                   -   Canonicalize operations
      --canonicalize-dataflow                          -   Canonicalize handshake IR
      --convert-llhd-to-llvm                           -   Convert LLHD to LLVM
      --create-dataflow                                -   Convert standard MLIR into dataflow IR
      --create-pipeline                                -   Create StaticLogic pipeline operations.
      --cse                                            -   Eliminate common sub-expressions
      --handshake-insert-buffer                        -   Insert buffers to break graph cycles.
      --inline                                         -   Inline function calls
        --disable-simplify                             - Disable running simplifications during inlining
        --max-iterations=<uint>                        - Maximum number of iterations when inlining within an SCC
      --llhd-early-code-motion                         -   Move side-effect-free instructions and llhd.prb up in the CFG
      --llhd-function-elimination                      -   Deletes all functions.
      --llhd-process-lowering                          -   Lowers LLHD Processes to Entities.
      --lower-firrtl-to-rtl                            -   Lower FIRRTL to RTL
      --lower-handshake-to-firrtl                      -   Lowering to FIRRTL Dialect
      --remove-block-structure                         -   Remove block structure in handshake IR
  --print-ir-before-all                                - Print IR before each pass
  --print-ir-module-scope                              - When printing IR for print-ir-[before|after]{-all} always print the top-level module operation
  Compiler passes to run
    --pass-pipeline                                    -   A textual description of a pass pipeline to run
    Passes:
      --analyze-dataflow                               -   Print resource (operation) statistics
      --canonicalize                                   -   Canonicalize operations
      --canonicalize-dataflow                          -   Canonicalize handshake IR
      --convert-llhd-to-llvm                           -   Convert LLHD to LLVM
      --create-dataflow                                -   Convert standard MLIR into dataflow IR
      --create-pipeline                                -   Create StaticLogic pipeline operations.
      --cse                                            -   Eliminate common sub-expressions
      --handshake-insert-buffer                        -   Insert buffers to break graph cycles.
      --inline                                         -   Inline function calls
        --disable-simplify                             - Disable running simplifications during inlining
        --max-iterations=<uint>                        - Maximum number of iterations when inlining within an SCC
      --llhd-early-code-motion                         -   Move side-effect-free instructions and llhd.prb up in the CFG
      --llhd-function-elimination                      -   Deletes all functions.
      --llhd-process-lowering                          -   Lowers LLHD Processes to Entities.
      --lower-firrtl-to-rtl                            -   Lower FIRRTL to RTL
      --lower-handshake-to-firrtl                      -   Lowering to FIRRTL Dialect
      --remove-block-structure                         -   Remove block structure in handshake IR
  --show-dialects                                      - Print the list of registered dialects
  --split-input-file                                   - Split the input file into pieces and process each chunk independently
  --verify-diagnostics                                 - Check that emitted diagnostics match expected-* lines on the corresponding line
  --verify-each                                        - Run the verifier after each transformation pass

Generic Options:

  --help                                               - Display available options (--help-hidden for more)
  --help-list                                          - Display list of available options (--help-list-hidden for more)
  --version                                            - Display the version of this program
```
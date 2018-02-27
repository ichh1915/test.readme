# HLP Project
## Hao Hao(hh1915)

## 1.Compatibility and Contribution to the Group

* To ensure the interface is compatible during the group deliverable:
The program have three main high-level function with their interfaces shown as below:
* `ParseOperands:DataPath->Token List-><self-define operand type>`
* `ParseInstr:DataPath->LineData->Result<Parse<Instr>,string> option`
* `Execution:DataPath->Parse<Instr>->DataPath`

* The overall interface is shown in the flow chart below:

![Diagram](https://github.com/ichh1915/ARM/blob/master/flowchart.png)

* The following functions are compatible with other group members' module and can be conveniently adopted with little modification:

Function | Description
------------ | -------------
`tokenize:string->TokenList` | Tokenize a string of all operands
`flexOp2:Datapath->FlexOp2->uint32` | Calculate the flexible second operand as `uint32`
`Op2SetCFlag:Datapath->FlexOp2->bool option` |  Parse the `C flag` update status during the calculation of the flexible Op2
`checkCond:DataPath->Condition->bool` | Check whether the flags status match the condition during the conditional operation

## 2.Code Specification
### 2.1 Functionalities:

Operations | Syntax
------------ | -------------
`LSL,LSR,ASR,ROR,RRX` | `op{S}{cond} Rd, Rn, Rs` or  `op{S}{cond} Rd, Rm, #n` or `RRX{S}{cond} Rd, Rm`
`AND,ORR,BIC,EOR` | `op{S}{cond} Rd, Rn, FlexOperand2`
`MOV,NVN` | `op{S}{cond} Rd, FlexOperand2`
`TST,TEQ` |`op{cond} Rn, FlexOperand2`

* Flexible Second Operand(`FlexOperand2`) is calculated as `uint32`, before being parsed to `parse<Instr>` ;`SetC` is parsed as `bool option` representing the `C flag` update status when calculating `FlexOperand2`.
* For `Rs` during shift operations, only the least significant `byte` is used and can be in the range of `0-255`.
* For `#n` during shift operations, only the least siginificant `5-bits` are used and can be in the range of `0-31`, this feature is not implimented in VisUAL, the reasoning is to eliminate redundant number of shift(`#n`).
* The immediate literals are tested to be creatable by rotating a 8-bit number right within a 32-bit word.
* Updating Flags:
* `Shift Operations`:The `C flag` is updated to the last bit shifted out, except when the shift length is 0. `N and C` are updated according to the result.

* `Bitwise Operations` & `mov/mvn` & `tst/teq`:The `C flag` can be updated during the calculation of the `FlexOperand2`. `N and C` are updated according to the result.

### 2.2 Directories of main functions:
#### `TokenizeOperands.fs`
* `ParsesomeOperationOps`: parse `Token List` to `self-defined operand type` with additional input of `DataPath`
* `tokenize: string->token List`: tokenize a `string` of all operands to `token List`
* `FlexOp2: DataPath->FlexOp2->uint32`: calculate the flexible second operand as `uint32`
* `Op2SetCFlag: DataPath->FlexOp2->bool option`: return the `C flag` status updated when calculating `FlexOp2`


####  `SFT.fs`
* `ShiftExecute: DataPath->Parse<Instr>->DataPath`: execute shift operation and update the `DataPath`
* `updateFlRegs`: update flag and register contents after an operation
* `CheckCond: DataPath->Condition->bool`: check whether the flags matches the condition for conditional operation
* `SFparse`: parse `LineData` to `Result<Parse<Instr>,string> option` with additional input of `DataPath`
#### `BIT.fs`
* `BitwiseExecute: DataPath->Parse<Instr>->DataPath`
* `updateFlRegs`
* `BTparse`
####  `TST.fs`
* `TestExecute: DataPath->Parse<Instr>->DataPath`
* `updateFlRegs`
* `TTparse`
####  `MOV.fs`
* `MovsExecute: DataPath->Parse<Instr>->DataPath`
* `updateFlRegs`
* `MVparse`

## 3.Test Plan
**The Tests are desgined as unit tests with randomized initail states under `VTest.fs`**
* The specific Test method is as following:
* Generate random initial `R0-R14` register contents;
* Generate random initial `NZCV` Flags;
* fail with `Error` when N and V are noth true.
* Initialise assembly instructions covering all realised operations and operation modes;
* Feed the above initial states as DataPath and LineData in to F#'s `someExecution:DataPath->Parse<Instr>->DataPath`;
* Input the initial states as Params and string to visUAL `RunVisualWithFlagsOut->Params->string->Flags->VisOutput`;
* Test the output states for `F# assembler` and `visUAL` are **equal**, namingly the updated `register contents` and the updated `flags` are **equal** .

The four types of operations are represented as symbols:

Operations | Symbol
------------ | -------------
`LSL,LSR,ASR,ROR,RRX`| `SFT`
`AND,ORR,BIC,EOR` | `BIT`
`MOV,NVN` | `MOV`
`TST,TEQ` | `TST`

The specific unit tests and their status are as following:

Test assembly line | Status
------------ | -------------
`SFT{S} Rd, Rn, #n` with `#n>=32` | Expected to `Fail`,due to only `n modulo 32` are used for shift operations in the F# program, whereas visUAL allow number of shift to be `>32`
`SFT{S} Rd, Rn, #n` with `#n<32` | `Success`
`SFT{S} Rd, Rn, Rs` | `Success`
`RRX{S} Rd, Rm` | `Success`
`SFT{S} Rd, Rn, Rs` in the cases `Rd=Rn` | `Success` corner case
`BIT{S} Rd, Rn, #n` | `Success`
`BIT{S} Rd, Rn, Rs` | `Success`
`BIT{S} Rd, Rn, Rs, SHIFT, #n` | `Success`
`BIT{S} Rd, Rn, Rs, SHIFT, Rm` | `Success`
`MOV{S} Rd, #n` | `Success`
`MOV{S} Rd, Rs` | `Success`
`MOV{S} Rd, Rs, SHIFT, #n` | `Success`
`MOV{S} Rd, Rs, SHIFT, Rm` | `Success`
`TST{S} Rn, #n` | `Success`
`TST{S} Rn, Rs` | `Success`
`TST{S} Rn, Rs, SHIFT, #n` | `Success`
`TST{S} Rn, Rs, SHIFT, Rm` | `Success`
`MVNS{Cond} "R5,R10,ROR R6`| Test all conditions. `Success` for sufficient number of test runs, each run with random initial flags
`MVNSNV R5,R10,ROR R6` | Expected to have `error`, NV condition is omitted by visUAL
`ORRS R1,R1,R3,RRX #1` | Expected to have `error`, the #1 for RRX is omitted by visUAL













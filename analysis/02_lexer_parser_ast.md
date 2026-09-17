# Betterado Front-End: Lexer, Preprocessor, Resilient Parser & AST

## 1. Overview and Design Objectives

The `betterado-syntax` crate implements the front-end HDL ingestion engine. Its objectives are:
1. **High Throughput**: Lexing and parsing millions of lines of Verilog and SystemVerilog per second.
2. **Resilience & Fault Tolerance**: Syntax errors do not halt parsing; the parser recovers and parses remaining modules to provide comprehensive IDE feedback.
3. **Lossless Source Mapping**: Accurate byte spans for every token and AST node to power IDE hover, jump-to-definition, and diagnostic squiggles.

---

## 2. Streaming Lexer & Tokenizer

The lexer uses a zero-allocation streaming architecture directly over UTF-8 string slices (`&str`).

### 2.1. Token Specification
```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum TokenKind {
    // Keywords
    Module, EndModule, Input, Output, Inout, Wire, Reg, Logic,
    Assign, Always, AlwaysComb, AlwaysFf, AlwaysLatch, Initial,
    Begin, End, If, Else, Case, EndCase, Default,
    Parameter, LocalParam, Generate, EndGenerate, For,
    Posedge, Negedge,

    // Operators
    Plus, Minus, Star, Slash, Percent,
    Amp, Pipe, Caret, Tilde, Bang,
    AmpAmp, PipePipe, EqEq, BangEq, EqEqEq, BangEqEq,
    Lt, LtEq, Gt, GtEq,
    Shl, Shr, ShlArithmetic, ShrArithmetic,
    AssignEq, AssignLe, // = and <=
    Question, Colon,

    // Delimiters
    LParen, RParen, LBracket, RBracket, LBrace, RBrace,
    Semicolon, Comma, Dot, Hash, At,

    // Literals & Identifiers
    Ident(Spur),
    IntLiteral { width: Option<u32>, signed: bool, value: u128 },
    StringLiteral(Spur),

    // Special
    Eof,
    Error,
}

#[derive(Debug, Clone, Copy)]
pub struct Token {
    pub kind: TokenKind,
    pub span: Span, // start and end byte offsets
}
```

### 2.2. Preprocessor Engine
The preprocessor runs transparently ahead of the parser:
- **Macro Definitions**: `` `define WIDTH 32 ``, `` `define CLOG2(x) ... ``
- **Conditional Compilation**: `` `ifdef ``, `` `ifndef ``, `` `elsif ``, `` `else ``, `` `endif ``
- **File Inclusion**: `` `include "header.vh" `` with circular dependency detection.
- **Timescale Directives**: `` `timescale 1ns/1ps `` captured to configure the simulation time resolution.

---

## 3. AST (Abstract Syntax Tree) Design

The AST represents synthesizable and behavioral RTL constructs in a clean, strongly typed hierarchy:

```rust
pub struct SourceFile<'ast> {
    pub modules: Vec<ModuleDef<'ast>>,
    pub packages: Vec<PackageDef<'ast>>,
}

pub struct ModuleDef<'ast> {
    pub name: Spur,
    pub params: Vec<ParamDecl<'ast>>,
    pub ports: Vec<PortDecl<'ast>>,
    pub items: Vec<ModuleItem<'ast>>,
    pub span: Span,
}

pub enum ModuleItem<'ast> {
    NetDecl(NetDecl<'ast>),
    ContinuousAssign(AssignStmt<'ast>),
    ProceduralBlock(ProceduralBlock<'ast>),
    Instance(InstanceDef<'ast>),
    GenerateBlock(GenerateBlock<'ast>),
}

pub struct ProceduralBlock<'ast> {
    pub kind: ProceduralKind, // Initial, Always, AlwaysComb, AlwaysFf
    pub sensitivity: Option<SensitivityList<'ast>>,
    pub body: Statement<'ast>,
    pub span: Span,
}

pub enum Statement<'ast> {
    Block(Vec<Statement<'ast>>),
    BlockingAssign { target: LValue<'ast>, expr: Expr<'ast> },
    NonBlockingAssign { target: LValue<'ast>, expr: Expr<'ast> },
    If { cond: Expr<'ast>, then_branch: Box<Statement<'ast>>, else_branch: Option<Box<Statement<'ast>>> },
    Case { expr: Expr<'ast>, items: Vec<CaseItem<'ast>> },
    For { init: Box<Statement<'ast>>, cond: Expr<'ast>, step: Box<Statement<'ast>>, body: Box<Statement<'ast>> },
    Null,
}
```

---

## 4. Parser Implementation: Recursive Descent with Pratt Expressions

1. **Declarations & Statements**: Parsed via recursive descent. When encountering unexpected tokens, the parser skips forward to statement terminators (`;`, `end`, `endmodule`) using synchronization sets.
2. **Expressions**: Parsed using **Pratt Parsing** (Top-Down Operator Precedence). This avoids deep call stack recursion for deeply nested binary and ternary operations (`a ? b : c`, bitwise ops) and naturally encodes Verilog precedence rules:
   - Level 1: Unary operators (`+`, `-`, `~`, `!`, `&`, `|`, `^`)
   - Level 2: Multiplication/Division (`*`, `/`, `%`)
   - Level 3: Addition/Subtraction (`+`, `-`)
   - Level 4: Shifts (`<<`, `>>`, `<<<`, `>>>`)
   - Level 5: Relational & Equality (`<`, `>`, `==`, `!=`, `===`, `!==`)
   - Level 6: Bitwise (`&`, `^`, `|`)
   - Level 7: Logical (`&&`, `||`)
   - Level 8: Conditional Ternary (`?:`)

---

## 5. Performance Targets

- Parse speed: $> 2.5 \text{ million lines/sec}$ on modern multi-core x86_64 / Apple Silicon.
- Memory footprint: $< 40 \text{ bytes}$ per AST node due to `bumpalo` arena allocation and identifier interning.

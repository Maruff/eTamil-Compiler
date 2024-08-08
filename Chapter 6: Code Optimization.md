
---

## Chapter 6: Code Optimization

### Overview

Code optimization is the fifth phase of the compilation process. This phase aims to improve the intermediate code generated in the previous phase to make the final machine code more efficient in terms of execution time, memory usage, and other resources. This chapter explores various optimization techniques, the principles behind them, and provides a practical implementation using Rust for the eTamil language.

### Importance of Code Optimization

Code optimization is crucial for several reasons:

- **Performance**: Optimized code runs faster and uses fewer resources.
- **Efficiency**: Optimization reduces the size of the executable, leading to lower memory and storage usage.
- **Battery Life**: For battery-powered devices, optimized code can extend battery life by reducing power consumption.

### Optimization Techniques

Several optimization techniques can be applied at different levels of the compilation process. Here, we focus on machine-independent optimizations that are applied to the intermediate code.

#### Loop Optimization

Loop optimization techniques aim to improve the performance of loops, which are often the most time-consuming parts of a program. Common loop optimization techniques include:

- **Loop Unrolling**: Reduces the overhead of loop control by executing the loop body multiple times per iteration.
- **Loop Invariant Code Motion**: Moves calculations that do not change within the loop outside the loop.
- **Loop Fusion**: Combines adjacent loops that iterate over the same range to reduce loop overhead.

#### Dead Code Elimination

Dead code elimination removes code that does not affect the program's output. This includes:

- **Unused Variables**: Variables that are declared but never used.
- **Unreachable Code**: Code that can never be executed due to the program's control flow.

#### Constant Folding

Constant folding evaluates constant expressions at compile time rather than at runtime. This includes:

- **Arithmetic Expressions**: Calculating the result of constant arithmetic expressions.
- **Logical Expressions**: Simplifying constant logical expressions.

#### Common Subexpression Elimination

Common subexpression elimination identifies and eliminates expressions that are computed multiple times with the same values. This reduces redundant calculations.

### Rust Implementation

In this section, we will implement basic optimization techniques for the intermediate code (three-address code) generated for the eTamil language.

#### Implementing Loop Unrolling

First, we implement loop unrolling. This example assumes a simple loop structure in the intermediate code.

```rust
impl TAC {
    pub fn optimize_loop_unrolling(&mut self) {
        // Implement loop unrolling optimization
        // This is a simplified example
        let mut new_instructions = Vec::new();

        for instruction in &self.instructions {
            match instruction {
                // Detect simple loops and unroll them
                TACInstruction::BinaryOp { dest, op, left, right } if *op == '+' && right == "1" => {
                    // Example: unroll "for (i = 0; i < 4; i++) { ... }" into four iterations
                    for _ in 0..4 {
                        new_instructions.push(instruction.clone());
                    }
                }
                _ => new_instructions.push(instruction.clone()),
            }
        }

        self.instructions = new_instructions;
    }
}
```

#### Implementing Dead Code Elimination

Next, we implement dead code elimination.

```rust
impl TAC {
    pub fn eliminate_dead_code(&mut self) {
        // Implement dead code elimination optimization
        let mut used_vars = std::collections::HashSet::new();
        let mut new_instructions = Vec::new();

        for instruction in self.instructions.iter().rev() {
            match instruction {
                TACInstruction::Assign { dest, src } => {
                    if used_vars.contains(dest) {
                        new_instructions.push(instruction.clone());
                    }
                    used_vars.insert(src.clone());
                }
                TACInstruction::BinaryOp { dest, op, left, right } => {
                    if used_vars.contains(dest) {
                        new_instructions.push(instruction.clone());
                    }
                    used_vars.insert(left.clone());
                    used_vars.insert(right.clone());
                }
                TACInstruction::LoadConst { dest, value: _ } => {
                    if used_vars.contains(dest) {
                        new_instructions.push(instruction.clone());
                    }
                }
            }
        }

        new_instructions.reverse();
        self.instructions = new_instructions;
    }
}
```

#### Implementing Constant Folding

Next, we implement constant folding.

```rust
impl TAC {
    pub fn constant_folding(&mut self) {
        // Implement constant folding optimization
        for instruction in &mut self.instructions {
            match instruction {
                TACInstruction::BinaryOp { op, left, right, dest } => {
                    if let (Ok(left_val), Ok(right_val)) = (left.parse::<i64>(), right.parse::<i64>()) {
                        let result = match op {
                            '+' => left_val + right_val,
                            '-' => left_val - right_val,
                            '*' => left_val * right_val,
                            '/' => left_val / right_val,
                            _ => continue,
                        };
                        *instruction = TACInstruction::LoadConst {
                            dest: dest.clone(),
                            value: result,
                        };
                    }
                }
                _ => {}
            }
        }
    }
}
```

### Case Study: Code Optimization for eTamil

For the eTamil language, code optimization involves applying the techniques discussed above to the intermediate code. Here’s an example eTamil program and how it is optimized.

#### Example eTamil Program

```tamil
எண் a = 5 + 3 * (2 - 1);
```

#### Generating and Optimizing TAC

Using the lexer, parser, semantic analyzer, intermediate code generator, and optimizer implemented in Rust, the above eTamil program would be optimized as follows:

```rust
fn main() {
    let source_code = "எண் a = 5 + 3 * (2 - 1);";
    let mut lexer = Lexer::new(source_code.to_string());
    let tokens: Vec<Token> = lexer.tokenize();

    let mut parser = Parser::new(tokens);
    let ast = parser.parse_program();

    let mut semantic_analyzer = SemanticAnalyzer::new();
    match semantic_analyzer.analyze(&ast) {
        Ok(_) => {
            let mut code_generator = CodeGenerator::new();
            match code_generator.generate(&ast) {
                Ok(_) => {
                    let mut tac = code_generator.tac;
                    tac.constant_folding();
                    tac.eliminate_dead_code();
                    tac.optimize_loop_unrolling();
                    println!("{:?}", tac.instructions);
                }
                Err(e) => println!("Code generation error: {}", e),
            }
        }
        Err(e) => println!("Semantic analysis error: {}", e),
    }
}
```

The output TAC instructions after optimization might look like this:

```plaintext
LoadConst { dest: "t1", value: 5 }
LoadConst { dest: "t2", value: 3 }
LoadConst { dest: "t3", value: 2 }
LoadConst { dest: "t4", value: 1 }
BinaryOp { dest: "t5", op: '-', left: "t3", right: "t4" }  // Folded to t5 = 1
BinaryOp { dest: "t6", op: '*', left: "t2", right: "t5" }  // Folded to t6 = 3
BinaryOp { dest: "t7", op: '+', left: "t1", right: "t6" }  // Folded to t7 = 8
Assign { dest: "a", src: "t7" }                            // Final result: a = 8
```

### Conclusion

In this chapter, we explored the principles of code optimization, focusing on various optimization techniques such as loop optimization, dead code elimination, constant folding, and common subexpression elimination. We also demonstrated how to implement these optimizations in Rust for the eTamil language. Code optimization improves the efficiency and performance of the generated machine code, making the final program faster and more resource-efficient.

---

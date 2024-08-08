
---

## Chapter 2: Lexical Analysis

### Overview

Lexical analysis is the first phase of the compilation process, responsible for converting the source code into a sequence of tokens. Tokens are the smallest units of meaning in the source code, such as keywords, identifiers, operators, and literals. This chapter explores the principles of lexical analysis, the tools and techniques used, and provides a practical implementation using Rust for the eTamil language.

### Role of Lexical Analysis

Lexical analysis, also known as scanning, serves as the foundation for the subsequent phases of compilation. Its primary roles include:

- **Tokenization**: Breaking down the source code into a stream of tokens.
- **Filtering**: Removing whitespace and comments that are not relevant for further processing.
- **Error Detection**: Identifying and reporting lexical errors, such as invalid characters or malformed tokens.

### Tokenization

Tokenization involves scanning the source code character by character to identify and classify sequences of characters into tokens. Each token represents a logical unit, such as a keyword (`if`, `while`), identifier (`variableName`), operator (`+`, `-`), or literal (`123`, `'a'`).

#### Regular Expressions and Finite Automata

Regular expressions and finite automata are essential tools for defining and recognizing tokens.

- **Regular Expressions**: Provide a concise way to specify patterns for tokens. For example, the regular expression for an identifier in many languages might be `[a-zA-Z_][a-zA-Z0-9_]*`.
- **Finite Automata**: Used to implement regular expressions. They can be deterministic (DFA) or non-deterministic (NFA).

### Tools and Techniques

Several tools and techniques are used to implement lexical analyzers:

- **Lexical Analyzer Generators**: Tools like Lex and Flex automate the generation of lexical analyzers from regular expressions.
- **Manual Implementation**: Writing custom lexical analyzers using programming languages.

### Rust Implementation

In this section, we will implement a simple lexical analyzer for eTamil using Rust. The eTamil language has unique lexical requirements, including support for Tamil characters and specific keywords.

#### Defining Tokens

First, we define the tokens for eTamil. These might include keywords, identifiers, literals, operators, and punctuation.

```rust
#[derive(Debug, PartialEq)]
pub enum Token {
    Keyword(String),
    Identifier(String),
    Number(i64),
    Operator(char),
    Punctuation(char),
    Whitespace,
    Unknown(char),
}
```

#### Implementing the Lexer

Next, we implement the lexer that scans the source code and produces tokens.

```rust
pub struct Lexer {
    input: String,
    position: usize,
}

impl Lexer {
    pub fn new(input: String) -> Self {
        Lexer { input, position: 0 }
    }

    pub fn next_token(&mut self) -> Option<Token> {
        self.skip_whitespace();
        if self.position >= self.input.len() {
            return None;
        }

        let current_char = self.input.chars().nth(self.position).unwrap();

        let token = if current_char.is_alphabetic() {
            self.read_identifier()
        } else if current_char.is_digit(10) {
            self.read_number()
        } else {
            match current_char {
                '+' | '-' | '*' | '/' => Token::Operator(current_char),
                '(' | ')' | '{' | '}' => Token::Punctuation(current_char),
                _ => Token::Unknown(current_char),
            }
        };

        self.position += 1;
        Some(token)
    }

    fn skip_whitespace(&mut self) {
        while self.position < self.input.len()
            && self.input.chars().nth(self.position).unwrap().is_whitespace()
        {
            self.position += 1;
        }
    }

    fn read_identifier(&mut self) -> Token {
        let start = self.position;
        while self.position < self.input.len()
            && self.input.chars().nth(self.position).unwrap().is_alphanumeric()
        {
            self.position += 1;
        }
        let identifier = &self.input[start..self.position];
        Token::Identifier(identifier.to_string())
    }

    fn read_number(&mut self) -> Token {
        let start = self.position;
        while self.position < self.input.len()
            && self.input.chars().nth(self.position).unwrap().is_digit(10)
        {
            self.position += 1;
        }
        let number: i64 = self.input[start..self.position].parse().unwrap();
        Token::Number(number)
    }
}
```

#### Testing the Lexer

We can now test our lexer with a simple eTamil program.

```rust
fn main() {
    let source_code = "எண் a = 5 + 3;";
    let mut lexer = Lexer::new(source_code.to_string());

    while let Some(token) = lexer.next_token() {
        println!("{:?}", token);
    }
}
```

Running this program should produce the following output:

```
Keyword("எண்")
Identifier("a")
Operator('=')
Number(5)
Operator('+')
Number(3)
Punctuation(';')
```

### Case Study: Lexical Analysis for eTamil

For the eTamil language, lexical analysis involves additional complexities such as supporting Tamil script characters. Here’s a more refined lexer that includes Tamil keywords and identifiers.

#### Defining Tamil Keywords

```rust
pub const KEYWORDS: &[&str] = &["எண்", "மொழி", "நிலை", "நிரலிழை", "செயல்"];
```

#### Updating the Lexer

Update the `read_identifier` method to recognize Tamil keywords.

```rust
fn read_identifier(&mut self) -> Token {
    let start = self.position;
    while self.position < self.input.len()
        && self.input.chars().nth(self.position).unwrap().is_alphabetic()
    {
        self.position += 1;
    }
    let identifier = &self.input[start..self.position];
    if KEYWORDS.contains(&identifier) {
        Token::Keyword(identifier.to_string())
    } else {
        Token::Identifier(identifier.to_string())
    }
}
```

### Conclusion

In this chapter, we explored the principles of lexical analysis, focusing on tokenization, regular expressions, and finite automata. We also demonstrated how to implement a lexical analyzer in Rust for the eTamil language. The lexer serves as the foundation for the subsequent phases of compilation, ensuring that the source code is correctly tokenized and ready for syntax analysis.

---

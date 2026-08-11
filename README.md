# .AdaptML-Book4
📘 AdaptML — अध्याय 4
AdaptML Lexer — Source Code
अब हम AdaptML compiler का पहला वास्तविक हिस्सा बनाएँगे: Lexer।
4.1 Lexer क्या करता है?
Lexer AdaptML source code को छोटे-छोटे tokens में बदलता है।
उदाहरण:
<text color="blue">Hello</text>
Lexer इसे लगभग ऐसे पहचान सकता है:
TAG_OPEN   <
TAG_NAME   text
ATTRIBUTE  color
EQUAL      =
STRING     "blue"
TEXT       Hello
TAG_CLOSE  </text>
इस प्रक्रिया को Lexical Analysis कहते हैं।
4.2 Lexer का Flow
app.adaptml
     ↓
   Lexer
     ↓
   Tokens
     ↓
   Parser
Lexer अभी यह तय नहीं करता कि document सही structure में है या नहीं। वह मुख्यतः source को पहचानने योग्य tokens में बदलता है।
4.3 lexer.js
यह हमारा शुरुआती AdaptML Lexer है:
class AdaptMLLexer {
  constructor(input) {
    this.input = input;
    this.position = 0;
    this.tokens = [];
  }

  tokenize() {
    while (this.position < this.input.length) {
      const char = this.input[this.position];

      // Whitespace
      if (/\s/.test(char)) {
        this.position++;
        continue;
      }

      // Opening tag
      if (char === '<') {
        if (this.input[this.position + 1] === '/') {
          this.tokens.push({
            type: "CLOSE_TAG_START",
            value: "</"
          });

          this.position += 2;
        } else {
          this.tokens.push({
            type: "OPEN_TAG_START",
            value: "<"
          });

          this.position++;
        }

        continue;
      }

      // Closing angle bracket
      if (char === '>') {
        this.tokens.push({
          type: "TAG_END",
          value: ">"
        });

        this.position++;
        continue;
      }

      // Equal sign
      if (char === '=') {
        this.tokens.push({
          type: "EQUAL",
          value: "="
        });

        this.position++;
        continue;
      }

      // String
      if (char === '"' || char === "'") {
        const quote = char;
        this.position++;

        let value = "";

        while (
          this.position < this.input.length &&
          this.input[this.position] !== quote
        ) {
          value += this.input[this.position];
          this.position++;
        }

        this.position++;

        this.tokens.push({
          type: "STRING",
          value
        });

        continue;
      }

      // Identifier
      if (/[a-zA-Z0-9_-]/.test(char)) {
        let value = "";

        while (
          this.position < this.input.length &&
          /[a-zA-Z0-9_-]/.test(this.input[this.position])
        ) {
          value += this.input[this.position];
          this.position++;
        }

        this.tokens.push({
          type: "IDENTIFIER",
          value
        });

        continue;
      }

      // Unknown character
      this.tokens.push({
        type: "UNKNOWN",
        value: char
      });

      this.position++;
    }

    return this.tokens;
  }
}

module.exports = AdaptMLLexer;
4.4 Lexer को चलाना
एक नई file बनाएँ:
test-lexer.js
उसमें:
const AdaptMLLexer = require("./lexer");

const source = `
<page title="Home">
  <text>Hello AdaptML</text>
</page>
`;

const lexer = new AdaptMLLexer(source);

const tokens = lexer.tokenize();

console.log(tokens);
Run करें:
node test-lexer.js
4.5 Tokens का उदाहरण
Output कुछ इस तरह दिखाई देगा:
[
  { type: 'OPEN_TAG_START', value: '<' },
  { type: 'IDENTIFIER', value: 'page' },
  { type: 'IDENTIFIER', value: 'title' },
  { type: 'EQUAL', value: '=' },
  { type: 'STRING', value: 'Home' },
  { type: 'TAG_END', value: '>' },

  { type: 'OPEN_TAG_START', value: '<' },
  { type: 'IDENTIFIER', value: 'text' },
  { type: 'TAG_END', value: '>' },

  { type: 'IDENTIFIER', value: 'Hello' },
  { type: 'IDENTIFIER', value: 'AdaptML' },

  { type: 'CLOSE_TAG_START', value: '</' },
  { type: 'IDENTIFIER', value: 'text' },
  { type: 'TAG_END', value: '>' }
]
यह अभी पहला सरल Lexer है। आगे हम इसे बेहतर बनाएँगे ताकि text, comments, self-closing tags और errors को सही तरीके से संभाल सके।
4.6 Lexer के बाद क्या होगा?
अब हमारा architecture:
┌─────────────────┐
│ app.adaptml     │
└────────┬────────┘
         ↓
┌─────────────────┐
│ lexer.js        │
│ Source → Tokens │
└────────┬────────┘
         ↓
┌─────────────────┐
│ parser.js       │
│ Tokens → AST    │
└────────┬────────┘
         ↓
┌─────────────────┐
│ compiler.js     │
│ AST → HTML      │
└─────────────────┘
📗 अगला अध्याय — Chapter 5
AdaptML Parser + AST
हम parser.js बनाएँगे और Lexer से मिले tokens को इस तरह structured AST में बदलेंगे:
Document
└── Page
    ├── Text
    └── Button
फिर यही AST हमारे AdaptML Compiler का आधार बनेगा।

# O que é um `lex` ou `avaliador lexico` ?

`Lexical analyzer` ou `lex` ou `avaliador léxico` é um processo fundamental na arquitetura de qualquer interpretador ou compilador.  Ele é responsável por analisar o `codigo fonte` ou `entrada de texto` iterando sobre cada caractere é gerando `tokens` ou `simbolos` que vão ser importante para os próximos passos de um compilador/interpretador.

##  Como escrever um lex ?

Exemplo: LexEstupido.js
```js
class Token {
  constructor(type, value) {
    this.type = type;
    this.value = value;
  }
}

class Lexer {
  constructor(code) {
    this.code = code;
    this.position = 0;
  }

  isWhitespace(char) {
    return /\s/.test(char);
  }

  isDigit(char) {
    return /\d/.test(char);
  }

  isAlpha(char) {
    return /[a-zA-Z_]/.test(char);
  }

  getNextToken() {
    while (this.position < this.code.length) {
      const char = this.code[this.position];

      if (this.isWhitespace(char)) {
        this.position++;
        continue;
      }

      switch (true) {
        case this.isDigit(char):
          return this.parseNumber();
        case this.isAlpha(char):
          return this.parseIdentifier();
        case /[\+\-\*\/\(\)]/.test(char):
          this.position++;
          return new Token(char, char);
        case char === '=':
          this.position++;
          return new Token('EQUALS', '=');
        default:
          console.error(`Erro léxico: Caractere inesperado '${char}' na posição ${this.position}`);
          return null;
      }
    }

    // Fim do código
    return null;
  }

  parseNumber() {
    let value = '';

    while (this.position < this.code.length && this.isDigit(this.code[this.position])) {
      value += this.code[this.position];
      this.position++;
    }

    return new Token('NUMBER', parseInt(value, 10));
  }

  parseIdentifier() {
    let value = '';

    while (this.position < this.code.length && (this.isAlpha(this.code[this.position]) || this.isDigit(this.code[this.position]))) {
      value += this.code[this.position];
      this.position++;
    }

    return new Token('ID', value);
  }

  tokenize() {
    const tokens = [];

    let token;
    while ((token = this.getNextToken()) !== null) {
      tokens.push(token);
    }

    return tokens;
  }
}

// Exemplo de uso
const codigoFonte = "x = 10 + 20 * (30 - 5)";
const lexer = new Lexer(codigoFonte);
const tokensResultantes = lexer.tokenize();
console.log(tokensResultantes);

```

Primeiramente, criamos um 'Token', uma estrutura que armazena a identidade de cada caractere. É importante observar que um 'Token' pode conter mais atributos do que os exemplificados aqui; meu exemplo é simplificado.

Em seguida, começamos com a classe que recebe o código no construtor. Posteriormente, o método 'tokenize' é implementado como o principal da classe, responsável por percorrer o código fonte. O método 'getNextToken' é encarregado de identificar o caractere atual e atribuí-lo a um 'Token'. Os demais métodos servem apenas como validadores de strings. Ao final, o vetor de 'Tokens' é retornado."
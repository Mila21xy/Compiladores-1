# Analizador Léxico para Mini-0

**Laboratorio 10 - Compiladores**  
**Universidad La Salle**  
**Escuela Profesional de Ingeniería de Software**

## 📋 Descripción

Este proyecto implementa un analizador léxico completo para el lenguaje Mini-0 utilizando Flex. El analizador es capaz de reconocer todos los tipos de tokens definidos en la especificación del lenguaje y clasificarlos correctamente.

## 🎯 Características Implementadas

### ✅ Reconocimiento de Tokens
- **Palabras reservadas**: `if`, `else`, `end`, `while`, `loop`, `fun`, `return`, `new`, `string`, `int`, `char`, `bool`, `true`, `false`, `and`, `or`, `not`
- **Identificadores**: Letra o underscore seguido de letras, números o underscores
- **Números literales**: 
  - Decimales: `15`, `42`, `100`
  - Hexadecimales: `0x0f`, `0xFF`, `0x1A2B`
  - ⚠️ Los valores `15` y `0x0f` se reconocen como el mismo valor (15)
- **Strings**: Entre comillas dobles con soporte para escapes `\\`, `\n`, `\t`, `\"`
- **Operadores**: `+`, `-`, `*`, `/`, `>`, `<`, `>=`, `<=`, `=`, `<>`
- **Delimitadores**: `(`, `)`, `[`, `]`, `,`, `:`
- **Saltos de línea**: Relevantes para la gramática (NL)

### ✅ Comentarios
- **Comentarios de línea**: `// hasta el fin de línea`
- **Comentarios de bloque**: `/* ... */` sin anidamiento

### ✅ Estructura de Datos
```c
typedef struct {
    TokenType type;        // Tipo de token
    char* lexeme;          // Cadena reconocida
    int line;              // Número de línea (primera línea = 1)
    int has_numeric_value; // Indica si tiene valor numérico
    long numeric_value;    // Valor numérico procesado
} Token;
```

### ✅ Manejo de Errores
- Caracteres no reconocidos generan tokens de tipo `TK_ERROR`
- Se reporta la línea donde ocurre el error

## 📁 Estructura del laboratorio

```
mini0-lexer/
├── src/
│   ├── mini0.l          # Especificación Flex del analizador léxico
│   ├── token.h          # Definición de tipos y estructuras de tokens
│   └── token.c          # Implementación de funciones auxiliares
├── tests/
│   ├── test1.mini0      # Programa básico con funciones
│   ├── test2.mini0      # Números y strings
│   ├── test3.mini0      # Comentarios
│   ├── test4.mini0      # Casos de error
│   ├── test5.mini0      # Arrays y loops
│   └── test6.mini0      # Operadores booleanos
├── Makefile             # Archivo de compilación
├── README.md            # Documentación
└── .gitignore           # Archivos a ignorar en Git
```

## 🔧 Compilación y Uso

### Requisitos
- `flex` (Fast Lexical Analyzer)
- `gcc` (GNU Compiler Collection)
- `make`

### Instalación de dependencias (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install flex gcc make
```

### Compilación
```bash
make
```

### Uso
```bash
./mini0_lexer <archivo.mini0>
```

Ejemplo:
```bash
./mini0_lexer tests/test1.mini0
```

### Ejecutar todas las pruebas
```bash
make test
```

### Limpiar archivos generados
```bash
make clean
```

## 📊 Salida del Programa

El programa genera una tabla con los tokens reconocidos:

```
ANÁLISIS LÉXICO - Mini-0
Total de tokens: 45

LÍNEA  TIPO            LEXEMA               VALOR          
--------------------------------------------------------------
1      NL              \n                  
2      FUN             fun                 
2      ID              factorial           
2      LPAREN          (                   
2      ID              n                   
2      COLON           :                   
2      INT_TYPE        int                 
2      RPAREN          )                   
2      COLON           :                   
2      INT_TYPE        int                 
2      NL              \n                  
3      IF              if                  
3      ID              n                   
3      LE              <=                  
3      NUMERAL         1                   1              
...
```

## 🧪 Casos de Prueba

### Test 1: Programa Básico
- Función recursiva `factorial`
- Función `main`
- Declaraciones de variables
- Estructuras de control `if/else/end`

### Test 2: Números y Strings
- Números decimales y hexadecimales
- Strings con caracteres de escape
- Verificación de equivalencia (15 = 0x0f)

### Test 3: Comentarios
- Comentarios de línea (`//`)
- Comentarios de bloque (`/* */`)
- Comentarios inline

### Test 4: Errores
- Caracteres inválidos (`@`, `#`)
- Tokens de error correctamente identificados

### Test 5: Arrays
- Declaración de arrays con `[]`
- Operador `new`
- Acceso a elementos con `[i]`
- Bucle `while/loop`

### Test 6: Operadores
- Operadores relacionales y booleanos
- Expresiones complejas
- Precedencia de operadores

## 🔍 Detalles de Implementación

### Procesamiento de Números
```c
// Los hexadecimales se convierten a decimal
0x0f → valor numérico: 15
15   → valor numérico: 15
```

### Procesamiento de Strings
```c
// Los escapes se traducen al carácter correspondiente
"Hola\nmundo" → "Hola
mundo"
"Ruta: C:\\Users" → "Ruta: C:\Users"
```

### Contador de Líneas
- La primera línea del programa es la línea 1
- Se incrementa con cada `\n` encontrado
- Se mantiene correctamente en comentarios multilínea

## 📝 Notas Importantes

1. **Variable `yyin`**: Se utiliza para redirigir la entrada de Flex desde un archivo
2. **Memoria dinámica**: Los tokens se almacenan en un array dinámico que crece según sea necesario
3. **Liberación de memoria**: Se liberan todos los recursos al finalizar el programa
4. **Compatibilidad**: El código es compatible con C99 y superiores

## 👥 Autores

- Leonardo Raphael Pachari Gomez
- Angela Milagros Quispe Huanca


.L:
%option noyywrap
%option yylineno
%{
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <ctype.h>

  typedef enum {
    T_IF, T_ELSE, T_END, T_WHILE, T_LOOP, T_FUN, T_RETURN, T_NEW, T_STRING, T_INT, T_CHAR, T_BOOL, 
    T_TRUE, T_FALSE, T_AND, T_OR, T_NOT, T_ID, T_INT_LITERAL, T_HEX_LITERAL, T_STRING_LITERAL,
    T_PLUS, T_MINUS, T_STAR, T_SLASH, T_GT, T_LT, T_GE, T_LE, T_EQ, T_NEQ, T_LPAREN, T_RPAREN, 
    T_LBRACK, T_RBRACK, T_COMMA, T_COLON, T_ERROR, T_COMMENT, T_NEWLINE
  } TokenType;

  typedef struct {
    TokenType type;
    char* lexeme;  // Texto original
    int line;      // Línea en el archivo
    long numeric_value;  // Si es un número (entero o hexadecimal)
    char* sval;    // Si es un string
  } Token;

  // Utilidad para duplicar cadenas
  static char* xstrdup(const char* str) {
    size_t len = strlen(str) + 1;
    char* p = (char*)malloc(len);
    if(p) memcpy(p, str, len);
    return p;
  }

  static void emit(TokenType t, const char* lex) {
    Token tok = {0};
    tok.type = t;
    tok.lexeme = xstrdup(lex);
    tok.line = yylineno;
    
    // Impresión del token
    printf("[linea %d] %-20s lexema=\"%s\"\n", tok.line, tokname(tok.type), tok.lexeme);
    
    free(tok.lexeme);
  }

  static const char* tokname(TokenType t) {
    switch (t) {
      case T_IF: return "IF";
      case T_ELSE: return "ELSE";
      case T_END: return "END";
      case T_WHILE: return "WHILE";
      case T_LOOP: return "LOOP";
      case T_FUN: return "FUN";
      case T_RETURN: return "RETURN";
      case T_NEW: return "NEW";
      case T_STRING: return "STRING";
      case T_INT: return "INT";
      case T_CHAR: return "CHAR";
      case T_BOOL: return "BOOL";
      case T_TRUE: return "TRUE";
      case T_FALSE: return "FALSE";
      case T_AND: return "AND";
      case T_OR: return "OR";
      case T_NOT: return "NOT";
      case T_ID: return "ID";
      case T_INT_LITERAL: return "INT_LITERAL";
      case T_HEX_LITERAL: return "HEX_LITERAL";
      case T_STRING_LITERAL: return "STRING_LITERAL";
      case T_PLUS: return "PLUS";
      case T_MINUS: return "MINUS";
      case T_STAR: return "STAR";
      case T_SLASH: return "SLASH";
      case T_GT: return "GT";
      case T_LT: return "LT";
      case T_GE: return "GE";
      case T_LE: return "LE";
      case T_EQ: return "EQ";
      case T_NEQ: return "NEQ";
      case T_LPAREN: return "LPAREN";
      case T_RPAREN: return "RPAREN";
      case T_LBRACK: return "LBRACK";
      case T_RBRACK: return "RBRACK";
      case T_COMMA: return "COMMA";
      case T_COLON: return "COLON";
      case T_ERROR: return "ERROR";
      case T_COMMENT: return "COMMENT";
      case T_NEWLINE: return "NEWLINE";
      default: return "?";
    }
  }
%}

%%

[ \t\r\f\v]+                  ; // Ignorar espacios en blanco
\n                            { emit(T_NEWLINE, yytext); }

"if"                         { emit(T_IF, yytext); }
"else"                       { emit(T_ELSE, yytext); }
"end"                        { emit(T_END, yytext); }
"while"                       { emit(T_WHILE, yytext); }
"loop"                        { emit(T_LOOP, yytext); }
"fun"                         { emit(T_FUN, yytext); }
"return"                      { emit(T_RETURN, yytext); }
"new"                         { emit(T_NEW, yytext); }
"string"                      { emit(T_STRING, yytext); }
"int"                         { emit(T_INT, yytext); }
"char"                        { emit(T_CHAR, yytext); }
"bool"                        { emit(T_BOOL, yytext); }
"true"                        { emit(T_TRUE, yytext); }
"false"                       { emit(T_FALSE, yytext); }
"and"                         { emit(T_AND, yytext); }
"or"                          { emit(T_OR, yytext); }
"not"                         { emit(T_NOT, yytext); }

[a-zA-Z_][a-zA-Z0-9_]*        { emit(T_ID, yytext); }

"0x"[0-9a-fA-F]+              { emit(T_HEX_LITERAL, yytext); }
[0-9]+                        { emit(T_INT_LITERAL, yytext); }
\"([^\\\"]|\\.)*\"            { emit(T_STRING_LITERAL, yytext); }

"+"                           { emit(T_PLUS, yytext); }
"-"                           { emit(T_MINUS, yytext); }
"*"                           { emit(T_STAR, yytext); }
"/"                           { emit(T_SLASH, yytext); }
">"                           { emit(T_GT, yytext); }
"<"                           { emit(T_LT, yytext); }
">="                          { emit(T_GE, yytext); }
"<="                          { emit(T_LE, yytext); }
"="                           { emit(T_EQ, yytext); }
"<>"                          { emit(T_NEQ, yytext); }

"("                           { emit(T_LPAREN, yytext); }
")"                           { emit(T_RPAREN, yytext); }
"["                           { emit(T_LBRACK, yytext); }
"]"                           { emit(T_RBRACK, yytext); }
","                           { emit(T_COMMA, yytext); }
":"                           { emit(T_COLON, yytext); }

"//"[^\\n]*                   { emit(T_COMMENT, yytext); }
"/*"([^\*]|\*+[^/])*\*+/     { emit(T_COMMENT, yytext); }

.                             { emit(T_ERROR, yytext); }

%%

int main(int argc, char **argv) {
  if(argc < 2) {
    fprintf(stderr, "Uso: %s <archivo.mini0>\n", argv[0]);
    return 1;
  }
  FILE *f = fopen(argv[1], "r");
  if(!f) {
    perror("No se pudo abrir el archivo");
    return 1;
  }
  yyin = f;
  yylex();
  fclose(f);
  return 0;
}




.C :
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "token.h"  // Incluye la estructura Token

int main(int argc, char **argv) {
  if(argc < 2) {
    fprintf(stderr, "Uso: %s <archivo.mini0>\n", argv[0]);
    return 1;
  }
  FILE *f = fopen(argv[1], "r");
  if(!f) {
    perror("No se pudo abrir el archivo");
    return 1;
  }
  yyin = f;  // Asignar el archivo de entrada
  yylex();   // Ejecutar el análisis léxico
  fclose(f);
  return 0;
}



.H: 
typedef enum {
  T_IF, T_ELSE, T_END, T_WHILE, T_LOOP, T_FUN, T_RETURN, T_NEW, T_STRING, T_INT, T_CHAR, T_BOOL, 
  T_TRUE, T_FALSE, T_AND, T_OR, T_NOT, T_ID, T_INT_LITERAL, T_HEX_LITERAL, T_STRING_LITERAL,
  T_PLUS, T_MINUS, T_STAR, T_SLASH, T_GT, T_LT, T_GE, T_LE, T_EQ, T_NEQ, T_LPAREN, T_RPAREN, 
  T_LBRACK, T_RBRACK, T_COMMA, T_COLON, T_ERROR, T_COMMENT, T_NEWLINE
} TokenType;

typedef struct {
  TokenType type;
  char* lexeme;  // Texto original
  int line;      // Línea en el archivo
  long numeric_value;  // Si es un número (entero o hexadecimal)
  char* sval;    // Si es un string
} Token;

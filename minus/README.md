## Simple interpreter

### ECMA 6 version:

``` 
➜  minus git:(master) npm run test-mjs

> hello-jison@1.0.0 test-mjs
> npm run build; ./minus/use_minus.mjs ./minus/input.minus


> hello-jison@1.0.0 build
> jison minus/minus.jison minus/minus.l -o minus/minus.js

[ 3, 2, 96, 4 ]
```

### CommonJS version:

```
➜  minus git:(master) ✗ npm run test-simple

> hello-jison@1.0.0 test-simple
> npm run build; ./minus/use_minus.js


> hello-jison@1.0.0 build
> jison minus/minus.jison minus/minus.l -o minus/minus.js

[
  0
]
```

## Location: @1, @2, etc.

See https://www.gnu.org/software/bison/manual/html_node/Actions-and-Locations.html#Actions-and-Locations

```
➜  minus git:(20250205) ✗ cat minus-location.jison 
%right '-'
%%
e: 
    e '-' e  { console.log(`e -> e - e ${JSON.stringify(@2)}`) }
  | N        { console.log(`e ->N ${JSON.stringify(@1)}`) }
;
```                                                                                                                                                                                                                                

```      
➜  minus git:(20250205) ✗ npx jison minus-location.jison minus.l -o minus.js
➜  minus git:(20250205) ✗ ./use_minus.js
output for 2-1-1:
e ->N {"first_line":1,"last_line":1,"first_column":0,"last_column":1}
e ->N {"first_line":1,"last_line":1,"first_column":2,"last_column":3}
e ->N {"first_line":1,"last_line":1,"first_column":4,"last_column":5}
e -> e - e {"first_line":1,"last_line":1,"first_column":3,"last_column":4}
e -> e - e {"first_line":1,"last_line":1,"first_column":1,"last_column":2}
true
```

## .parser.yy

The parser has a `yy` property which is exposed to actions as the `yy` free variable. 
Any functionality attached to this property is available in both lexical 
and semantic actions through the `yy` free variable.

```
➜  minus git:(20250205) cat minus-yy.jison 
%right '-'
%%
e: 
    e '-' e  { console.log(`e -> e - e ${yy.myval}`)}
  | N        {console.log(`e ->N ${yy.myval}`)}
;
```

``` 
➜  minus git:(20250205) cat minus.l
%%
\s     /* skip whites */;
\d+([.]\d+)?  return 'N';
'-'           return '-';
','           return ',';
  /* <<EOF>>       return 'EOF';*/
➜  minus git:(20250205) ✗ npx jison minus-yy.jison minus.l -o minus.js
```

```
➜  minus git:(20250205) ✗ node
Welcome to Node.js v23.5.0.
Type ".help" for more information.
> minus = require("./minus.js")
{
  parser: { yy: {} },
  Parser: [Function: Parser],
  parse: [Function (anonymous)],
  main: [Function: commonjsMain]
}
> minus.parser.yy.myval = 99
99
> minus.parse('3-2-1')
e ->N 99
e ->N 99
e ->N 99
e -> e - e 99
e -> e - e 99
true
```

## .main

The property `main` is a function that can be used to run the parser. Receives an array of two elements. 
The second element is the name of the file to be parsed.The first is never used.

For the same code above:

```
➜  minus git:(20250205) ✗ node                               
> minus = require("./minus.js")
{
  parser: { yy: {} },
  Parser: [Function: Parser],
  parse: [Function (anonymous)],
  main: [Function: commonjsMain]
}
> minus.parser.yy.myval = "hi!"
'hi!'
> minus.main([true,'input.calc'])
e ->N hi!
e ->N hi!
e ->N hi!
e -> e - e hi!
e -> e - e hi!
```

## .Parser

The `Parser` property is a constructor that can be used to create a new parser object.

```js
// ...
function Parser() {
        this.yy = {};
}
Parser.prototype = parser; 
parser.Parser = Parser;
```

Here is an example with the same [code minus-yy.jison](minus-yy.jison) above:

```js
minus git:(20250205) ✗ node 
Welcome to Node.js v23.5.0.
Type ".help" for more information.
> minus = require("./minus.js")
{
  parser: { yy: {} },
  Parser: [Function: Parser],
  parse: [Function (anonymous)],
  main: [Function: commonjsMain]
}
> q = new minus.Parser()
{ yy: {} }
> q.yy.myval = 4
4
> q.parse("3-1")
e ->N 4
e ->N 4
e -> e - e 4
true
```
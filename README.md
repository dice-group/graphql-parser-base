# SPARQL-parser-base

This repository generates a [ANTLR-v4-based](https://github.com/antlr/antlr4) GraphQL parser in C++. The ANTLR v4 code generator is called by CMake.

## requirements

- C++20 compatible compiler
- only tested on linux, x86_64
- see [Dockerfile](Dockerfile) for details 

## build it

```shell
#get it 
git clone https://github.com/dice-group/sparql-parser-base.git
cd graphql-base-parser
#build it
mkdir build
cd build
cmake  -DCMAKE_BUILD_TYPE=Release ..
make -j graphql-parser-base
```

There are three project-specific options you can set for CMake:

- `GRAPHQL_BASE_PARSER_MARCH`: Allows you to set the -march parameter. If you are building for your local machine, you should set it to `-DSPARQL_BASE_PARSER_MARCH=native`

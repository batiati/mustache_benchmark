# Mustache template engine, rendering in different languages

Render speed results in milliseconds, for 3 different templates, number of runs 1M.

### No template precompilation

<table>

<tr>
<th>Data</th>
<th>Erlang/bbmustache</th>
<th>Erlang/fast</th>
<th>Go</th>
<th>Scala</th>
<th>OCaml</th>
</tr>

<tr>
<td>Template 1</td>
<td>23,971</td>
<td>33,649</td>
<td>35,620</td>
<td>80,622</td>
<td>19,541</td>
</tr>

<tr>
<td>Template 2</td>
<td>16,471</td>
<td>8,330</td>
<td>11,600</td>
<td>17,357</td>
<td>5,865</td>
</tr>

<tr>
<td>Template 3</td>
<td>31,246</td>
<td>12,442</td>
<td>16,188</td>
<td>33,220</td>
<td>9,034</td>
</tr>

</table>


### From precompiled templates

<table>

<tr>
<th>Data</th>
<th>Erlang/bbmustache</th>
<th>Erlang/erlydtl</th>
<th>Go</th>
<th>Scala/scalate</th>
<th>Scala/mustache</th>
<th>OCaml</th>
</tr>

<tr>
<td>Template 1</td>
<td>1,814</td>
<td>29,495</td>
<td>19,406</td>
<td>11,156</td>
<td>5,646</td>
<td>1,007</td>
</tr>

<tr>
<td>Template 2</td>
<td>2,525</td>
<td>7,293</td>
<td>7,545</td>
<td>6,250</td>
<td>3,482</td>
<td>1,452</td>
</tr>

<tr>
<td>Template 3</td>
<td>2,265</td>
<td>x</td>
<td>9,854</td>
<td>4,947</td>
<td>3,965</td>
<td>2,269</td>
</tr>

</table>


## Templates

[template 1](data/template1.html) -- 65 lines, 5020 bytes, variables only

[template 2](data/template2.html) -- 11 lines, 383 bytes, variables only

[template 3](data/template3.html) -- 20 lines, 207 bytes, with if, loop etc constructs


## Erlang

The first *Erlang/bbmustache* implementation uses the bbmustache library https://github.com/soranoba/bbmustache

- Erlang/OTP 18 erts-7.1
-bbmustache, "1.0.4"

[mustache_bm.erl](erl_bbmustache/src/mustache_bm.erl)

The second *Erlang/fast* implementation uses a simple function based on binary:split which can replace variables in a template.

[mustache_bm2.erl](erl_fast/src/mustache_bm2.erl)

bbmustache precompiled:
[mustache_pc_bm.erl](erl_bbmustache/src/mustache_pc_bm.erl)

It is important for bbmustache to use the _{key\_type, string}_ option (or no options, this will be the default). _{key\_type, binary}_ slows down rendering, significantly in some cases.

And to compare ErlyDTL https://github.com/erlydtl/erlydtl with precompilation:
[mustache_bm3.erl](erl_erlydtl/src/mustache_bm3.erl)
(For the 3rd template, the test was not done, because it is incompatible with the DTL).

## Go

go1.3.3 linux/amd64

https://github.com/cbroglie/mustache (In Go tradition -- unknown version.
Commit 6857e4b493bdb8d4b1931446eb41704aeb4c28cb at the time of the test)

Tests, of course, on the compiled binary, not in the interpreter.

[mustache_bm.go](go_mustache/src/mbm/mustache_bm.go)

Tried 3 more libs:
- https://github.com/aymerick/raymond -- this one is very slow
- https://github.com/ChrisBuchholz/gostache -- does not work with maps, requires specific bindings
- https://github.com/alexkappa/mustache -- very slow


## Scala

First I took the Scalate library

- scala 2.10.6
- "org.fusesource.scalate" %% "scalate-core" % "1.6.1"
- https://github.com/scale/scale

The option without precompiling the template [MustacheBM.scala](scala_scalate/src/main/scala/MustacheBM.scala) is terribly slow:
- 10 renders -- 3970ms
- 100 renders -- 19254ms

But with precompilation [MustachePC_BM.scala](scala_scalate/src/main/scala/MustachePC_BM.scala) shows good results.

Tried another library: https://github.com/vspy/scala-mustache

[MustacheBM.scala](scala_mustache/src/main/scala/MustacheBM.scala)
[MustachePC_BM.scala](scala_mustache/src/main/scala/MustachePC_BM.scala)

The results are comparable to other languages.


##OCaml

- Ocaml 4.02.1
- ezjsonm 0.4.1
- mustache 1.0.1 https://github.com/rgrinberg/ocaml-mustache

Tests on a binary compiled into native code.

[mustache_bm.ml](ocaml_mustache/src/mustache_bm.ml)
[mustache_pc_bm.ml](ocaml_mustache/src/mustache_pc_bm.ml)


The library generates an error if the binding does not contain a value for a variable in the template.
Maybe it's configurable.


## Conclusions

OCaml is the fastest.

Erlang/bbmustache with precompilation works just as well as OCaml. Without precompilation, it's worse, but not bad compared to other languages.
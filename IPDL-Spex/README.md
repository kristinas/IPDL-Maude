# SpeX

SpeX is a rewriting-based environment that facilitates the experimental development of formal-specification languages and tools.
It is language agnostic, so it's not geared towards any particular syntax or semantics; instead, it provides libraries that support the continuous integration of parsers and information processors – one for each language.
Currently, it brings together structured specifications over propositional, equational, or first-order logic, hidden algebra, or dynamic networks of interactions, among others.

## Obtaining SpeX

SpeX is implemented in [Maude 3](http://maude.cs.illinois.edu/w/index.php/The_Maude_System).
The latest version of the environment is available [here](https://gitlab.com/ittutu/spex/-/raw/main/dist/spex-23.12.tar.gz).
On GNU/Linux machines, the simplest way to install it is by running the following scripts from the directory containing the SpeX source tree:

```shell
./configure
make
sudo make install
```

## Using SpeX

The SpeX interpreter can be launched from the command line by typing `spex`, or it can be explicitly loaded into Maude by executing `maude -no-banner -allow-files run-SpeX.maude`.

The utility of any instance of SpeX is given by the processors and corresponding languages it hosts.
Besides such processors, SpeX supports the following four core commands:

* `load file` reads the contents of a given file;
* `eof` stops reading input from the current input stream; if that stream is the standard input (not a file accessed via `load`), then it also ends the execution of the interpreter;
* `quit` ends the execution of the interpreter;
* `lang l` selects one of the languages defined in the SpeX knowledge base, thus allowing subsequent input to be processed according to the definition of that language.

To illustrate the use of the processor `Th[PL]`, which handles theory presentations over propositional logic, consider the following input where standard axioms of propositional calculus are declared:

```
lang Th[PL]

th T is
  props p, q, r .
  ax p implies (q implies p) .
  ax (p implies (q implies r)) implies ((p implies q) implies (p implies r)) .
  ax (not p implies not q) implies (q implies p) .
endth
```

A much more complex processor and language, based on hidden algebra, is `COMP`.
You can find out more about it at the [`COMP` homepage](http://www.imar.ro/~diacon/COMPproject/COMP.html).

## License

The SpeX source code is licensed under the [GNU General Public License v2.0 or later](https://www.gnu.org/licenses/old-licenses/lgpl-2.0.html).

This repository is a fork from [`tremps/Infosec-Coding-Project`](https://github.com/tremps/Infosec-Coding-Project).
This fork is a modification of the original repository, so that it can be used as an in-class exercise.


# Trusting Trust Project

This project introduces a bug into the [TinyCC](https://github.com/TinyCC/tinycc) compiler.
The bug is introduced by a variation of Ken Thompson's "Trusting Trust" attack.

The source for the TinyCC compiler can be found at https://github.com/TinyCC/tinycc and is licensed under the GNU Lesser General Public License.


## Requirements

Before running the steps below, it is expected that you have:

1. Cloned the sources from the [official TinyCC repository](https://repo.or.cz/w/tinycc.git): <git://repo.or.cz/tinycc.git>
2. Built a version of this tiny C compiler on a local Linux host according to the description in the respective [README file](https://repo.or.cz/tinycc.git/blob_plain/HEAD:/README) in the root of the repository.
3. For the reminder of this document, it is assumed that you installed that version of TinyCC to `$HOME/tcc/`.
   If you installed it somewhere else, you have to replace all occurrences of `$HOME/tcc/bin` with your installation path.
4. To execute this project, the `configure` script in the TinyCC source tree shipped in this repository has to be executable.
   Usually, a `git clone` does not preserve these flags, so make the script executable again:

  ```bash
   chmod u+x configure
  ```

5. Lastly, it is assumed that you cloned this repository and changed your working directory to the root of it.


## Testing our project

1. You have met the requirements above.
2. Compile `test.c` with the TinyCC compiler that you have setup on your host (see section above).

  ```bash
  PATH="$HOME/tcc/bin:$PATH" tcc -o test.out test.c
  ```

3. Experiment with running the program `test.out` given various passwords as argument

  ```bash
  ./test.out <password>
  ```

4. Note, that the program fails, unless `password123` is provided as argument.


The previous steps illustrate that the example program works as intended when compiled with a trusted compiler.
Now, let's experiment with a trusting trust attack.
You will first build a trusted TinyCC compiler from the sources that are shipped in this repository under [./tinycc](./tinycc).
Thereafter, we will introduce a bug into this compiler, rebuild

1. To start, let's create some directories for this experiment:

  ```bash
  mkdir $HOME/tcc_$(head -1 tinycc/VERSION)/
  mkdir $HOME/tcc_$(head -1 tinycc/VERSION)_bugged/
  ```
2. Make a "clean" version of TinyCC from the sources in this repository.
   For technical reasons, we can only build it with the TinyCC compiler, that was installed previously in the requirements.
   Consequently, point the respective program to the directory containing the respective `tcc` binary (see modifications of `PATH` variable in the commands below).

  ```bash
  cd tinycc
  make clean
  PATH="$HOME/tcc/bin:$PATH" ./configure --prefix=$HOME/tcc_$(head -1 VERSION)
  PATH="$HOME/tcc/bin:$PATH" make
  make install
  cd -
  ```
  - As an intermediate exercise, see that the test program behaves as expected when compiled with this version of the TinyCC compiler.
  - First, compile a new version of the test program:
  ```bash
  PATH="$HOME/tcc_$(head -1 tinycc/VERSION)/bin:$PATH" tcc -o test.out test.c
  ```
  - Then, realize that the program still only succeeds with `password123` is provided as argument.
3. Now, we will create a compromised "bugged" version of the TinyCC compiler.
   We will build that compiler with the compiler from the previous step (2).
  - First, copy `libtcc_bugged.c` into the `tinycc` directory and replace `libtcc.c` with it.
    ```bash
    cp libtcc_bugged.c tinycc/libtcc.c
    ```
  - Within the `tinycc` directory, make and install compromised "bugged" compiler

    ```bash
    cd tinycc
    make clean
    PATH="$HOME/tcc_$(head -1 VERSION)/bin:$PATH" ./configure --prefix=$HOME/tcc_$(head -1 VERSION)_bugged
    PATH="$HOME/tcc_$(head -1 VERSION)/bin:$PATH" make
    make install
    cd -
    ```
  - As an intermediate exercise, see how the behavior of the test program changes, when compiled with the compromised version of the TinyCC compiler.
  - First, compile a new version of the test program:
  ```bash
  PATH="$HOME/tcc_$(head -1 tinycc/VERSION)_bugged/bin:$PATH" tcc -o test.out test.c
  ```
  - Then, realize that the program `test.out` succeeds no matter which password is provided as argument.
4. To complete the attack, we have to "clean up" the sources of the compromised compiler.
   We do that by checking out the original version of `tinycc/libtcc.c`.

  ```bash
  git checkout -- tinycc/libtcc.c
  ```
5. Now with the compromised compiler, we can rebuild the "clean" sources.
   The compromised compiler will always reintroduce the malicious change into newly compiled versions.

  ```bash
  cd tinycc
  make clean
  PATH="$HOME/tcc_$(head -1 VERSION)_bugged/bin:$PATH" ./configure --prefix=$HOME/tcc_$(head -1 VERSION)_bugged
  PATH="$HOME/tcc_$(head -1 VERSION)_bugged/bin:$PATH" make
  make install
  cd -
  ```
  - First, compile a new version of the test program with a compromised compiler, which in turn was build with a compromised compiler from clean compiler sources:
  ```bash
  PATH="$HOME/tcc_$(head -1 tinycc/VERSION)_bugged/bin:$PATH" tcc -o test.out test.c
  ```
  - Now, verify that the program still `test.out` succeeds no matter which password is provided as argument, even though neither its sources nor the compiler sources contain respective instructions.


<!-- TODO: Continue here! -->

```bash
$ sha256sum ~/tcc_*/bin/tcc
b56a9c59cb821eb9f0131d3de23f9ea862d9f76787ffb31c5c1ca0560c713efb  /home/user/tcc_0.9.27/bin/tcc
0f77e205500a41bfc176036a62d3adbddc39380f203d0ede281295df297e2412  /home/user/tcc_0.9.27_bugged/bin/tcc
```





7. Now, replace libtcc.c with its original, unmodified source code provided in the base directory of the project



8. Within the tinycc directory, make and install tcc as many time as you would like
```
make clean
./configure
make
make test
make install
```
9. You can now repeat step 2, you should notice that every string entered is accepted




### Note

The `configure` and `Makefile` within the tinycc directory we provide are modified to compile with tcc.
This is vital for the code to work as intended.
The first time compiling our code with the bugged version, you create a bugged version of the `tcc` binary that will target compilation of `test.c` files and that
will target libtcc.c to reintroduce itself into all preceding versions of the compiler.
This bugged version is only designed to compile the original tcc source code.
Do not try compiling the bugged version of libtcc.c with a bugged version of the compiler. Step 8 can
be repeated as many times as you want because the bugged version is continuously reintroducing itself.
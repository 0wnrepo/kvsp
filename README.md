# KVSP; Kyoto Virtual Secure Platform

KVSP is the first virtual secure platform in the world,
which makes your life better.

On VSP you can run your encrypted code as is.
No need to decrypt while running. See [here](https://anqou.net/poc/2019/10/18/post-3106/)
for more details (in Japanese).

KVSP consists of many other sub-projects.
`kvsp` command, which this repository serves, is
a simple interface to use them easily.

## Quick Start

Download a KVSP release and unzip it.
(It has been compiled on Ubuntu 18.04 LTS. If it doesn't work in the following steps,
please read __Build__ section and try to build KVSP on your own.
It may be time-consuming, but not so hard.)

```
$ wget 'https://github.com/virtualsecureplatform/kvsp/releases/latest/download/kvsp.tar.gz'
$ tar xf kvsp.tar.gz
$ cd kvsp_v10/bin    # The directory's name depends on the file you download.
```

Write some C code...

```
$ vim fib.c

$ cat fib.c
static int fib(int n) {
  int a = 0, b = 1;
  for (int i = 0; i < n; i++) {
    int tmp = a + b;
    a = b;
    b = tmp;
  }
  return a;
}

int main() {
  // The result will be set in the register x8.
  return fib(5);
}
```

...like so. This program (`fib.c`) computes the 5th term of the Fibonacci sequence, that is, 5.

Compile the C code (`fib.c`) to an executable file (`fib`).

```
$ ./kvsp cc fib.c -o fib
```

Let's check if the program is correct by emulator, which runs
without encryption.

```
$ ./kvsp emu fib
#1      done. (1759 us)
#2      done. (1365 us)
#3      done. (1390 us)
#4      done. (1417 us)
#5      done. (1423 us)
#6      done. (1407 us)
#7      done. (1421 us)
#8      done. (1449 us)
#9      done. (1416 us)
break.
#cycle  9

...

x8  5

...
```

We can see `x8  5` here, so the program above seems correct.
Also we now know it takes 9 clocks by `#cycle  9`.

Now we will run the same program with encryption.

Generate a secret key (`secret.key`).

```
$ ./kvsp genkey -o secret.key
```

Encrypt `fib` with `secret.key` to get an encrypted executable file (`fib.enc`).

```
$ ./kvsp enc -k secret.key -i fib -o fib.enc
```

Run `fib.enc` for 9 clocks to get an encrypted result (`result.enc`).
Notice that we DON'T need the secret key (`secret.key`) here,
which means the encrypted program (`fib.enc`) runs without decryption!

```
$ ./kvsp run -i fib.enc -o result.enc -c 9 ## Use option `-g num_of_gpus` if you have GPUs.
#1      done. (15333972 us)
#2      done. (23033477 us)
#3      done. (26517776 us)
#4      done. (26593093 us)
#5      done. (26475888 us)
#6      done. (26555916 us)
#7      done. (26632725 us)
#8      done. (26579704 us)
#9      done. (26549935 us)
```

Decrypt `result.enc` with `secret.key` to print the result.

```
$ ./kvsp dec -k secret.key -i result.enc
...

x8  5

...
```

We could get the correct answer using secure computation!

## More examples?

See the directory `examples/`.

## System Requirements

We ensure that KVSP works on the following cloud services:

- [さくらインターネット 高火力コンピューティング Tesla V100（32GB）モデル](https://www.sakura.ad.jp/koukaryoku/)
- [GCP n1-standard-96 with 8 x NVIDIA Tesla V100](https://cloud.google.com/compute/docs/machine-types?hl=ja#n1_standard_machine_types)
- [AWS EC2 m5.metal](https://aws.amazon.com/ec2/instance-types/c5/)

If you run KVSP locally, prepare a machine with the following devices:

- Intel CPU with AVX2 support (e.g. Intel Core i7-8700)
- 8GB RAM
- NVIDIA GPU (not required but highly recommended)
    - Only NVIDIA Tesla V100 is supported.
    - Other GPUs _may_ work but are not supported.

## Build

Clone this repository:

```
$ git clone https://github.com/virtualsecureplatform/kvsp.git
```

Clone submodules recursively:

```
$ git submodule update --init --recursive
```

Build KVSP:

```
$ make  # It may take a while.
```

Use option `ENABLE_CUDA` if you build KVSP with GPU support:

```
$ make ENABLE_CUDA=1 CUDACXX="/usr/local/cuda/bin/nvcc" CUDAHOSTCXX="/usr/bin/clang-8"
```

## Build KVSP Using Docker

Based on Ubuntu 18.04 LTS image.

```
# docker build -t kvsp-build .
# docker run -it -v $PWD:/build -w /build kvsp-build:latest
```

## Detailed Explanation on KVSP

Kyoto Virtual Secure Platform (KVSP) enables to execute a standard C program while protecting its code on cloud computing platforms.

The main idea of KVSP is emulating a CPU on TFHE, which is one kind of fully homomorphic encryption. By using TFHE we can perform logical operations on ciphertexts.

KVSP runs at 4 second per clock on NVIDIA V100, 2.5s on AWS's c5.metal and 1.5s on 8 V100.

KVSP consists of 6 sub-projects:

- Iyokan: A parallel execution engine for logic gates using homomorphic encryption.
- CAHPv3: Our original ISA for the processor which is emulated in KVSP. Most significant features are 16bit data width and 16/24bit variable instruction width.
- cahp-diamond: Our original CPU which implements CAHPv3. It consists of about 4K logic gates.
- llvm-cahp: LLVM backend for CAHPv3.
- TFHEpp & cuFHE: C++ libraries of TFHE. TFHEpp is a full-scratched library for CPU and cuFHE is modified version of an existing implementation for GPU. They make KVSP faster from a cryptographical point of view.
- kvsp: KVSP's CUI interface.

This is a research project. We are planning to publish a paper.

## Code Owners

![Code Owners](code-owners.png)

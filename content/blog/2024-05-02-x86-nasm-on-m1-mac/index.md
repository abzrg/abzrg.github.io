+++
title = "x86 NASM on M1 Mac"
+++

We need an x86 Linux Docker image, and I prefer using the [Ubuntu 22.04 Jammy](https://releases.ubuntu.com/jammy/) release for this purpose.

```sh
docker pull --platform linux/amd64 ubuntu:jammy
```

Now, we can run it,

```sh
# We didn't pass '--rm' to retain it after exiting the Docker terminal/shell. Instead, we assigned it a name ('--name x86asm') for easy attachment later.
docker run -it --name x86asm --platform linux/amd64 ubuntu:jammy

# To attach
docker start --interactive --attach x86asm

# To delete this container
docker container rm x86asm
```

Next, we need to install some packages,

```sh
apt-get update
apt install -y binutils vim-tiny nasm
```

Next, we are ready to write assembly code and build it. Let's write it to a file using `vim-tiny`: `vi hello.asm`,

```asm
; hello.asm

section .data
    hello_msg db "Hello, World!", 10

section .text
    global _start

_start:
    mov rax, 1
    mov rdi, 1
    mov rsi, hello_msg
    mov rdx, 14
    syscall

    mov rax, 60
    xor rdi, rdi
    syscall
```

Finally, compile, link and run,

```sh
# Compile
$ nasm -f elf64 hello.asm -o hello.o

# Link
$ ld hello.o -o hello

# Run
$ ./hello
Hello, World!
```

+++
title = "How to Practice CUDA Locally Without an NVIDIA GPU"
description = "Are you too poor to buy NVIDIA GPUs? and are you still living in CLI?"
draft = false
date = "2026-07-14"

[taxonomies]
tags = ["hpc", "cuda", "cli"]

[extra]
math = false
+++


If you're learning CUDA without access to an NVIDIA GPU, you're no longer blocked.
Compiler Explorer can compile and even execute CUDA programs on remote hardware.
Just head over to [godbolt.org](https://godbolt.org), choose a CUDA compiler, and you're ready to experiment.

This post, however, aims to set up a local CLI development environment.
With it, you'll be able to code in the _comfort_ of your favorite terminal and editor.
{% marginnote(id="mn-0", offset="-1.5em") %}
The only catch is that there's a few milliseconds of delay between when you run the command and get your outputs, due to the network latency.
{% end %}

This setup is good for practicing CUDA syntax, checking that kernels and memory management compile correctly, and confirming small programs behave as expected, all without installing the CUDA toolkit or owning NVIDIA hardware.
It's _not_ a good fit for serious performance benchmarking, since you're sharing remote hardware and timing won't be reliable, or for large, stateful projects, since each run starts _fresh_.

To set this up locally, we'll use a small CLI that talks to Compiler Explorer and, if needed, a tool to flatten{% sidenote(id="") %} Since Compiler Explorer receives a single source file, projects that rely on local headers need to be flattened first.{% end %} multi-file projects into a single source file.
I picked [`cexpl`](https://github.com/xfgusta/cexpl) and [`quom`](https://github.com/Viatorus/quom), respectively

Set up a virtual environment and install `cexpl`. If your project spans multiple files, install `quom` as well.

```console
$ python -m venv venv
$ source venv/bin/activate   # on Windows: venv\Scripts\activate
$ pip install cexpl quom
```

Next, write your CUDA code in a file, say `main.cu`.

```cpp, hl_lines=9 10
#include <cstdio>
#include <cuda_runtime.h>

__global__ void hello() {
    printf("Hello from thread %d\n", threadIdx.x);
}

int main() {
    hello<<<1, 8>>>();
    cudaDeviceSynchronize();
    return 0;
}
```

<p style="margin-bottom: 0;">
{% marginnote(id="", offset="-6.5em") %}
This kernel just prints a message from 8 GPU threads.
Also, in practice you should check the return value of <code>cudaDeviceSynchronize</code>.
{% end %}
</p>

<hr style="border: none; margin: 0;">


> [!NOTE]
> If your code includes local header files, make a combined file using `quom`.
>
> ```console
> $ quom main.cu combined.cu
> ```

Finally, compile and run it _remotely_ with

```console
$ cexpl --exec --skip-asm --lang cuda main.cu
```

<span></span>
{% marginnote(id="mn-1", offset="-3em") %}
<code>--exec</code> runs the compiled program, not just compile it;
<code>--skip-asm</code> hides the generated assembly output, so you only see program results;
<code>--lang cuda</code> tells Compiler Explorer to treat the file as CUDA code.
{% end %}

After a short delay, you should see something like:

```console
STDOUT:
Hello from thread 0
Hello from thread 1
Hello from thread 2
Hello from thread 3
Hello from thread 4
Hello from thread 5
Hello from thread 6
Hello from thread 7
```

That output was produced by an NVIDIA GPU running on Compiler Explorer's servers.


> [!IMPORTANT]
> Keep in mind that the execution happens <em>in a fresh environment every run</em>.


> [!NOTE]
> `--exec` can sometimes crash instead of showing errors. If your code has a compilation error, `cexpl --exec` should normally print the compiler errors to stderr and stop there, but depending on your system or network, it sometimes crashes outright instead of showing you anything useful. If that happens, just drop `--exec` and run:
>
>   ```console
>   $ cexpl --skip-asm --lang cuda main.cu
>   ```
>
>   This only compiles the code and prints the compiler output, including any errors, without trying to execute it. Once your code compiles cleanly, add `--exec` back to actually run it.

---

That's all there is to it.
With that, you can practice and learn CUDA a bit more easily.

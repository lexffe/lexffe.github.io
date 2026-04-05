+++
title = "Tech I appreciate from the software engineering world for the past n years."
description = "I attempt to appreciate things in software engineering"
date = 2026-03-23
+++

Over time I see more and more negativity in the software engineering and computer science community. I myself become opinionated and polarised as well. But I realised that this is not good for anyone, especially in the mental health department.

For this post, I want to list out things that bring me joy when using it. It is not just things that I have used before but things I understand and see the potential.

## Portable SIMD in Rust

I think mechanical sympathy is very underrated. We spend so much time wielding code and sometimes we leave so much performance on the table. My ideal world is that the compiler is smart enough to do autovectorisation. Couple that with instruction level parallelism and out of order execution and your application would be flying. In general, if your code is laid out obvious enough, the compiler does tend to recognise patterns and autovectorise your code. But sometimes your code is just complicated enough, with enough for loops and if branches, that the compiler can’t figure out. Or sometimes you figured out how to calculate all the branches at once.

Recently at $work I have had the privilege and luxury to write high performance code. It is compute heavy and benefits heavily from vectorisation. Unfortunately I made the sin of optimising before measurement. The original code was more or less scalar but I did not check the emitted assembly nor did I do any profiling. So there was no way to tell if autovectorisation ever worked. That said, moving from scalar code to portable SIMD did yield a substantial boost. I did not use external libraries and used `std::simd`.

Things I appreciate: 

- I have never really written SIMD code before so I pair cargo asm with Claude. This creates a nice feedback loop where you can guide optimisation based on the assembly.  
- The syntax and abstraction works. I look at `u32x4` and I understand what it means. I don’t need to remember mnemonics.  
- You can mix and match portable SIMD code with unsafe platform intrinsics. So if you see a codegen behaviour that you don’t think is efficient, then you can use code blocks guarded with platform detection and write with intrinsics.

Things it could improve on:

- The codegen is generally really good, but kind of falls short on ARM targets. AVX codegen works better than NEON, specifically when it comes to swizzling. It is possible that it’s specific to my code. In the end, I prompted Claude to use the portable SIMD code to generate a NEON specific code block.  
- Portable SIMD is still in nightly...
- If I build std during build-time as well (`-Zbuild-std`), it yields better assembly. I did not spend much time investigating this behaviour.

(I could have picked C/C++ but I have not had good experience writing those before. Dealing with portable SIMD and/or intrinsics in C/C++ just isn’t on my wishlist. My wish is to write SIMD/vectorised code with Zig vectors in some other project.)

References:

- [https://nnethercote.github.io/perf-book/introduction.html](https://nnethercote.github.io/perf-book/introduction.html)  
- [https://en.algorithmica.org](https://en.algorithmica.org) 

## Zig CC

Zig as a C compiler felt like the same GOARCH+GOOS magic I had. Combining that with cargo zigbuild and we are cross-compiling to every target. The only slight issue I have is that Zig target triples are not the same as the one in Rust.

## Jujutsu

My workflow used to be:

- Checkout new branch  
- write code  
- Forgot to commit all the files so I commit all the files into a single commit  
- Painstakingly refactor the history so the commits make sense  
- Rebase to main  
- Colleague reviews my PR  
- Interactive rebase but pick that one commit that my colleague suggested changes and edits that commit  
- Stuck in rebase hell and painstakingly resolve all the conflicts  
- Force push

Now, I am not saying that jujutsu doesn’t have the same steps, but my workflow has been way more pleasant recently. I started off by initialising jujutsu on the main codebase I work on, and I used VisualJJ to move files across changesets. I can put files into their changesets and reordering commits becomes trivial. I was not too happy that I have to open up VSCode just to do jj operations (because right now JJ has lackluster support in IntelliJ), so I moved over to jjui and jj cli. It took me a second to understand changesets (commits) and bookmarks (branches) but it feels so natural now.

I can also prompt Claude Code to create a changeset every time it does something, so I can see the changes individually.

## Fast tooling

People used to write tooling for scripted languages in that specific language. Gulp, babel, webpack were written in JS. Poetry, pipenv, conda were written in Python.  
We are definitely seeing a surge of people (re)writing tooling to improve developer experience. UV, TypeScript compiler Go rewrite, esbuild, ripgrep, oxc, are all good examples. Now, I have yet to use any of these tools except uv, but it is really nice to see that there are groups of people dedicating their time to make DX nicer for other devs. UV is now my preferred tool to work with Python.

## GPU accelerated UI

I recently moved from VSCode to Zed and it feels really fast. I don’t have any scientific measurements that can back my claim but clicking around and typing feels responsive. It’s the power of Emacs/Neovim responsiveness but in a GUI. Ghostty boasts a GPU accelerated terminal as well, just like kitty, alacritty, wezterm, et cetera. I use ghostty mainly for the drop down terminal on macOS but the GPU acceleration is a nice bonus. (https://github.com/ghostty-org/ghostty\#competitive-performance)

It makes so much sense. Why waste the CPU cycles on drawing glyphs in the terminal, when you could offload that to the GPU?

(Although, for heavy coding I am still using the JetBrains suite, which even though it uses way more resources, still manages to be responsive. I guess this is years of optimisation with Java/Swing?)

## WebAssembly (?)

My friend reminded me that we were talking about WebAssembly around 10 years ago (we were 17 at the time…). Unfortunately I do not have the chance to use WASM consciously in my day to day life but this might be the original Java vision. Instead of going and downloading a JRE / .NET runtime, you just have a browser, which most consumer electronic devices in the world have.

People are using WASM at the server-side as well\! Again, I haven’t had the chance of using it but it seems like a good alternative to container based solutions. You can even run WASM/WASI on kubernetes.

## EFI and EFI stub

I had bad memories when I was messing around dual booting Windows and Linux when I was a kid, messing with the bootloader and disk sectors so BIOS could find the bootloader. Grub rescue is really not something I know how to deal with.

Booting the Linux kernel directly from UEFI feels like magic. There’s no bootloader anymore and there’s one less thing to go through when booting a computer. (When Chrome OS first came out, I kept going back to this video about [Chromium OS Fast Boot](https://www.youtube.com/watch?v=mTFfl7AjNfI).

Even though my home server uses Grub nowadays, it’s a nifty knowledge.

## Kubernetes (and Infrastructure as Code)

This section might deserve its own post. I used to see k8s as this extra thing you need to run on top of IaaS but now I see the potential. I don’t even think that the control plane reconciliation with your desired state (GitOps or whatever) is the main feature. K8s provided a single API and abstraction for the big clouds. Every cloud has their idiom of doing things on their platform, but k8s is just this one standard and big clouds implement it. I can declare a deployment, a persistent volume and a load balancer and the big cloud translates it to their parlance. Pair that with terraform and you  
Of course when you go into minute details this argument tends to break down (IAM is cloud specific, PostgresOperator \!= managed DB, object storage, etc.), but this taught me that Kubernetes can be a powerful abstraction. 

## SQLite

Nothing much to say other than it is my default flat-file database that just works.

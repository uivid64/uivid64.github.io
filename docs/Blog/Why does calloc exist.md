# Why does calloc exist?

MON 05 DECEMBER 2016

[Edit: Welcome Hacker News readers! Before we dive into the neat memory management esoterica, I want to briefly note that as engineers we have an [ethical obligation](https://en.wikipedia.org/wiki/Engineering_ethics#Obligation_to_society) in our work to consider the ["safety, health, and welfare of the public"](http://www.ieee.org/about/corporate/governance/p7-8.html), because if we don't, [terrible things](https://en.wikipedia.org/wiki/Engineering_ethics#Case_studies_and_key_individuals) [happen](http://www.jewishvirtuallibrary.org/jsource/Holocaust/IBM.html). This is a challenging responsibility that requires we all stay thoughtful and informed – but that's difficult if popular technical news aggregators choose to [censor links and discussions about the societal implications of technology](https://news.ycombinator.com/item?id=13108404). I sympathize with their moderation challenges, but this idea of creating a politics-free safe space is the cowards' way out, quite literally choosing the ["absence of tension" over "the presence of justice"](https://www.africa.upenn.edu/Articles_Gen/Letter_Birmingham.html). I hope the HN moderators find a way to step up to the responsibility their position entails; in the mean time, you might consider also subscribing to to [The Recompiler](https://recompilermag.com/) and [Model View Culture](https://modelviewculture.com/), and checking out [Safety Pin Box](https://www.safetypinbox.com/), [techsolidarity.org](https://sfbay.techsolidarity.org/), or [Fund Club](http://joinfundclub.com/). Anyway, thanks for listening! We now return to our regularly scheduled `calloc`-related programming, and I hope you enjoy my essay. And if you like this, you might also enjoy [Cory Benfield's related post](https://lukasa.co.uk/2016/12/Debugging_Your_Operating_System/).]

------

When programming in C, there are two standard ways to allocate some new memory on the heap:

```
void* buffer1 = malloc(size);
void* buffer2 = calloc(count, size);
```

`malloc` allocates an *uninitialized* array with the given number of bytes, i.e., `buffer1` could contain anything. In terms of its public API, `calloc` is different in two ways: first, it takes two arguments instead of one, and second, it returns memory that is pre-initialized to be all-zeros. So there are lots of books and webpages out there that will claim that the `calloc` call above is equivalent to calling `malloc` and then calling `memset` to fill the memory with zeros:

```
/* Equivalent to the calloc() call above -- OR IS IT?? */
void* buffer3 = malloc(count * size);
memset(buffer3, 0, count * size);
```

So... why does `calloc` exist, if it's equivalent to these 2 lines? The C library is not known for its excessive focus on providing convenient shorthands!

It turns out the answer is less widely known than I had realized! If I were [Julia Evans](https://drawings.jvns.ca/) at this point I'd make a neat little comic 😊. But I'm not, so... here's a wall of text.

It turns out there are actually two differences between calling `calloc`, versus calling `malloc` + `memset`.

## Difference #1: computers are bad at arithmetic

When `calloc` multiplies `count * size`, it checks for overflow, and errors out if the multiplication returns a value that can't fit into a 32- or 64-bit integer (whichever one is relevant for your platform). This is good. If you do the multiplication the naive way I did it above by just writing `count * size`, then if the values are too large then the multiplication will silently wrap around, and `malloc` will happily allocate a smaller buffer than we expected. That's bad. "*This* part of the code thought the buffer was *this* long but *that* part of the code thought it was *that* long" is the beginning of, like, eleventy-billion security advisories every year. ([Example](http://undeadly.org/cgi?action=article&sid=20060330071917))

I wrote a little program to demonstrate. It tries to allocate an buffer containing 263 × 263 = 2126 bytes, first using `malloc` and then using `calloc`:

Download source: [calloc-overflow-demo.c](https://vorpus.org/blog/why-does-calloc-exist/calloc-overflow-demo.c)

```
 6int main(int argc, char** argv)
 7{
 8    size_t huge = INTPTR_MAX;
 9
10    void* buf = malloc(huge * huge);
11    if (!buf) perror("malloc failed");
12    printf("malloc(huge * huge) returned: %p\n", buf);
13    free(buf);
14
15    buf = calloc(huge, huge);
16    if (!buf) perror("calloc failed");
17    printf("calloc(huge, huge) returned: %p\n", buf);
18    free(buf);
19}
```

On my computer, I get:

```
~$ gcc calloc-overflow-demo.c -o calloc-overflow-demo
~$ ./calloc-overflow-demo
malloc(huge * huge) returned: 0x55c389d94010
calloc failed: Cannot allocate memory
calloc(huge, huge) returned: (nil)
```

So yeah, apparently `malloc` successfully allocated a 73786976294838206464 exbiyte array? I'm sure that will work out well. This is a nice thing about `calloc`: it helps avoid terrible security flaws.

But, it's not *that* exciting. (I mean, let's be honest: if we really cared about security we wouldn't be writing in C.) It only helps in the particular case where you're deciding how much memory to allocate by multiplying two numbers together. This happens, it's an important case, but there are lots of other cases where we either aren't doing any arithmetic at all, or where we're doing some more complex arithmetic and need a more general solution. Plus, if we wanted to, we could certainly write our own wrapper for `malloc` that took two arguments and multiplied them together with overflow checking. And in fact if we want an overflow-safe version of `realloc`, or if we don't want the memory to be zero-initialized, then... we still have to do that. So, it's... nice? But it doesn't really justify `calloc`'s existence.

The other difference, though? Is super, super important.

## Difference #2: lies, damned lies, and virtual memory

Here's [a little benchmark program](https://vorpus.org/blog/why-does-calloc-exist/calloc-1GiB-demo.c) that measures how long it takes to `calloc` a 1 gibibyte buffer versus `malloc+memset` a 1 gibibyte buffer. (Make sure you compile without optimization, because modern compilers are clever enough to know that `free(calloc(...))` is a no-op and optimize it out!) On my laptop I get:

```
~$ gcc calloc-1GiB-demo.c -o calloc-1GiB-demo
~$ ./calloc-1GiB-demo
calloc+free 1 GiB: 3.44 ms
malloc+memset+free 1 GiB: 365.00 ms
```

i.e., `calloc` is more than 100x faster. Our textbooks and manual pages says they're equivalent. What the heck is going on?

The answer, of course, is that `calloc` is cheating.

For small allocations, `calloc` literally will just call `malloc+memset`, so it'll be the same speed. But for larger allocations, most memory allocators will for various reasons make a special request to the operating system to fetch more memory just for this allocation. ("Small" and "large" here are determined by some heuristics inside your memory allocator; for glibc "large" is anything >128 KiB, [at least in its default configuration](https://www.gnu.org/software/libc/manual/html_node/Malloc-Tunable-Parameters.html)).

When the operating system hands out memory to a process, it always zeros it out first, because otherwise our process would be able to peek at whatever detritus was left in that memory by the last process to use it, which might include, like, crypto keys, or embarrassing fanfiction. So that's the first way that `calloc` cheats: when you call `malloc` to allocate a large buffer, then *probably* the memory will come from the operating system and already be zeroed, so there's no need to call `memset`. But you don't know that for sure! Memory allocators are pretty inscrutable. So *you* have to call `memset` every time just in case. But `calloc` lives inside the memory allocator, so *it* knows whether the memory it's returning is fresh from the operating system, and if it is then it skips calling `memset`. And this is why `calloc` has to be built into the standard library, and you can't efficiently fake it yourself as a layer on top of `malloc`.

But this only explains part of the speedup: `memset+malloc` is actually clearing the memory twice, and `calloc` is clearing it once, so we might expect `calloc` to be 2x faster at best. Instead... it's 100x faster. What the heck?

It turns out that the kernel is also cheating! When we ask it for 1 GiB of memory, it doesn't actually go out and find that much RAM and write zeros to it and then hand it to our process. Instead, it fakes it, using virtual memory: it takes a single 4 KiB [page](https://drawings.jvns.ca/pagetable/) of memory that is already full of zeros (which it keeps around for just this purpose), and maps 1 GiB / 4 KiB = 262144 [copy-on-write](https://drawings.jvns.ca/copyonwrite/) copies of it into our process's address space. So the first time we actually *write* to each of those 262144 pages, then at that point the kernel has to go and find a real page of RAM, write zeros to it, and then quickly swap it in place of the "virtual" page that was there before. But this happens lazily, on a page-by-page basis.

So in real life, the difference won't be as stark as it looks in our benchmark up above – part of the trick is that `calloc` is shifting some of the cost of zero'ing out pages until later, while `malloc+memset` is paying the full price up front. BUT, at least we aren't zero'ing them out twice. And at least we aren't trashing the cache hierarchy up front – if we delay the zero'ing until we were going to write to the pages anyway, then that means both writes happen at the same time, so we only have to pay one set of TLB / L2 cache / etc. misses. And, most importantly, it's possible we might never get around to writing to all of those pages at all, in which case `calloc` + the kernel's sneaky trickery is a huge win!

Of course, the exact set of optimizations `calloc` uses will vary depending on your environment. A neat trick that used to be popular was that the kernel would go around and speculatively zero out pages when the system was idle, so that they'd be fresh and ready when needed – but this is [out of fashion](http://lists.dragonflybsd.org/pipermail/commits/2016-August/624202.html) on current systems. Tiny embedded systems without virtual memory obviously won't use virtual memory trickery. But in general, `calloc` is never worse than `malloc+memset`, and on mainstream systems it can do much better.

One real life example is a recent [bug in requests](https://github.com/kennethreitz/requests/issues/3729), where doing streaming downloads over HTTPS with a large receive block size was chewing up 100% CPU. It turns out that the problem was that when the user said they were willing to handle up to 100 MiB chunks at a time, then requests passed that on [to pyopenssl](https://github.com/pyca/pyopenssl/issues/577), and then pyopenssl used `cffi.new` to allocate a 100 MiB buffer to hold the incoming data. But most of the time, there wasn't actually 100 MiB ready to read on the connection; so pyopenssl would allocate this large buffer, but then would only use a small part of it. Except... it turns out that `cffi.new` [emulates calloc by doing malloc+memset](https://bitbucket.org/cffi/cffi/issues/295/cffinew-is-way-slower-than-it-should-be-it), so they were paying to allocate and zero the whole buffer anyway. If `cffi.new` had used `calloc` instead, then the bug never would have happened! Hopefully they'll fix that soon.

Or here's another example that comes up in [numpy](http://www.numpy.org/): suppose you want to make a big [identity matrix](https://en.wikipedia.org/wiki/Identity_matrix), one with 16384 rows and 16384 columns. That requires allocating a buffer to hold 16384 * 16384 floating point numbers, and each float is 8 bytes, so that comes to 2 GiB of memory total.

Before we create the matrix, our process is using 24 MiB of memory:

```
>>> import numpy as np
>>> import resource
>>> # this way of fetching memory usage probably only works right on Linux:
>>> def mebibytes_used():
...     return resource.getrusage(resource.RUSAGE_SELF).ru_maxrss / 1024
...
>>> mebibytes_used()
24.35546875
```

Then we allocate a 2 GiB dense matrix:

```
>>> big_identity_matrix = np.eye(16384)
>>> big_identity_matrix
array([[ 1.,  0.,  0., ...,  0.,  0.,  0.],
       [ 0.,  1.,  0., ...,  0.,  0.,  0.],
       [ 0.,  0.,  1., ...,  0.,  0.,  0.],
       ...,
       [ 0.,  0.,  0., ...,  1.,  0.,  0.],
       [ 0.,  0.,  0., ...,  0.,  1.,  0.],
       [ 0.,  0.,  0., ...,  0.,  0.,  1.]])
>>> big_identity_matrix.shape
(16384, 16384)
```

How much memory is our process using now? The Answer May Surprise You (Learn This One Weird Trick To Slim Down Your Processes Now):

```
>>> mebibytes_used()
88.3515625
```

Numpy allocated the array using `calloc`, and then it wrote 1s in the diagonal... but most of the array is still zeros, so it isn't actually taking up any memory, and our 2 GiB matrix fits into ~60 MiB of actual RAM. Of course there are other ways to accomplish the same thing, like using a real [sparse matrix library](https://en.wikipedia.org/wiki/Sparse_matrix), but that's not the point. The point is that *if* you do something like this, `calloc` will magically make everything more efficient – and it's always at least as fast as the alternative.

So basically, `calloc` exists because it lets the memory allocator and kernel engage in a sneaky conspiracy to make your code faster and use less memory. You should let it! Don't use `malloc+memset`!


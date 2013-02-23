Entropy RANDOMNESS LIBRARY

Generates 16 bytes of random data and writes one byte per word to 16 sequential
words, starting at a given location in memory.

Yes, I know that this technically doesn't have a lot of entropy because it is
expanding 4 bytes into 16. However, due to the avalanche effect of the hash and
the fact that the system clock has good enough time resolution for our
purposes, the data generated will certainly be good enough for in game usage.

Use is very simple. Set the A register to where you want the first out of 16
words of data to be written, then call lib_rng_wrapper.
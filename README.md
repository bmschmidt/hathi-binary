# Hathi-Trust Binary features

I recently posted an [observable notebook](https://observablehq.com/@bmschmidt/similarity-search-on-millions-of-books-in-browser) looking at 2 million books from Hathi in a fancy binary representation.

This is a repo with a small demonstration python notebook and the underlying data for *all* of Hathi and a short ipython notebook that you can edit to do searches on *any* arbitrary text.

Note before cloning: this is pretty big! 3 GB of data, to be precise.


Keyword search remains dominant for books, but at some point, whether they know it or not, everyone will probably be searching vectorized representations. This notebook tries out some methods for textual similarity search across a large corpus of books based on vectorized representations.

Back in 2018, it took me a lot of effort to set up an approximate nearest-neighbors search on a server. Now in 2022, new technologies and new tricks that make it possible to search across 2 million+ books in dozens of languages without even having a server. In this demo notebook, I load exactly 2 million books; it would be quite easy to scale up significantly higher, although it might take a minute to download representations of ten to twenty million books.

Note that I don't actually use learned representations in this notebook; instead, I use simple, language-agnostic features as described in Schmidt, Benjamin. 2018. “Stable Random Projection: Lightweight, General-Purpose Dimensionality Reduction for Digitized Libraries.” Journal of Cultural Analytics 3 (1). https://doi.org/10.22148/16.025.

Read below for more details.

This starts with a single random seed book: you can then refine your search by adding or subtracting matches. It's not perfect, obviously. But since this uses absolutely no metadata at all, it will find different sorts of books.

## Details

This is a multistep process.

Each book is reduced to a 1280-dimensional vector based on its words as described in Schmidt, Benjamin. 2018. “Stable Random Projection: Lightweight, General-Purpose Dimensionality Reduction for Digitized Libraries.” Journal of Cultural Analytics 3 (1). https://doi.org/10.22148/16.025.

Rather than use the full floats, I then reduce those each dimension to a single bit by determining if it is greater or less than zero. This gets us down from 5120 bytes per book to just 160. That loses some information, but there's still something there.

I then take those bytes and pack them into 20 big integers, and put them into parquet files as separate columns.

That all happens server-side. Using these files, it's then very easy and fast to calculate the distance between any two books in Hamming space by using binary operators: first xor (to see which bits have the same values) and then BIT_COUNT (to see how many bits are set). There's a probably a slightly faster way to do this, but this gets it done in about 3 seconds on my laptop. Not fast for reading more books than are in almost any physical library! 


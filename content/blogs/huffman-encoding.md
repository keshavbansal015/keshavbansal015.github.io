---
title: "Huffman Encoding Benchmark"
date: 2026-01-15
draft: false
tags: ["huffman", "benchmark", "performance", "compression"]
summary: "Benchmarking Huffman encoding implementations across C++, Go, Java, and Python on a 3.4MB text corpus."
---

Huffman encoding implementations in C++, Go, Java, and Python. Benchmarked on a 3.4MB text file.

## Overview

Huffman coding is a greedy algorithm that assigns variable-length codes to characters based on their frequencies. More frequent characters get shorter codes, resulting in optimal prefix-free compression.

## Implementations

I implemented the same Huffman encoder in four different languages to compare performance characteristics:

- **C++**: Raw pointers and manual memory management
- **Go**: Channels for frequency counting
- **Java**: PriorityQueue and object-oriented design
- **Python**: heapq module and dictionaries

## Results

The benchmark results on a 3.4MB text corpus showed interesting performance differences:

| Language | Encoding Time | Memory Usage |
|----------|----------------|--------------|
| C++      | 45ms           | 12MB         |
| Go       | 78ms           | 18MB         |
| Java     | 120ms          | 45MB         |
| Python   | 340ms          | 89MB         |

## Key Insights

1. **C++** dominates in raw performance due to zero-cost abstractions
2. **Go** offers a good balance of performance and simplicity
3. **Java's** GC pauses become noticeable at scale
4. **Python** is excellent for prototyping but needs C extensions for production

The complete source code is available on [GitHub](https://github.com/keshavbansal015/huffman-benchmark).

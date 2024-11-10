---
layout: post-no-feature
title: Notes on learning Openmp
description: "Introduction about OpenMP API"
categories: articles
tags: [sample post, images, test]
---
### Introduction
OpenMP API is used for writing parallel applications for shared memory computers for CPU and GPU architechtures. MPI+OpenMP for CPUs and OpenMP device offload are recommended.

#### OpenMP Overview
Managing threads and assigning loop distribution to threads.
OpenMP: shared-memory parallel programming model
All processors/cores access a shared main memory

The OpenMP Memory model:<br>
All threads have access to the same, global shared memory<br>
Data in private memory is only accessible to its particular thread.<br>
Data transfer is through shared memory and is 100% transparent to the application.<br>
<img width="622" alt="image" src="https://github.com/Hstellar/Hstellar.github.io/assets/22677436/89183aea-2a22-4e19-ad8d-cc6134637c82">

OpenMP execution model:
OpenMP programs just starts with just one thread: The Master<br>
Worker threads are spawned at parallel region, together with master they form team of threads 
In below figure, the dotted lines indicates OpenMP Runtime taking care of thread management 

<img width="338" alt="image" src="https://github.com/Hstellar/Hstellar.github.io/assets/22677436/2487289b-e497-4aa0-a081-d632a949f3c3">

Concept of Fork-Join in computer science: The idea of going parallel and then going sequential again<br>
OpenMP implements incremental parallelization: We can start from sequential program at parallel region and over time make it a parallel program.<br>
Never forget Amdahl's law!

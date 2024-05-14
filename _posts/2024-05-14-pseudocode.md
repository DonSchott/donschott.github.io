---
layout: post
title: Helloworld example post.
date: 2024-04-15 00:01:00
description: What included pseudo code could look like. Real posts soon to follow :-)
tags: formatting code
categories: sample-posts
pseudocode: true
---

This is just a **"helloworld" example post**. Stay tuned for my first real posts. Coming soon!

It shows how pseudocode for the quicksort algorithm is rendered by [pseudocode](https://github.com/SaswatPadhi/pseudocode.js). 

````markdown
```pseudocode
% This quicksort algorithm is extracted from Chapter 7, Introduction to Algorithms (3rd edition)
\begin{algorithm}
\caption{Quicksort}
\begin{algorithmic}
\PROCEDURE{Quicksort}{$$A, p, r$$}
    \IF{$$p < r$$}
        \STATE $$q = $$ \CALL{Partition}{$$A, p, r$$}
        \STATE \CALL{Quicksort}{$$A, p, q - 1$$}
        \STATE \CALL{Quicksort}{$$A, q + 1, r$$}
    \ENDIF
\ENDPROCEDURE
\PROCEDURE{Partition}{$$A, p, r$$}
    \STATE $$x = A[r]$$
    \STATE $$i = p - 1$$
    \FOR{$$j = p$$ \TO $$r - 1$$}
        \IF{$$A[j] < x$$}
            \STATE $$i = i + 1$$
            \STATE exchange
            $$A[i]$$ with $$A[j]$$
        \ENDIF
        \STATE exchange $$A[i]$$ with $$A[r]$$
    \ENDFOR
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
```
````

This is generated:

```pseudocode
% This quicksort algorithm is extracted from Chapter 7, Introduction to Algorithms (3rd edition)
\begin{algorithm}
\caption{Quicksort}
\begin{algorithmic}
\PROCEDURE{Quicksort}{$$A, p, r$$}
    \IF{$$p < r$$}
        \STATE $$q = $$ \CALL{Partition}{$$A, p, r$$}
        \STATE \CALL{Quicksort}{$$A, p, q - 1$$}
        \STATE \CALL{Quicksort}{$$A, q + 1, r$$}
    \ENDIF
\ENDPROCEDURE
\PROCEDURE{Partition}{$$A, p, r$$}
    \STATE $$x = A[r]$$
    \STATE $$i = p - 1$$
    \FOR{$$j = p$$ \TO $$r - 1$$}
        \IF{$$A[j] < x$$}
            \STATE $$i = i + 1$$
            \STATE exchange
            $$A[i]$$ with $$A[j]$$
        \ENDIF
        \STATE exchange $$A[i]$$ with $$A[r]$$
    \ENDFOR
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
```
Super cool! :-)
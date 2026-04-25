---
title: "Kitchen Sink — Random Test Post"
date: 2026-04-25
draft: false
math: true
tags: ["test", "random", "kitchen-sink"]
summary: "A random test post to verify markdown, code highlighting, and LaTeX rendering on the site. The content is deliberately nonsensical."
---

> **Heads up:** the content of this post is **intentionally random**. It exists to exercise every rendering path — markdown formatting, code blocks, tables, and LaTeX — in one page. Ignore the meaning; evaluate the look.

## Markdown formatting

This paragraph contains **bold**, *italic*, ***bold-italic***, `inline code`, and a [link to nowhere](https://example.invalid). Em-dashes — en-dashes – and ellipses… should all render cleanly.

A short list:

- First item, mildly interesting
- Second item, moderately interesting
- Third item, extremely interesting (not really)

An ordered list:

1. Preheat the oven to 200°C
2. Add distributed systems
3. Stir until linearizable
4. Serve with a side of CAP theorem

A nested list:

- Outer item
  - Inner item A
  - Inner item B
    - Deeper still

A blockquote within a blockquote:

> This is the outer quote.
>
> > And this is nested. Probably said by someone famous, but actually made up for this test.

---

## Tables

| Language | Year    | Paradigm            | Score (1–10) |
| -------- | ------- | ------------------- | ------------:|
| Go       | 2009    | Concurrent, C-like  |            9 |
| Rust     | 2010    | Systems, ownership  |            8 |
| Python   | 1991    | Multi-paradigm      |            7 |
| Scala    | 2004    | Functional + OO     |            6 |

---

## Code blocks

Shell:

```bash
# Count lines of Go code in a project, excluding vendor and tests
find . -name '*.go' \
  -not -path './vendor/*' \
  -not -name '*_test.go' \
  | xargs wc -l \
  | sort -n
```

Go:

```go
package main

import (
	"context"
	"fmt"
	"sync"
)

// fanOut runs fn over inputs with at most n workers.
func fanOut[T any, R any](ctx context.Context, inputs []T, n int, fn func(context.Context, T) R) []R {
	results := make([]R, len(inputs))
	var wg sync.WaitGroup
	sem := make(chan struct{}, n)
	for i, in := range inputs {
		wg.Add(1)
		sem <- struct{}{}
		go func(i int, in T) {
			defer wg.Done()
			defer func() { <-sem }()
			results[i] = fn(ctx, in)
		}(i, in)
	}
	wg.Wait()
	return results
}

func main() {
	ctx := context.Background()
	out := fanOut(ctx, []int{1, 2, 3, 4}, 2, func(_ context.Context, x int) int {
		return x * x
	})
	fmt.Println(out) // [1 4 9 16]
}
```

Python:

```python
from dataclasses import dataclass
from typing import Iterable

@dataclass(frozen=True)
class Point:
    x: float
    y: float

def centroid(points: Iterable[Point]) -> Point:
    pts = list(points)
    n = len(pts)
    if n == 0:
        raise ValueError("no points")
    return Point(sum(p.x for p in pts) / n, sum(p.y for p in pts) / n)

print(centroid([Point(0, 0), Point(4, 0), Point(0, 3)]))
```

SQL:

```sql
WITH claimed AS (
  SELECT id
  FROM jobs
  WHERE status = 'ready' AND run_at <= now()
  ORDER BY priority DESC, run_at ASC
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
UPDATE jobs
SET status = 'in_flight',
    lease_expires_at = now() + interval '30 seconds',
    attempts = attempts + 1
FROM claimed
WHERE jobs.id = claimed.id
RETURNING jobs.*;
```

---

## LaTeX / Math

Inline math: the probability mass function of a Poisson with rate $\lambda$ is $P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!}$, and $e^{i\pi} + 1 = 0$ remains as smug as ever.

Block math — the Fourier transform:

$$
\hat{f}(\xi) = \int_{-\infty}^{\infty} f(x)\, e^{-2\pi i x \xi}\, dx
$$

A matrix and a system:

$$
A = \begin{pmatrix} a_{11} & a_{12} \\ a_{21} & a_{22} \end{pmatrix},
\qquad
A \mathbf{x} = \mathbf{b}
$$

Softmax, because every test post needs one:

$$
\sigma(\mathbf{z})_i = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}
\qquad \text{for } i = 1, \dots, K
$$

The quadratic formula, unnecessarily:

$$
x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$

---

## Task list

- [x] Render markdown
- [x] Highlight code
- [x] Render LaTeX
- [ ] Achieve enlightenment
- [ ] Return library books

---

## Closing note

If you're reading this on the live site and everything above looks right — tables aligned, code coloured, math typeset, bold clearly bold — the rendering pipeline is working end to end.

This post will be deleted (or kept as a developer sanity check) at the author's whim.

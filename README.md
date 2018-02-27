# CS43: Functional Programming in Clojure

## Problem Set 3: Parallelism and Concurrency

### Problem 1 (Parallelism with one letter)

Consider the following function:

```clojure
(defn sleepy-sqrt [x]
   (Thread/sleep 2000)
   (Math/sqrt x))
```

- Map this function over `(repeat 3 20)` and measure the execution time.

- Leverage "parallelism on the cheap" and use `pmap` instead of `map` on the same sequence.  Measure the execution time.

- Use the [Claypoole](https://github.com/TheClimateCorporation/claypoole) library to run this in a thread pool with a thread count equal to the current number of CPUs.  Experiment with different thread counts and observe how the execution time varies.

### Problem 2 (Parallelized data processing)

- Download the IRS Statistics of Income Dataset (around 100MB) from http://www.irs.gov/pub/irs-soi/12zpallagi.csv.  Place it into the `data` subdirectory, and rename it to `soi.csv`, so that it works with the provided code samples.

- To ensure you've downloaded the data correctly, print out the header using

```clojure
(-> (slurp "data/soi.csv")
    (str/split #"\n")
    (first))
```

- This is enormously inefficient, because it has to load the entire file as a string into memory.  Instead, write a function that returns a lazy sequence of each line of the file, and grab the first element to print out the header.

You should get something like:

```
"STATEFIPS,STATE,zipcode,AGI_STUB,N1,MARS1,MARS2,MARS4,PREP,N2,NUMDEP,A00100,N00200,A00200,N00300,A00300,N00600,A00600,N00650,A00650,N00900,A00900,SCHF,N01000,A01000,N01400,A01400,N01700,A01700,N02300,A02300,N02500,A02500,N03300,A03300,N00101,A00101,N04470,A04470,N18425,A18425,N18450,A18450,N18500,A18500,N18300,A18300,N19300,A19300,N19700,A19700,N04800,A04800,N07100,A07100,N07220,A07220,N07180,A07180,N07260,A07260,N59660,A59660,N59720,A59720,N11070,A11070,N09600,A09600,N06500,A06500,N10300,A10300,N11901,A11901,N11902,A11902"
```

There are 77 columns total.  We will primarily consider ourselves with

- `A00200`: The salaries and wages amount
- `A02300`: The unemployment compensation amount

For more on what is in the data, check out https://www.stats.indiana.edu/taxes/12zpdoc.doc.

- `clojure.core.reducers` contains parallel implementations of reduce operation.  The first step is to reduce over several sequences in parallel.  The resulting values are then combined serially to produce the final result.

![alt text](/img/reduce-combine.png)

(Image by Henry Garner, Clojure Data Science)

We can implement a reduce over the sequence to count the lines as follows:

```clojure
(->> (io/reader "data/soi.csv")
       (line-seq)
       (reduce (fn [i x]
                 (inc i)) 0))
```

Use `r/fold` in `clojure.core.reducers` that counts the number of lines in parallel.

- The builtin Clojure IO utilities must load the entire CSV file into memory before create chunks for parallel processing, which can produce significant overhead.  The Iota library provides optimized data structures for reduction and folding, to improve on this performance.  Further, it can handle files larger than RAM using memory mapping.  Use `iota/seq` to load `data/soi.csv`.

- At this stage, your pipeline will process the file as a lazy sequence of strings.  Extend your code to parse each row in the CSV file into Clojure vectors, converting all numeric values to `Double` types.  Use `r/map` to perform this in parallel.

- Extend this pipeline to return a lazy sequence of maps representing each row, associating column names to values.  Use `r/map` to perform this in parallel.

For example, a given row might look like this:

#[{:N2 1505430.0, :A19300 181519.0, :MARS4 256900.0 ...}]

- Check out the excellent [tesser](https://github.com/aphyr/tesser) library, which focuses on providing parallel reduce operators for local and distributed execution.  Using `tesser.math`, compute the correlation and covariance between the `:A02300` and `:A00200` fields in parallel.  Draw conclusions about the relationship between salaries / wages and unemployment compensation.


### Problem 3 (Dining philosophers)

Recall Edsger Dijkstra’s classic “dining philosophers problem” - those of you who have taken CS110 will have “fond” memories of this notorious puzzle.

<img src="img/dining_phil.png" alt="Drawing" width="400px" />


(image: Wikipedia)

From Wikipedia, the problem description is as follows:

“Five silent philosophers sit at a round table with bowls of spaghetti. Forks are placed between each pair of adjacent philosophers.

Each philosopher must alternately think and eat. However, a philosopher can only eat spaghetti when they have both left and right forks. Each fork can be held by only one philosopher and so a philosopher can use the fork only if it is not being used by another philosopher. After an individual philosopher finishes eating, they need to put down both forks so that the forks become available to others. A philosopher can take the fork on their right or the one on their left as they become available, but cannot start eating before getting both forks.

Eating is not limited by the remaining amounts of spaghetti or stomach space; an infinite supply and an infinite demand are assumed.”

- Implement a concurrent simulation of the dining philosophers using Clojure software transactional memory primitives - refs and dosync will be helpful here.  Ensure that no philosopher starves (and that deadlock never occurs).

**References**

Problem 2 adapted from Clojure Data Science by Henry Garner


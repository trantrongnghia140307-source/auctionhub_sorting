# REFLECTION: AuctionHub Sorting

## 1. Why each algorithm fits its module

**A. Insertion Sort (Live Auction Monitor).** Bids arrive almost in order, so each new element only has to shift past a few positions. The cost is O(n + I), where I is the number of inversions, so on nearly sorted data it is close to linear. It also sorts in place and keeps equal elements in their original order.

**B. Selection Sort (Top-K Bid Finder).** Each pass picks exactly the largest element of the remaining range, so after k passes the Top-k is ready without sorting everything. It performs at most n − 1 swaps, far fewer than other O(n²) algorithms. On a small set, O(n²) comparisons are still acceptable.

**C. Interchange Sort (Price Anomaly Validator).** A batch has at most 50 records, so n² is about 2,500 comparisons, which is negligible. The code is short and easy to get right and to verify, which suits a QA tool that needs to be trustworthy rather than fast. The swap count shows how disordered the batch is.

**D. Bubble Sort with early stop (Stabilization Detector).** A single pass with no swaps is enough to know the data is already sorted. If the data has stabilized, it takes exactly one pass, i.e. O(n). The `swapped` flag is used to stop early, while the STABLE/UNSTABLE result depends only on the first pass.

**E. Quick Sort (Full Historical Sorter).** Average O(n log n) and in-place, so it suits millions of records. Median-of-three keeps already sorted or reversed data from becoming the worst case.

## 2. Quick Sort runtime on 100k records

Compiled with `g++ -O2 -std=c++17`, using 100,000 bids generated with `mt19937` (seed 42).

| Input | Time |
|---|---|
| 100,000 random bids | 26 ms |
| 100,000 already sorted bids | 11 ms |
| 100,000 reversed bids | 27 ms |

The result after sorting was verified with the `isSorted` function. Screenshot: `screenshot_runtime.png`.

## 3. A bug I ran into

**Symptom:** after running `gen.exe`, no error was shown but the file `large.txt` was not created.

**Cause:** `gen.exe` was run while the working directory was `test_data`, but `gen.cpp` writes to the relative path `test_data/large.txt`. The program looked for `test_data/test_data/`, so opening the file for writing failed, and because the code did not check the `ofstream`, it silently ignored the failure.

**How I found it:** I opened the folder and saw only the two `.exe` files and no `large.txt`.

**Fix:** put the code in the parent folder `auctionhub-sorting` and run it from there, keeping `test_data` for data only.

**Lesson:** code that reads or writes files should check `if (!f)` and print an error message.

## 4. Example prompts that follow the policy (Green Zone)

| # | Prompt | Level |
|---|---|---|
| 1 | "How do Selection Sort and Interchange Sort work, and how do they differ? Please use the array 3 1 4 2 as an example" | Green |
| 2 | "Compare the time complexity of Quick Sort and Insertion Sort in the best, average, and worst cases" | Green |
| 3 | "How do I commit and push files to a GitHub repository" | Green |

## 5. Worst case of Quick Sort with median-of-three

It is still O(n²). Median-of-three only keeps already sorted or reversed data from being the worst case. There are still specially constructed sequences where the median always falls near one end, so each partition removes only a few elements:

T(n) = T(n − 1) + O(n) = O(n²)

To guarantee O(n log n), you need a random pivot, introsort, or median-of-medians.

---
title: "Algorithm Performance"
date: 2010-07-01 18:20:24Z
draft: false
aliases:
- /post/algorithm-performance/
categories:
- Algorithms
---
I was chatting to a colleague recently who although he was a very good programmer, did not have a computer science/maths background – this was fine until he wanted to use bubble sort on a large(ish) (100k) number of records and I had to explain to him about how algorithms that take n<sup>2</sup> time are not your friend for big (or even little) n.

One of the most important decisions you can make when optimizing an application is the up-front choice of algorithm to use, performance tuning and optimization can make a difference but you are very lucky to achieve 2x-5x improvement, to be able to make 10x or 100x improvement in your application you need decent algorithm choice.

Here’s a little table I put together to help explain why you have to be careful and know the time/space performance of any algorithm you put in your applications.

| Items | const | ln n | log n | n | n<sup>2</sup> | n<sup>3</sup>| 2<sup>n</sup> | 3<sup>n</sup> | n! |
|--|--|--|--|--|--|--|--|--|--|
| 1     | 1     | 0 | 0 | 1 | 1 | 1 | 2 | 3 | 1 | 
| 10    | 1     | 2 | 1 | 10 | 100 | 1,000 | 1,024 | 59049 | 3.63x10<sup>6</sup> |
| 100   | 1     | 5 | 2 | 100 | 10<sup>4</sup> | 10<sup>6</sup> | 1.268x10<sup>30</sup> | 5.15x10<sup>47</sup> | 9.33x10<sup>157</sup> |
| 1,000 | 1     | 7 | 3 | 10<sup>3</sup> | 10<sup>6</sup> | 10<sup>9</sup> | 1.07x10<sup>301</sup> | | | 
| 10,000 | 1    | 9 | 4 | 10<sup>4</sup> | 10<sup>8</sup> | 10<sup>12</sup> | | | |
| 100,000 | 1   | 12 | 5 | 10<sup>5</sup> | 10<sup>10</sup> | 10<sup>15</sup> | | | |
| 1,000,000 | 1  | 14 | 6 | 10<sup>6</sup> | 10<sup>12</sup> | 10<sup>18</sup> | | | | 
| 10,000,000 | 1 | 16 | 7 | 10<sup>7</sup> | 10<sup>14</sup> | 10<sup>21</sup> | | | |
 
Now is this the approximate time/space growth you will get; how long your process will take will depend on the cost of each operation and to give you a guide, here’s now many units there are in one year.

| | Seconds | Milliseconds | Microseconds |
|--|--|--|--|
| 1 year | 3.16x10<sup>7</sup> | 3.16x10<sup>10</sup> | 3.16x10<sup>13</sup> | 

So if you happen to pick O(n<sup>3</sup>) algorithm you are going to have to waiting between 16 minutes and around 35 years to solve for just a 1,000 items! 

Notice also that as the notation power grows, even micro-second execution times (or expanding onto Cloud infrastructure) isn’t going to help too much.
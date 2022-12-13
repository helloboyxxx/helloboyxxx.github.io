---
layout: post
title: "Kahan summation algorithm"
subtitle: "Add up all numbers!"
date: 2022-12-11 02:11:01 -0400
background: '/img/posts/Rachmaninov/yuja.png'
---

Adding a [sequence](https://en.wikipedia.org/wiki/Sequence) of finite-[precision](https://en.wikipedia.org/wiki/Decimal_precision) [floating-point numbers](https://en.wikipedia.org/wiki/Floating-point_number)

adding a sequence of finite-precision floating-point number.

Adding a bunch of real numers together is simple in math, right? Real numers gives you nice property such as **Commutative property of addition** and **Associative property of addition**:

$$
a + b = b + a
$$

$$
(a + b) + c = a + (b + c)
$$

But when it comes to real life, on our computers, the second properties may not hold. Suppose we are using six-digit decimal floating-point system, and we have three variables: $$a = 10000.0, b = 3.14159, c = 2.71828$$.

$$a + b = 10003.14159$$, which rounds to $$10003.1$$

$$b + c = 5.85987$$

$$(a + b) + c = 10003.1 + 2.71828 = 10005.81828$$, rounds to $$10005.8$$

$$a + (b + c) = 10000.0 + 5.85987 = 10005.85987$$, rounds to $$10005.9$$

As long as we are trying to represent real numbers as machine numbers, we will experience rounding. And the error between the real number and the machine number is called **[roundoff errors](https://en.wikipedia.org/wiki/Round-off_error)**. 

Therefore, when we try to add a sequence of finite-precision floating-point numbers together, roundoff errors may accumulate. So we need to think carefully and try to minimize the error it creates. 

[Kahan summation algorithm](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) significantly reduces the error I discussed above. I am here to record my understanding of this algorithm, and I hope that this can help you in some degree.


```python
import numpy as np
import math
import matplotlib.pyplot as plt

data_len = 1e6
# mu, sigma = 0, 1
# data = np.random.normal(mu, sigma, data_len)
data = np.random.uniform(low = -1, high = 1, size = int(data_len))
print(data.dtype)

count, bins, ignored = plt.hist(data, 25, density=True)
plt.show()


# try calculate the sum using naive algorithm
def naive_sum(data):
  data_sum = 0.0
  for x in data:
    data_sum += x
  return data_sum

# Kahan's algorithm
def kahan_sum(data):
  data_sum = 0.0
  correction = 0.0
  for x in data:
      corrected_x = x - correction
      nu_sum = data_sum + corrected_x
      correction = (nu_sum - data_sum) - corrected_x
      data_sum = nu_sum
  
  return data_sum


print("naive_sum: \t", naive_sum(data))
print("np.sum: \t", np.sum(data))
print("Kahan's: \t", kahan_sum(data))
print("math.fsum: \t", math.fsum(data))

```








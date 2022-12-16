---
layout: post
title: "Kahan's summation algorithm"
subtitle: "Add up all numbers!"
date: 2022-12-15 22:06:54 -0400
background: '/img/posts/Kahan/Principia_Mathematica_54-43.png'
---
### Intro

Adding a bunch of real numers together is simple in math, right? Real numers gives you nice property such as **Commutative property of addition** and **Associative property of addition**:

$$
a + b = b + a
$$

$$
(a + b) + c = a + (b + c)
$$

But when it comes to real life, on our computers, the second properties may not hold. Suppose we are using six-digit decimal floating-point system, and we have three variables: $$a = 10000.0, b = 3.14159, c = 2.71828$$.

$$a + b = 10003.14159$$, which rounds to $$10003.1$$

$$b + c = 5.85987$$, there is no rounding.

$$(a + b) + c = 10003.1 + 2.71828 = 10005.81828$$, rounds to $$10005.8$$

$$a + (b + c) = 10000.0 + 5.85987 = 10005.85987$$, rounds to $$10005.9$$

qAs long as we are trying to represent real numbers as machine numbers, we will experience rounding. And the error between the real number and the machine number is called **[roundoff errors](https://en.wikipedia.org/wiki/Round-off_error)**. 

Therefore, when we try to add a sequence of finite-precision floating-point numbers together, roundoff errors may accumulate. So we need to think carefully and try to minimize the error it creates. 

### Kahan's algorithm

[Kahan summation algorithm](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) significantly reduces the error I discussed above. I am here to record my understanding of this algorithm, and I hope that this can help you in some degree. 

Let's first find what naive summation algorithm is doing. The following codes are all written in python. 

```python
def naive_sum(data):
  data_sum = 0.0
  for x in data:
    data_sum += x  # This is where roundoff error may happen
  return data_sum
```

We see that as `data_sum` becomes larger, roundoff error may occur and it accumulates in every cycle. Even though we can use some [rounding](https://en.wikipedia.org/wiki/Rounding) algorithms (such as half-even rounding mode) to reduce the expected value of the error created, we don't know how much error it creates, and therefore cannot control the error. 

Now let's stare at what just happened. Let $$S_{old}, S_{new}$$ represent the sum, and $$a$$ being the new value to be added. 

$$
S_{new} := S_{old} + a \\ 
a_{eff} := S_{new} - S_{old}
$$

In the second line, we are subtracting two machine numbers, so $$a_{eff}$$ will be the exact value of the difference between $$S_{new}$$ and $$S_{old}$$. Now lets try to calculate the difference between $$a$$ and $$a_{eff}$$. 

$$
c := a_{eff} - a
$$

$$c$$, which is used to keep track of the error created by our addition. We may observe that when $$c>0$$, it is rounding up; when $$c<0$$, it is rounding down.

<img class="img-fluid" src="/img/posts/Kahan/kahan01.png">

What's the next step? Accumulate all the error together and correct them in the end? No, that will create roundoff error when $$c$$ gets larger. So we should try to correct this error in every iteration. Now that is what Kahan's algorithm is doing: 

$$
a_c := a - c_{old}\\
S_{new} := S_{old} + a_c\\
a_{eff} := S_{new} - S_{old}\\
c_{new} := a_{eff} - a_c
$$



$$a_c$$ is the "corrected input". Since $$c_{old}$$ is error created from previous iterations (this is a small value), $$-c_{old}$$ will do the correction for our data. We add the "corrected input" to our sum. Then, again, calculate $$a_{eff}$$ and $$c_{new}$$ as we suggested before. Notice how $$c_{new}$$ is calculated by subtracting $$a_c$$ from $$a_{eff}$$ --- $$a_c$$ is our corrected input at that point. By trying to correct the error created in every iteration, the absolute value of our error will never become a large value. 

Here is my python code for the algorithm thus far:

```python
# Kahan's algorithm
def kahan_sum(data):
  data_sum = 0.0
  correction = 0.0
  for x in data:
      corrected_x = x - correction
      new_sum = data_sum + corrected_x
      correction = (new_sum - data_sum) - corrected_x
      data_sum = new_sum
  
  return data_sum
```


### Further enhancements

This "naive" Kahan's algorithm may somtimes fail to keep track of error. 

In the line $$a_{eff} := S_{new} - S_{old}$$, we are assuming that $$|S_{old}| > |a_c|$$. 
But it usually fails at the beginning of our iterations. Consider the given data:

$$
[1.0, 10^{100}, 1.0, -10^{100}]
$$

In the second iteration, the line $$S_{new} := S_{old} + a_c$$ will make $$S_{new} = 10^{100}$$. Then the following line will make $$a_{eff} = 10^{100} - 1 = 10^{100}$$, and $$c_{new}$$ will be zero.

So Neumaier provides an "improved Kahan–Babuška algorithm". It "effectively swapping the role of what is large and what is small" so that in each iteration, smaller values will be used for calculating error values. 

I didn't get too deep on this topic, but if you are interested, you can start from reading the refrences below. 

### References:

YouTube: [Fixing Roundoff Error, part 2: Kahan summation & double doubles](https://youtu.be/fojaJcAk1sQ)

Wikipedia: [Kahan summation algorithm](https://en.wikipedia.org/wiki/Kahan_summation_algorithm)

[ASPN cookbook recipes for accurate floating point summation](https://code.activestate.com/recipes/393090/)
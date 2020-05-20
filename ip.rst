
.. code:: ipython3

    %logstop
    %logstart -rtq ~/.logs/ip.py append
    %matplotlib inline
    import matplotlib
    import seaborn as sns
    sns.set()
    matplotlib.rcParams['figure.dpi'] = 144

.. code:: ipython3

    from static_grader import grader

Program Flow exercises
======================

The objective of these exercises is to develop your ability to use
iteration and conditional logic to build reusable functions. We will be
extending our ``get_primes`` example from the `Program Flow
notebook <../PY_ProgramFlow.ipynb>`__ for testing whether much larger
numbers are prime. Large primes are useful for encryption. It is too
slow to test every possible factor of a large number to determine if it
is prime, so we will take a different approach.

Exercise 1: ``mersenne_numbers``
--------------------------------

A Mersenne number is any number that can be written as :math:`2^p - 1`
for some :math:`p`. For example, 3 is a Mersenne number
(:math:`2^2 - 1`) as is 31 (:math:`2^5 - 1`). We will see later on that
it is easy to test if Mersenne numbers are prime.

Write a function that accepts an exponent :math:`p` and returns the
corresponding Mersenne number.

.. code:: ipython3

    def mersenne_number(p):
        return 2**p - 1
    mersenne_number(2)




.. parsed-literal::

    3



Mersenne numbers can only be prime if their exponent, :math:`p`, is
prime. Make a list of the Mersenne numbers for all primes :math:`p`
between 3 and 65 (there should be 17 of them).

Hint: It may be useful to modify the ``is_prime`` and ``get_primes``
functions from `the Program Flow notebook <PY_ProgramFlow.ipynb>`__ for
use in this problem.

.. code:: ipython3

    # we can make a list like this
    my_list = [0, 1, 2]
    print(my_list)


.. parsed-literal::

    [0, 1, 2]


.. code:: ipython3

    # we can also make an empty list and add items to it
    another_list = []
    print(another_list)
    
    for item in my_list:
        another_list.append(item)
    
    print(another_list)


.. parsed-literal::

    []
    [0, 1, 2]


.. code:: ipython3

    def is_prime(number):
        if number <= 1:
            return False
        
        for factor in range(2, number):
            if number % factor == 0:
                return False
    
        return True
    
         
    
    
    def get_primes(n_start, n_end):
        prime_list = []
        for number in range(n_start, n_end):
            if is_prime(number):
                prime_list.append(number)
        return prime_list
        

.. code:: ipython3

    get_primes(3, 65)




.. parsed-literal::

    [3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61]



The next cell shows a dummy solution, a list of 17 sevens. Alter the
next cell to make use of the functions you’ve defined above to create
the appropriate list of Mersenne numbers.

.. code:: ipython3

    mersennes = [mersenne_number(p) for p in get_primes(3,65)]
    mersennes




.. parsed-literal::

    [7,
     31,
     127,
     2047,
     8191,
     131071,
     524287,
     8388607,
     536870911,
     2147483647,
     137438953471,
     2199023255551,
     8796093022207,
     140737488355327,
     9007199254740991,
     576460752303423487,
     2305843009213693951]



.. code:: ipython3

    grader.score.ip__mersenne_numbers(mersennes)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


Exercise 2: ``lucas_lehmer``
----------------------------

We can test if a Mersenne number is prime using the `Lucas-Lehmer
test <https://en.wikipedia.org/wiki/Lucas%E2%80%93Lehmer_primality_test>`__.
First let’s write a function that generates the sequence used in the
test. Given a Mersenne number with exponent :math:`p`, the sequence can
be defined as

.. math::  n_0 = 4 

.. math::  n_i = (n_{i-1}^2 - 2) mod (2^p - 1) 

Write a function that accepts the exponent :math:`p` of a Mersenne
number and returns the Lucas-Lehmer sequence up to :math:`i = p - 2`
(inclusive). Remember that the `modulo
operation <https://en.wikipedia.org/wiki/Modulo_operation>`__ is
implemented in Python as ``%``.

.. code:: ipython3

    def lucas_lehmer(p):
        lehmer_list = [4]
        for i in range(1,p-1):
            lehmer_list.append((lehmer_list[i-1]**2-2)%(2**p-1))
        return lehmer_list
    
    lucas_lehmer(5)
        




.. parsed-literal::

    [4, 14, 8, 0]



Use your function to calculate the Lucas-Lehmer series for
:math:`p = 17` and pass the result to the grader.

.. code:: ipython3

    ll_result = lucas_lehmer(17)
    
    grader.score.ip__lucas_lehmer(ll_result)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


.. code:: ipython3

    ll_result




.. parsed-literal::

    [4,
     14,
     194,
     37634,
     95799,
     119121,
     66179,
     53645,
     122218,
     126220,
     70490,
     69559,
     99585,
     78221,
     130559,
     0]



Exercise 3: ``mersenne_primes``
-------------------------------

For a given Mersenne number with exponent :math:`p`, the number is prime
if the Lucas-Lehmer series is 0 at position :math:`p-2`. Write a
function that tests if a Mersenne number with exponent :math:`p` is
prime. Test if the Mersenne numbers with prime :math:`p` between 3 and
65 (i.e. 3, 5, 7, …, 61) are prime. Your final answer should be a list
of tuples consisting of ``(Mersenne exponent, 0)`` (or ``1``) for each
Mersenne number you test, where ``0`` and ``1`` are replacements for
``False`` and ``True`` respectively.

*HINT: The ``zip`` function is useful for combining two lists into a
list of tuples*

.. code:: ipython3

    
    def ll_prime(p):
        def is_prime(number):
            if number <= 1:
                return False
        
            for factor in range(2, number):
                if number % factor == 0:
                    return False
    
            return True
    
       
        lehmer_list = [4]    
        for i in range(1,p-1):
            lehmer_list.append((lehmer_list[i-1]**2-2)%(2**p-1))
            
        if lehmer_list[p-2]==0:
            return True
        else:
            return False
            
            
    
        
        

.. code:: ipython3

    Mersenne_exponent = []
    for num in range(3,65):
            if is_prime(num):
                Mersenne_exponent.append(num)
    zero_one_list=[] 
    for i in Mersenne_exponent:
        if ll_prime(i):
            zero_one_list.append(1)
        else:
            zero_one_list.append(0)
    zero_one_list       
    mersenne_primes = list(zip(Mersenne_exponent,zero_one_list))
    mersenne_primes





.. parsed-literal::

    [(3, 1),
     (5, 1),
     (7, 1),
     (11, 0),
     (13, 1),
     (17, 1),
     (19, 1),
     (23, 0),
     (29, 0),
     (31, 1),
     (37, 0),
     (41, 0),
     (43, 0),
     (47, 0),
     (53, 0),
     (59, 0),
     (61, 1)]



.. code:: ipython3

    mersenne_primes = mersenne_primes
    
    grader.score.ip__mersenne_primes(mersenne_primes)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


Exercise 4: Optimize ``is_prime``
---------------------------------

You might have noticed that the primality check ``is_prime`` we
developed before is somewhat slow for large numbers. This is because we
are doing a ton of extra work checking every possible factor of the
tested number. We will use two optimizations to make a ``is_prime_fast``
function.

The first optimization takes advantage of the fact that two is the only
even prime. Thus we can check if a number is even and as long as its
greater than 2, we know that it is not prime.

Our second optimization takes advantage of the fact that when checking
factors, we only need to check odd factors up to the square root of a
number. Consider a number :math:`n` decomposed into factors
:math:`n=ab`. There are two cases, either :math:`n` is prime and without
loss of generality, :math:`a=n, b=1` or :math:`n` is not prime and
:math:`a,b \neq n,1`. In this case, if :math:`a > \sqrt{n}`, then
:math:`b<\sqrt{n}`. So we only need to check all possible values of
:math:`b` and we get the values of :math:`a` for free! This means that
even the simple method of checking factors will increase in complexity
as a square root compared to the size of the number instead of linearly.

Lets write the function to do this and check the speed!
``is_prime_fast`` will take a number and return whether or not it is
prime.

You will see the functions followed by a cell with an ``assert``
statement. These cells should run and produce no output, if they produce
an error, then your function needs to be modified. Do not modify the
assert statements, they are exactly as they should be!

.. code:: ipython3

    import math
    from math import sqrt
    
    def is_prime_fast(number):
        if number <= 1:
            return False
        elif number == 2:
            return True
        
        elif number > 2 and number % 2 == 0:
            return False
        for factor in range(3, int(math.sqrt(number))+1,2):
                if number % factor == 0:
                    return False
        return True
    





.. parsed-literal::

    False



Run the following cell to make sure it finds the same primes as the
original function.

.. code:: ipython3

    for n in range(10000):
        assert is_prime(n) == is_prime_fast(n)

Now lets check the timing, here we will use the ``%%timeit`` magic which
will time the execution of a particular cell.

.. code:: ipython3

    %%timeit
    is_prime(67867967)


.. parsed-literal::

    4.94 s ± 52.9 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


.. code:: ipython3

    %%timeit
    is_prime_fast(67867967)


.. parsed-literal::

    300 µs ± 9.9 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)


Now return a function which will find all prime numbers up to and
including :math:`n`. Submit this function to the grader.

.. code:: ipython3

    def get_primes_fast(n):
        return [number for number in range(2, n+1) if is_prime_fast(number)]
    
    get_primes_fast(11)




.. parsed-literal::

    [2, 3, 5, 7, 11]



.. code:: ipython3

    grader.score.ip__is_prime_fast(get_primes_fast)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


Exercise 5: sieve
-----------------

In this problem we will develop an even faster method which is known as
the Sieve of Eratosthenes (although it will be more expensive in terms
of memory). The Sieve of Eratosthenes is an example of dynamic
programming, where the general idea is to not redo computations we have
already done (read more about it
`here <https://en.wikipedia.org/wiki/Dynamic_programming>`__). We will
break this sieve down into several small functions.

Our submission will be a list of all prime numbers less than 2000.

The method works as follows (see
`here <https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes>`__ for more
details)

1. Generate a list of all numbers between 0 and N; mark the numbers 0
   and 1 to be not prime
2. Starting with :math:`p=2` (the first prime) mark all numbers of the
   form :math:`np` where :math:`n>1` and :math:`np <= N` to be not prime
   (they can’t be prime since they are multiples of 2!)
3. Find the smallest number greater than :math:`p` which is not marked
   and set that equal to :math:`p`, then go back to step 2. Stop if
   there is no unmarked number greater than :math:`p` and less than
   :math:`N+1`

We will break this up into a few functions, our general strategy will be
to use a Python ``list`` as our container although we could use other
data structures. The index of this list will represent numbers.

We have implemented a ``sieve`` function which will find all the prime
numbers up to :math:`n`. You will need to implement the functions which
it calls. They are as follows

-  ``list_true`` Make a list of true values of length :math:`n+1` where
   the first two values are false (this corresponds with step 1 of the
   algorithm above)
-  ``mark_false`` takes a list of booleans and a number :math:`p`. Mark
   all elements :math:`2p,3p,...n` false (this corresponds with step 2
   of the algorithm above)
-  ``find_next`` Find the smallest ``True`` element in a list which is
   greater than some :math:`p` (has index greater than :math:`p` (this
   corresponds with step 3 of the algorithm above)
-  ``prime_from_list`` Return indices of True values

Remember that python lists are zero indexed. We have provided assertions
below to help you assess whether your functions are functioning
properly.

.. code:: ipython3

    def list_true(n):
        mylist = [False,False]
      
        for i in range(2,n+1):
            mylist.append(True)
        return mylist   
        
    len(list_true(31) )  




.. parsed-literal::

    32



.. code:: ipython3

    assert len(list_true(20)) == 21
    assert list_true(20)[0] is False
    assert list_true(20)[1] is False

Now we want to write a function which takes a list of elements and a
number :math:`p` and marks elements false which are in the range
:math:`2p,3p ... N`.

.. code:: ipython3

    def mark_false(bool_list, p):
        i = 2*p
        while i <  len(bool_list):   
            bool_list[i] = False
            i = i + p
         
        return bool_list
    
    # the function is not just marking but also returning a list 
    # Error was because i <= len(bool_list) is wrong and it must be < only . for exanple lets say p = 16 and n =32, 
    #  then 2p = 32 and bool_list[32] does not exist although the length is 32 but last element will be bool_list[31]


.. code:: ipython3

    assert mark_false(list_true(6), 2) == [False, False, True, True, False, True, False]

.. code:: ipython3

    mark_false(list_true(31), 2)




.. parsed-literal::

    [False,
     False,
     True,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True,
     False,
     True]



Now lets write a ``find_next`` function which returns the smallest
element in a list which is not false and is greater than :math:`p`.

.. code:: ipython3

    def find_next(bool_list, p):
        for i in range(p+1,len(bool_list)):
            if bool_list[i] == True:
                return i


.. code:: ipython3

    assert find_next([True, True, True, True], 2) == 3
    assert find_next([True, True, True, False], 2) is None

Now given a list of ``True`` and ``False``, return the index of the true
values.

.. code:: ipython3

    def prime_from_list(bool_list):
        primelist = []
        for i in range (0, len(bool_list)):
            if bool_list[i] == True:
                primelist.append(i)
        
        return primelist
    
    prime_from_list([False, False, True, True, False])





.. parsed-literal::

    [2, 3]



.. code:: ipython3

    assert prime_from_list([False, False, True, True, False]) ==  [2, 3]

.. code:: ipython3

    def sieve(n):
        bool_list = list_true(n)
        p = 2
        while p is not None:
            bool_list = mark_false(bool_list, p)
            p = find_next(bool_list, p)
        return prime_from_list(bool_list)
    
    sieve(30)




.. parsed-literal::

    [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]



.. code:: ipython3

    assert sieve(1000) == get_primes(0, 1000)

.. code:: ipython3

    %%timeit 
    sieve(1000)


.. parsed-literal::

    624 µs ± 14.3 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)


.. code:: ipython3

    %%timeit 
    get_primes(0, 1000)


.. parsed-literal::

    4.86 ms ± 39 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)


.. code:: ipython3

    grader.score.ip__eratosthenes(sieve)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


*Copyright © 2020 The Data Incubator. All rights reserved.*

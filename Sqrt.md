### Floored Square Root of an Integer

| Input | Output | Time Complexity| Space Complexity|
| ----------- | ----------- | ----------- | ----------- |
| integer: $n$ | integer: $\lfloor{ \sqrt n} \rfloor$ | $O(log \ n)$| $O(1)$|

> In *Structure and Interpretation of Computer Programs* by Abelson and Sussman there is a description of solving for the square root of a number. I give credit to this book for influencing my problem solving style for this problem.

The objective is to find a number  that when squared will be equal to the parameter `number` of the `sqrt` function. Mathematically the goal is to find
$y$ in the equation $y^2 = n$. Where $n$ is the input of the `sqrt` function.
Since $y$ could be so many possible values, it will be more efficient to logically limit the search space.
For square roots of integer values, the max value that the square root can have is the number itself.
The minimum value that can be the square root of an integer is $0$. With the lower and upper bounds are established,
testing if the mid point of the upper and lower bounds is the square root is a good guess. If the guess is too high,
the guess becomes the upper bound and the lower bound stays the same. If the guess is too low, the guess becomes the
lower bound and the upper bound stays the same. Generating a guess will continue until the square of the guess is accurate
within a specified error. The helper function `good_enough` has the error preset to $0.01$. Repeatedly cutting the search
space in half means the time complexity is $O(log \ n)$, where n is the integer input of the funtion. The memory needed
for this algorithm does not grow with the size of the input, which indicates space complexity of $O(1)$.

~~~python
import math
def sqrt(number):
    """
    Calculate the floored square root of a number

    Args:
       number(int): Number to find the floored squared root
    Returns:
       int: Floored Square Root
    """
    assert(number >= 0)
    assert(type(number) == float or type(number) == int)
    def good_enough(number, guess, eps = 0.01):
        diff = number - guess ** 2
        if diff < 0:
            diff *= -1
        if diff <= eps:
            return True
        else:
            return False

    def sqrt_helper(number, low, high, guesses=0):
        
        guess = (low + high) / 2.0

        if good_enough(number, guess):
            # print(guesses) good for observing the time complexity
            return math.floor(guess + 0.01)
        elif guess ** 2 > number:
            return sqrt_helper(number, low, guess, guesses= guesses + 1)
        else:
            return sqrt_helper(number, guess, high, guesses= guesses + 1)

    return sqrt_helper(number, 0, number)

~~~

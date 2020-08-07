# Simplex Algorithm

First, we import the packages we need.  Numpy will help with dealing with arrays.


```python
import numpy as np
import math
import time
```

A function is needed to take max (cx) subject to Ax <= b, x >= 0and convert it to standard form: max(c'x) subject to A'x = b', x >= 0. 


```python
def standard_form(A,b,c):
    N = A.shape[0]
    M = A.shape[1]
    A = np.hstack((A,np.eye(N)))
    c = np.hstack((c,np.zeros(N)))
    for i in range(N):
        if b[i] < 0:
            b[i] = -1*b[i]
            A[i] = -1*A[i]
    return A, b, c
```

Next, we need a function that will take the arrays and create the initial tableau.


```python
def create_table(A,b,c):
    Z_col = np.zeros(b.shape)
    A_prime = np.hstack((A, Z_col))
    c_prime = np.hstack((-c, np.array([1, 0])))
    table = np.vstack((np.hstack((A_prime, b)),c_prime))
    return table
```

To iterate through the Simplex Algorithm, we need to find the pivots at each iteration for the given tableau.


```python
def find_pivot(table):
    last_row = table[table.shape[0]-1]
    last_row = last_row[:-1]
    min_value_r = min(last_row)
    pivot_col = last_row.tolist().index(min_value_r)
    ratios = [table[i][table.shape[1]-1] / table[i][pivot_col] if table[i][pivot_col] != 0 else math.inf 
              for i in range(table.shape[0]-1)]
    min_value_c = min(ratios)
    pivot_row = ratios.index(min_value_c)
    return(pivot_row, pivot_col)
```

Once the pivot is found, we need to perform the row operations in the tableau that constitute a move.


```python
def move(table, pivot_row, pivot_col):
    pivot_value = table[pivot_row][pivot_col]
    table[pivot_row] = table[pivot_row]/pivot_value
    loop_rows = [i for i in range(table.shape[0]) if i != pivot_row]
    for i in loop_rows:
        new_val = table[i][pivot_col]
        table[i] = table[i] - new_val*table[pivot_row]
    return table
```

In order to know when we have arrived at a solution and need to terminate, we need a function that will check the tableau to see if we need to iterate again or terminate.


```python
def check(table):
    last_row = table[table.shape[0]-1]
    last_row = last_row[:-1]
    return not all(i >= 0 for i in last_row)
```

Once we have arrived at a solution, we need to find the basic and non-basic columns in our tableau.  This will allow us to derive the optimal values of each decision variable.  The optimal objective value will be in the lower right corner of the tableau.


```python
def read_solution(table):
    optimal_value = table[-1][-1]
    solution_vector = []
    orig_c = table[:,-1]
    orig_c = orig_c[:-1]
    for i in range(table.shape[1]):
        curr_col = table[:,i].tolist()
        remove_zeros = list(filter(lambda a: a != 0, curr_col))
        if remove_zeros == [1]:
            #these are the basic variables
            basic_col = np.array(curr_col)
            basic_col = basic_col[:-1]
            solution_vector.append(np.dot(orig_c,basic_col))
        else:
            #these are the non-basic variables
            solution_vector.append(0)
    return optimal_value, solution_vector
```

Now we have everything we need to solve a linear program using the Simplex algorithm! Let's try it out with this example problem.

"Suppose you have 2 production plants for making furniture. Due to specifications, each plant's operations are different in terms of labor needed, materials, and pollution produced per car.  These are summarized for each plant for one unit of furniture produced:

Plant 1: 2 labor, 2 materials, 13 pollution

Plant 2: 3 labor, 4 materials, 9 pollution

We have 200 hours of labor and 250 units of material available. We do not want to produce more than 900 units of pollution.  The goal is to maximize the total amount of furnature produced between all the plants."

We can formulate this as a linear programming model.  

Decision Variables:
Let x_i = the number of furnature produced at plant i.

Constraints:
We have 200 hours of labor
2*x_1 + 3*x_2 <= 200
We have 250 units of material available:
2*x_1 + 4*x_2 <= 250
We are allowed to produce 900 units of pollution
13*x_1 + 9*x_2 <= 900
We cannot produce a negative amount of furniture
x_i >= 0 for i = 1,2

Objective Function:
Maximize the number of furniture produced:
x_1 + x_2 


```python
A = np.array([[2, 3],
              [2, 4],
              [13, 9]])
b = np.array([[200],
              [250],
              [900]])
c = np.array([1, 1])
```

Now we can convert the arrays into standard form, create the tableau, find the pivot, move to the next vertex, iterate until finding a solution, then reporting the solution!


```python
start_time = time.time()
A, b, c = standard_form(A,b,c)
table = create_table(A,b,c)
iters = 0
max_iters = 1000
proceed = True
while proceed and iters < max_iters:
    pivot_row, pivot_col = find_pivot(table)
    table = move(table, pivot_row, pivot_col)
    proceed = check(table)
    iters += 1
if iters == max_iters:
    print("Solution not found")
else:
    optimal_value, solution_vector = read_solution(table)
    total_time = time.time() - start_time
    print("Optimal solution found in " + str(iters) + " iterations in " + str(total_time) + " seconds")
    print("The optimal objective value is: " + str(optimal_value))
    print("The optimal solution vector for the decision variables is:")
    print(solution_vector)
```

    Optimal solution found in 2 iterations in 0.0009975433349609375 seconds
    The optimal objective value is: 80.95238095238095
    The optimal solution vector for the decision variables is:
    [42.85714285714285, 38.0952380952381, 0, 11.904761904761898, 0, 0, 11.904761904761898, 0, 0.0, 0]
    

Thus, the optimal solution is for factory 1 to produce 42.9 units of furniture and for factory 2 to produce 38.1 units of furniture.  The Simplex algorithm is extremely fast in finding a solution, but obviously fractional pieces of furniture does not make sense in a practical world.  Thus, the Simplex algorithm is very effective at solving LP's.

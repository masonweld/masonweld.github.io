# Sudoku Solver

Problem Description: Given a 9 x 9 grid with some numbers already given, each row, column, and 3 x 3 sub-grid must have exactly one of each numbers from 1 to 9.

Solution Method: Binary Integer Program using Python.  The first solver uses CPLEX, and the second solver uses Gurobi.

# 1) Solving with CPLEX

## Step 1.1: Import Packages and Store Given Sudoku Grid

The grid will be stored as a Numpy Array, where the empty cells will be denoted with a zero.  The grid can be initialized by calling a file or explicitly forming it.


```python
from docplex.mp.model import Model
from docplex.util.environment import get_environment
import numpy as np
from numpy import genfromtxt
import pandas as pd
import time
```


```python
# read csv sudoku grid
#GRID = genfromtxt("sudoku_grid1.csv", delimiter=',')

# or directly declare grid
GRID = np.array([[8, 7, 6, 9, 0, 0, 0, 0, 0],
                 [0, 1, 0, 0, 0, 6, 0, 0, 0],
                 [0, 4, 0, 3, 0, 5, 8, 0, 0],
                 [4, 0, 0, 0, 0, 0, 2, 1, 0],
                 [0, 9, 0, 5, 0, 0, 0, 0, 0],
                 [0, 5, 0, 0, 4, 0, 3, 0, 6],
                 [0, 2, 9, 0, 0, 0, 0, 0, 8],
                 [0, 0, 4, 6, 9, 0, 1, 7, 3],
                 [0, 0, 0, 0, 0, 1, 0, 0, 4]])
```

## Step 1.2: Build CPLEX Binary Integer Program


```python
def build_sudoku_model_cplex(grid, **kwargs):
    # create model
    m = Model(name='Sudoku', **kwargs)
    # create variables
    # x(i,j,k) = 1 if number k is in row i col j
    v  = {(i,j,k): m.binary_var(name="x_{0}_{1}_{2}".format(i,j,k)) for i in range(9) for j in range(9) for k in range(9)}
    
    # constrain given numbers
    for i in range(9):
        for j in range(9):
            if grid[i][j] != 0:
                c = m.add_constraint(v[i,j,grid[i][j]-1] == 1)
    
    # constrain each cell to have one number 
    for i in range(9):
        for j in range(9):
            m.add_constraint(m.sum(v[i,j,k] for k in range(9)) == 1)
    
    # constrain each column to have one of each number
    for j in range(9):
        for k in range(9):
            m.add_constraint(m.sum(v[i,j,k] for i in range(9)) == 1)
    
    # constrain each row to have one of each number
    for i in range(9):
        for k in range(9):
            m.add_constraint(m.sum(v[i,j,k] for j in range(9)) == 1)
     
    # constrain each 3x3 box to have one of each number
    for a in range(3):
        for b in range(3):
            for k in range(9):
                box_row = [3*a, 3*a+1, 3*a+2]
                box_col = [3*b, 3*b+1, 3*b+2]
                m.add_constraint(m.sum(v[i,j,k] for i in box_row
                                       for j in box_col) == 1)
    
    # arbitrary objective funcion
    m.minimize(1)
    return m, v
```

## Step 1.3: Function for Printing Solution


```python
def print_solution(variables):
    # create pandas df to store solution and print solution
    opt_df = pd.DataFrame.from_dict(variables, orient="index", columns = ["variable_object"])
    opt_df.index = pd.MultiIndex.from_tuples(opt_df.index, names=["i", "j", "k"])
    opt_df.reset_index(inplace=True)
    opt_df["solution_value"] = opt_df["variable_object"].apply(lambda item: item.solution_value)
    solution = np.zeros((9,9))
    for i, row in opt_df.iterrows():
        if row["solution_value"] == 1:
            solution[int(row["i"])][int(row["j"])] = int(row["k"]) + 1
    print(solution)
```

## Step 1.4: Run, Solve, and Print Solution


```python
if __name__ == '__main__':
    # Build the model
    start_time = time.time()
    model, variables = build_sudoku_model_cplex(GRID)
    model.print_information()
    # Solve the model.
    if model.solve():
        t = time.time() - start_time
        print("Solution Found in " + str(t) + " seconds.")
        print_solution(variables)
    else:
        print("Sudoku grid given has no solution")
```

    Model: Sudoku
     - number of variables: 729
       - binary=729, integer=0, continuous=0
     - number of constraints: 354
       - linear=354
     - parameters: defaults
     - objective: none
     - problem type is: MILP
    Solution Found in 0.07341623306274414 seconds.
    [[8. 7. 6. 9. 1. 4. 5. 3. 2.]
     [3. 1. 5. 2. 8. 6. 7. 4. 9.]
     [9. 4. 2. 3. 7. 5. 8. 6. 1.]
     [4. 3. 8. 7. 6. 9. 2. 1. 5.]
     [6. 9. 1. 5. 2. 3. 4. 8. 7.]
     [2. 5. 7. 1. 4. 8. 3. 9. 6.]
     [1. 2. 9. 4. 3. 7. 6. 5. 8.]
     [5. 8. 4. 6. 9. 2. 1. 7. 3.]
     [7. 6. 3. 8. 5. 1. 9. 2. 4.]]
    

# 2) Solving With Gurobi

## Step 2.1: Import Packages and Store Given Sudoku Grid


```python
from gurobipy import *
import numpy as np
from numpy import genfromtxt
import csv
import time

GRID = np.array([[8, 7, 6, 9, 0, 0, 0, 0, 0],
                 [0, 1, 0, 0, 0, 6, 0, 0, 0],
                 [0, 4, 0, 3, 0, 5, 8, 0, 0],
                 [4, 0, 0, 0, 0, 0, 2, 1, 0],
                 [0, 9, 0, 5, 0, 0, 0, 0, 0],
                 [0, 5, 0, 0, 4, 0, 3, 0, 6],
                 [0, 2, 9, 0, 0, 0, 0, 0, 8],
                 [0, 0, 4, 6, 9, 0, 1, 7, 3],
                 [0, 0, 0, 0, 0, 1, 0, 0, 4]])
```

## Step 1.2: Build Gurobi Binary Integer Program


```python
def build_sudoku_model_gurobi(grid):
    
    m = Model("Solve_Sudoku")
    
    # Add Binary Variables
    # x_i,j,k = 1 if cell in row i column j is number k
    rows = range(9)
    columns = range(9)
    numbers = range(9)
    
    v = m.addVars(rows, columns, numbers, vtype = GRB.BINARY, name = 'v')
    
    # Assign values for filled in cells
    for i in rows:
        for j in columns:
            if GRID[i][j] != 0:
                v[i,j,GRID[i][j]-1].lb = 1
    
    # Each cell contains one number
    each_cell = m.addConstrs(v.sum(i,j,'*') == 1 for i in rows for j in columns)
    
    # Each row has one of each number
    each_row = m.addConstrs(quicksum([v[i,j,k] for j in columns]) 
    == 1 for i in rows for k in numbers)
    
    # Each column has one of each number
    each_col = m.addConstrs(quicksum([v[i,j,k] for i in rows]) 
    == 1 for j in columns for k in numbers)
    
    # Each box has one of each number
    for a in range(3):
        for b in range(3):
            box_row = [3*a, 3*a+1, 3*a+2]
            box_col = [3*b, 3*b+1, 3*b+2]
            each_box = m.addConstrs(quicksum([v[i,j,k] for i in box_row
                                              for j in box_col]) == 1 for k in numbers)
    return m, v, GRID
```

## Step 1.3: Function for Printing Solution


```python
def print_solution(m, GRID):  
    index = []
    for z in m.getVars():
        if z.x == 1:
            index.append(z.varName) 
    for c in range(len(index)):
        new_list = list(str(index[c]))
        i = int(new_list[2])
        j = int(new_list[4])
        k = int(new_list[6])
        GRID[i][j] = k + 1
    print(GRID)
```

## Step 1.4: Run, Solve, and Print Solution 


```python
if __name__ == '__main__':
    # Build the model
    start_time = time.time()
    model, variables, GRID = build_sudoku_model_gurobi(GRID)
    model.update()
    # Solve the model
    model.optimize()
    t = time.time() - start_time
    print("Solution Found in " + str(t) + " seconds.")
    print_solution(model, GRID)
```

    Optimize a model with 324 rows, 729 columns and 2916 nonzeros
    Variable types: 0 continuous, 729 integer (729 binary)
    Coefficient statistics:
      Matrix range     [1e+00, 1e+00]
      Objective range  [0e+00, 0e+00]
      Bounds range     [1e+00, 1e+00]
      RHS range        [1e+00, 1e+00]
    Presolve removed 324 rows and 729 columns
    Presolve time: 0.00s
    Presolve: All rows and columns removed
    
    Explored 0 nodes (0 simplex iterations) in 0.01 seconds
    Thread count was 1 (of 4 available processors)
    
    Solution count 1: 0 
    
    Optimal solution found (tolerance 1.00e-04)
    Best objective 0.000000000000e+00, best bound 0.000000000000e+00, gap 0.0000%
    Solution Found in 0.02991938591003418 seconds.
    [[8 7 6 9 1 4 5 3 2]
     [3 1 5 2 8 6 7 4 9]
     [9 4 2 3 7 5 8 6 1]
     [4 3 8 7 6 9 2 1 5]
     [6 9 1 5 2 3 4 8 7]
     [2 5 7 1 4 8 3 9 6]
     [1 2 9 4 3 7 6 5 8]
     [5 8 4 6 9 2 1 7 3]
     [7 6 3 8 5 1 9 2 4]]
    

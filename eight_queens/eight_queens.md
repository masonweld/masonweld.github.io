# Eight (N) Queens Problem

Problem Description: Place eight (or N) queens on an 8 by 8 (or N by N) chess board in such a way that all the queens do not "attack" any other queen.

Solution Method: Binary Integer Program using Python and Gurobi

# # Step 1: Import Packages


```python
from gurobipy import *
import gurobipy as grb
from gurobipy import GRB
import time
```

# # Step 2: Build Binary Integer Program


```python
def build_eight_queens(n):
    # create model
    Q = Model("Queens")
    N = range(n)
    
    # create variables
    x = Q.addVars(N,N,vtype = GRB.BINARY, name = "x")
    
    # contrain such that no queen attacks another
    
    #1) one queen per row
    Q.addConstrs(grb.quicksum(x[i,j] for j in N) == 1 for i in N)
    #2) one queen per column
    Q.addConstrs(grb.quicksum(x[i,j] for i in N) == 1 for j in N)
    #3) exactly n queens on board
    Q.addConstr(grb.quicksum(x[i,j] for i in N for j in N) == n)
    #4) one queen per diagnoal
    Q.addConstrs(grb.quicksum(x[i+k,i] for i in range(n-k)) <= 1 for k in N)
    Q.addConstrs(grb.quicksum(x[i,i+k] for i in range(n-k)) <= 1 for k in N)
    Q.addConstrs(grb.quicksum(x[i+k,n-i-1] for i in range(n-k)) <= 1 for k in range(n))
    Q.addConstrs(grb.quicksum(x[i,n-i-k-1] for i in range(n-k)) <= 1 for k in range(n))
    #5) arbitrary objective function
    Q.setObjective(0)
    
    return Q, x
```

# # Step 3: Function for Printing Solution


```python
def print_solution(x,N):
    for i in N:
        line = ""
        for j in N:
            if x[i,j].x > 0.1:
                line = line + "Q "
            else:
                line = line + ". "
        print(line)
```

# # Step 4: Run, Solve, and Print Solution


```python
if __name__ == '__main__':
    # Build the model
    for n in [8, 10, 20, 30]:
        start_time = time.time()
        model, variables = build_eight_queens(n)
        model.setParam("OutputFlag", 0)
        model.optimize()
        status = model.status
        # Solve the model
        t = time.time() - start_time
        print("Solution for Board Size "+ str(n) +" Found in " + str(t) + " seconds.")
        print_solution(variables,range(n))
        print()
```

    Solution for Board Size 8 Found in 0.008981704711914062 seconds.
    . . . . Q . . . 
    . . . . . . Q . 
    . Q . . . . . . 
    . . . . . Q . . 
    . . Q . . . . . 
    Q . . . . . . . 
    . . . . . . . Q 
    . . . Q . . . . 
    
    Solution for Board Size 10 Found in 0.008973836898803711 seconds.
    . . . . . Q . . . . 
    . . . Q . . . . . . 
    . Q . . . . . . . . 
    . . . . . . . . . Q 
    . . . . . . . Q . . 
    . . Q . . . . . . . 
    Q . . . . . . . . . 
    . . . . . . . . Q . 
    . . . . . . Q . . . 
    . . . . Q . . . . . 
    
    Solution for Board Size 20 Found in 0.01695418357849121 seconds.
    . . . . . . . . Q . . . . . . . . . . . 
    . . . . . Q . . . . . . . . . . . . . . 
    . . . . . . . . . . . . Q . . . . . . . 
    . . . . . . . . . . . . . . . . . . Q . 
    . . . . . . . . . . . . . Q . . . . . . 
    . . . . . . . . . . Q . . . . . . . . . 
    . . . . . . . . . . . . . . . . . Q . . 
    . . . . Q . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . Q . . . . . 
    . Q . . . . . . . . . . . . . . . . . . 
    . . . Q . . . . . . . . . . . . . . . . 
    . . . . . . . . . Q . . . . . . . . . . 
    . . . . . . Q . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . Q . . . . 
    . . Q . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . Q . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . Q 
    . . . . . . . . . . . . . . . . Q . . . 
    . . . . . . . Q . . . . . . . . . . . . 
    Q . . . . . . . . . . . . . . . . . . . 
    
    Solution for Board Size 30 Found in 0.02592921257019043 seconds.
    . . . . . . . . . . . . . . . . Q . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . Q . . . . . . . . . . . . . . . 
    . . . . . . . . Q . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . Q . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . . . . Q . . . . . 
    . . . . . . . . . . . . . . . . . . . . Q . . . . . . . . . 
    . . . . . . . . . . . . . . . . . Q . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . . . . . . Q . . . 
    . . . . . . . . . . . . . Q . . . . . . . . . . . . . . . . 
    Q . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . . . . . . . Q . . 
    . . . . . . . . . . . . . . . . . . . . . . . . . . . . . Q 
    . . . . . . . . . . . . . . . . . . . . . . Q . . . . . . . 
    . . . . . Q . . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . Q . . . . . . . . . . . . . . . . . . . 
    . . Q . . . . . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . Q . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . Q . . . . . . . . . . . 
    . Q . . . . . . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . Q . . . . . . . . . . . . . . . . . 
    . . . . . . . . . Q . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . . . . . Q . . . . 
    . . . . . . . . . . . . . . . . . . . Q . . . . . . . . . . 
    . . . Q . . . . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . . . Q . . . . . . 
    . . . . . . . Q . . . . . . . . . . . . . . . . . . . . . . 
    . . . . Q . . . . . . . . . . . . . . . . . . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . Q . . . . . . . . 
    . . . . . . . . . . . . . . . . . . . . . . . . . . . . Q . 
    . . . . . . . . . . . . . . . Q . . . . . . . . . . . . . . 
    
    

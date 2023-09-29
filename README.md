## DA 350 SP23 Lab 4 - Bicycle Production

![image](https://github.com/trinhkhoa/Lab-4--Bicycle-Production/assets/77667121/546d6b3d-ed59-4bc0-9180-28aaa60faed4)

**This Lab is due Wednesday night, February 15.**

Please use this .ipynb file as a starting point and write your code and fill in your answers directly in the notebook. I will ask specific questions in bolded font . For written answers, please create the cell as a Markdown type so the text displays neatly. For code, comment sufficiently so I can understand your process. Please try to delete unnecessary, old code so it is easy for me to evluate. When you have finished, displayed all code outputs and written all answers, please save and submit to Canvas as a .html file.

## Background and Objective:
Bicycle sales increased dramatically over the Covid-19 pandemic. The industry saw a 121% rise in sales from March 2020 to March 2021. Grand View Research predicts the compound annual growth rate (CAGR) of the market as 9.7% per year from 2023 to 2030, see https://www.grandviewresearch.com/industry-analysis/bicycle-market.

This week's lab will be very similar to the Lego furniture lab from class to practice using Gurobi through Python. The data are fictional to minimize friction as you learn the software and get familiar with the syntax. Subsequent week labs will be more nuanced and operating with real world data.

## Data:
You are planning the manufacturing agenda of a data and psychology -themed bicycle manufacturing company, "Cycle Analysis". You have the following bicycles that you can produce, and the associated number of wheels, metal, cable, and production hours that each requires, as well as the selling price:

| Cycle	| Wheels	| Metal	| Cable	| Hours	| Price |
|---|---|---|---|---|---|
|Street  Bike	|2	|300g	|3m	|5	|$240 |
|Electric Bike	 |2	 |200g |	5m| 4	| $370|
|Mountain Bike| 2	|450g |	2m	|4	| $290 |
|Tricycle| 3|	100g |	1.5m |	1	|$190 |
|Unicycle|	1|	200g|	1m|	2	|$120 |

This week, you have 80 wheels, 25kg of metal, and 120m of cable on hand. You employ 2 workers, each of whom works 40 hours a week.
```python
#Import packages
!pip install gurobipy  # install gurobipy, if not already installed
import gurobipy as gp
from gurobipy import GRB
```
```
Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
Collecting gurobipy
  Downloading gurobipy-10.0.1-cp38-cp38-manylinux2014_x86_64.whl (12.8 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 12.8/12.8 MB 35.1 MB/s eta 0:00:00
Installing collected packages: gurobipy
Successfully installed gurobipy-10.0.1
```
```python
# Create an environment with your WLS license
params = {
"WLSACCESSID": '9e8555ab-a7bf-4be2-b678-00c5cf100cb1',
"WLSSECRET": '005c0f56-5a52-44cb-b8d2-a25c6b974b67',
"LICENSEID": 930631,
}
env = gp.Env(params=params)

# Create the model within the Gurobi environment
model = gp.Model(env=env)
```
```
Set parameter WLSAccessID
Set parameter WLSSecret
Set parameter LicenseID to value 930631
Academic license - for non-commercial use only - registered to trinh_k1@denison.edu
```
**1) Setup the linear programming optimization model to find the best combination of cycles of produce. Solve the problem - how many of each type of cycle should you produce, and what revenue do you make?**

**Decision Variables:**

  - `variables[0]`: number of street bike sold
  - `variables[1]`: number of electric bike sold
  - `variables[2]`: number of mountain bike sold
  - `variables[3]`: number of tricycle sold
  - `variables[4]`: number of unicycle sold

**Objective:**
Maximization:  240\*`variables[0]`  +370\*`variables[1]`  +290\*`variables[2]`  +190\*`variables[3]`  +120\*`variables[4]`

**Constraints:**

Subject to `Wheels`:  2\*`variables[0]` +  2\*`variables[1]` +  2\*`variables[2]` +  2\*`variables[3]` + `variables[4]`  ≤80
 
Subject to `Metal`:  300\*`variables[0]` +  200\*`variables[1]` +  450\*`variables[2]` +  100\*`variables[3]` +  200\*`variables[4]`  ≤25000
 
Subject to `Cable`:  3\*`variables[0]` +  5\*`variables[1]` +  2\*`variables[2]` +  1.5\*`variables[3]` + `variables[4]`  ≤120
 
Subject to `Hours`:  5\*`variables[0]` +  4\*`variables[1]` +  4\*`variables[2]` + `variables[3]` +  2\*`variables[4]`  ≤80

 ```python
from gurobipy import *

# Create a new model
m = gp.Model("Cycle_Analysis")
m.Params.LogToConsole = 0

##### Data
cycles = ['street bike','electric bike','mountain bike','tricycle','unicycle']

components = ['wheels','metal','cable','hours']

component_data =  [[2, 2, 2, 3, 1],
                   [300, 200, 450, 100, 200],
                   [1, 5, 2, 1.5, 1],
                   [5, 4, 4, 1, 2]]

pieces_data = [80, 25000, 120, 80]

##### Variables
#one loop for each variable type
#make a list to store all of the variables in
variables = []

#create all of the variables
for x in range(len(cycles)):
    #Assigns the revenue of furniture x, continuous variable type, and appropriate name
    variables.append(m.addVar(vtype = "C", name = cycles[x]))

##### Constraints
#linear expressions

#constraints for our piece counts
#one for each component type
for i in range(len(component_data)):
    #Add the constraint
    m.addConstr(gp.LinExpr(component_data[i], variables) <= pieces_data[i])

#set the model sense
#0 = max, 1 = min
#use 0 because our objective is to maximize the revenue
m.setObjective(240*variables[0] + 370*variables[1] + 290*variables[2] + 190*variables[3] + 120*variables[4])
m.ModelSense = 0

######### Solve the model
m.optimize()
```
```
Restricted license - for non-production use only - expires 2024-10-28
```
```python
# Print the results
if m.Status == gp.GRB.OPTIMAL:
    for v in m.getVars():
        print(v.varName, int(v.X))

print("Max Revenue:",'$',m.objVal)
```
```
street bike 0
electric bike 16
mountain bike 0
tricycle 16
unicycle 0
Max Revenue: $ 8960.0
```
**2) How many of each resource do you use in the optimal solution?**
```python
total_wheels = 2*variables[0].x + 2*variables[1].x + 2*variables[2].x + 3*variables[3].x + variables[4].x
total_metal = 300*variables[0].x + 200*variables[1].x + 450*variables[2].x + 100*variables[3].x + 200*variables[4].x
total_cable = 3*variables[0].x + 5*variables[1].x + 2*variables[2].x + 1.5*variables[3].x + variables[4].X
total_hour = 5*variables[0].x + 4*variables[1].x + 4*variables[2].x + variables[3].x + 2*variables[4].x
temp = [[2,300,3,5],
        [2,200,5,4],
        [2,450,2,4],
        [3,100,1.5,1],
        [1,200,1,2]]
i = 0
for v in m.getVars():
    print(v.varName, 'produced', int(v.x),'(',int(v.x*temp[i][0]),'-',int(v.x*temp[i][1]),'-',int(v.x*temp[i][2]),'-',int(v.x*temp[i][3]),')')
    i = i+1
print('Total wheels used in the optimal solution is',int(total_wheels))
print('Total metal used in the optimal solution is',int(total_metal))
print('Total cable used in the optimal solution is',int(total_cable))
print('Total hour used in the optimal solution is',int(total_hour))
```
```
street bike produced 0 ( 0 - 0 - 0 - 0 )
electric bike produced 16 ( 32 - 3200 - 80 - 64 )
mountain bike produced 0 ( 0 - 0 - 0 - 0 )
tricycle produced 16 ( 48 - 1600 - 24 - 16 )
unicycle produced 0 ( 0 - 0 - 0 - 0 )
Total wheels used in the optimal solution is 80
Total metal used in the optimal solution is 4800
Total cable used in the optimal solution is 104
Total hour used in the optimal solution is 80
```
**3) What assumption is being made about consumer demand in the model? How might this be viewed as a realistic assumption?**

The assumption being made about consumer demand in the model just only works in the ideal world, that 16 electric bike and 16 tricycle was produced will be sold at the given price without any changes of any causation. Furthermore, a business cannot just sell only electric bike or tricycle instead of selling all of them, which in the ideal world, some people will need street bike or mountain bike or unicycle, which they don't have to sell and may cause inventory from making too much of 1 products that optimize the profit.

**4) What is the lowest price that you could sell Unicycles at to want to produce any in the optimal solution?**
```python
reduced_costs = m.getAttr(GRB.Attr.RC)
print("Reduced cost of unicycles is: $", int(reduced_costs[4]))
print("The lowest price that we could sell Unicycles at to want to produce any in the optimal solution is: $", 120-int(reduced_costs[4]))
```
```
Reduced cost of unicycles is: $ -65
The lowest price that we could sell Unicycles at to want to produce any in the optimal solution is: $ 185
```
**5) What is the most money you would be willing to pay for 1 extra wheel? For 1 extra gram of Metal?**
```python
shadow_price = m.getAttr(GRB.Attr.Pi)
pc = ['wheels','metal']
for i in range(len(pc)):
    print("The most money I would be willing to pay for 1 extra", pc[i], "is : $", int(shadow_price[i]))
```
```
The most money I would be willing to pay for 1 extra wheels is : $ 39
The most money I would be willing to pay for 1 extra metal is : $ 0
```
**6) You can gain up to 20 more production hours by paying your workers overtime at $10 per hour. Re-solve the model with this extra possibility - now what cycles do you produce, and how much overtime do you pay?**

You will need to add one new variable, one new constraint, and modify one existing constraint to allow for this possibility.
```python
# Create a new model
n = gp.Model("Overtime_Analysis")
n.Params.LogToConsole = 0

##### Data
cycles = ['street bike','electric bike','mountain bike','tricycle','unicycle','overtime']

components = ['wheels','metal','cable','hours']

component_data =  [[2, 2, 2, 3, 1, 0],
                   [300, 200, 450, 100, 200, 0],
                   [1, 5, 2, 1.5, 1, 0],
                   [5, 4, 4, 1, 2, -1]]


pieces_data = [80, 25000, 120, 80]

##### Variables
#one loop for each variable type
#make a list to store all of the variables in
variables = []

#create all of the variables
for x in range(len(cycles)):
    #Assigns the revenue of furniture x, continuous variable type, and appropriate name
    variables.append(n.addVar(vtype = "c", name = cycles[x]))

##### Constraints
#linear expressions

#constraints for our piece counts
#one for each component type
for i in range(len(component_data)):
    #Add the constraint
    n.addConstr(gp.LinExpr(component_data[i], variables) <= pieces_data[i])
n.addConstr(variables[5] <=20)

#set the model sense
#0 = max, 1 = min
#use 0 because our objective is to maximize the revenue
n.setObjective(240*variables[0] + 370*variables[1] + 290*variables[2] + 190*variables[3] + 120*variables[4] - 10* variables[5])
n.ModelSense = 0

######### Solve the model
n.optimize()
# Print the results
if n.Status == gp.GRB.OPTIMAL:
    for v in n.getVars():
        print(v.varName, int(v.X))
```
```
print("Max Revenue:",'$',round(n.objVal, 2))
street bike 0
electric bike 19
mountain bike 2
tricycle 11
unicycle 0
overtime 20
Max Revenue: $ 10006.67
```
There was a massive tropical storm while I was out riding my bike. I decided to cyclone. -Lance Calfstrong

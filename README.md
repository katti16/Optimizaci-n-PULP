# Optimizaci-n-PULP
En este repositorio se puede encontrar una guía de la utilización del solver PULP

# 1. Iniciar el programa.
El primer paso es abrir un cuaderno en Python. Para ello, se puede utilizar "Google Collaboratory".
Antes de cargar los datos, es necesario instalar los paquetes que se van a utilizar para la simulación:
```shell
import pandas as pd
import numpy as np
!pip install pulp
import pulp
from google.colab import files
```

# 2. Cargar los datos.
```shell
uploaded = files.upload()
```

# 3. Definir las variables
````shell
viviendas = data.comunidad.unique()
max_capacities = {comunidad: data.loc[data.comunidad == comunidad, 'capacidad'].values[0] for comunidad in viviendas}
viviendas_nuevas = {comunidad: pulp.LpVariable(name=comunidad, cat=pulp.LpInteger, lowBound=0, upBound=max_capacities[comunidad] ) for comunidad in viviendas}
coeficiente = data.set_index('comunidad').loc[viviendas, 'coef1'].values
coste = data.set_index('comunidad').loc[viviendas,'coste construcción' ].values
poblacion = data.set_index('comunidad').loc[viviendas,'población' ].values
intercept = data.set_index('comunidad').loc[viviendas, 'intercept'].values
parque = data.set_index('comunidad').loc[viviendas, 'parque viviendas'].values
````

# Definir funciones
````shell
total_viviendas_nuevas = pulp.LpAffineExpression()
for i, comunidad in enumerate(viviendas):
    total_viviendas_nuevas += intercept[i] + coeficiente[i] * (viviendas_nuevas[comunidad])
coste_construccion = pulp.LpAffineExpression()
for i, comunidad in enumerate(viviendas):
    coste_construccion +=  coste[i] * viviendas_nuevas[comunidad]
````

# Definir restricciones
````shell
constraint1 = pulp.LpConstraint(
    e=pulp.lpSum(list(viviendas_nuevas.values())),
    sense=pulp.LpConstraintEQ,
    name='total viviendas',
    rhs=46000
)
constraint2=pulp.LpConstraint(
    e=pulp.lpSum(coste_construccion),
    name='presupuesto',
    rhs=4000000000
)

constraint3 = pulp.LpConstraint(
    e=pulp.lpSum(viviendas_nuevas.values()),
    sense=pulp.LpConstraintGE, 
    name='non-negative constraint',
    rhs=0
)

constraint4 = pulp.LpConstraint(
    e=pulp.lpSum(total_viviendas_nuevas.values()),
    sense=pulp.LpConstraintEQ, 
    name='cos',
    rhs=5.1
)

constraint5 = {comunidad: pulp.LpConstraint(
    e=viviendas_nuevas[comunidad],
    sense=pulp.LpConstraintLE,
    name=f"capacity_{comunidad}",
    rhs=max_capacities[comunidad]
) for comunidad in viviendas}
````

# Optimizar el problema
````shell
problem = pulp.LpProblem("distribución", pulp.LpMinimize)
problem += coste_construccion


problem += constraint3
problem += constraint2
problem += constraint1
problem += constraint4

for constraint in constraint5.values():
    problem += constraint

status = problem.solve()

print(f"Status: {pulp.LpStatus[problem.status]}")
for v in problem.variables():
    print(v.name, "=", v.varValue)
    
print(pulp.value(problem.objective))

````

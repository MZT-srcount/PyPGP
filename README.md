# HyperGP: A high performance heterogeneous parallel GP framework

[![License: BSD 3-Clause License](https://img.shields.io/badge/License-BSD%203--Clause-red)](https://github.com/MZT-srcount/HyperGP/blob/main/LICENSE)
[![PyPI](https://img.shields.io/badge/PyPI-support-blue)](https://pypi.org/project/HyperGP/)
[![readthedocs](https://img.shields.io/badge/docs-passing-green)]()

PyGP is an open-source high performance framework, providing convenient distributed heterogeneous accleration for the custom prototyping of Genetic Programming (GP) and its variants. To ensure both flexibility and high performance, PyGP encompasses a variety of technologies for GP characteristics to provide convenient prototyping and efficient acceleration of various custom algorithms. To enable quick prototyping within PyGP for research on different types of genetic programming and different application fields, adaptability is also emphasized in building the PyGP framework, to support a wide range of potential applications, such as symbolic regression and image classification. 

- HyperGP document: xxxxx


# Installation
HyperGP is available on PyPI and can be installed with:

```
pip install HyperGP
```

# Quick Start for Symbolic Regression

1. **import modoule**: Three types module should be import to run:  
  
   - *basic components*:  
      - ``Population`` to initialize population
      - ``PrimitiveSet`` to set the primitives and terminals
      - ``executor`` to execute the expression
      - ``Tensor`` to store and compute datas
      - ``GpOptimizer`` a workflow manager, to iter overall process 

   - *operators*:
      - such as: ``RandTrCrv``, ``RandTrMut``

   - *states*:
      - such as ``ProgBuildStates``, ``ParaStates``

```
    import random, HyperGP
    from HyperGP import Population, PrimitiveSet, executor, Tensor, GpOptimizer
    from HyperGP.library.operators import RandTrCrv, RandTrMut
    from HyperGP.states import ProgBuildStates, ParaStates
```

2. **generate the training data**: We can use ``Tensor`` module to generate the array, or use to encapsulate the ``numpy.ndarray`` or the ``list``
```
    # Generate training set
    input_array = Tensor.uniform(0, 10, size=(2, 10000))
    target = HyperGP.exp((input_array[0] + 1) ** 2) / (input_array[1] + input_array[0])
```
3. **build the primitive set**: To run the program, we will need  the ``PrimitiveSet`` module to define the used primitives and terminals

```
    # Generate primitive set
    pset = PrimitiveSet(input_arity=1,  primitive_set=[('add', HyperGP.add, 2),('sub', HyperGP.sub, 2),('mul', HyperGP.mul, 2),('div', HyperGP.div, 2)])
```

4. **initialize population**: with the ``PrimitiveSet``, we can use ``Population`` to initialize the population
```
    # Init population
    pop = Population(pop_size=100, prog_paras=ProgBuildStates(pset=pset, depth_rg=[2, 3], len_limit=10000), parallel=False)
```
5. **initialize** ``GpOptimizer`` **workflow module**: To run a workflow, we should first initialize it and set the states we use to the GpOptimizer.
```
    # Init workflow
    optimizer = GpOptimizer()

    # Register relevant states
    optimizer.status_init(
        p_list=pop.states['progs'].indivs,
        input=input_array,pset=pset,output=None,
        fit_list = pop.states['progs'].fitness)
```

6. **build the evaluation function**
```
    def evaluation(output, target):
        r1 = HyperGP.sub(output, target, dim_0=1)
        return (r1 * r1).sum(dim=1).sqrt()
```

7. **set mask**

```   
    # Set Mask
    def set_prmask(size):
        cdd = random.sample(range(size), size)
        return [[cdd[i] for i in range(0, size, 2)], [cdd[i] for i in range(1, size, 2)]]
```

8. **add the component user want to iteratively run**
```
    # Add components
    optimizer.iter_component(
        ParaStates(func=RandTrCrv(), source=["p_list", "p_list"], to=["p_list", "p_list"],
                    mask=set_prmask(100)),
        ParaStates(func=RandTrMut(), source=["p_list", ProgBuildStates(pset=pset, depth_rg=[2, 3], len_limit=10000), True], to=["p_list"],
                    mask=[random.sample(range(100), 100), 1, 1]),
        ParaStates(func=ExecGPU(), source=["p_list", "input", "pset"], to=["output", None],
                    mask=[1, 1, 1]),
        ParaStates(func=evaluation, source=["output", "target"], to=["fit_list"],
                    mask=[1, 1]))
```
9. **run the optimizer**
```
    # Iteratively run
    optimizer.run(100)
```


## Main Features

A rich acceleration mode are supported.

| **Features**                | **Stable-Baselines3** |
| --------------------------- | ----------------------|
| Documentation               | :heavy_check_mark: |
| Custom environments         | :heavy_check_mark: |
| Acceleration for Custom algorithms           | :heavy_check_mark: |
| Acceleration for Custom monitors             | :heavy_check_mark: |
| Acceleration for Custom representation | :heavy_check_mark: |
| Multi-node parallel         | :heavy_check_mark: |
| GPU-Acceleration            | :heavy_check_mark: |
| Hybrid Acceleration with other library   | :heavy_check_mark: |
| High code coverage          | :heavy_check_mark: |

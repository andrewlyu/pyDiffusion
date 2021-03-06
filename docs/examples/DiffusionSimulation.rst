====================
Diffusion Simulation
====================

Here are few examples of how to use ``pydiffusion.mphSim`` to perform diffusion simulations.

.. code-block:: python

    import numpy as np
    import matplotlib.pyplot as plt
    import pandas as pd
    from pydiffusion.core import DiffSystem
    from pydiffusion.utils import step, mesh
    from pydiffusion.simulation import mphSim
    from pydiffusion.plot import profileplot, DCplot
    from pydiffusion.io import read_csv, save_csv

Diffusion simulation by easy setups
-----------------------------------

pyDiffusion can simulate simple diffusion systems by easy setups.

Usually, a diffusion simulation requires two inputs:

1. A initial profile (``pydiffusion.core.DiffProfile``)
2. A diffusion system with well-defined diffusion coefficients (``pydiffusion.core.DiffSystem``)

Define a diffusion system with constant diffusion coefficients 10\ :sup:`-14`\ m\ :sup:`2`\/s with concentration range from 0 to 1. To accomplish this, we should create a ``DiffSystem`` object. Input parameters include `Xr`: solubility range, ``X`` and ``DC``: concentration and corresponding diffusion coefficients data.

.. code-block:: python

    diffsys = DiffSystem(Xr=[0, 1], X=[0, 1], DC=[1e-14, 1e-14], name='Constant D')

Then create a initial step profile (step at 500 micron, length = 1000 micron) before simulation. To accomplish this, function ``mesh`` is used to mesh grids, and ``step`` is used to create a step profile upon meshed grids.

.. code-block:: python

    dis = mesh(0, 1000, 501)
    profile_init = step(dis, 500, diffsys, name='Intitial step profile')

    fig = plt.figure(figsize=(16, 6))
    ax1, ax2 = fig.add_subplot(121), fig.add_subplot(122)
    ax1.set_title('Diffusion Coefficients', fontsize=15)
    ax2.set_title('Initial Step Profile', fontsize=15)
    DCplot(diffsys, ax1)
    profileplot(profile_init, ax2)

.. image:: https://github.com/zhangqi-chen/pyDiffusion/blob/master/docs/examples/DiffusionSimulation_files/DiffusionSimulation_1.png

Simulate the diffusion process for 200 hours.

.. code-block:: python

    time = 200 * 3600
    profile_final = mphSim(profile_init, diffsys, time)

Plot the results

.. code-block:: python

    ax = profileplot(profile_init, ax, ls='--')
    profileplot(profile_final, ax, c='r')

.. image:: https://github.com/zhangqi-chen/pyDiffusion/blob/master/docs/examples/DiffusionSimulation_files/DiffusionSimulation_2.png

Diffusion simulation from diffusivity data
------------------------------------------

The data required for diffusion simulation is the diffusion coefficients and solubility limits. Here we read concentration dependent diffusion coefficients for Ni-Mo system at 1100 degree C from datafile, and input solubility information mannually.

.. code-block:: python

    data = pd.read_csv('examples/data/NiMo.csv')
    X, DC = data['X'], data['DC']
    Xr = np.array([[0, 0.25],
                [0.49, 0.53],
                [0.97, 1]])
    diffsys_NiMo = DiffSystem(Xr=Xr, X=X, DC=DC)

As long as using `1d-array like` type, you can read ``X`` and ``DC`` data from any file type.

You can also use `pydiffusion.io.read_csv` to read diffusion coefficients directly from saved data.

.. code-block:: python

    _, diffsys_NiMo = read_csv('examples/data/NiMo.csv', [0, 1])

Then create initial step profile as above.

.. code-block:: python

    dis = mesh(0, 400, 301)
    profile_NiMo_init = step(dis, 300, diffsys_NiMo)

Perform diffusion simulation for 800 hours.

.. code-block:: python

    profile_NiMo = mphSim(profile_NiMo_init, diffsys_NiMo, 800*3600)

Plot results

.. code-block:: python

    fig = plt.figure(figsize=(16, 6))
    ax1, ax2 = fig.add_subplot(121), fig.add_subplot(122)
    ax1.set_title('Diffusion Coefficients of Ni-Mo at 1100C', fontsize=15)
    ax2.set_title('Diffusion Simulation for 800 hours', fontsize=15)
    DCplot(diffsys_NiMo, ax1)
    profileplot(profile_NiMo_init, ax2, ls='--')
    profileplot(profile_NiMo, ax2, c='r')

.. image:: https://github.com/zhangqi-chen/pyDiffusion/blob/master/docs/examples/DiffusionSimulation_files/DiffusionSimulation_3.png

Save simulation results
-----------------------

Both diffusion profile and diffusion coefficient information can be saved into .csv format by pydiffusion. So that you can read it next time by ``read_csv``. It is recommended to save both profile and diffusion coefficients at the same time.

.. code-block:: python

    save_csv('examples/data/NiMo_800h.csv', profile=profile_NiMo, diffsys=diffsys_NiMo)

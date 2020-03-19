# Marlim R3D

Marlim R3D is a realistic resistivity model with corresponding
controlled-source electromagnetic data. The model was created by Carvalho and
Menezes, 2017, and the resulting CSEM data, computed with *SBLwiz* from *EMGS*,
was presented by Correa and Menezes, 2019.


### References

- B. R. Carvalho and P. T. L. Menezes, 2017, Marlim R3D: a realistic model for
  CSEM simulations - phase I: model building: Brazilian Journal of Geology, 47,
  633-644; DOI:
  [10.1590/2317-4889201720170088](https://doi.org/10.1590/2317-4889201720170088).
- Correa, J. L. and P. T. L. Menezes, 2019, Marlim R3D: A realistic model for
  controlled-source electromagnetic simulations - Phase 2: The
  controlled-source electromagnetic data set: Geophysics, 84(5), E293-E299;
  DOI: [10.1190/geo2018-0452.1](https://doi.org/10.1190/geo2018-0452.1).


## Loading the model and the survey

```python
import discretize
import xarray as xr

# Load model and survey
ds = xr.load_dataset('../block_model_and_survey.nc', engine='h5netcdf')

# Mesh
hx, hy, hz = ds.attrs['hx'], ds.attrs['hy'], ds.attrs['hz']
x0 = ds.attrs['x0']
mesh_model = discretize.TensorMesh([hx, hy, hz], x0=x0)

# Models
resh_bg, resh_bg = ds.attrs['resh_bg'], ds.attrs['resv_bg']
resh_tg, resv_bg = ds.attrs['resh_tg'], ds.attrs['resv_tg']

# Survey
src = ds.attrs['src']
strength = ds.attrs['strength']
freq = ds.attrs['freq']
rec_x = ds.x.data
rec_y = ds.attrs['rec_y']
rec_z = ds.attrs['rec_z']
```


## Saving the data

```python
# Save the three lines
ds.line_1_re.data = ... # y =-3000 RE (req. for layered and block model)
ds.line_1_im.data = ... # y =-3000 IM (req. for layered and block model)
ds.line_2_re.data = ... # y =    0 RE (req. for layered and block model)
ds.line_2_im.data = ... # y =    0 IM (req. for layered and block model)
ds.line_3_re.data = ... # y = 3000 RE (only req. for block model)
ds.line_3_im.data = ... # y = 3000 IM (only req. for block model)

# Add info
ds.attrs['runtime'] = ...     # Elapsed real time (wall time) [s]
ds.attrs['cputime'] = ...     # Total time [s] (for parallel comp. >> runtime)
ds.attrs['nthreads'] = ...    # Number of threads used
ds.attrs['maxram'] = ...      # Max RAM used
ds.attrs['ncells'] = ...      # Number of cells (FD codes, else 'N/A')
ds.attrs['nnodes'] = ...      # Number of nodes (FE codes, else 'N/A')
ds.attrs['ndof'] = ...        # Number of dof (FE codes, else 'N/A')
ds.attrs['extent'] = ...      # (xmin, xmax, ymin, ymax, zmin, zmax) mesh ext.
ds.attrs['min_cwidth'] = ...  # (hxmin, hymin, hzmin) smallest cell
ds.attrs['max_cwidth'] = ...  # (hxmax, hymax, hzmax) largest cell
ds.attrs['machine'] = ...     # Machine info, e.g.
#                             # "laptop with an i7-6600U CPU@2.6 GHz (x4)
#                             #  and 16 GB of memory, using Ubuntu 18.04"
ds.attrs['version'] = ...     # Version number of your code
ds.attrs['date'] = datetime.today().isoformat()

# Add other meta data: add whatever you think is important for your code
ds.attrs['...'] = ...

# Save it under <{model}_{code}.nc>
model = ...  # 'layered' or 'block'
code = ...   # 'custEM', 'emg3d', 'PETGEM', or 'SimPEG'
#            # custEM/PETGEM: you can add a '_{p}', where p = 'p1' or 'p2'
ds.to_netcdf(f"../results/{model}_{code}.nc", engine='h5netcdf')
```

A note regarding `runtime`, `cputime`, and also `maxram`: Only profile the
solution of the actual system `Ax=b`. Mesh creation, model and field
interpolation, and all other pre- and post-processing steps do not fall under
this measure. If you have doubts regarding the difference of `runtime` and
`cputime` please read https://en.wikipedia.org/wiki/Elapsed_real_time. In
short: runtime is the real-world time it takes. If it starts at 14:14:38 and
finishes at 14:15:48 then the runtime is 70 seconds. Now if you run the process
on one thread then cputime will be the same or less than runtime. However, if
you run your process in parallel then your cputime will be higher than runtime.

=> **PETGEM**: Please save data as `data.conj()`. PETGEM has, as far as I could
see, the opposite Fourier definition than custEM/emg3d/SimPEG. It is best we
save it all with the same definition.


## Info

Every code should store its info directly in the data (`ds.attrs[]`) as shown
in the code snippet above.

**Please make sure to add all info-data as indicated!**

If you want to get an idea of the content and format of the info-data to
provide have a look at the `BlockModel-Comparison.ipynb`, there you see the
info provided from `emg3d`.
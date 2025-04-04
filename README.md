# GEOSgcm_Irrigation_Wiki
## Instructions Couple Model
#### AGCM + Irrigation

### Step 1: AGCM base
Follow instructions from https://github.com/GEOS-ESM/GEOSgcm/wiki to build the
model

### Step 2: Add Irrigation
Replace your
```
./GEOSgcm/src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/GEOSsurface_GridComp
``` 
with
```
/discover/nobackup/scasirat/Projects/Coupled/GEOSgcmIrrigV2/GEOSgcm/src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/GEOSsurface_GridComp
``` 

```
cp -r /discover/nobackup/scasirat/Projects/Coupled/GEOSgcmIrrigV2/GEOSgcm/src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/GEOSsurface_GridComp ./GEOSgcm/src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/GEOSsurface_GridComp 
```

Rebuild the model

```./parallel_build.sh -mil```

### Step 3: Create Restarts and Boundary conditions

```
cd install-SLES15/bin
```

#### Boundary Conditions

Run make_bcs.py and follow instructions
```
./make_bcs.py
```
The correct BCs should be something like v13 - v12 + Irrigation

#### Restarts
Run remap_restarts.py and follow instructions (the grids and resolution should
be coherent with the boudnary conditions)
```
./remap_restarts.py
```

### Step 4: Create the model setup

```
./gcm_setup
```

### Step 5: Modify inputs

In your experiment directory ```EXPDIR``` we need to apply a few changes: 

#### linkbcs
Add Irrigation BCs path, similar to this:
```
/bin/ln -sf $BCSDIR/land/$BCRSLV/visdf_180x1080.dat visdf.dat
/bin/ln -sf $BCSDIR/land/$BCRSLV/nirdf_180x1080.dat nirdf.dat
/bin/ln -sf $BCSDIR/land/$BCRSLV/vegdyn_180x1080.dat vegdyn.data
/bin/ln -sf $BCSD.IR/land/$BCRSLV/lai_clim_180x1080.data lai.data
/bin/ln -sf $BCSDIR/land/$BCRSLV/green_clim_180x1080.data green.data
/bin/ln -sf $BCSDIR/land/$BCRSLV/ndvi_clim_180x1080.data ndvi.data
# Add Irrigation BCs
/bin/ln -sf /discover/nobackup/scasirat/Projects/Coupled/BCs/C180_1440_720/land/$BCRSLV/irrigation_180x1080.dat irrigation.data
```

Double check that the Ozone Data. If the simulation starts after 06-2017, you
cannot use Merra2OX. You can use for example the CMIP5, but, if so, you need to
change also the number of years pchem_clim_years in the AGCM.rc file from e.g. 39 to 228

#### AGCM.rc
As mentioned above, (irrelevant to irrigation), if the simulation starts after
06-2017, you cannot use MERRA20X. If you use CMIP5 the total years of the
dataset are 228. So substitute pchem_clim_years: 29 with pchem_clim_years:228
```
pchem_clim: species.data
pchem_clim_years: 228
```

##### Add irrigation
We can add Irrigation "restart" by adding below VEGDYN_INTERNAL RESTART_FILE,
the IRRIGATION_INTERNAL_RESTART_FILE, that correspond to the boundary conditions added in ```linkbcs```
```
VEGDYN_INTERNAL_RESTART_FILE:  vegdyn.data
IRRIGATION_INTERNAL_RESTART_FILE:  irrigation.data
```

#### RC/GEOS_SurfaceGridComp.rc
Here we activate Irrigation, by setting the GEOSldas Irrigation parameters
(e.g., RUN_IRRIG : 1)

```
RUN_IRRIG : 1
```

#### HISTORY.rc
If we want to print any of the irrigation variables, we can add them to the
history file. For example if we want the total water provided by irrigation, we
can add to a collection:

```
 'IRRLAND'        , 'SURFACE'     ,
```

#### Restarts
Add your restarts and the ```cap_restart``` file to the setup folder as mentioned in
the AMIP instructions https://github.com/GEOS-ESM/GEOSgcm/wiki/Running-an-AMIP-Experiment 

The model requires the restarts with the following (naming) format:
```
catch_internal_rst
fvcore_internal_rst
lake_internal_rst
landice_internal_rst
moist_internal_rst
openwater_internal_rst
pchem_internal_rst
seaicethermo_internal_rst
```

#### Additional Files to Change
##### CAP.rc
Here we need to set up the END_DATE. Don't change the BEG_DATE since it is
already given by cap_restart. Here you can change also the parameters related
to the job segments

##### gcm_run.j
Here we can change the Batch Parameters for Run Job 

### Step 6: Run the model
Run the model:
```sbatch gcm_run.j```

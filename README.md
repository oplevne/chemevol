# ChemEvol
[![Build Status](https://travis-ci.org/zemogle/chemevol.svg?branch=master)](https://travis-ci.org/zemogle/chemevol)

Python package to read in a star formation history file, input galaxy parameters and run a chemical evolution model to determine the evolution of gas, metals and dust in galaxies.

Running the script following the instructions below will produce:

1. a results data file with galaxy parameters in (filename.dat with name provided
  by user from inits file - see instructions below)

The code is based on Morgan & Edmunds 2003 (MNRAS, 343, 427)
and described in detail in Rowlands et al 2014 (MNRAS, 441, 1040), with latest features discussed in De Vis et al (submitted 2016).

If you use this code, please do cite the above papers.  The license is provided with this package.

## Installation

Download this repository and do the following:
```
python setup.py install
```

## Requirements

### Python packages
- numpy
- astropy
- matplotlib [Not a strict requirement]

### Input files needed
The code reads in a star formation history from a file called filename.sfh.  This needs to be in the form: time (yr), SFR (Msolar/yr) and in the directory where you are running the code.   An example is provided with this code `Milkyway.sfh` based on the SFH for the Milky Way in Yin et al 2009 (A & A, 505, 497).

### Input data needed
The code also requires a dictionary of initial parameters. This can be done by providing a .json or .csv file and using the package installed, or by running the example python script provided with this package.

There are example data files in the folder `<Download Dir>/chemevol/chemevol/examples/` which show the correct format of each type of file.  Feel free to copy these example data files into the directory where you wish to run the code and follow the instructions.  A detailed breakdown of the parameter files and inputs needed are found at the bottom of this readme.

There are helper functions for loading batch files in CSV and valid JSON format.
*Note*: Valid JSON uses double quotes for definitions and lower case for booleans, e.g. `"myvalue" : false`

## Running the code as a package using json or csv data files

```python
import chemevol as ch
galaxies = ch.BulkEvolve('<File directory>/data.json')
galaxies.upload_json()
galaxies.evolve_all()
```
Or for CSV files:
```python
import chemevol as ch
galaxies = ch.BulkEvolve('<File directory>/data.csv')
galaxies.upload_csv()
galaxies.evolve_all()
```
See the files in `<Download Dir>/chemevol/chemevol/examples/` for the correct format of each type of file (.py, .json, .csv examples are provided).

### Viewing the results
Once the code is run you will have an array called `galaxies.results` with all the parameters in.  To look at this data try:
```python
[g['dust_all'] for g in galaxies.results] #will print out the dust_all
[g['mgas'] for g in galaxies.results] #will print out all the gasmasses
gasmass  = galaxies.results[0]['mgas'] #etc
```

The code writes data to a file (if you use the example in `<Download Dir>/chemevol/chemevol/examples/data.json` the code writes out two files called `Model_I.dat` and `Model_VI.dat`).  To read in this data for plotting or playing with you can use `astropy.table`:
```python
from astropy.table import Table
import matplotlib.pyplot as plt

t = Table.read('Model_VI.dat', format='ascii')
plt.semilogy(t['fg'],t['dustmass']/(t['mgas']+t['mstars']))
```

Alternatively you can run the code without the bulkevolve class above:

## Running the code using the example python script provided
Copy the example_multi.py file to the desired directory (where your .sfh file is and where you want to have your results). Then edit the parameters in the init dictionaries within this script.

 python example_multi.py

## Full list of Input Parameters to Model

|   Parameter Name   |   Description   |   If not specified, will revert to default   |
|   --------------   |   :-----------  |   -------:  |
|   name             |   to identify your galaxy/run, output file will be called this   |   None   |
|   gasmass_init     |   initial gas mass in solar masses  |    None    |
|   SFH       			 |	 filename for star formation history file (filename.sfh)   |   MilkyWay.sfh : MW-like SFH    |
|   t_end       	   |	 end of time array for chemical integrals in Gyrs, try 20Gyrs   |   None    |
|   gamma            | 	 power law for extrapolation of SFH if using SFH generated by MAGPHYS code, otherwise set to zero |    None   |
|   IMF_fn         	 |	 choice of IMF function: Chab/chab/c, TopChab/topchab/tc, 	Kroup/kroup/k or Salp/salp/s   |   None    |
|   dust_source 		 |	 choice of dust sources to be included in run:	SN: supernova dust only; LIMS: low intermediate mass stars dust only; LIMS+SN: both SN and LIMS included; ALL: includes supernovae, LIMS and grain growth combined    |   None   |
|   reduce_sn_dust   |	 reduce the contribution from SN dust (currently based on Todinin & Ferrara 2001).  If leave as is, specify False. To reduce dust mass, quote number to reduce by eg 5 or 20  |    None    |
|   destroy 	       |	 add dust destruction from SN shocks: True or False   |   None   |
|   inflows 		     |	 inflows_metals = metallicity of inflow: input a number appropriate for the metallicity of primordial gas inflows eg 1e-4. inflows_xSFR = inflow rate of gas is X * SFR: input a number X.  inflows_dust = amount of dust inflow: input a number appropriate for dust/gas ratio of primordial inflows  |   None   |
|   outflows         | 	 outflows_metals = metallicity of inflow: input True or False where True = current metallicity of system, False = 0.  outflows_xSFR = outflow rate of gas is X * SFR: input a number. outflows_dust = amount of dust in outflow: input True of False where	True = dust/gas of system, False = 0   |   None    |
|   cold_gas_fraction    |	fraction of gas in cold dense state for grain growth, typically 0.5 (Asano et al 2012)  |    None    |
|   epsilon_grain   |		grain growth parameter from Mattsson & Andersen 2012, typically 500 for t_grow ~ 10Myr  consistent with MW |    None    |
|   destruct        | 			amount of material destroyed by each supernova, typically 1000 or 100Msun for diffuse or dense interstellar gas   |   None   |

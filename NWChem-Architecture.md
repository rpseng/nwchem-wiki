# NWChem Architecture

As described in the [Getting Started](Getting-Started.md) section, NWChem consists of independent modules that perform the
various functions of the code. Examples include the input parser,
self-consistent field (SCF) energy, SCF analytic gradient, and density
functional theory (DFT) energy modules. The independent NWChem modules
can share data only through a disk-resident database, which is similar
to the GAMESS-UK dumpfile or the Gaussian checkpoint file. This allows
the modules to share data, or to share access to files containing data.

It is not necessary for the user to be intimately familiar with the
contents of the
[database](#database-structure) in order
to run NWChem. However, a nodding acquaintance with the design of the
code will help in clarifying the logic behind the input requirements,
especially when restarting jobs or performing multiple tasks within one
job.

As detailed in the section describing the ([input file
structure](Getting-Started.md#input-file-structure)), all
start-up directives are processed at the beginning of the job by the
main program, and then the input module is invoked. Each input directive
usually results in one or more entries being made in the database. When
a TASK directive is encountered, control is passed to the appropriate
module, which extracts relevant data from the database and any
associated files. Upon completion of the task, the module will store
significant results in the database, and may also modify other database
entries in order to affect the behavior of subsequent computations.

## Database Structure

Data is shared between modules of NWChem by means of the database. Three
main types of information are stored in the data base: (1) arrays of
data, (2) names of files that contain data, and (3) objects. Arrays are
stored directly in the database, and contain the following information:

1.  the name of the array, which is a string of ASCII characters (e.g.,
    "reference energies")
2.  the type of the data in the array (i.e., real, integer, logical, or
    character)
3.  the number (N) of data items in the array (Note: A scalar is stored
    as an array of unit length.)
4.  the N items of data of the specified type

It is possible to enter data directly into the database using the [SET
directive](SET.md). For example, to store a (64-bit precision)
three-element real array with the name "reference energies" in the
database, the directive is as follows:

`set "reference energies" 0.0 1.0 -76.2`

NWChem determines the data to be real (based on the type of the first
element, 0.0), counts the number of elements in the array, and enters
the array into the database.

Much of the data stored in the database is internally managed by NWChem
and should not be modified by the user. However, other data, including
some NWChem input options, can be freely modified.

Objects are built in the database by storing associated data as multiple
entries, using an internally consistent naming convention. This data is
managed exclusively by the subroutines (or methods) that are associated
with the object. Currently, the code has two main objects: basis sets
and geometries. [GEOMETRY](Geometry.md) and
[BASIS](Basis.md) present a complete discussion of the input to
describe these objects.

As an illustration of what comprises a geometry object, the following
table contains a partial listing of the database contents for a water
molecule geometry named "test geom". Each entry contains the field test
geom, which is the unique name of the object.
```
Contents of RTDB h2o.db  
-----------------------  
Entry                                   Type[nelem]  
---------------------------  ----------------------  
geometry:test geom:efield             double[3]      
geometry:test geom:coords             double[9]      
geometry:test geom:ncenter               int[1]      
geometry:test geom:charges            double[3]      
geometry:test geom:tags                 char[6] 
...
```
Using this convention, multiple instances of objects may be stored with
different names in the same database. For example, if a user needed to
do calculations considering alternative geometries for the water
molecule, an input file could be constructed containing all the
geometries of interest by storing them in the database under different
names.

The runtime database contents for the file h2o.db listed above were
generated from the user-specified input directive,
```
 geometry "test geom"  
   O     0.00000000    0.00000000    0.00000000 
   H     0.00000000    1.43042809   -1.10715266 
   H     0.00000000   -1.43042809   -1.10715266  
 end
```
The [GEOMETRY](Geometry.md) directive allows the user to specify
the coordinates of the atoms (or centers), and identify the geometry
with a unique name.

Unless a specific name is defined for the geometry, (such as the name
`"test geom"` shown in the example), the default name of geometry is
assigned. This is the geometry name that computational modules will look
for when executing a calculation. The SET directive can be used in the
input to force NWChem to look for a geometry with a name other than
geometry. For example, to specify use of the geometry with the name
`"test geom"` in the example above, the SET directive is as follows:
```
set geometry "test geom"
```
NWChem will automatically check for such indirections when loading
geometries. Storage of data associated with basis sets, the other
database resident object, functions in a similar fashion, using the
default name `"ao basis"`.

## Persistence of data and restart

The database is persistent, meaning that all input data and output data
(calculation results) that are not destroyed in the course of execution
are permanently stored. These data are therefore available to subsequent
tasks or jobs. This makes the input for restart jobs very simple, since
only new or changed data must be provided. It also makes the behavior of
successive restart jobs identical to that of multiple tasks within one
job.

Sometimes, however, this persistence is undesirable, and it is necessary
to return an NWChem module to its default behavior by restoring the
database to its input-free state. In such a case, the
[UNSET](UNSET.md) directive can be used to delete all database
entries associated with a given module (including both inputs and
outputs).

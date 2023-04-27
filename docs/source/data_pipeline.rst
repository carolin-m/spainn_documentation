.. _data_pipeline:

Data pipeline
----------------

The data pipeline of **SpaiNN** mainly consists of :py:class:`DatabaseUtils` and involves the creation of a database using the :py:class:`GenerateDB` class or the conversion of existing databases (*e.g.*, SchNarc database) using the :py:class:`ConvertDB` class. 

When generating a database, the following properties are parsed from SHARC output, *i.e.*, ``Qm.in`` and ``Qm.out`` files: energies (:math:`\mathsf{E_{i}}`), forces (:math:`\mathsf{\mathbf{F}_{i}}`), non-adiabatic coupling (:math:`\mathsf{\mathbf{NAC}_{ij}}`), Dipoles (:math:`\mathsf{\mu_{i}}`), and spin-orbit couplings (:math:`\mathsf{\mathbf{SOC}_{ij}}`).
These properties obtain valuable information about the molecules, which are employed in non-adiabatic molecular dynamics simulations using SHARC (see :ref:`main`).
The respective properties are stored in form of an Atomic Simulation Environment (ASE) databse.


By default, a dictionary of units of all properties and the atomic coordinates is added to the database. 
This ensures that the units of properties can be easily converted without requiring internal default units.
The default values are summarized in the following dictionary.

.. code-block:: console
 
  distance_unit = "Bohr"
 
  property_unit_dict = {
            "energy": "Hartree",
            "forces": "Hartree/Bohr",
            "nacs": "1",
            "dipoles": "1",
            "socs": "1",
        }      

Moreover, during the generation of a databse, the total number of electronic states as well as the number of singlet, doublet and triplet states are stored in form of metadata to the **SpaiNN** database.

The following code snippet shows how to generate a database (``iso_butene.db``) from SHARC output stored in the folder ``sample_data``.

>>> import spainn
>>> from spainn.asetools import GenerateDB
>>> datapath = os.path.join(os.getcwd(), 'sample_data')
>>> dbname = os.path.join(datapath, 'iso_butene.db')
>>> genDB = GenerateDB()
>>> genDB.generate(path=datapath, dbname=dbname, smooth_nacs=True)
>>> 
INFO:spainn.asetools.generate_db:Found following state list: ['3']
INFO:spainn.asetools.generate_db:Found following properties: energy forces dipoles nacs
INFO:spainn.asetools.generate_db:Wrote 101 geometries to ./sample_data/iso_butene.db

Conversion of an existing databse, e.g., SchNarc database, into a **SpaiNN** database can be performed using the following code snippet.
Noteworthy, in subsequent steps of the data pipeline expect a dictionary of units (properties and atomic coordinates) and states (number of singlet, doublet, and triplet states).
The following code snippet demonstrates how to add this information during the database conversion:

>>> import spainn
>>> from spainn.asetools import ConvertDB
>>> oldDB = os.path.join(os.getcwd(), 'sample_DB', 'schnarc_trans_butene.db')
>>> newDB = os.path.join(os.getcwd(), 'sample_DB', 'trans_butene.db')
>>> metadata = {
     '_distance_unit': 'Bohr', 
     '_property_unit_dict': {'energy': 'Hartree', 'forces': 'Hartree/Bohr', 'nacs': '1', 'smooth_nacs': '1'},
     'n_singlets': 3, 'n_doublets': 0, 'n_triplets': 0, 
     'phasecorrected': False, 'states': 'S S S'
     }
>>> convDB = ConvertDB()
>>> convDB.convert(olddb=oldDB, newdb=newDB, copy_metadata=False, smooth_nacs=True)
>>> db_new = connect(newDB)
>>> db_new.metadata = metadata
>>>
INFO:spainn.asetools.convert_db:Converting ./sample_DB/schnarc_trans_butene.db into ./sample_DB/trans_butene.db
INFO:spainn.asetools.convert_db: ./sample_DB/schnarc_trans_butene.db has 101 entries
INFO:spainn.asetools.convert_db: ./sample_DB/schnarc_trans_butene.db keys: energy has_forces forces dipoles nacs


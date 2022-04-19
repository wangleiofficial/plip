# Protein-Ligand Interaction Profiler (PLIP)

![PLIP Build](https://github.com/pharmai/plip/workflows/PLIP%20Build/badge.svg)
![GitHub](https://img.shields.io/github/license/pharmai/plip?style=social)
![GitHub All Releases](https://img.shields.io/github/downloads/pharmai/plip/total?style=social)
![Docker Pulls](https://img.shields.io/docker/pulls/pharmai/plip?style=social&logo=docker)
![Docker Image Size (tag)](https://img.shields.io/docker/image-size/pharmai/plip/latest?style=social&logo=docker)

Analyze noncovalent protein-ligand interactions in 3D structures with ease.

<img src="pliplogo.png"  alt="PLIP Logo" height="100">


| Use Case                                                                  | [Web Server](https://plip-tool.biotec.tu-dresden.de)         | Docker             | Singularity        | Python Module       |
|---------------------------------------------------------------------------|--------------------|--------------------|--------------------|--------------------|
| "I want to analyze my protein-ligand complex!"                            | :heavy_check_mark: | :heavy_check_mark: | :yellow_circle:    | :x:                |
| "I want to analyze *a billion* protein-ligand complexes!"                 | :x:                | :yellow_circle:    | :heavy_check_mark: | :yellow_circle:    |
| "I love the Linux command line and want to build a workflow around PLIP!" | :x:                | :heavy_check_mark: | :heavy_check_mark: | :yellow_circle:    |
| "I'm a Python programmer and want to use PLIP in my project!"             | :x:                | :yellow_circle:    | :yellow_circle:    | :heavy_check_mark: |

---

## Quickstart

### Docker
If you have Docker installed, you can run a PLIP analysis for the structure `1vsn` with the following shell command:

On Linux / MacOS:
```bash
$ docker run --rm \
    -v ${PWD}:/results \
    -w /results \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    pharmai/plip:latest -i 1vsn -yv
```

On Windows:
```bash
$ docker run --rm \
    -v ${PWD}:/results \
    -w /results \
    -u $(id -u ${USER}):$(id -g ${USER}) \
    pharmai/plip:latest -i 1vsn -yv
```
### Singularity

The equivalent command for our pre-built [Singularity](https://singularity.lbl.gov/) image for Linux (available under [Releases](https://github.com/pharmai/plip/releases)) is as follows:

```bash
$ ./plip.simg -i 1vsn -yv
```

Singularity allows to use PLIP with ease in HPC environments. Note that you need to have Singularity installed on your base system.

---

## Usage

This README provides instructions for setup and using basic functions of PLIP.
For more details, see the [Documentation](DOCUMENTATION.md).

### 1. Install PLIP

#### Containerized Image (recommended)
:exclamation: We ship PLIP as pre-built containers for multiple architectures (amd64/ARM), available on the [Docker Hub](https://hub.docker.com/r/pharmai/plip) or as pre-built Singularity image under [Releases](https://github.com/pharmai/plip/releases). See the quickstart section above for usage instructions.

#### Dependencies
If you cannot use the containerized bundle or want to use PLIP sources, make sure you have the following requirements installed:
- Python >= 3.6.9
- [OpenBabel](#Installing-OpenBabel) >= 3.0.0 with [Python bindings](https://open-babel.readthedocs.io/en/latest/UseTheLibrary/PythonInstall.html)
- PyMOL >= 2.3.0 with Python bindings (optional, for visualization only)
- ImageMagick >= 6.9 (optional)

**Python:** If you are on a system where Python 3 is executed using `python3` instead of just `python`, replace the `python` and `pip` commands in the following steps with `python3` and `pip3` accordingly.

**OpenBabel:** Many users have trouble setting up OpenBabel with Python bindings correctly. We therefore provide some [installation help for OpenBabel](#Installing-OpenBabel) below.

#### From Source

Open a terminal and clone this repository using
```bash
$ git clone https://github.com/pharmai/plip.git
```

Either set your `PYTHONPATH` environment variable to the root directory of your PLIP repository or run the following command in it

```bash
$ python setup.py install
```

#### Pip install from git repo
The pypi package of OpenBabel 3.1.1.1 was wrongly built, please refer to [issue](https://github.com/openbabel/openbabel/issues/2408#issuecomment-1014582279). You can install PLIP as Python module with:

```bash
$ pip install git+https://github.com/pharmai/plip.git
```

**Note:** Be aware that you still have to install the above mentioned dependencies and link them correctly.

### 2. Run PLIP

#### Command Line Tool

Run the `plipcmd.py` script inside the PLIP folder to detect, report, and visualize interactions. The following example creates a PYMOL visualization for the interactions between the inhibitor [NFT](https://www.rcsb.org/ligand/NFT) and its target protein in the PDB structure [1vsn](https://www.rcsb.org/structure/1VSN).

**Note:** If you have installed PLIP with `python setup.py install` or PyPi, you will not have to set an alias for the `plip` command.

```bash
# Set an alias to make your life easier and create and enter /tmp/1vsn
$ alias plip='python ~/plip/plip/plipcmd.py'
$ mkdir /tmp/1vsn && cd /tmp/1vsn
# Run PLIP for 1vsn and open the resulting visualization in PyMOL
$ plip -i 1vsn -yv
$ pymol 1VSN_NFT_A_283.pse
```

#### Python Module
In your terminal, add the PLIP repository to your `PYTHONPATH` variable. For our example, we also download a PDB file for testing.
```bash
$ export PYTHONPATH=~/plip:${PYTHONPATH}
$ cd /tmp && wget http://files.rcsb.org/download/1EVE.pdb
$ python
```
In python, import the PLIP modules, load a PDB structure and run the analysis.
This small example shows how to print all numbers of residues involved in pi-stacking:

```python
from plip.structure.preparation import PDBComplex

my_mol = PDBComplex()
my_mol.load_pdb('/tmp/1EVE.pdb') # Load the PDB file into PLIP class
print(my_mol) # Shows name of structure and ligand binding sites
my_bsid = 'E20:A:2001' # Unique binding site identifier (HetID:Chain:Position)
my_mol.analyze()
my_interactions = my_mol.interaction_sets[my_bsid] # Contains all interaction data

# Now print numbers of all residues taking part in pi-stacking
print([pistack.resnr for pistack in my_interactions.pistacking]) # Prints [84, 129]
```

### 3. Investigate the Results
PLIP offers various output formats, ranging from renderes images and PyMOL session files to human-readable text files and XML files. By default, all files are deposited in the working directory unless and output path is provided. For a full documentation of running options and output formats, please refer to the [Documentation](DOCUMENTATION.md).

## Versions and Branches
For production environments, you should use the latest tagged commit from the `master` branch or refer to the [Releases](https://github.com/pharmai/plip/releases) page. Newer commits from the `master` and `development` branch may contain new but untested and not documented features.

## Contributors
- Sebastian Salentin (original author) | [github.com/ssalentin](https://github.com/ssalentin)
- Joachim Haupt | [github.com/vjhaupt](https://github.com/vjhaupt)
- Melissa F. Adasme Mora |  [github.com/madasme](https://github.com/madasme)
- Alexandre Mestiashvili | [github.com/mestia](https://github.com/mestia)
- Christoph Leberecht  | [github.com/cleberecht](https://github.com/cleberecht)
- Florian Kaiser  | [github.com/fkaiserbio](https://github.com/fkaiserbio)
- Katja Linnemann | [github.com/kalinni](https://github.com/kalinni)

## PLIP Web Server
Visit our PLIP Web Server on [plip-tool.biotec.tu-dresden.de](https://plip-tool.biotec.tu-dresden.de).

## License Information
PLIP is published under the GNU GPLv2. For more information, please read the `LICENSE.txt` file.
Using PLIP in your commercial or non-commercial project is generally possible when giving a proper reference to this project and the publication in NAR.

## Citation Information
If you are using PLIP in your work, please cite
> Adasme,M. et al. PLIP 2021: expanding the scope of the protein-ligand interaction profiler to DNA and RNA.
> Nucl. Acids Res. (05 May 2021), gkab294. doi: 10.1093/nar/gkab294
 
or

> Salentin,S. et al. PLIP: fully automated protein-ligand interaction profiler.
> Nucl. Acids Res. (1 July 2015) 43 (W1): W443-W447. doi: 10.1093/nar/gkv315

## FAQ
> I try to run PLIP, but I'm getting an error message saying:
> ValueError: [...] is not a recognised Open Babel descriptor type
>
Make sure OpenBabel is correctly installed. This error can occur if the installed Python bindings don't match the OpenBabel version on your machine.
We don't offer technical support for installation of third-party packages but added some [installation help for OpenBabel](#Installing-OpenBabel) below.
Alternatively you can refer to their [website](https://openbabel.org/docs/dev/Installation/install.html).

> I'm unsure on how to run PLIP and don't have much Linux experience.
>
You should consider running PLIP as Docker image, as we describe above.

> PLIP is reporting different interactions on several runs!
>
Due to the non-deterministic nature on how hydrogen atoms can be added to the input structure, it cannot be guaranteed that each run returns exactly the same set of interactions. If you want to make sure to achieve consistent results, you can:

- protonate the input structure once with PLIP or your tool of preference
- run PLIP with `--nohydro`

> How does PLIP handle NMR structures?
>
By default PLIP uses the first model it sees in a PDB file. You can change this behavior with the flag `--model`.

## Installing OpenBabel
As many users encounter problems with installing the required OpenBabel tools, we want to provide some help here. However, we cannot offer technical support. Comprehensive information about the installation of OpenBabel for Windows, Linux, and macOS can be found in the [OpenBabel wiki](http://openbabel.org/wiki/Category:Installation) and the [OpenBabel Docs](https://open-babel.readthedocs.io/en/latest/Installation/install.html).
Information about the installation of [OpenBabel Python bindings](https://open-babel.readthedocs.io/en/latest/UseTheLibrary/PythonInstall.html) can also be found there.

### Using Conda intall OpenBabel and Python bindings

Install OpenBabel using the [binary from GitHub](https://github.com/openbabel/openbabel/releases/latest) or with
```bash
# For Conda users
$ conda install openbabel -c conda-forge
```

**Note:** If you have trouble, make sure the OpenBabel version matches the one for the python bindings!

### Using your Package Manager (Example for Ubuntu 20.04)

```bash
$ apt-get update && apt-get install -y \
    libopenbabel-dev \
    libopenbabel6 \
    python3-openbabel \
    openbabel
```
### From Source (Example for Ubuntu 18.04)
Clone the OpenBabel repository into the /src directory
```bash
$ git clone -b openbabel-3-0-0 \
https://github.com/openbabel/openbabel.git
```

Within /src/openbabel create end enter a directory /build and configure the build using
```bash
$ cmake .. \
-DPYTHON_EXECUTABLE=/usr/bin/python3.6 \
-DPYTHON_BINDINGS=ON \
-DCMAKE_INSTALL_PREFIX=/usr/local \
-DRUN_SWIG=ON
```
From within the same directory (/src/openbabel/build) compile and install using
```bash
$ make -j$(nproc --all) install
```
## Contact / Maintainer
As of April 2020 PLIP is now officially maintained by [PharmAI GmbH](https://pharm.ai). Do you have feature requests, found a bug or want to use  PLIP in your project? Commercial support is available upon request.

 ![](https://www.pharm.ai/wp-content/uploads/2020/04/PharmAI_logo_color_no_slogan_500px.png)

 Please get in touch: `hello@pharm.ai`

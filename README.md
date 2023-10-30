# Artefact readme

**Artefact URL: https://doi.org/10.5281/zenodo.10048157**

This document outlines the necessary steps for using the *BN Classifier* artefact within the TACAS '23 VM. It also lists other instructions necessary to install/build *BN Classifier* from the internet or from the attached source files. The pre-built binaries can be used with almost no network access, but network is necessary for downloading new system dependencies. Network is also required when building from source.

WARNING: In it's default configuration, TACAS'23 VM comes with network access disabled. Make sure you attach a network adapter in the VM settings before starting the VM. Also note that some of the benchmarks may require up to 64GB of RAM. However, we provide a "reduced" set of benchmarks that should be runnable with just 8GB of RAM that is provided by the default VM configuration. Finally, note that the performance within the VM can be limited. At the end of this README, you can find instructions for installing BN Classifier locally (requires only Python; supports macOS, Linux and Windows).

### Artefact structure

First, we briefly describe the function of each file/folder in the artefact. Detailed instructions are then given later in the document.

* `README.md` | This markdown readme.
* `env` | A pre-built Python virtual environment that can be used in the TACAS '23 virtual machine.
* `requirements.txt` | List of Python dependencies used to build the `env` environment.
* `bin` | Pre-built binaries of *BN Classifier* that can be used in the TACAS '23 virtual machine.
* `tutorial` | Simple Jupyter notebook and sample input/output files that demonstrate how to use *BN Classifier*.
* `benchmarks` | All scripts and input files used to produce the performance claims in the tool paper.
* `manual.pdf` | A more detailed manual describing all the functionality of *BN Classifier*.
* `sources` | The source code of the relevant Rust/Python packages. Each folder contains a README with build instructions and references to documentation. You can also find these sources on Github.
  * `sources/hctl-model-checker` | The source code of the "main" Rust library responsible for the model checking.
  * `sources/bn-classifier` | The source code of the *BN classifier*, both the Rust engine responsible for property classification, and a desktop application build using Rust, HTML and JavaScript.
  * `sources/aeon-py` | The source code of the updated AEON.py package which now provides access to the `hctl-model-checker` and `bn-classifier` through Python bindings.

## Set-up the environment (quick test)

Here, we outline the steps necessary to set-up the artefact and verify that all components work as expected.

1. First, download `artefact.zip` and extract the `artefact` folder into the home folder of the virtual machine. Open a terminal window in this folder.

2. Verify that `pwd` returns `/home/tacas23/artefact`. In the following, we assume all commands are executed from this folder. (The pre-built Python virtual environment requires this path, so the `artefact` folder really needs to be placed here. If you want to run the experiments at some other location, see the instructions for recreating the environment at the end of this readme).

3. Install the Python virtual environments package: `sudo apt update` and `sudo apt install python3-venv`.

4. Activate the pre-built Python environment: `source ./env/bin/activate`.

   * If you start a new terminal session, always activate the virtual environment to enable BN Classifier.

5. Test that the Python environment works as expected: `python3 tutorial/quick-start.py`

   This operation can run up to 10 minutes in the VM (running natively should be faster) and you should see an output similar to the following:

   ```
   BooleanNetwork(variables = 18, parameters = 0, regulations = 60)
   Loaded model and properties out of `./tutorial/case-study-mapk/mapk-annotated.aeon`.
   Parsing formulae and generating symbolic representation...
   Successfully parsed all 1 required property (assertion) and all 4 classification properties.
   Successfully encoded model with 18 variables and 4 parameters.
   Model admits 16 instances.
   Evaluating required properties (this may take some time)...
   Required properties successfully evaluated.
   15 instances satisfy all required properties.
   Evaluating classification properties (this may take some time)...
   Classification properties successfully evaluated.
   Generating classification mapping based on model-checking results...
   Results saved to `./tutorial/classification-archive.zip`.
   ```

6. Next, test that the explorer GUI is working as expected: `./bin/hctl-explorer ./tutorial/classification-archive.zip`

   This should open a new explorer window showing a single "Mixed outcomes" node overlayed with a help message. At this point, you can close the window. We will return to the explorer GUI in the tutorial.

7. Finally, we test the jupyter notebook environment: `python3 -m jupyter notebook --no-browser`

   This should start a jupyter notebook environment that you can access through the web browser. You need to find a "Or copy and paste one of these URLs:" message. Then copy the URL under this message and paste it into a browser window (Firefox is installed in the VM). The URL should look like this: `http://localhost:8888/tree?token=SOME_RANDOM_STRING_OF_CHARACTERS`. This will open a jupyter notebook user interface. You can keep this window open, because it will be used in the tutorial.


At this point, we have validated that all components are usable, which concludes the quick test part of the artefact. 

## Tutorial

Next, to continue, please return to the jupyter notebook session that we previously opened in the browser (or open it again). Navigate to the `tutorial` folder and open the `tutorial.ipynb`  notebook. The notebook contains all the relevant instructions on how to use *BN Classifier* as a Python library as well as the explorer GUI.  

>  If you are unfamiliar with Jupyter notebooks, you can execute a code cell by selecting it and pressing `Shift+Enter`. Alternatively, you can also press the "run" button in the top toolbar (using the "play" arrow icon).

## Benchmarks

After you have completed the tutorial, you can also execute the benchmarks. Note that you have already replicated the results of our case study in the tutorial.
We have prepared two bash scripts that you can execute to run (A) a reduced version or (B) a full version of our experiments.
The reduced version involves subsets of benchmarks with lower resource requirements (as discussed below).
The scripts internally run Python procedures, hence you should run them in a terminal where you've activated the Python environment first (`source ./env/bin/activate`). 

Both scripts list the progress on the standard CLI output. That usually involves a message when a new benchmark instance is started/finished being processed. Simultaneously, the scripts produce the results on the fly.
The relevant raw results for each group of benchmarks (e.g., for a classification on fully specified models, or for each parameter scan) will gradually appear in separate `_run_*` sub-directories inside the `benchmarks` directory. For example, the results of a parameter scan on the model `077` will appear in a sub-directory with prefix `_run_models_parameter-scan_077_model-checking-p2`. The name reflects the folder with correponding models and the procedure we evaluate (in this case, model checking of phi2). Each `_run_*` directory will contain:
- `*_out.txt` files for each processed model, with a computation time and short progress summary (appears immediately when a model is evaluated). Each classification summary contains a path to a corresponding results archive.
- `*_times.csv` file with a summary of computation times for all benchmark instances finished so far (updated on the fly)
- `*_agreggated.csv` file with a summary regarding how many benchmark instances finished before certain times (updated after all instances of the set are finished)

To make sure that all unsuccessful benchmark instances are correctly discarded from the result summaries, you can run `python3 benchmarks/helper-scripts/postprocess-mc.py <RESULTS_DIR_PATH>` on directories with model-checking results and `python3 benchmarks/helper-scripts/postprocess-classif.py <RESULTS_DIR_PATH>` on directories with classification results.

The produced results (when running the full version of the benchmarks) can be used to reproduce the claims in the paper.
In Figure 4 in Section 6.2 of the paper, each curve of the plot corresponds to data in one of the `*_aggregated.csv` files that will appear inside the result directories with the following prefixes:
- `_run_models_fully-specified-models_model-checking-pN_*` where N={1,2,3,4} (for model-checking of each property on fully specified models)
- `_run_models_fully-specified-models_classification_*` (for classification on fully specified models)

You can also reproduce the data from Table 2 in Section 6.3 of the paper.
- use times of all 7 benchmark instances that will appear in `_run_models_parameter-scan_PSBNs_*` for the coloured model checking on PSBN models
- each model will have its own parameter scan results in `_run_models_parameter-scan_<MODEL-ID>_*`. You will need the time of the fastest instance of each scan (the first time listed in the corresponding `*_aggregated.csv`). Then follow the methodology described in the paper.

> Note that the computation times on the `TACAS'23 VM` might be slightly higher.

#### A) Executing reduced benchmarks
The reduced version involves:
- model-checking (for all 4 formulae) and classification experiments on a subset of 100 (out of 230) fully specified models with fastest computation times
- parameter scan experiments for the model `077`
- coloured model checking for a subset of 5 (out of 7) PSBN models with fastest computation times and lowest memory requirements
- classification experiments on all 5 PSBNs models involving higher arity function symbols

To run the reduced benchmarks, execute the following commands from the `artefact` directory. This script is expected to take up to `1 hour` (depending on the machine) to successfully finish on the `TACAS'23 VM` with 8GB of RAM.
```
cd benchmarks
bash run-benchmarks-subset.sh
```

#### B) Executing full benchmarks
The full version of the benchmarks involves:
- model-checking (for all 4 formulae) and classification experiments on all 230 fully specified models
- parameter scan experiments for each of the selected 7 PSBN models
- coloured model checking for all 7 selected PSBN models
- classification experiments on all 5 PSBNs models involving higher arity function symbols

Note that we have run all the experiments on on a workstation with a 32-core AMD Threadripper 2990WX CPU and 64GB of RAM. Every experiment was executed in a separate process, and its runtime was measured using the standard UNIX time utility. Due to the high number of considered experiments, we typically executed up to 16 benchmarks in parallel to speed up the evaluation. This is because, in our experience, the memory bandwidth of this particular CPU does not scale sufficiently to all 32 cores for the considered workloads (symbolic algorithms typically perform a lot of sequential random memory accesses, resulting in contention among cores). For each benchmark instance, we consider a timeout of one hour.

To run the full benchmarks, execute the following commands from the `artefact` directory. This script `sequentially` runs thousands of benchmark instances, and it is thus expected to take up to 20 days in total to finish (but the results are produced on the fly, as described above). Also note that some of the benchmarks may require up to 64GB of RAM.
```
cd benchmarks
bash run-benchmarks-all.sh
```

#### Further information regarding benchmarks
All the details regarding the benchmarking process, the models we use, and pre-computed results (and how they correspond to the paper) are provided in `./benchmarks/README.md`.
There are also more examples of how to run experiments for specific benchmark sets or individual models, and how to utilize parallelism during benchmarking.


## Installing locally

These steps can be used to install BN classifier on your own machine instead of the TACAS VM. The remaining steps (quick test, tutorial and benchmarks) should then work using the same instructions as within the VM.

### Create the Python `venv`

First, we create a Python virtual environment. For this, we assume you have Python 3 installed with support for virtual environments (current minimal supported version is Python 3.9). For example, on debian-based linux, this corresponds to system packages `python3`, `python3-pip` and `python3-venv` (i.e. `sudo apt install python3 python3-pip python3-venv`).

Then, navigate to the `artefact` folder, and run the following to create a fresh Python environment with all the dependencies:

1. `rm -r ./env` to delete the pre-build Python environment if it is still present.
2. `python3 -m venv ./env` to create a new blank environment in the `env` folder.
3. Next, run `source ./env/bin/activate` to activate the environment. 
4. Finally, run `pip3 install -r requirements.txt` to install all the relevant dependencies.

**Working with python virtual environments:** Note that in every terminal session where you want to work with BN classifier, you have to run `source ./env/bin/activate` to enable the virtual environment. To deactivate the environment, simply execute the `deactivate` command.

### Install the HCTL explorer app

The BN Classifier GUI (also called HCTL explorer) is available as a desktop application for all three major operating systems. The pre-built installer can be downloaded from the [Github release page](https://github.com/sybila/biodivine-bn-classifier/releases/tag/v0.2.3). Simply pick the type of installer which is suitable for your platform (`dmg` for macOS, `exe`/`msi` for Windows, or `deb`/`AppImage`/`app` for Linux). There should not be any additional steps required.

## Building from source

Building from source is possible, but requires network access to download the necessary dependencies. More detailed instructions for building each component of BN Classifier are given in the README of the corresponding source files. These instructions are tested on Linux, but should work for macOS too. Note that if unsure, each source folder has a github actions directory with a complete build workflow for each OS, hence you can also consult this file to inspect the necessary steps.

For both parts, you will need the Rust compiler installed on your system. Hence first run ``curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` to install Rust (does not require superuser rights; default settings should be sufficient; you can also use other method that is preferred by your linux distro, but some of these only provide a very outdated version of Rust, hence we prefer this official setup script). 

### Building the Python bindings

* Create a Python virtual environment and activate it as described in the "Installing locally" section.
* Go to the `./sources/aeon-py` folder.
* Install `maturin` , a build tool for building Rust-Python bindings: `pip3 install maturin`.
* Run `maturin develop --release --features static-z3`. This will build the Python bindings from source and install them in your current virtual environment.
* You can also use `maturin build --features static-z3` to build a Python `wheel` file that can be installed on other machines.

### Building HCTL explorer

* Go to the `./sources/bn-classifier` folder.
* (Only on Linux) Run `sudo apt-get install -y libgtk-3-dev libwebkit2gtk-4.0-dev librsvg2-dev patchelf` to install necessary system dependencies.
* Run `cargo install tauri-cli` to install the Tauri desktop app builder.
* Finally, run `cargo tauri build`.
* The `hctl-explorer` binary should be available in `src-tauri/target/release/`. Similarly, the `.deb` installation package should be available in `src-tauri/target/release/bundle/deb` (different installation packages will be available depending on your current operating system).




# Session 2

Our goal for today is to become familiar with the MC tutorial Docker container, generate events with `Pythia8`, and analyse and compare the predictions to experimental data using `Rivet`. We begin by generating events with `Pythia8`, analysing them, and comparing its predictions to experimental data using `Rivet`. We then improve our predictions by upgrading the hard scattering component with the help of `MG5_aMC@NLO`. We will go through the material together; the examples will be discussed and demonstrated at the same time. You are encouraged to try the examples immediately, but don't worry if you fall behind. You can always finish all exercises at your own pace after session.

Our goal for today is to become familiar with the MC tutorial Docker container, generate events with `Pythia8`, and analyse and compare the predictions to experimental data using `Rivet`. This document contains basic instructions along with several examples. We will go through the material together; the examples will be discussed and demonstrated at the same time. You are encouraged to try the examples immediately, but don't worry if you fall behind. You can always finish all exercises at your own pace after session. 


## Event generation with `Pythia8`
Let us start with a very simple example. Our goal is to generate some events, inspect them, and save them in a file on hard-drive.

---
### Example 1: First $l^+l^-$ event
Let us start by generating some events with opposite-charge lepton pairs using `Pythia`'s `Python` interface. 

_This example includes a few `Python` code snippets; refer to the [docker_notes.md](docker_notes.md) if you are unsure how to run them._

```python
# This is Python

# Import the Pythia module.
import pythia8

# Create a Pythia object.
pythia = pythia8.Pythia()

# Configure the Pythia object.
pythia.readString("WeakSingleBoson:ffbar2gmZ = on")  # Switch on pp > photon/Z
pythia.readString("23:onMode = off") # Turn off all Z decays ... 
pythia.readString("23:onIfAny = 11 13") # ... except to electrons and muons
pythia.readString("Beams:eCM = 13000.") # Set 13 TeV CM energy.

# Initialize, incoming pp beams are default.
pythia.init()

# Generate an(other) event. Fill event record.
pythia.next()
```

The first few events will be printed to the standard output (subject to the `Next:numberShowEvent` setting), and further events can be printed using `pythia.event.list()`. The event will be saved in `pythia.event`. We can print the number of particles in the event record or iterate over them and print all occurrences of the $Z$ boson, for example:
```python
# This is Python

# Print the number of particles in the event
print(pythia.event.size())

# Iterate over particles in the event
for particle in pythia.event:
   # If the particle is the Z boson
   if (particle.id() == 23):
      # Print a message
      print("found Z boson: {}".format(particle.p()))
```

The event record contains anywhere from a few hundred to a couple of thousand entries. This is because, in addition to the hard event $pp \to \gamma/Z \to l^+ l^-$, `Pythia8` adds initial and final state parton showers (ISR & FSR), multi-parton interactions (MPI), hadronization and hadron decays, beam remnants, etc.. Moreover, the same particle can appear multiple times in the event record at different stages of the collision. To visually verify our settings for the hard interaction, it can be useful to disable the other stages:

```python
# This is Python

# Pass settings to Pythia
pythia.readString("PartonLevel:FSR = off") # disable final-state radiation
pythia.readString("PartonLevel:ISR = off") # disable initial-state radiation
pythia.readString("PartonLevel:MPI = off") # disable multi-parton interactions 
pythia.readString("PartonLevel:Remnants = off") # disable beam remnants
pythia.readString("HadronLevel:all = off") # disable hadronization
pythia.readString("Check:event = off") # disable event checking (required when beam remnants are disabled)
```
Note that after changing these settings, the `pythia` object needs to be re-initialised with `pythia.init()`. Regenerating the event with everything disabled except for the hard scattering yields:
```
 --------  PYTHIA Event Listing  (complete event)  ---------------------------------------------------------------------------------
 
    no         id  name            status     mothers   daughters     colours      p_x        p_y        p_z         e          m 
     0         90  (system)           -11     0     0     0     0     0     0      0.000      0.000      0.000  13000.000  13000.000
     1       2212  (p+)               -12     0     0     3     0     0     0      0.000      0.000   6500.000   6500.000      0.938
     2       2212  (p+)               -12     0     0     4     0     0     0      0.000      0.000  -6500.000   6500.000      0.938
     3         -2  (ubar)             -21     1     0     5     0     0   101      0.000      0.000      0.137      0.137      0.000
     4          2  (u)                -21     2     0     5     0   101     0      0.000      0.000   -213.621    213.621      0.000
     5         23  (Z0)               -22     3     4     6     7     0     0      0.000      0.000   -213.484    213.758     10.812
     6         13  mu-                 23     5     0     0     0     0     0     -1.182      2.385    -13.742     13.998      0.106
     7        -13  mu+                 23     5     0     0     0     0     0      1.182     -2.385   -199.743    199.760      0.106
                                   Charge sum:  0.000           Momentum sum:      0.000      0.000   -213.484    213.758     10.812

 --------  End PYTHIA Event Listing  -----------------------------------------------------------------------------------------------
```

A sample of $n$ events can be generated by repeatedly calling `pythia.next()` $n$ times.

#### Note on $ep$ collisions in `Pythia`
`Pythia` offers a selection of processes in $ep$ collisions, all within the framework of collinear factorisation:
- Deep Inelastic Scattering (DIS), neutral and charge current, including SIDIS
- Photoproducton of jets and heavy flavour
- ...

DIS example:
```python
# This is python

# Import the Pythia module.
import pythia8

# Create a Pythia object.
pythia = pythia8.Pythia()

# Configure the Pythia object.
pythia.readString("Beams:idA = 11") # electron (e-) on one beam
pythia.readString("Beams:idB = 2212") # proton (p) on the other
pythia.readString("Beams:frameType = 2") # fixed energies for both beams
pythia.readString("Beams:eA = 18.") # 18 GeV e- (EIC-like setup)
pythia.readString("Beams:eB = 275.") # 275 GeV p (EIC-like setup)
pythia.readString("WeakBosonExchange:ff2ff(t:gmZ) = on") # enable Neutral Current DIS (gamma*/Z exchange)
pythia.readString("PhaseSpace:Q2Min = 1.0") # setup Q2_min cut

# Initialize
pythia.init()

# Generate an(other) event. Fill event record.
pythia.next()
```

Going beyond collinear factorisaion is possible: `Cascade3` MCEG implements hard scattering and parton shower within the TMD framework and can hand over to `Pythia` for non-perturbative effects.

---
### Example 2: $l^+l^-$ event sample

The `Python` interface is useful for exploring `Pythia8`'s capabilities. However, if we want to pass the generated events along, we need to switch to its default operating mode: compiled code linked against `libpythia8`.

_This example includes a couple of `C++` snipets. Instructions on how to compile and link these against `libpythia8` and `HepMC3` are provided in [docker_notes.md](docker_notes.md)._

The following code snippet demonstrates a `C++` program that configures `Pythia8` in the same way as Example 1. Instead of generating just one event, this program generates one hundred events and writes them to a file named `DY.hepmc` in `HepMC3` format.

```c++
// This is C++

#include "Pythia8/Pythia.h"
#include "Pythia8Plugins/HepMC3.h"
using namespace Pythia8;

int main() {

  // Interface for conversion from Pythia8::Event to HepMC event.
  // Specify file where HepMC events will be stored.
  Pythia8ToHepMC toHepMC("DY.hepmc");

  // Generator. Process selection. LHC initialization.
  Pythia pythia;
  pythia.readString("WeakSingleBoson:ffbar2gmZ = on");  // Switch on pp > photon/Z
  pythia.readString("23:onMode = off"); // Turn off all Z decays ...
  pythia.readString("23:onIfAny = 11 13"); // ... except to electrons and muons
  pythia.readString("Beams:eCM = 13000."); // Set 13 TeV CM energy.

  // If Pythia fails to initialize, exit with error.
  if (!pythia.init()) return 1;

  // Begin event loop. Generate event. Skip if error.
  for (int iEvent = 0; iEvent < 100; ++iEvent) {
    if (!pythia.next()) continue;

    // Construct new empty HepMC event, fill it and write it out.
    toHepMC.writeNextEvent( pythia );

  // End of event loop. Statistics.
  }
  pythia.stat();

  // Done.
  return 0;
}
```
The content of `DY.hepmc` is human readable and I invite you to look at it. 
```bash
# This is shell

# Inspect the DY.hepmc file
dexec less DY.hepmc
```

_If you want to learn more `HepMC3` file format, have a look in the Appendix._

---
### Example 3: Fancier version of Example 2
The program from Example 2 can be further improved:
1. Instead of passing commands to `Pythia8` using the `readString` method, you can also use a command file, which is parsed with the `readFile` method.
   ```c++
   // This is C++
   pythia.readFile("<filename>.cmnd");
   ```
   where the content of the `<filename>.cmnd` file would be for example:
   ```
   WeakSingleBoson:ffbar2gmZ = on ! Switch on pp > photon/Z
   23:onMode = off ! Turn off all Z decays ...
   23:onIfAny = 11 13 ! ... except to electrons and muons
   Beams:eCM = 13000 ! Set 13 TeV CM energy.
   ```

2. The name of the output file, and optionally also the _command file_, could be passed as command line arguments. For example
   ```c++
   // This is C++
   int main(int argc, char* argv[]) {

     // Check that correct number of command-line arguments
     if (argc != 2) {
       cerr << " Unexpected number of command-line arguments. \n You are"
            << " expected to provide one output file name. \n"
            << " Program stopped! " << endl;
       return 1;
     }
     // Confirm that external files will be used for input and output.
     cout << " <<< \n >>> HepMC events will be written to file "
          << argv[1] << " <<< \n" << endl;

     // Interface for conversion from Pythia8::Event to HepMC event.
     // Specify file where HepMC events will be stored.
     Pythia8ToHepMC toHepMC(argv[1]);
   ```
---

## Analysing events with `Rivet`

`Rivet` provides a framework for data analysis over an event sample. It offers methods to tag particles of interest, apply cuts, calculate kinematic observables, and create histograms. This makes it a universal tool for bridging theoretical and experimental communities. Notably, `Rivet` is actively used: many LHC experiments encourage their analysis teams to implement the event selection used in measurements within `Rivet` and even provide the relevant data. Consequently, theorists primarily need to focus on providing event samples.

A list of all available analyses can be found on the [`Rivet` website](https://rivet.hepforge.org/analyses/). The following analyses will be useful in the remainder of this tutorial:
- `CMS_2019_I1753680`: measurements of the differential cross sections for $Z$ bosons produced in proton-proton collisions at $\sqrt{s}=13$ TeV, decaying to muons and electrons. The data analysed were collected in 2016 with the CMS detector at the LHC and correspond to an integrated luminosity of 35.9/fb. It provides differential cross sections for the dilepton pair, most notably the transverse momentum ($p_t$) and the rapidity ($y$).
- `CMS_2016_I1413748`: measurements of the top quark-antiquark spin correlation and the top quark polarization for top pairs produced in proton-proton collisions at $\sqrt{s}=8$ TeV. The data correspond to an integrated luminosity of 19.5/fb
collected with the CMS detector at the LHC. The measurements are performed using events with two oppositely charged leptons (electrons or muons) and two or more jets, where at least one of the jets is identified as originating from a bottom quark. The spin correlations and polarization are extracted from the angular distributions of the two selected leptons, both inclusively and differentially, with respect to the invariant mass, rapidity, and transverse momentum of the $t\bar{t}$ system.

---
### Example 4: Analysing $l^+l^-$ event sample with `Rivet`
The `.hepmc` file generated in the previous example can be directly used as input for `Rivet`:
```bash
# This is shell

# Launch the analysis 
dexec rivet --analysis=CMS_2019_I1753680 DY.hepmc
```
Here, the `--analysis` option specifies which analysis to apply to the event sample. Running this command will create a `Rivet.yoda` file, which contains the histograms of the observables defined in the specified analysis.

Note that generating only 100 events will not be sufficient to construct a reliable differential distribution, as the statistical uncertainty in each bin is proportional to $1/\sqrt{N}$, where $N$ is the number of entries in that bin. To obtain meaningful results, Example 2 should be modified to generate a larger event sample.

The results stored in the `.yoda` file can be plotted using the following command:
```bash
# This is shell

# Do the plots
dexec rivet-mkhtml-tex --err Rivet.yoda
```
This command generates a `rivet-plots` directory containing all the plots, along with a simple HTML website that compiles them in `index.html`. You can open this file in your favourite browser. The `--err` switch switches on the plotting of the MC errorbars (which is invaluable for samples with low stats). 

Here’s what you should see:
![A screenshot of the index.html webpage generated by Rivet.](pics/rivetScreenshot.png)

#### Note on $ep$ collisions in `Rivet`
`Rivet` is collision agnostic. As of today it provides 43 HERA analyses (including comparison to data). It focuses on published measurements, so far no future measurements like EIC, but a lot of examples to draw inspiration from. 

---

### Customizing `Rivet` Plots

1. __Modifying Plotting Styles__: You can customize the plotting style (e.g., line style, legend label) by adding colon-separated modifiers after the `.yoda` file name. For example:
   ```bash
   # This is shell

   # Do the plots
   dexec rivet-mkhtml-tex Rivet.yoda:"LineColor=blue:LineWidth=0.05:Label=Pythia8"
   ```
   For more details, refer to the [`plotting.md`](https://gitlab.com/hepcedar/rivet/-/blob/rivet-4.1.2/doc/tutorials/plotting.md) file from the [`Rivet` tutorials directory](https://gitlab.com/hepcedar/rivet/-/blob/rivet-4.1.2/doc/tutorials).

2. __Configuring Plotting Area Settings__: You can adjust plotting area settings (e.g., sizes, limits, axis labels) using a configuration file:
   ```bash
   # This is shell

   # Do the plots
   dexec rivet-mkhtml-tex -c plot.conf Rivet.yoda
   ```
   In the example below we adjust the ratio plot $y$-axis limits using the `plot.conf` file:
   ```
   # BEGIN PLOT .*
   RatioPlotYMin=0
   RatioPlotYMax=2
   # END PLOT
   ```
   More information is available in the [`makeplots.md`](https://gitlab.com/hepcedar/rivet/-/blob/rivet-4.1.2/doc/tutorials/makeplots.md) file from the [`Rivet` tutorials directory](https://gitlab.com/hepcedar/rivet/-/blob/rivet-4.1.2/doc/tutorials).

3. __Using Matplotlib for Plotting__: `Rivet` also supports plotting with `matplotlib` in `Python` using the `rivet-mkhtml` command. This is however, not supported by the container.

---
### Example 5: Using FIFO pip in Examples 2 and 4
When showering events with MPI and hadronization enabled, the event records can become very large. For instance, the sample of 100k events from Example 2 occupies about 18GB of hard drive space. If you don't need to keep the full output but only need to pass it to an analysis program on-the-fly, using a `FIFO` pipe is recommended.

Here’s how you can set up a `FIFO` pipe to handle this:
```bash
# This is shell
# Setup a FIFO pipe
mkfifo DY.hepmc
# Launch the generation of the events 
dexec ./mysource &
# Launch the analysis of the events
dexec rivet --analysis=CMS_2019_I1753680 DY.hepmc
```
In this setup, `mysource` is the executable created in Example 2. The event records are written to a `FIFO` pipe named `DY.hepmc`. Data generated by `mysource` is buffered temporarily in the `FIFO` pipe until it is read by `rivet`. Once the data is read, it is removed from the buffer, allowing `mysource` to continue generating new data. This approach ensures a smooth data flow between the programs without using excessive hard drive space.

## Appendix

Here we provide more information on topics we may have assumed that you already knew including a couple of useful references. 

### PDG IDs

The Particle Data Group (PDG) assigns unique identifiers, known as PDG IDs, to all elementary particles and many composite particles. These IDs are standardized numerical codes used in particle physics to unambiguously identify particles in simulations, experiments, and theoretical studies. PDG IDs are widely used in particle physics software.

The PDG ID is a signed integer that encodes the particle’s properties, such as its type (quark, lepton, boson), charge, and generation. For example, positive values are typically assigned to particles, while negative values denote their corresponding antiparticles.

Here is a list of PDG IDs for Standard Model Particles

| Particle              | Symbol     | PDG ID |
|-----------------------|------------|--------|
| Down quark            | $d$        | 1      |
| Up quark              | $u$        | 2      |
| Strange quark         | $s$        | 3      |
| Charm quark           | $c$        | 4      |
| Bottom quark          | $b$        | 5      |
| Top quark             | $t$        | 6      |
| Electron              | $e^-$      | 11     |
| Electron neutrino     | $\nu_e$    | 12     |
| Muon                  | $\mu^-$    | 13     |
| Muon neutrino         | $\nu_\mu$  | 14     |
| Tau                   | $\tau^-$   | 15     |
| Tau neutrino          | $\nu_\tau$ | 16     |
| Gluon                 | $g$        | 21     |
| Photon                | $\gamma$   | 22     |
| Z boson               | $Z$        | 23     |
| W boson (positive)    | $W^+$      | 24     |
| Higgs boson           | $H$        | 25     |

#### Examples

- **Electron (PDG ID = 11):** The electron is a negatively charged lepton. Its PDG ID is 11, and its antiparticle, the positron, has a PDG ID of -11.
- **Top quark (PDG ID = 6):** The top quark is the heaviest quark in the Standard Model, and its PDG ID is 6. Its antiparticle has a PDG ID of -6.

### `Pythia` (excerpt from the [Pythia 8.3 worksheet](https://pythia.org/pdfdoc/worksheet8312.pdf))

The `Pythia 8.3` program is a standard tool for the generation of high-energy collisions (specifically, it focuses on centre-of-mass energies greater than about 10 GeV), comprising a coherent set of physics models for the evolution from a few-body high-energy ("hard") scattering process to a complex multihadronic final state. The particles are produced in vacuum. Simulation of the interaction of the produced particles with detector material is not included in Pythia but can, if needed, be done by interfacing to external detector-simulation codes.
The `Pythia 8.3` code package contains a library of hard interactions and models for initial- and final-state parton showers, multiple parton-parton interactions, beam remnants, string fragmentation and particle decays. It also has a set of utilities and interfaces to external programs.

### `Rivet` (excerpt from the [Rivet website](https://rivet.hepforge.org/))

The `Rivet` toolkit (Robust Independent Validation of Experiment and Theory) is a system for validation of Monte Carlo event generators. It provides a large (and ever growing) set of experimental analyses useful for MC generator development, validation, and tuning, as well as a convenient infrastructure for adding your own analyses.

`Rivet` is the most widespread way by which analysis code from the LHC and other high-energy collider experiments is preserved for comparison to and development of future theory models. It is used by phenomenologists, MC generator developers, and experimentalists on the LHC and other facilities.

### `HepMC3` (excerpt from from the [HepMC3 website](https://gitlab.cern.ch/hepmc/HepMC3))

`HepMC3` is a new version of the `HepMC` event record. It uses shared pointers for in-memory navigation and the POD concept for persistency.

`HepMC` is a data format and corresponding software library designed to store and manipulate event records in high-energy physics simulations. It is widely used in particle physics for representing the events generated by MC event generators, such as Pythia, Herwig, and MadGraph.

The first few lines of the `DY.hepmc` file from Example 2, but generated with only hard scattering enabled, look like this:
```
HepMC::Version 3.02.07
HepMC::Asciiv3-START_EVENT_LISTING
W Weight
E 0 4 7
U GEV MM
W 1.0000000000000000000000e+00
A 0 GenCrossSection 9.28704901e+03 9.28704901e+03 -1 -1
A 0 GenPdfInfo -2 2 2.10456285e-05 3.28648051e-02 1.08115966e+01 2.73311568e+00 5.91053583e-01 0 0
A 0 alphaQCD 0.196444505005227
A 0 alphaQED 0.0076330162100602
A 0 event_scale 10.8115965996342
A 2 flow1 0
A 4 flow1 101
A 2 flow2 101
A 4 flow2 0
A 0 signal_process_id 221
P 1 0 2212 0.0000000000000000e+00 0.0000000000000000e+00 6.4999999322807234e+03 6.5000000000000000e+03 9.3827000000000005e-01 4
P 2 1 -2 0.0000000000000000e+00 0.0000000000000000e+00 1.3679658520103094e-01 1.3679658520103094e-01 0.0000000000000000e+00 21
P 3 0 2212 0.0000000000000000e+00 0.0000000000000000e+00 -6.4999999322807234e+03 6.5000000000000000e+03 9.3827000000000005e-01 4
P 4 3 2 0.0000000000000000e+00 0.0000000000000000e+00 -2.1362123342012433e+02 2.1362123342012433e+02 0.0000000000000000e+00 21
V -3 0 [2,4]
P 5 -3 23 0.0000000000000000e+00 0.0000000000000000e+00 -2.1348443683492329e+02 2.1375803000532534e+02 1.0811596599634182e+01 22
V -4 0 [5] @ 0.0000000000000000e+00 0.0000000000000000e+00 -1.3793324689150972e-12 1.3811001665927541e-12
P 6 -4 13 -1.1816265629138398e+00 2.3852070816818842e+00 -1.3741902648092829e+01 1.3997732194250242e+01 1.0566000000000000e-01 1
P 7 -4 -13 1.1816265629138398e+00 -2.3852070816818842e+00 -1.9974253418682008e+02 1.9976029781106473e+02 1.0566000000000000e-01 
```
Here, `E 0` indicates the beginning of the first event (numbered from 0), followed by the number of vertices (plus 2) and the number of particles. Particle entries begin with `P` and vertex entries with `V`, each followed by an index. Each particle record contains information about the particle, such as the PDG ID and momentum, while each vertex record lists the incoming particles in square brackets.


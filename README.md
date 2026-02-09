# IITB-CNFS-CTEQ Monte Carlo Tutorial, February 2026

The purpose of this tutorial is to introduce essential tools for making predictions at collider experiments (EIC, LHC, ...). Through a series of simple examples, we will cover the basics of Monte Carlo (MC) event generators, including how to generate events, store them, and analyse the results using a standard analysis toolchain. 

## Session 1 - Monday, 09.02.2026

We opened with a compact introduction to the basic components of MC event generators (see the slides). We then briefly explained the idea behind the essential technical prerequisite: `Docker`. 
Throughout the tutorial, `Docker` containers provide all the required software, which would otherwise be challenging and time-consuming to install on your own host systems. 
It is therefore essential that everyone is familiar with how to operate them. We continue with a hands-on introduction to using `Docker`: [docker_notes.md](docker_notes.md).

## Session 2 - Tuesday, 10.02.20206

We begin by generating events with `Pythia8`, analysing them, and comparing its predictions to experimental data using `Rivet`. We then improve our predictions by upgrading the hard scattering component with the help of `MG5_aMC@NLO`. The learning materials for this session can be found [here](session2.md). This document contains basic instructions along with several examples. We will go through the material together; the examples will be discussed and demonstrated at the same time. You are encouraged to try the examples immediately, but don't worry if you fall behind. You can always finish all exercises at your own pace after session. 

Note that [docker-notes.md](docker_notes.md) now contains extra material relevant for Today's session.

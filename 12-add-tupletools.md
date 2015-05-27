---
layout: page
title: First steps in LHCb
subtitle: TupleTools and branches
minutes: 15
---

> ## Learning Objectives {.objectives}
>
> * Add extra TupleTools to the default DecayTreeTuple
> * Configure the extra TupleTools
> * Use branches
> * Find useful TupleTools

Usually, the default information stored by `DecayTreeTuple` as shown in our [minimal DaVinci job](09-minimal-dv-job.html) is not enough for physics analysis. 
Fortunately, most of the information we need can be added by adding C++ tools (known as `TupleTools`) to `dtt`;
there is an extensive library of these, some of which will be briefly discussed during the lesson.

> ## Default DecayTreeTuple tools {.callout}
> The default tools added in `DecayTreeTuple` are:
>
>  - `TupleToolKinematic`, which fills the kinematic information of the decay.
>  - `TupleToolPid`, which stores DLL and PID information of the particle.
>  - `TupleToolANNPID`, which stores the new NeuralNet-based PID information of the particle.
>  - `TupleToolGeometry`, which stores the geometrical variables (IP, vertex position, etc) of the particle.
>  - `TupleToolEventInfo`, which stores general information (event number, run number, GPS time, etc) of the event.

In order to add `TupleTools` to `dtt`, we have to use the `addTupleTool` method of `DecayTreeTuple` (only available when we have `from DecayTreeTuple.Configuration import *` in our script).
This method instantiates the tool, adds it to the list of tools to execute and returns it.
For example, if we want to fill the tracking information of our particles, we can add the `TupleToolTrackInfo` tool in the following way:

```python
track_tool = dtt.addTupleTool('TupleToolTrackInfo')
```

Some tools can be configured. For example, if we wanted further information from the tracks, such as the number of degrees of freedom of the track fit, we would have to turn on the verbose mode of the tool:

```python
track_tool.Verbose = True
```

If we don't need to configure the tool or we want to use the defaults, there's no need for storing the returned tool in a variable.
For example, if we wanted the information of the PV associated to our particle, we could just add the `TupleToolPrimaries` with no further configuration:

```python
dtt.addTupleTool('TupleToolPrimaries')
```

The way the `DecayTreeTuple.Decay` is written in in our [minimal DaVinci job](09-minimal-dv-job.html), 

```python
dtt.Decay = '[D*(2010)+ -> (D0 -> K- pi+) pi+]CC'
```

means that the configured `TupleTools` will only run on the head of the decay chain, that is, the D*.
In order to select the particles for which we want the information stored, we need to mark them with a `^` symbol in the decay descriptor.
For example, if we want to fill the information of the D0 and its children, we would modify the `dtt` to look like this:

```python
dtt.Decay = '[D*(2010)+ -> ^(D0 -> ^K- ^pi+) pi+]CC'
```

This will run all the configured `TupleTools` on the marked particles, with the caveat that some tools are only run on certain types of particles (eg, tracking tools on particles that have an associated track).
This configuration is not optimal, since there may be tools which we only want to run on the D's and some only on the children. Enter `Branches`, which allow us to specify which tools get applied to which particle in the decay (in addition to the `TupleTools` configured at the top level).

Branches let you define custom namespaces in your ntuple by means of a `dict`.
Its keys define the name of each branch (and, as a consequence, the prefix of the corresponding leaves in the ntuple), while the corresponding values are decay descriptors that specify which particles you want to include in the branch.

```python
dtt.addBranches({'Dstar' : '^[D*(2010)+ -> (D0 -> K- pi+) pi+]CC',
                 'D0'    : '[D*(2010)+ -> ^(D0 -> K- pi+) pi+]CC',
                 'Kminus': '[D*(2010)+ -> (D0 -> ^K- pi+) pi+]CC',
                 'piplus': '[D*(2010)+ -> (D0 -> K- ^pi+) pi+]CC',
                 'pisoft': '[D*(2010)+ -> (D0 -> K- pi+) ^pi+]CC'})
```

Once the branches have been configured, they can be accessed as `dtt.PARTICLENAME`and `TupleTools` can be added as discussed before.
For example, if we want to store the proper time information of the D0, we would do

```python
dtt.D0.addTupleTool('TupleToolPropertime')
```

The usage of `Branches` is very important (and strongly encouraged) to keep the size of your ntuples small, since it prevents us from storing unneeded information (for example trigger information, which will be discussed at a later lesson).

> ## Where to find TupleTools {.callout}
> One of the most difficult things is to know which tool we need to add to our `DecayTreeTuple` in order to get the information we want.
> For this, it is necessary to know where to find `TupleTools` and their code.
> `TupleTools` are spread in 9 packages under `Analysis/Phys` (see the head of `svn` [here](https://svnweb.cern.ch/trac/lhcb/browser/Analysis/trunk/Phys)), all starting with the prefix `DecayTreeTuple`, according to the type of information they fill in our ntuple:
>
> - `DecayTreeTuple` for the more general tools.
> - `DecayTreeTupleANNPID` for the NeuralNet-based PID tools.
> - `DecayTreeTupleDalitz` for Dalitz analysis.
> - `DecayTreeTupleJets` for obtaining information on jets.
> - `DecayTreeTupleMC` gives us access to MC-level information.
> - `DecayTreeTupleMuonCalib` for muon calibration tools.
> - `DecayTreeTupleReco` for reconstruction-level information, such as `TupleToolTrackInfo`.
> - `DecayTreeTupleTracking` for more detailed tools regarding tracking.
> - `DecayTreeTupleTrigger` for accessing to the trigger information of the candidates.
>
> The `TupleTools` are placed in the `src` folder within each package and it's usually easy to get what they do just by looking at their name.
> However, the best way to know what a tool does is check its documentation, either by opening its `.h` file or be searching for it in the latest [`doxygen`](http://lhcb-release-area.web.cern.ch/LHCb-release-area/DOC/davinci/releases/latest/doxygen/index.html).
> Most tools are very well documented and will also inform you of their configuration options.
> As an example, to get the information on the `TupleToolTrackInfo` we used before we could either check its [source code](https://svnweb.cern.ch/trac/lhcb/browser/Analysis/trunk/Phys/DecayTreeTupleReco/src/TupleToolTrackInfo.h) or its [web documentation](http://lhcb-release-area.web.cern.ch/LHCb-release-area/DOC/analysis/releases/latest/doxygen/da/ddd/class_tuple_tool_track_info.html).
> In case we need more information or need to know *exactly* what the code does, the `fill` method is the one we need to look at.

The updated options `ntuple_options.py`, which can be found [here](./code/12-add_tupletools/ntuple_options.py) can be run in the same way as in the [minimal DaVinci job](09-minimal-dv-job.html) lesson.
We will obtain a `DVntuple.root` file, which we can open and inspect with `ROOT`'s `TBrowser`:

```
$ root DVntuple.root
root [0]
Attaching file DVntuple.root as _file0...
root [1] TBrowser *b = new TBrowser()
root [2]
```

Now you can try to locate the branches we have added, which are placed in the `TupleDstToD0pi_D0ToKpi/DecayTree`, and plot some distributions by double-clicking the leaves.
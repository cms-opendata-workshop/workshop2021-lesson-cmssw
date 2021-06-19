---
title: "The Configuration"
teaching: 20
exercises: 30
questions:
- "What are the key elements of a CMSSW configuration file?"
- "What kind of elements can I control at the configuration level?"
objectives:
- "Learn the basic structure of the Python implementation of a CMSSW configuration file."
- "Learn how to modify a config file in order to change parameters and/or run additional code."
keypoints:
- "The Python implementation of the configuration of CMSSW is fundamentally modular."
- "The CMSSW configuration allows for the usage of already available code and/or make changes to yours without having to recompile."
---

## CMSSW configuration Framework

The CMS software framework uses a “software bus” model.  A single executable, `cmsRun`, is used, and the modules are loaded at runtime.  A configuration file, fully written in Python, defines which modules are loaded, in which order they are run, and with which configurable parameters they are run. Note that this is not an interactive system. The entire configuration is defined once, at the beginning of the job, and cannot be changed during running.  This is the file that you "feed" `cmRun` when it is executed.

## Playing with the demoanalyzer_cfg.py file

In the case of our `DemoAnalyzer` we have been working with, its configuration file is the `demoanalyzer_cfg.py`, which resides in the `Demo/DemoAnalyzer/` directory of your CMSSW `Demo` package.  Note that it does not reside in the `Demo/DemoAnalyzer/python` directory (which is usually the case for python configuration files).  This does not really matter though.  There is a specific situation where this is important, and we will look at it later.

Meanwhile, if we explore what is in the `Demo/DemoAnalyzer/python` directory:

~~~
ls Demo/DemoAnalyzer/python
~~~
{: .language-bash}

we will get:

~~~
__init__.py  __init__.pyc  demoanalyzer_cfi.py  demoanalyzer_cfi.pyc
~~~
{: .output}

You will note that there is a `demoanalyzer_cfi.py` in there.  We will not pay attention to this file now, but it instructive to point out that the `_cfg` and `_cfi` descriptors are meaningful.  While the former one defines a top level configuration, the latter works more like a *module initialization* file.  There are also `_cff` files which bear pieces of configuration and so they are dubbed *config fragments*.  You can read a bit more about it in [this subsection](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookConfigFileIntro#PythonConfigExamples) of the Workbook.

Ok, so the file we will play around with is just the `Demo/DemoAnalyzer/demoanalyzer_cfg.py`.  Let's take a look with `nano` or `vi` ([here](https://www.thegeekdiary.com/basic-vi-commands-cheat-sheet/) you can find a good cheatsheet for that editor):

~~~
vi Demo/DemoAnalyzer/demoanalyzer_cfg.py
~~~
{: .language-bash}

The first instructions that you will find in all the top level CMSSW config files are the lines

~~~
import FWCore.ParameterSet.Config as cms

process = cms.Process("Demo")
~~~
{: .language-python}

The first line imports our CMS-specific Python classes and functions, and the second one creates a **process object**.  This refers to a CMSSW process (the one we will be configuring, of course).  Essentially, the main idea is that we will be *feeding* our process all the tasks that we need done by the CMSSW software or its plugins.  The process needs always a name. It could be any short word, but it is usually chosen so it is meaningful.  For instance, if the main task would be to process the high level trigger information, an adequate name will be "HLT"; if the process is actually the full reconstruction of the data, then it is most likely assigned the name "RECO".  For our demo, we will leave our creativity aside and just call it "Demo" (you can, of course, change it to your linking).

Then, you will notice a line that *loads* something:

~~~
process.load("FWCore.MessageService.MessageLogger_cfi")
~~~
{: .language-python}

Actually, because of the `_cfi` tag, we know it is presumably a piece of Python code that initializes some module.  Indeed, it is the *MessageLogger* service.  As the name describes, it controls how the message logging is handled during the job execution.  The string ``"FWCore.MessageService.MessageLogger_cfi"`` tells you exactly where to look for it on [Github](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/FWCore/MessageService/python/MessageLogger_cfi.py) if you needed it.  Note the structure matches the repository's except that the `python` directory name is always omitted when loading modules this way (this is why it is often important to put config files in the python directory).

There is a whole [Workbook section](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideMessageLogger) regarding this module, but let's just look at a simple example.

> ## Changing the logging frequency in our CMSSW job
>
> Suppose you want the Framework to report every 5 events instead of each event.  Then one can simply add this line
>
> `process.MessageLogger.cerr.FwkReport.reportEvery = 5`
>
> right below the *load* line:
>
> ~~~
process.load("FWCore.MessageService.MessageLogger_cfi")
process.MessageLogger.cerr.FwkReport.reportEvery = 5
> ~~~
> {: .language-python}
{: .challenge}

Note that all we are doing is loading the MessageLogger module and changing just one parameter, the one in [this line](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/FWCore/MessageService/python/MessageLogger_cfi.py#L29), instead of going with the default value, which is one.

For the next line

~~~
process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(10) )
~~~
{: .language-python}

it is easy to guess that it controls the number of events that are going to be processed in our CMSSW job.  It is worth noting that `maxEvents` is a *untracked* variable within the Framework.  In general, the system keeps track of what parameters are used to create each data item in the Event and saves this information in the output files. This can be used later to help understand how the data was made. However, sometimes a parameter will have no effect on the final objects created. Such parameters are declared *untracked*.  More information can be found in the [Workbook](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Parameters).

Let's change the number of events to `100`:

~~~
process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(100) )
~~~
{: .language-python}

Next, there is the first module (also an object by itself) we are attaching it to our process object:

~~~
process.source = cms.Source("PoolSource",
    # replace 'myfile.root' with the source file you want to use
    fileNames = cms.untracked.vstring(
    #    'file:myfile.root'
        'root://eospublic.cern.ch//eos/opendata/cms/Run2011A/ElectronHad/AOD/12Oct2013-v1/20001/001F9231-F141-E311-8F76-003048F00942.root'
    )
)
~~~
{: .language-python}


Inside the process object there must be exactly one object assigned that has Python type `Source` and is used for data input. There may be zero or more objects for each of many other Python types. In the official production configurations there can be hundreds or even thousands of objects attached to the process. Your job is configured by your choice of objects to construct and attach to the process, and by the configuration of each object. (This may be done via [import statements](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#TheImportStatment) or calls to the load function, instead of or in addition to object construction.) Some of the Python types that may be used to create these objects are listed in the [Workbook](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Attribute_Declarations_for_the_P).

> ## Explore the PoolSource C++
>
> Generally, all modules in our python configuration are associated with its corresponding C++ code.  By searching the [CMSSW Github repository](https://github.com/cms-sw/cmssw), will you be able to point exactly to the line of C++ code where the `fileNames` parameter is read?
>
> > ## solution
> >
> > You will find that the label first appearing in a Python CMSSW module is actually the name of the C++ code.  So, we would expect that there be a class `PoolSource.h` (and perhaps its implementation as `PoolSource.C`) associated with the "PoolSource" label.  Let's then go to the [Github CMSSW repository](https://github.com/cms-sw/cmssw) and simply search for `PoolSource.C` using the search field of that page.  Immediately, in the [search results](https://github.com/cms-sw/cmssw/search?q=PoolSource.C&unscoped_q=PoolSource.C), we notice there is a `IOPool/Input/src/PoolSource.cc` file that we can browse.  After looking for the variable `fileNames`, we find that this parameter is read in [this line](https://github.com/cms-sw/cmssw/blob/abc9134327fff8fbb378b3c81a56c5850195a9bb/IOPool/Input/src/PoolSource.cc#L68).  Note that it is in the constructor of the object where the `ParameterSet` objects are read.
> {: .solution}
{: .challenge}

Note also that the `fileNames` variable is a `vstring`, i.e., a vector of strings in the C++ sense.  In Python, it is a list, so you can very well input a comma separated list of files.  There is a drawback, though.  In general, our open datasets will contain more than 255 files, which is the limit for the number of arguments a Python function can take, so very long vstrings cannot be created in one step.  There are [various alternatives](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuidePythonTips#Running_on_more_than_255_files) to circumvent this problem.  However, in our workshop, we will use the [FileUtils](https://github.com/cms-sw/cmssw/blob/master/FWCore/Utilities/python/FileUtils.py) module to load all the required files.

## Conditions Data

Let's keep exploring the `demoanalyzer_cfg.py` config file.  The next few lines

~~~
#needed to cache the conditions data
process.load('Configuration.StandardSequences.FrontierConditions_GlobalTag_cff')
#process.GlobalTag.connect = cms.string('sqlite_file:/cvmfs/cms-opendata-conddb.cern.ch/FT_53_LV5_AN1_RUNA.db')
process.GlobalTag.globaltag = 'FT_53_LV5_AN1::All'
~~~
{: .language-python}

have to do with being able to read CMSSW database information.  We call this the [Conditions Data](http://opendata.cern.ch/docs/cms-guide-for-condition-database) as we may find values for calibration, alignment, trigger info, etc., in these database snapshots.  One can think of the `GlobalTag` as a label that contains a set of database snapshots that need to be adequate for a point in time in the history of the CMS detector.  For 2011/2012 open data release, the global tag is `FT_53_LV5_AN1` (the `::All` string is a flag that tells the frameworks to read *All* the information associated with the tag).  

The `connect` variable just modifies they way in which the framework is going to access these snapshots. You may notice that, if you are using the Docker environment, this line is commented out (like above), while if you are using the Virtual Machine it is not commented out.  This is because when using VMs, wee read these conditions from the shared files system area at CERN (CVMFS), so we need it active.  Read in the latter way, the conditions will be cached locally in your virtual machine the first time you run and so the CMSSW job will be slow.  Fortunately, we already did this while setting up our VM, so our jobs will run much faster.  This does not really matter for the Docker container (keep this line commented out).

## Configure our DemoAnalyzer

The second to last line in our configuration
~~~
process.demo = cms.EDAnalyzer('DemoAnalyzer'
)
~~~
{: .language-python}

has to do with our recently created `DemoAnalyzer`.  This module is now just a *declaration of existance* because it is empty.  Let's put it to work:

> ## Making our EDAnalyzer configurable
>
> Let's pretend that for some reason you will need to run your job either for extracting the energy of RECO muons from beam collisions or from cosmic rays.  Note that CMS has information from both.  If you go back to [the guide](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/#access-methods) we used earlier to help us with the muon information extraction, you will notice that there is the possibility to use an `InputTag` that is `muonsFromCosmics`  instead of just `muons`.  Your job is to make this configurable in our `demoanalyzer_cfg.py` so we don't have to re-compile every time we want to make the switch.
>
> > ## Solution
> >
> > Since we are going to make our DemoAnalyzer configurable, the first thing we need to do is to modify the C++ source of our analyzer in order to accommodate configurability. Let's modify then the `Demo/DemoAnalyzer/src/DemoAnalyzer.cc` file.  Again, following the logic in the [Physics Ojects guide](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/#access-methods) and using an editor, we should add the declaration for a muon InputTag.  We could include this declaration right below the declaration of our member functions:
> >
> > ~~~
> > virtual void endLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);
> >
> > //declare the input tag for MuonCollection
> > edm::InputTag muonInput;
> >
> > // ----------member data ---------------------------
> > std::vector<float> muon_e;
> > ~~~
> > {: .language-cpp}
> >
> > Then we will have to read this InputTag from the configuration.  As it was noted above, this is done in the constructor.  It will become:
> >
> > ~~~
> > //constructors and destructor
> > //
> > DemoAnalyzer::DemoAnalyzer(const edm::ParameterSet& iConfig)
> >
> > {
> >   //now do what ever initialization is needed
> >   muonInput = iConfig.getParameter<edm::InputTag>("InputCollection");
> > }
> > ~~~
> > {: language-cpp}
> >
> > Here we will be reading the `InputCollection` variable from configuration (which is of type `edm::InputTag`, which is [essentially a string](https://github.com/cms-sw/cmssw/blob/52ef6482b221be8c1516bcc6eab63015d4e1fb72/FWCore/Utilities/interface/InputTag.h#L15)) and will store it in the `muonInput` container.
> >
> > Next, we will modify the `analyze` function replacing this line
> >
> > `iEvent.getByLabel("muons", mymuons);`,
> >
> >  where the InputTag is hard-coded as "muons", with
> >
> > `iEvent.getByLabel(muonInput, mymuons);`,
> >
> >  where we use the configurable `muonInput` variable.
> >
> > The section of interest in the `analyze` function will then look like:
> >
> > ~~~
> > void DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup){
> >   using namespace edm;
> >   //clean the container
> >   muon_e.clear();
> >
> >  //define the handler and get by label
> >  Handle<reco::MuonCollection> mymuons;
> >  iEvent.getByLabel(muonInput, mymuons);
> >
> >  //if collection is valid, loop over muons in event
> >  if(mymuons.isValid()){
> >    for (reco::MuonCollection::const_iterator itmuon=mymuons->begin(); itmuon!=mymuons->end(); ++itmuon){
> >        muon_e.push_back(itmuon->energy());
>     }
> > }
> > //print the vector
> >  . . .
> > ~~~
> > {: .language-cpp}
> >
> > Finally, let's change the `Demo/DemoAnalyzer/demoanalyzer_cfg.py` by replacing our empty module statement:
> >
> > `process.demo = cms.EDAnalyzer('DemoAnalyzer')`
> >
> > with
> >
> > ~~~
> > process.demo = cms.EDAnalyzer('DemoAnalyzer',
> >        InputCollection = cms.InputTag("muons")
> > )
> > ~~~
> > {: .language-python}
> >
> > In this way, we are now able to enter "muons" or "muonsFromCosmics", depending on our needs.
> {: .solution}
{: .challenge}

Now, before re-compiling our code, let's check that our python configuration is ok.  We can validate the syntax of your configuration using python:

~~~
python Demo/DemoAnalyzer/demoanalyzer_cfg.py
~~~
{: .language-bash}

If there are no errors, you are good to go.

Now let's compile the code; again, with `scram`:

~~~
scram b
~~~
{: .language-bash}

If everything goes well, you should see something like:

~~~
Reading cached build data
>> Local Products Rules ..... started
>> Local Products Rules ..... done
>> Building CMSSW version CMSSW_5_3_32 ----
>> Subsystem FT_53_LV5_AN1 built
>> Entering Package Demo/DemoAnalyzer
>> Creating project symlinks
  src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
>> Compiling edm plugin /home/cmsusr/test/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc
>> Building edm plugin tmp/slc6_amd64_gcc472/src/Demo/DemoAnalyzer/src/DemoDemoAnalyzer/libDemoDemoAnalyzer.so
Leaving library rule at Demo/DemoAnalyzer
@@@@ Running edmWriteConfigs for DemoDemoAnalyzer
--- Registered EDM Plugin: DemoDemoAnalyzer
>> Leaving Package Demo/DemoAnalyzer
>> Package Demo/DemoAnalyzer built
>> Subsystem Demo built
>> Local Products Rules ..... started
>> Local Products Rules ..... done
gmake[1]: Entering directory `/home/cmsusr/test/CMSSW_5_3_32'
>> Creating project symlinks
  src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
>> Done python_symlink
>> Compiling python modules cfipython/slc6_amd64_gcc472
>> Compiling python modules python
>> Compiling python modules src/Demo/DemoAnalyzer/python
>> All python modules compiled
@@@@ Refreshing Plugins:edmPluginRefresh
>> Pluging of all type refreshed.
>> Done generating edm plugin poisoned information
gmake[1]: Leaving directory `/home/cmsusr/test/CMSSW_5_3_32'
~~~
{: .output}

Finally, let's run the CMSSW job:

~~~
cmsRun Demo/DemoAnalyzer/demoanalyzer_cfg.py > mylog.log 2>&1 &
~~~
{: language-bash}

If you check the development of the job with

~~~
tail -f mylog.log
~~~

Eventually, you will see something like:

~~~
...
Begin processing the 21st record. Run 166782, Event 340669175, LumiSection 309 at 21-Sep-2020 03:54:42.464 CEST
Begin processing the 26th record. Run 166782, Event 340969031, LumiSection 309 at 21-Sep-2020 03:54:42.465 CEST
Muon # 0 with E = 5.69502 GeV.
Begin processing the 31st record. Run 166782, Event 340976903, LumiSection 309 at 21-Sep-2020 03:54:42.467 CEST
Begin processing the 36th record. Run 166782, Event 341180751, LumiSection 309 at 21-Sep-2020 03:54:42.468 CEST
Muon # 0 with E = 3.34225 GeV.
Muon # 1 with E = 3.58025 GeV.
Muon # 2 with E = 6.38618 GeV.
Begin processing the 41st record. Run 166782, Event 340390232, LumiSection 309 at 21-Sep-2020 03:54:42.469 CEST
Muon # 0 with E = 3.11859 GeV.
Muon # 1 with E = 4.65139 GeV.
Muon # 2 with E = 6.14445 GeV.
Muon # 3 with E = 10.3529 GeV.
Muon # 4 with E = 12.1988 GeV.
Muon # 5 with E = 4.36989 GeV.
Muon # 6 with E = 8.17477 GeV.
Begin processing the 46th record. Run 166782, Event 340721280, LumiSection 309 at 21-Sep-2020 03:54:42.471 CEST
Muon # 0 with E = 9.15747 GeV.
Muon # 1 with E = 7.45448 GeV.
Begin processing the 51st record. Run 166782, Event 340800336, LumiSection 309 at 21-Sep-2020 03:54:42.472 CEST
Muon # 0 with E = 13.085 GeV.
Begin processing the 56th record. Run 166782, Event 341075240, LumiSection 309 at 21-Sep-2020 03:54:42.473 CEST
Muon # 0 with E = 28.4378 GeV.
Muon # 1 with E = 4.59856 GeV.
..
~~~
{: .output}

> ## Change the InputTag
>
> Now, change the name of the `InputColletion` from "muons" to "muonsFromCosmics" in your configuration and run again **without** re-compiling the code.  Do you see any difference?
>
> > ## Solution
> >
> > For a part of your output you should see something like:
> >
> > ~~~
> > Begin processing the 21st record. Run 166782, Event 340669175, LumiSection 309 at 21-Sep-2020 04:36:50.314 CEST
> > Begin processing the 26th record. Run 166782, Event 340969031, LumiSection 309 at 21-Sep-2020 04:36:50.315 CEST
> > Muon # 0 with E = 1.85208 GeV.
> > Begin processing the 31st record. Run 166782, Event 340976903, LumiSection 309 at 21-Sep-2020 04:36:50.317 CEST
> > Muon # 0 with E = 12.831 GeV.
> > Begin processing the 36th record. Run 166782, Event 341180751, LumiSection 309 at 21-Sep-2020 04:36:50.318 CEST
> > Muon # 0 with E = 5.89144 GeV.
> > Muon # 1 with E = 0.238398 GeV.
> > Muon # 2 with E = 1.36712 GeV.
> > Begin processing the 41st record. Run 166782, Event 340390232, LumiSection 309 at 21-Sep-2020 04:36:50.320 CEST
> > Begin processing the 46th record. Run 166782, Event 340721280, LumiSection 309 at 21-Sep-2020 04:36:50.321 CEST
> > Muon # 0 with E = 0.42504 GeV.
> > Muon # 1 with E = 0.244313 GeV.
> > Begin processing the 51st record. Run 166782, Event 340800336, LumiSection 309 at 21-Sep-2020 04:36:50.322 CEST
> > Muon # 0 with E = 0.97245 GeV.
> > Begin processing the 56th record. Run 166782, Event 341075240, LumiSection 309 at 21-Sep-2020 04:36:50.324 CEST
> > Muon # 0 with E = 4.1584 GeV.
> > Muon # 1 with E = 3.54916 GeV.
> > Muon # 0 with E = 10.8804 GeV.
> > Muon # 1 with E = 4.60771 GeV.
> > Muon # 0 with E = 10.1483 GeV.
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Running some already-available CMSSW code

The last line in our `Demo/DemoAnalyzer/demoanalyzer_cfg.py` is

~~~
process.p = cms.Path(process.demo)
~~~
{: .language-python}

The “software bus” model that was mentioned in the introduction of this episode can be made evident in this line.  CMSSW executes its code using *Paths* (which in turn could be arranged in [Schedules](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Schedule_statements)).  Each Path can execute a series of modules (or [Sequences](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Module_sequences) of modules).  In our example we have just one Path named `p` that executes the `demo` process, which corresponds to our DemoAnalyzer.

In general, however, we could add more modules.  For instance, the Path line could look like

~~~
process.mypath = cms.Path (process.m1+process.m2+process.s1+process.m3)
~~~
{: .language-python}

, where `m1`, `m2`, `m3` could be CMSSW modules (individual EDAnalyzers, EDFilters, EDProducers, etc.) and `s1` could be a [modules Sequence](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile#Module_sequences).

> ## Adding a Trigger Filter
>
> In CMSSW, there are other types of code one can execute.  Some of these are known as EDFilters.  As the name implies, they can be used to filter events.  For instance, one could use the [HLTHighLevel](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/HLTrigger/HLTfilters/interface/HLTHighLevel.h) filter class to only run over events that have passed a certain kind of trigger.  
>
> We can get a hint of its usage by scooping around the corresponding [python](https://github.com/cms-sw/cmssw/tree/CMSSW_5_3_X/HLTrigger/HLTfilters/python) directory of that package.  We immediately notice the [hltHighLevel_cfi.py](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/HLTrigger/HLTfilters/python/hltHighLevel_cfi.py) module, so let's load it with the line
>
> `process.load("HLTrigger.HLTfilters.hltHighLevel_cfi")`
>
> Let's configure the `HLTPaths` parameter in that module so it will only pass events that fired any trigger bit with the pattern `HLT_Mu15*`:
>
> `process.hltHighLevel.HLTPaths = cms.vstring('HLT_Mu15*')`
>
> Now, let's add the module to our running Path:
>
> `process.p = cms.Path(process.hltHighLevel+process.demo)`
>
> > ## The full config file
> >
> > **Do not forget to switch back to "muons"* for the InputTag.  The full config file will then look like:
> >
> > ~~~
> > import FWCore.ParameterSet.Config as cms
> >
> > process = cms.Process("Demo")
> >
> > process.load("FWCore.MessageService.MessageLogger_cfi")
> > process.MessageLogger.cerr.FwkReport.reportEvery = 5
> >
> > process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(100) )
> >
> > process.source = cms.Source("PoolSource",
> >    # replace 'myfile.root' with the source file you want to use
> >    fileNames = cms.untracked.vstring(
> >    #    'file:myfile.root'
> >        'root://eospublic.cern.ch//eos/opendata/cms/Run2011A/ElectronHad/AOD/12Oct2013-v1/20001/001F9231-F141-E311-8F76-003048F00942.root'
> >    )
> > )
> > #needed to cache the conditions data
> > process.load('Configuration.StandardSequences.FrontierConditions_GlobalTag_cff')
> > process.GlobalTag.connect = cms.string('sqlite_file:/cvmfs/cms-opendata-conddb.cern.ch/FT_53_LV5_AN1_RUNA.db')
> > process.GlobalTag.globaltag = 'FT_53_LV5_AN1::All'
> >
> > process.demo = cms.EDAnalyzer('DemoAnalyzer',
> >        InputCollection = cms.InputTag("muons")
> > )
> >
> > process.load("HLTrigger.HLTfilters.hltHighLevel_cfi")
> > process.hltHighLevel.HLTPaths = cms.vstring('HLT_Mu15*')
> >
> > process.p = cms.Path(process.hltHighLevel+process.demo)
> > ~~~
> > {: .language-python}
> {: .solution}
>
> Without even having to compile again, the execution of the trigger path will stop if the `hltHighLevel` filter module throws a `False` result. The output becomes
>
> ~~~
> Begin processing the 1st record. Run 166782, Event 340184599, LumiSection 309 at 21-Sep-2020 05:44:20.327 CEST
> Begin processing the 6th record. Run 166782, Event 340256207, LumiSection 309 at 21-Sep-2020 05:44:20.335 CEST
> Begin processing the 11th record. Run 166782, Event 340439831, LumiSection 309 at 21-Sep-2020 05:44:20.340 CEST
> Begin processing the 16th record. Run 166782, Event 340635879, LumiSection 309 at 21-Sep-2020 05:44:20.341 CEST
> Begin processing the 21st record. Run 166782, Event 340669175, LumiSection 309 at 21-Sep-2020 05:44:20.344 CEST
> Begin processing the 26th record. Run 166782, Event 340969031, LumiSection 309 at 21-Sep-2020 05:44:22.771 CEST
> Begin processing the 31st record. Run 166782, Event 340976903, LumiSection 309 at 21-Sep-2020 05:44:22.774 CEST
> Begin processing the 36th record. Run 166782, Event 341180751, LumiSection 309 at 21-Sep-2020 05:44:22.777 CEST
> Begin processing the 41st record. Run 166782, Event 340390232, LumiSection 309 at 21-Sep-2020 05:44:22.779 CEST
> Begin processing the 46th record. Run 166782, Event 340721280, LumiSection 309 at 21-Sep-2020 05:44:22.782 CEST
> Begin processing the 51st record. Run 166782, Event 340800336, LumiSection 309 at 21-Sep-2020 05:44:22.784 CEST
> Begin processing the 56th record. Run 166782, Event 341075240, LumiSection 309 at 21-Sep-2020 05:44:22.786 CEST
> Begin processing the 61st record. Run 166782, Event 340174236, LumiSection 309 at 21-Sep-2020 05:44:22.787 CEST
> Muon # 0 with E = 22.5233 GeV.
> Begin processing the 66th record. Run 166782, Event 340343684, LumiSection 309 at 21-Sep-2020 05:44:41.220 CEST
> Begin processing the 71st record. Run 166782, Event 340623812, LumiSection 309 at 21-Sep-2020 05:44:41.224 CEST
> Begin processing the 76th record. Run 166782, Event 340743484, LumiSection 309 at 21-Sep-2020 05:44:41.228 CEST
> Begin processing the 81st record. Run 166782, Event 341031628, LumiSection 309 at 21-Sep-2020 05:44:41.231 CEST
> Begin processing the 86th record. Run 166782, Event 341213420, LumiSection 309 at 21-Sep-2020 05:44:41.233 CEST
> Begin processing the 91st record. Run 166782, Event 340198962, LumiSection 309 at 21-Sep-2020 05:44:41.235 CEST
> Begin processing the 96th record. Run 166782, Event 340375546, LumiSection 309 at 21-Sep-2020 05:44:41.236 CEST
> ~~~
> {: .output}
>
>  The filter clearly prevents the execution of our EDAnalyzer in most of the events.
{: .challenge}


Congratulations!!, you have made it to the end the lesson.


{% include links.md %}

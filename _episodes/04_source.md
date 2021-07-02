---
title: "The Source"
teaching: 10
exercises: 30
questions:
- "What are the elements of the source of an EDAnalyzer?"
- "How do I modify the source to get additional information?"
objectives:
- "Learn the basic structure of the C++ implementation of an EDAnalyzer."
- "Learn the basics on how to modify the source in order to perform analysis."
keypoints:
- "The C++ source file of an EDAnalyzer is taylored for particle physics analysis under the CMSSW Framework."
- "This source file needs to be modified according to the analyzer needs"
---

## Playing with the DemoAnalyzer.cc file

The `DemoAnalyzer.cc` file is the main file of our EDAnalyzer.  As it was mentioned, the default structure is always the same.  Let's look at what is inside using an editor like `nano`:

~~~
nano Demo/DemoAnalyzer/src/DemoAnalyzer.cc
~~~
{: .language-bash}

The first thing that you will see is a set of *includes*:

~~~
// system include files
#include <memory>

// user include files
#include "FWCore/Framework/interface/Frameworkfwd.h"
#include "FWCore/Framework/interface/EDAnalyzer.h"

#include "FWCore/Framework/interface/Event.h"
#include "FWCore/Framework/interface/MakerMacros.h"

#include "FWCore/ParameterSet/interface/ParameterSet.h"
~~~
{: .language-cpp}

These are the most basic *Framework* classes that are needed to mobilize the CMSSW machinery.  In particular, notice the `Event.h` class.  This class contains essentially all the *accessors* that are needed to extract information from *the Event*, i.e., from the particle collision.  Another important class is the `ParameterSet.h`.  This one will allow us to extract configuration parameters, which can be manipulated using the `Demo/DemoAnalyzer/demoanalyzer_cfg.py` python file.

Something important to take into account is that you can learn a lot about the kind of information you have access to by exploring the code in the CMSSW repository on Github.  For instance, you can look at the [Event.h](https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/FWCore/Framework/interface/Event.h) header and check all the available methods. You will notice, for instance, the presence of the `getByLabel` accessors; we will be using one these to access physics objects.

> When exploring CMSSW code on Github, remember to choose the CMSSW_5_3_X branch.
{: .testimonial}

> ## Including muon headers
>
> Let's pretend that we are interested in extracting the energy of all the muons in the event.  We would need to add the appropriate classes for this.  After quickly reviewing [this chapter](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/) of the CMS Open Data Guide (which is still under construction), we conclude that we need to add these two header lines to our analyzer:
>
> ```
> //classes to extract Muon information
> #include "DataFormats/MuonReco/interface/Muon.h"
> #include "DataFormats/MuonReco/interface/MuonFwd.h"
> ```
>
> Let's add them at the end of the header section together with the standard vector C++ library:
>
> ```
> #include<vector>
> ```
>
> So our header section becomes:
>
> ~~~
> // system include files
> #include <memory>
>
> // user include files
> #include "FWCore/Framework/interface/Frameworkfwd.h"
> #include "FWCore/Framework/interface/EDAnalyzer.h"
>
> #include "FWCore/Framework/interface/Event.h"
> #include "FWCore/Framework/interface/MakerMacros.h"
>
> #include "FWCore/ParameterSet/interface/ParameterSet.h"
>
>
> //classes to extract Muon information
> #include "DataFormats/MuonReco/interface/Muon.h"
> #include "DataFormats/MuonReco/interface/MuonFwd.h"
>
> #include<vector>
> ~~~
> {: .language-cpp}
{: .challenge}

Next, you will see the class declaration:

~~~
//
// class declaration
//

class DemoAnalyzer : public edm::EDAnalyzer {
   public:
      explicit DemoAnalyzer(const edm::ParameterSet&);
      ~DemoAnalyzer();

      static void fillDescriptions(edm::ConfigurationDescriptions& descriptions);


   private:
      virtual void beginJob() ;
      virtual void analyze(const edm::Event&, const edm::EventSetup&);
      virtual void endJob() ;

      virtual void beginRun(edm::Run const&, edm::EventSetup const&);
      virtual void endRun(edm::Run const&, edm::EventSetup const&);
      virtual void beginLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);
      virtual void endLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);

      // ----------member data ---------------------------
};
~~~
{: .language-cpp}

The first thing one notices is that our class inherits from the `edm::EDAnalyzer` class.  It follows the same structure as any class in C++.  The declaration of the methods reflect the functionality needed for particle physics analysis.  Their implementation are further below in the same file.

> ## Declaring info containers
>
> Let's add the declaration of a vector for our energy values:  
>
> ```
> std::vector<float> muon_e;
> ```
>
> This section becomes:
>
> ~~~
> //
> // class declaration
> //
>
> class DemoAnalyzer : public edm::EDAnalyzer {
>    public:
>       explicit DemoAnalyzer(const edm::ParameterSet&);
>       ~DemoAnalyzer();
>
>       static void fillDescriptions(edm::ConfigurationDescriptions& descriptions);
>
>
>   private:
>      virtual void beginJob() ;
>      virtual void analyze(const edm::Event&, const edm::EventSetup&);
>      virtual void endJob() ;
>
>      virtual void beginRun(edm::Run const&, edm::EventSetup const&);
>      virtual void endRun(edm::Run const&, edm::EventSetup const&);
>      virtual void beginLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);
>      virtual void endLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);
>
>      // ----------member data ---------------------------
>     
>      std::vector<float> muon_e; //energy values for muons in the event
> };
> ~~~
> {: .language-cpp}
{: .challenge}


Next, we can see the constructor and destructor of our DemoAnalyzer class:

~~~
// constructors and destructor
//
DemoAnalyzer::DemoAnalyzer(const edm::ParameterSet& iConfig)

{
   //now do what ever initialization is needed

}


DemoAnalyzer::~DemoAnalyzer()
{

   // do anything here that needs to be done at desctruction time
   // (e.g. close files, deallocate resources etc.)

}
~~~
{: .language-cpp}

Note that a `ParameterSet` object is passed to the constructor.  This is then the place where we will read any configuration we might end up implementing through our `Demo/DemoAnalyzer/demoanalyzer_cfg.py` python configuration file.

The heart of the source file is the `analyze` method:

~~~
// ------------ method called for each event  ------------
void
DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
{
   using namespace edm;



#ifdef THIS_IS_AN_EVENT_EXAMPLE
   Handle<ExampleData> pIn;
   iEvent.getByLabel("example",pIn);
#endif

#ifdef THIS_IS_AN_EVENTSETUP_EXAMPLE
   ESHandle<SetupData> pSetup;
   iSetup.get<SetupRecord>().get(pSetup);
#endif
}
~~~
{: .language-cpp}

Anything that goes inside this routine will loop over all available events.  The CMSSW Framework will take care of that, so you do not really have to write a `for` loop to go over all events.  Note that an `edm::Event` object and a `edm::EventSetup` object are passed by default.  While from the Event we can extract information like physics objects, from the EventSetup we can get information like trigger prescales.

> ## Get the muons energy
>
> Now let's add a few lines in the analyzer so we can retrieve the energy of all the muons in each event.  We will print out this information as an example. Again, after checking out [this guide](https://cms-opendata-guide.web.cern.ch/analysis/selection/objects/#access-methods), the analyze method becomes:
>
> ~~~
> // ------------ method called for each event  ------------
> void DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
> {
>  using namespace edm;
>  
>  //clean the container
>  muon_e.clear();
>  
>  //define the handler and get by label
>  Handle<reco::MuonCollection> mymuons;
>  iEvent.getByLabel("muons", mymuons);
>
>  //if collection is valid, loop over muons in event
>  if(mymuons.isValid()){
>     for (reco::MuonCollection::const_iterator itmuon=mymuons->begin(); itmuon!=mymuons->end(); ++itmuon){
>         muon_e.push_back(itmuon->energy());
>     }
>  }
>
>  //print the vector
>  for(unsigned int i=0; i < muon_e.size(); i++){
>     std::cout <<"Muon # "<<i<<" with E = "<<muon_e.at(i)<<" GeV."<<std::endl;
>  }
>
> #ifdef THIS_IS_AN_EVENT_EXAMPLE
>   Handle<ExampleData> pIn;
>   iEvent.getByLabel("example",pIn);
> #endif
>
> #ifdef THIS_IS_AN_EVENTSETUP_EXAMPLE
>   ESHandle<SetupData> pSetup;
>   iSetup.get<SetupRecord>().get(pSetup);
> #endif
> }
> ~~~
> {: .language-cpp}
{: .challenge}


The other methods are designed to execute instructions according to their own name description.  
~~~
// ------------ method called once each job just before starting event loop  ------------
void
DemoAnalyzer::beginJob()
{
}

// ------------ method called once each job just after ending the event loop  ------------
void
DemoAnalyzer::endJob()
{
}

// ------------ method called when starting to processes a run  ------------
void
DemoAnalyzer::beginRun(edm::Run const&, edm::EventSetup const&)
{
}

// ------------ method called when ending the processing of a run  ------------
void
DemoAnalyzer::endRun(edm::Run const&, edm::EventSetup const&)
{
}

// ------------ method called when starting to processes a luminosity block  ------------
void
DemoAnalyzer::beginLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&)
{
}

// ------------ method called when ending the processing of a luminosity block  ------------
void
DemoAnalyzer::endLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&)
{
}

~~~
{: .language-cpp}

For instance, any instructions placed inside the `beginRun` routine will be executed every time the Framework sees a new Run (a Run is determined by the start and stop of the acquisition of the CMS detector).  One may use the `beginJob` and `endJob` routines to, for example, book histograms or write output files.

> ## Let's compile
>  
> ~~~
> scram b
> ~~~
> {: .language-bash}
>
> > ## Look at the output
> >
> > Well, it fails.
> >
> > ~~~
> > >> Local Products Rules ..... started
> > >> Local Products Rules ..... done
> > >> Building CMSSW version CMSSW_5_3_32 ----
> > >> Entering Package Demo/DemoAnalyzer
> > >> Creating project symlinks
> >   src/Demo/DemoAnalyzer/python -> python/Demo/DemoAnalyzer
> > >> Compiling edm plugin /home/cmsusr/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc 
> > In file included from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/TrackingRecHit.h:4:0,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/RecSegment.h:17,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4D.h:16,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4DCollection.h:20,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonSegmentMatch.h:6,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonChamberMatch.h:5,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/Muon.h:17,
> >                  from /home/cmsusr/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc:34:
> > /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/CLHEP/interface/AlgebraicObjects.h:8:33: fatal error: CLHEP/Matrix/Vector.h: No such file or directory
> > compilation terminated.
> > In file included from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/TrackingRecHit.h:4:0,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/TrackingRecHit/interface/RecSegment.h:17,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4D.h:16,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/DTRecHit/interface/DTRecSegment4DCollection.h:20,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonSegmentMatch.h:6,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/MuonChamberMatch.h:5,
> >                  from /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/MuonReco/interface/Muon.h:17,
> >                  from /home/cmsusr/CMSSW_5_3_32/src/Demo/DemoAnalyzer/src/DemoAnalyzer.cc:34:
> > /opt/cms/slc6_amd64_gcc472/cms/cmssw/CMSSW_5_3_32/src/DataFormats/CLHEP/interface/AlgebraicObjects.h:8:33: fatal error: CLHEP/Matrix/Vector.h: No such file or directory
> > compilation terminated.
> > gmake: *** [tmp/slc6_amd64_gcc472/src/Demo/DemoAnalyzer/src/DemoDemoAnalyzer/DemoAnalyzer.o] Error 1
> > gmake: *** [There are compilation/build errors. Please see the detail log above.] Error 2
> > ~~~
> > {: .error}
> > This is because the Muon classes we added introduced some dependencies that need to be taken care of in the `BuildFile.xml`
> {: .solution}
{: .challenge}

So let's modify the `Demo/DemoAnalyzer/BuildFile.xml` to include `DataFormats/MuonReco` dependencies.  It should look like:

~~~
<use name="FWCore/Framework"/>
<use name="FWCore/PluginManager"/>
<use name="DataFormats/MuonReco"/>
<use name="FWCore/ParameterSet"/>
<flags EDM_PLUGIN="1"/>
<export>
   <lib name="1"/>
</export>
~~~
{: .language-xml}

Now, if you compile again, it should work.  Then, we can run with the `cmsRun` executable:

~~~
cmsRun Demo/DemoAnalyzer/demoanalyzer_cfg.py > mylog.log 2>&1 &
~~~
{: .language-bash}

Let's check the log file:

~~~
cat mylog.log
~~~
{: .language-bash}

~~~
02-Jul-2021 06:41:46 CEST  Initiating request to open file root://eospublic.cern.ch//eos/opendata/cms/Run2012B/DoubleMuParked/AOD/22Jan2013-v1/10000/1EC938EF-ABEC-E211-94E0-90E6BA442F24.root
02-Jul-2021 06:41:50 CEST  Successfully opened file root://eospublic.cern.ch//eos/opendata/cms/Run2012B/DoubleMuParked/AOD/22Jan2013-v1/10000/1EC938EF-ABEC-E211-94E0-90E6BA442F24.root
Begin processing the 1st record. Run 195013, Event 24425389, LumiSection 66 at 02-Jul-2021 06:42:00.346 CEST
Muon # 0 with E = 8.47069 GeV.
Begin processing the 2nd record. Run 195013, Event 24546773, LumiSection 66 at 02-Jul-2021 06:42:00.410 CEST
Muon # 0 with E = 36.431 GeV.
Muon # 1 with E = 32.3631 GeV.
Begin processing the 3rd record. Run 195013, Event 24679037, LumiSection 66 at 02-Jul-2021 06:42:00.411 CEST
Muon # 0 with E = 16.5536 GeV.
Muon # 1 with E = 12.5258 GeV.
Begin processing the 4th record. Run 195013, Event 24839453, LumiSection 66 at 02-Jul-2021 06:42:00.411 CEST
Muon # 0 with E = 5.19378 GeV.
Muon # 1 with E = 13.6549 GeV.
Begin processing the 5th record. Run 195013, Event 24894477, LumiSection 66 at 02-Jul-2021 06:42:00.412 CEST
Muon # 0 with E = 10.3546 GeV.
Begin processing the 6th record. Run 195013, Event 24980717, LumiSection 66 at 02-Jul-2021 06:42:00.412 CEST
Muon # 0 with E = 25.2733 GeV.
Muon # 1 with E = 10.7545 GeV.
Begin processing the 7th record. Run 195013, Event 25112869, LumiSection 66 at 02-Jul-2021 06:42:00.413 CEST
Muon # 0 with E = 165.026 GeV.
Muon # 1 with E = 145.994 GeV.
Begin processing the 8th record. Run 195013, Event 25484261, LumiSection 66 at 02-Jul-2021 06:42:00.413 CEST
Muon # 0 with E = 130.171 GeV.
Muon # 1 with E = 73.1873 GeV.
Begin processing the 9th record. Run 195013, Event 25702821, LumiSection 66 at 02-Jul-2021 06:42:00.414 CEST
Muon # 0 with E = 24.0889 GeV.
Muon # 1 with E = 14.4771 GeV.
Begin processing the 10th record. Run 195013, Event 25961949, LumiSection 66 at 02-Jul-2021 06:42:00.414 CEST
Muon # 0 with E = 189.134 GeV.
Muon # 1 with E = 38.8268 GeV.
02-Jul-2021 06:42:00 CEST  Closed file root://eospublic.cern.ch//eos/opendata/cms/Run2012B/DoubleMuParked/AOD/22Jan2013-v1/10000/1EC938EF-ABEC-E211-94E0-90E6BA442F24.root

=============================================

MessageLogger Summary

 type     category        sev    module        subroutine        count    total
 ---- -------------------- -- ---------------- ----------------  -----    -----
    1 fileAction           -s file_close                             1        1
    2 fileAction           -s file_open                              2        2

 type    category    Examples: run/evt        run/evt          run/evt
 ---- -------------------- ---------------- ---------------- ----------------
    1 fileAction           PostEndRun                        
    2 fileAction           pre-events       pre-events       

Severity    # Occurrences   Total Occurrences
--------    -------------   -----------------
System                  3                   3
~~~
{: .output}

{% include links.md %}

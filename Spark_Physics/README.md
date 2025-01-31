# Apache Spark for High Energy Physics
  
This collects a few simple examples of how Apache Spark can be used in the domain of High Energy Physics data analysis.  
Most of the examples are just for education purposes, use a small subset of data and can be run on laptop-sized computing resources.  
See also the blog post [Can High Energy Physics Analysis Profit from Apache Spark APIs?](https://db-blog.web.cern.ch/node/186)  

### Contents:
 1. **[Dimuon mass spectrum analysis](#1-dimuon-mass-spectrum-analysis)**
 2. **[HEP analysis benchmark](#2-hep-analysis-benchmark)**
 3. **[ATLAS Higgs analysis](#3-atlas-higgs-boson-analysis---outreach-style)**
 4. **[LHCb matter antimatter analysis](#4-lhcb-matter-antimatter-asymmetries-analysis---outreach-style)**
 - **[How to convert from ROOT format to Apache Parquet and ORC](#notes-on-reading-and-converting-from-root-format-to-parquet-and-orc)**
 - **[Physics references](#physics-references)**
---
## 1. Dimuon mass spectrum analysis
  
This is a sort of "Hello World!" example for High Energy Physics analysis.  
The implementations proposed here using Apache Spark APIs are a direct "Spark translation"
of a [tutorial using ROOT DataFrame](https://root.cern.ch/doc/master/df102__NanoAODDimuonAnalysis_8py.html)

### Data
  - The original data, converted to [nanoaod format](http://cds.cern.ch/record/2752849/files/Fulltext.pdf),
    is shared in [ROOT format](https://root.cern/about/). See [Notes](#Notes) on how to access it.
  - These examples use CMS open data from 2012, made available via the CERN opendata portal:
      [DOI: 10.7483/OPENDATA.CMS.YLIC.86ZZ](http://opendata.cern.ch/record/6004)
      and [DOI: 10.7483/OPENDATA.CMS.M5AD.Y3V3)](http://opendata.cern.ch/record/6030) 
  - Data is also provided (for this work) in Apache Parquet and Apache ORC format
  - You can download the following datasets:
    - **61 million events** (2GB)
      - original files in ROOT format: root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/Run2012BC_DoubleMuParked_Muons.root
      - dataset converted to **Parquet**: [Run2012BC_DoubleMuParked_Muons.parquet](https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012BC_DoubleMuParked_Muons.parquet/)
      - dataset converted to **ORC**: [Run2012BC_DoubleMuParked_Muons.orc](https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012BC_DoubleMuParked_Muons.orc/)
    - **6.5 billion events** (200 GB, this is the 2GB dataset repeast 105 times)
      - original files, in ROOT format root://eospublic.cern.ch//eos/root-eos/benchmark/CMSOpenDataDimuon
      - dataset converted to **Parquet**: [CMSOpenDataDimuon_large.parquet](https://sparkdltrigger.web.cern.ch/sparkdltrigger/CMSOpenDataDimuon_large.parquet)
          - download using `wget -r -np -R "index.html*" -e robots=off https://sparkdltrigger.web.cern.ch/sparkdltrigger/CMSOpenDataDimuon_large.parquet/`
      - dataset converted to **ORC**: [CMSOpenDataDimuon_large.orc](https://sparkdltrigger.web.cern.ch/sparkdltrigger/CMSOpenDataDimuon_large.orc)
        - download using `wget -r -np -R "index.html*" -e robots=off https://sparkdltrigger.web.cern.ch/sparkdltrigger/CMSOpenDataDimuon_large.orc/`
  
      
### Notebooks 
Multiple notebook solutions are provided, to illustrate different approaches with Apache Spark.  
Notes on the execution environment:
 - The notebooks use the dataset with 61 million events (Except the SCALE test that uses 6.5 billion events)
 - Spark version: Spark 3.2.1 (except the mapInArrow and Parquet examples that use Spark 3.3.0-SNAPSHOT versions)
 - The Apache ORC format is used to profit from vectorized read for complex types in Spark 3.2.1
 - The machine used for testing has 4 physical CPU cores

| <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/250px-Jupyter_logo.svg.png" height="50"> Notebook                                                                                                                                                                                                                                    | Run Time   | Short description                                                                                                                                                                                           |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [**1. DataFrame API**](Dimuon_mass_spectrum/1.Dimuon_mass_spectrum_histogram_Spark_DataFrame_ORC_vectorized.ipynb)                                                                                                                                                                                                                                                        | 10 sec     | The analysis is implemented using Apache Spark DataFrame API. This uses the dataset in Apache ORC format                                                                                                    |
| [**2. Spark SQL**](Dimuon_mass_spectrum/2.Dimuon_mass_spectrum_histogram_Spark_SQL_ORC_vectorized.ipynb)                                                                                                                                                                                                                                                                  | 10 sec     | Same as above but using Spark SQL                                                                                                                                                                           |
| [**3a. DataFrame API with Parquet**](Dimuon_mass_spectrum/3a.Dimuon_mass_spectrum_histogram_Spark_DataFrame_Parquet.ipynb)                                                                                                                                                                                                                                                | 30 sec     | Same as (1.) but considerably slower because Apache Spark 3.2.1 does not support vectorized reads for complex types in Apache Parquet                                                                       |
| [**3b. DataFrame API with Parquet vectorized**](Dimuon_mass_spectrum/3b.Dimuon_mass_spectrum_histogram_Spark_DataFrame_Parquet_vectorized_reader_for_complex_types.ipynb)                                                                                                                                                                                                 | 11 sec     | This uses a Spark SNAPSHOT version with support for vectorized reads for complex types in Apache Parquet, see [SPARK-34863](https://issues.apache.org/jira/browse/SPARK-34863)                              |
| [**4. Scala UDF**](Dimuon_mass_spectrum/4.Dimuon_mass_spectrum_histogram_Spark_Scala_UDF_ORC_vectorized.ipynb)                                                                                                                                                                                                                                                            | 10 sec     | Mixed DataFrame and Scala UDF. The dimuon invariant mass formula computation is done using a Scala UDF. Link to [Scala UDF code](Dimuon_mass_spectrum/scalaUDF/src/main/scala/ch/cern/udf/DimuonMass.scala) |
| [**5a. Pandas UDF flattened data**](Dimuon_mass_spectrum/5a.Dimuon_mass_spectrum_histogram_Spark_Pandas_flattened_data_UDF_ORC_vectorized.ipynb)                                                                                                                                                                                                                          | 15 sec     | Mixed DataFrame and Scala UDF. The dimuon invariant mass formula computation is done using a Python Pandas UDF                                                                                              |
| [**5b. Pandas UDF data arrays**](Dimuon_mass_spectrum/5b.Dimuon_mass_spectrum_histogram_Spark_Pandas_full_formula_with_arrays_UDF_ORC_vectorized.ipynb)                                                                                                                                                                                                                   | 58 sec     | Same as 5a, but the Pandas UDF in this case uses data in arrays and lists                                                                                                                                   |
| [**5c. Pandas UDF data arrays optimized**](Dimuon_mass_spectrum/5c.Dimuon_mass_spectrum_histogram_Spark_Pandas_optimized_formula_with_arrays_UDF_ORC_vectorized.ipynb)                                                                                                                                                                                                    | 42 sec     | Same as 5b, but using an optimized formula that substantially reduces the amount of calculations performed                                                                                                  |
| [**6a. MapInArrow flattened data**](Dimuon_mass_spectrum/6a.Dimuon_mass_spectrum_histogram_Spark_UDF_MapInArrow_flattened_data_ORC_vectorized.ipynb)                                                                                                                                                                                                                      | 19 sec     | This uses mapInArrow, introduced in [SPARK-37227](https://issues.apache.org/jira/browse/SPARK-37227)                                                                                                        |
| [**6b. MapInArrow data arrays**](Dimuon_mass_spectrum/6b.Dimuon_mass_spectrum_histogram_Spark_UDF_MapInArrow_with_arrays_ORC_vectorized.ipynb)                                                                                                                                                                                                                            | 84 sec     | Same as 6a but the mapInArrow UDF uses data arrays                                                                                                                                                          |
| [**7. RumbleDB on Spark**](Dimuon_mass_spectrum/7.Dimuon_mass_spectrum_histogram_RumbleDB_on_Spark.ipynb)                                                                                                                                                                                                                                                                 | 166 sec    | This implementation runs with RumbleDB query engine on top of Apache Spark. RumbleDB implements the JSONiq language.                                                                                        |
| [**<span style="color:red">8. DataFrame API at scale</span>**](Dimuon_mass_spectrum/8.Dimuon_mass_spectrum_histogram_Spark_DataFrame_ORC_vectorized-Large_SCALE.ipynb)                                                                                                                                                                                                    | (*)33 sec  | (*)This has processed 6.5 billion events, at scale on a cluster using 200 CPU cores.                                                                                                                        |
| **[<img src="https://raw.githubusercontent.com/googlecolab/open_in_colab/master/images/icon128.png" height="50"> Dimuon spectrum analysis on Colab](https://colab.research.google.com/github/LucaCanali/Miscellaneous/blob/master/Spark_Physics/Dimuon_mass_spectrum/Dimuon_mass_spectrum_histogram_Spark_DataFrame_Colab_version.ipynb)**                                | -          | You can run this on Google's Colaboratory                                                                                                                                                                   |
| **[<img src="https://swanserver.web.cern.ch/swanserver/images/badge_swan_white_150.png" height="30"> Dimuon spectrum analysis on CERN SWAN](https://cern.ch/swanserver/cgi-bin/go/?projurl=https://raw.githubusercontent.com/LucaCanali/Miscellaneous/master/Spark_Physics/Dimuon_mass_spectrum/Dimuon_mass_spectrum_histogram_Spark_DataFrame_CERNSWAN_version.ipynb)**  | -          | You can run this on CERN SWAN (requires CERN SSO credentials)                                                                                                                                               |               

---
## 2. HEP analysis benchmark
  
This provides implementations of the High Energy Physics benchmark tasks using Apache Spark.    
It follows the [IRIS-HEP benchmark](https://github.com/iris-hep/adl-benchmarks-index) specifications
and solutions linked there.  
Solutions to the benchmark tasks are also directly inspired by the article [Evaluating Query Languages and Systems for High-Energy Physics Data](https://arxiv.org/abs/2104.12615).  
      
### Data
  - This uses CMS open data from 2012, made available via the CERN opendata portal:
    [ DOI:10.7483/OPENDATA.CMS.IYVQ.1J0W](http://opendata.cern.ch/record/6021)
  - The original data, converted to [nanoaod format](http://cds.cern.ch/record/2752849/files/Fulltext.pdf), is shared in [ROOT format](https://root.cern/about/)
  - Data is also provided in Apache Parquet and Apache ORC format
  - Datasets you can download and use for this analysis:
  - 53 million events (16 GB), original files in ROOT format: root://eospublic.cern.ch//eos/root-eos/benchmark/Run2012B_SingleMu.root
  - **53 million events** (16 GB), converted to Parquet: [Run2012BC_DoubleMuParked_Muons.parquet](https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012B_SingleMu.parquet)
    - download using `wget -r -np -R "index.html*" -e robots=off https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012B_SingleMu_sample.parquet/` 
  - **53 million events** (16 GB), converted to ORC: [Run2012BC_DoubleMuParked_Muons.orc](https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012B_SingleMu.orc)
    - download using `wget -r -np -R "index.html*" -e robots=off https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012B_SingleMu_sample.orc/` 
  - **7 million events** (2 GB) ORC format [Run2012B_SingleMu_sample.orc](https://sparkdltrigger.web.cern.ch/sparkdltrigger/Run2012B_SingleMu_sample.orc)  
  
### Notebooks 

The notebooks use the dataset with 53 million events in Apache ORC format (to profit from vectorized read in Spark 3.2.1).

| <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/250px-Jupyter_logo.svg.png" height="50"> Notebook                                                                                                                                                                           | Short description                                                                                                                                                                                                                       |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [**Benchmark tasks 1 to 5**](HEP_benchmark/ADL_HEP_Query_Benchmark_Q1_Q5.ipynb)                                                                                                                                                                                                                                  | The analysis is implemented using Apache Spark DataFrame API. Uses the dataset in Apache ORC format                                                                                                                                     |
| [**Benchmark task 6**](HEP_benchmark/ADL_HEP_Query_Benchmark_Q6.ipynb)                                                                                                                                                                                                                                           | Three different solution are provided. This is the hardest task to implement in Spark. The proposed solutions use also Scala UDFs: link to [the Scala UDF code](HEP_benchmark/scalaUDF/src/main/scala/ch/cern/udf/HEPBenchmarkQ6.scala) |
| [**Benchmark task 7**](HEP_benchmark/ADL_HEP_Query_Benchmark_Q7.ipynb)                                                                                                                                                                                                                                           | Two different solutions provided, one using the explode function, the oder with Spark's higher order functions for array processing                                                                                                     |
| [**Benchmark task 8**](HEP_benchmark/ADL_HEP_Query_Benchmark_Q8.ipynb)                                                                                                                                                                                                                                           | This combines Spark DataFarme API for filtering and Scala UDFs for processing. Link to [the Scala UDF code](HEP_benchmark/scalaUDF/src/main/scala/ch/cern/udf/HEPBenchmarkQ8.scala)                                                     |
| **[<img src="https://raw.githubusercontent.com/googlecolab/open_in_colab/master/images/icon128.png" height="50"> Benchmark tasks 1 to 5 on Colab](https://colab.research.google.com/github/LucaCanali/Miscellaneous/blob/master/Spark_Physics/HEP_benchmark/ADL_HEP_Query_Benchmark_Q1_Q5_Colab_Version.ipynb)** | You can run this on Google's Colaboratory                                                                                                                                                                                               |
| **[<img src="https://swanserver.web.cern.ch/swanserver/images/badge_swan_white_150.png" height="30"> Benchmark tasks 1 to 5 on CERN SWAN](https://cern.ch/swanserver/cgi-bin/go/?projurl=https://raw.githubusercontent.com/LucaCanali/Miscellaneous/master/Spark_Physics/HEP_benchmark/ADL_HEP_Query_Benchmark_Q1_Q5_CERNSWAN_Version.ipynb)**         | You can run this on CERN SWAN (requires CERN SSO credentials)  |                     

---
## 3. ATLAS Higgs boson analysis - outreach-style
This is an example analysis of the Higgs boson detection via the decay channel H &rarr; ZZ* &rarr; 4l
From the decay products measured at the ATLAS experiment and provided as open data, you will be able to produce a few histograms,
comparing experimental data and Monte Carlo (simulation) data. From there you can infer the invariant mass of the Higgs boson.  
Disclaimer: this is for educational purposes only, it is not the code nor the data of the official Higgs boson discovery paper.  
It is based on the original work on [ATLAS outreach notebooks](https://github.com/atlas-outreach-data-tools/notebooks-collection-opendata/tree/master/13-TeV-examples/uproot_python)
and the derived [work at this repo](https://github.com/gordonwatts/pyhep-2021-SX-OpenDataDemo)   
Reference: ATLAS paper on the [discovery of the Higgs boson](https://www.sciencedirect.com/science/article/pii/S037026931200857X) (mostly Section 4 and 4.1)   

### Data
  - The original data in ROOT format is from the [ATLAS Open Datasets](http://opendata.atlas.cern/release/2020/documentation/)
    - direct link: [ATLAS open data events selected with at least four leptons (electron or muon)](https://atlas-opendata.web.cern.ch/atlas-opendata/samples/2020/4lep.zip)
    - Note: it is a small dataset (200 MB), this analysis is mostly to show the use of Spark API, rather than its performance and scalability.
  - The notebooks presented here use datasets in Apache Parquet format. 
    - Download from: [ATLAS Higgs notebook opendata in Parquet format](https://sparkdltrigger.web.cern.ch/sparkdltrigger/ATLAS_Higgs_opendata)
      - download all files (200 MB) using `wget -r -np -R "index.html*" -e robots=off https://sparkdltrigger.web.cern.ch/sparkdltrigger/ATLAS_Higgs_opendata/`

### Notebooks
| <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/250px-Jupyter_logo.svg.png" height="50"> Notebook                                                                                                                                                                                                                  | Short description                                                                                                                                                                              |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **[ATLAS opendata Higgs H-ZZ*-4l basic analysis](ATLAS_Higgs_opendata/H_ZZ_4l_analysis_basic_experiment_data.ipynb)**                                                                                                                                                                                                                                   | Basic analysis with experiment/detector data                                                                                                                                                   |
| **[H-ZZ*-4l analysis with additional filters and cuts](ATLAS_Higgs_opendata/H_ZZ_4l_analysis_extra_cuts_montecarlo_data.ipynb)**                                                                                                                                                                                                                        | Analysis with extra cuts and data operations, this uses Monte Carlo (simulation) data                                                                                                          |
| **[H-ZZ*-4l analysis with experiment data and monte carlo](ATLAS_Higgs_opendata/H_ZZ_4l_analysis_data_and_monte_carlo_Fig2_Higgs_paper.ipynb)**                                                                                                                                                                                    | Analysis with extra cuts and data operations, this uses experimental data and Monte Carlo (simulation) data for signal and backgroud. It roughly reproduces Figure 2 of the ATLAS Higgs paper. |
| **[<img src="https://raw.githubusercontent.com/googlecolab/open_in_colab/master/images/icon128.png" height="50">Run ATLAS opendata Higgs H-ZZ*-4l basic analysis on Colab](https://colab.research.google.com/github/LucaCanali/Miscellaneous/blob/master/Spark_Physics/ATLAS_Higgs_opendata/H_ZZ_4l_analysis_basic_experiment_data.ipynb)**             | This notebook opens on Google's Colab                                                                                                                                                          |
| **[<img src="https://swanserver.web.cern.ch/swanserver/images/badge_swan_white_150.png" height="30"> Run ATLAS opendata Higgs H-ZZ*-4l basic analysis on CERN SWAN](https://cern.ch/swanserver/cgi-bin/go/?projurl=https://raw.githubusercontent.com/LucaCanali/Miscellaneous/master/Spark_Physics/ATLAS_Higgs_opendata/H_ZZ_4l_analysis_basic.ipynb)** | This notebook opens on CERN's SWAN notebook service (requires CERN SSO credentials)                                                                                                            |

---
## 4. LHCb matter antimatter asymmetries analysis - outreach-style
This notebook provides an example of how to use Spark to perform a simple analysis using high energy physics data from a LHC experiment.
**Credits:**
* The original text of this notebook, including all exercises, analysis, explanations and data have been developed by the 
LHCb collaboration and are authored and shared by the LHCb collaboration in their opendata and outreach efforts. See links:
    * https://github.com/lhcb/opendata-project/blob/master/LHCb_Open_Data_Project.ipynb
    * "Undergraduate Laboratory Experiment: Measuring Matter Antimatter Asymmetries at the Large Hadron Collide" https://cds.cern.ch/record/1994172?ln=en
    * http://www.hep.manchester.ac.uk/u/parkes/LHCbAntimatterProjectWeb/LHCb_Matter_Antimatter_Asymmetries/Homepage.html
  
### Data
  - The notebook presented here uses datasets in Apache Parquet format:
    - (1.2 GB) download from: [LHCb_opendata_notebook_data](https://sparkdltrigger.web.cern.ch/sparkdltrigger/LHCb_opendata) 
    - download all files using `wget -r -np -R "index.html*" -e robots=off https://sparkdltrigger.web.cern.ch/sparkdltrigger/LHCb_opendata/`
  - The original work uses LHCb opendata made available via the CERN opendata portal:
  [PhaseSpaceSimulation.root](http://opendata.cern.ch/eos/opendata/lhcb/AntimatterMatters2017/data/PhaseSpaceSimulation.root),
  [B2HHH_MagnetDown.root](http://opendata.cern.ch/eos/opendata/lhcb/AntimatterMatters2017/data/B2HHH_MagnetDown.root)
  [B2HHH_MagnetUp.root](http://opendata.cern.ch/eos/opendata/lhcb/AntimatterMatters2017/data/B2HHH_MagnetUp.root)
  
### Notebooks
| <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/250px-Jupyter_logo.svg.png" height="50"> Notebook                                                                                                                                                                                                            | Short description                                                                 |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **[LHCb outreach analysis](LHCb_opendata/LHCb_OpenData_Spark.ipynb)**                                                                                                                                                                                                                                                                             | LHCb analysis notebook using open data                                            |
 | **[<img src="https://raw.githubusercontent.com/googlecolab/open_in_colab/master/images/icon128.png" height="50">Run LHCb opendata analysis notebook on Colab](https://colab.research.google.com/github/LucaCanali/Miscellaneous/blob/master/Spark_Physics/LHCb_opendata/LHCb_OpenData_Spark.ipynb)**                                              | This notebook opens on Google's Colab                                |
| **[<img src="https://swanserver.web.cern.ch/swanserver/images/badge_swan_white_150.png" height="30"> Run LHCb opendata analysis notebook on CERN SWAN](https://cern.ch/swanserver/cgi-bin/go/?projurl=https://raw.githubusercontent.com/LucaCanali/Miscellaneous/master/Spark_Physics/LHCb_opendata/LHCb_OpenData_Spark_CERNSWAN_Version.ipynb)** | This notebook opens on CERN SWAN notebook service (requires CERN SSO credentials) |

---

## Notes on reading and converting from ROOT format to Parquet and ORC
  - If you need to convert data in ROOT format to Apache Parquet or ORC:
     - You can use Spark and the Laurelin library, as detailed in [this note on converting from ROOT format](Spark_Root_data_preparation.md)
     - You can use Python toolkits, notably uproot and awkward arrays, as [in this example of using uproot](Uproot_example.md)
  - If you need to access data shared via the XRootD protocol, as it is the 
  case when reading from URLs like `root://eospublic.cern.ch/..`
    - You can use Apache Spark with the [Hadoop-XRootD connector](https://github.com/cerndb/hadoop-xrootd)
    - You can use the toolset from [XRootD project](https://xrootd.slac.stanford.edu/)
      - CLI example: `xrdcp root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/Run2012BC_DoubleMuParked_Muons.root .`

## Physics references
A few links with additional details on the terms and formulas used:  
  - https://github.com/iris-hep/adl-benchmarks-index/blob/master/reference.md
  - http://edu.itp.phys.ethz.ch/hs10/ppp1/2010_11_02.pdf
  - https://en.wikipedia.org/wiki/Invariant_mass
    

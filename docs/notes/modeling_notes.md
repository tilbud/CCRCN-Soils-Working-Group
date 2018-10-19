# Running List of notes for modeling

## Table of Contents
1. [Key](#key)  
2. [Notes on Common Assumptions from Frequentists Statistics](#notes-on-common-assumptions-from-frequentists-statistics)
3. [Environmental Covariates](#environmental-covariates)
4. [Parameters for Process Model](#parameters-for-process-model)
5. [Parameters for Spatial Model](#parameters-for-spatial-model)
6. [Parameters for Observation Model](#parameters-for-observation-model)
7. [Derived Quantities](#derived-quantities)

## Key

### indexing 
t - time interval OR sediment cohort  
lat  - latitude  
lon  - longitude  
d - depth interval  
i - core level  
j - site level OR watershed level OR hydrology modeling grid cell  
k - study level  

### functions
g() - process function  
h() - observation function

### variables / parameters
x - covariate  
y - response variable  
z - true unobserved variable / latent state  
σ - variance  
Σ - covariance matrix  
β - generic parameter  

### Abbreviations
BD - bulk density  
OM - fraction organic matter  
RSLR - relative sea level rise  
E - elevation  
GT - greater diurnal tide  
MSL - mean sea level  
MHW - mean high water  
MHHW - mean higher high water  
NAVD88 - North American Vertical Datum 1988  

## Notes on Common Assumptions from Frequentists Statistics
* Normal distributions - Commonly used frequentist statistical tests typically assume errors are normally distributed
  + Not all distributions are normal and other distributions have properties that are useful
  + Examples of other distributions we may need include:
    * Lognormal - positively skewed and always positive
    * Gamma - Always positive, can have variable shape
    * Beta - Can take any value between 0 and 1
  + Moments:
    * In normal distributions the mean and standard deviation are the ‘moments’, values which describe the shape of the distribution
    * In other distributions the moments may be simpler, more complex or a function of both mean and s.d. 
    * We can use ‘moment matching’ to approximate and fit more appropriate distribution
* Independence - Commonly used frequentist statistical tests typically assume errors are independent
  + In some cases variables may jointly vary. For example: bulk density and % organic matter
  + There may also be autocorrelation over space and through time 
* Homoscedasticity - Commonly used frequentist statistical tests typically assume errors are the same over the range of a variable
  + We don’t always need to assume constant errors. In some cases we can model variance as a function of something else. 
  + For example uncertainty in measuring the bulk density could be higher for organic soils than inorganic soils

## Environmental Covariates 

For environmental data at site (j) and time (t), or lat/lon and time (t), some variables could be treated as true known values. Others could have an unobserved ‘true’ value based on a probability distribution.

* Relative Sea Level Rise at t,j
  + We need annual time steps change in water level
  + Idea 1: Treat as known with no measurement uncertainty (big assumption)
  + Idea 2: Treat as normal with μ and σ krigged from local tide gauge network
  + Other assumptions?
    * Maybe: simplify rates using a segmented regression
    * Maybe: Integrate lunar nodal cycle to better represent annual-decadal scale dynamics
  + Idea 3: Same as 2 but instead of interpolating at x,y (very computationally expensive), maybe interpolate out to an existing hydrodynamic grid structure.
* Above Ground Biomass at t,i,j
  + May need specialized site data
  + MEM uses a parabola defined by max growing biomass
  + May need to be careful with how to integrate invasive species
    * Option 1: explicitly model a switch time
    * Option 2: Ignore invasive species in calibration and hindcasting and include in forecasting
  + We may want to find a set of simplified classes rather than model biomass explicitly for each site 
    * Idea 1: emergent (high, medium, low), scrub/shrub, forested (h,m,l)
    * Idea 2: break up by size and/or photosynthetic pathway C3, C4, and CAM 
    
* Average SSC t,j
  + Site specific SSC data is probably pretty common, we just need to put in the work to get good average values for each site.
  + How do we scale up?
    * Idea 1 - I have flow weighted average SSC data calculated according to Weston et al. (2014) for pretty much all coastal gauges
      + I don’t know how to best spatially interpolate between gauges
      + Maybe a simple model w/ watershed size
    * Idea 2 - Simplified categories 
      + Examples include deltaic / back marsh
      + Would need someone to map these or code our sites
      + We may end up being limited by our own ability to map
    * Idea 3 - Make SSC a decay function of distance from and size of river mouth / creek mouth.
    * Idea 4 - Is there an existing hydrodynamic grid model we can use?
    
* E<sub>top;i,j</sub> and/or E<sub>0;i,j</sub>
  + Uncertainty in Etop RTK-GPS is fairly minimal (~4 cm nominal error, ~0 bias). For scaling up would need to include bias and error from LiDAR
  + How do we model E0 initial elevation?
    * Do we run the model backwards from the present day?
    * Is ‘initial condition’ an unknown value that we need to account for?
      + Is it NAVD88 elevation of the depth interval relative to the surface elevation?
      + Is it NAVD88 elevation of the sample +/- some assumption of shallow subsidence?

* E<sub>t-1</sub>
  + Elevation can be both an input and an output variable. Having time dependency built in makes this a dynamic model.
  + Can we make random draws from a BACON distribution?
    * BACON accretion rates are represented as piecewise linear segments 
    * Amount of autocorrelation is represented by a beta distributed memory parameter (0 to 1) 

* Min and Max Depth t - Treat as known and used to calculate depth weighted averages of OM, BD, and ΔEt

* Tidal Datums (MSL relative to NAVD88 and MHHW relative to MSL)
  + Idea 1: Treat as known, use NOAA data or Holmquist et al., krigged surfaces
  + Idea 2: Use NOAA data or Holmquist et al. krigged surfaces with propagated uncertainty

## Parameters for Process Model
_Note that a lot of the complexity of this model will depend on whether the core or the depth interval are the fundamental unit of analysis._

* Root Shoot Ratio
  + MEM generally assumes 2:1 AGB to BGB
  + This should be highly variable site to site, and species to species
  + We may need some targeted lit review.
* Root mass shape
  + MEM assumes exponential relationship between depth and root mass
  + Not only shape available. I’ve also seen normal distributions.
  + May need targeted lit review.
* Maximum Root Depth - MEM
  + MEM runs typically go with 30 cm
  + This should be highly variable from site to site and species to species
  + Is there feedback between root depth and relative tidal elevation x sediment porosity?
* Root and Rhizome turnover rate
  + Should be higher in annual systems and lower in perennial
  + Different MEM papers have different turnover rates.
  + May need more targeted lit. review.
* Decay Rate - some evidence that model should be insensitive to decay rate at this temporal scale, recalcitrant fraction matters more
* Recalcitrant Fraction
  + In MEM it is assumed lignin content ultimately constraints decay. ~10% 
  + However we know that resistance to decay is highly dependent on biogeochemical context
  + Idea: 5 classes, 1 oxic, 4 anoxic
    * 2 salinity classes and 2 sediment classes
      + Assume saline wetlands are more dominated by sulfate reduction and fresh more by methane production
      + Assume sediments with high allochthonous input are more dominated by Fe or Mn reduction.
      + Where do we get priors?
      + What do we do w/ Nitrate reduction? Is it baked into framework? A remaining uncertainty?
    * Relative dominance of oxic and anoxic controlled by an oxygen diffusion parameter
      + Could be influenced by Z* elevation and root mass
      + Concern: root mass max can be a cause of AND affected by of oxygen penetration. Don’t know how to square this.
      + Mineral and organic soils could have inherently different oxygen penetration because of porosity. This could point towards positive feedback in soil formation.

* SSC capture efficiency and SSC settling velocity
  + In MEM these are calibrated at Jim’s sites 
  + Are these universal or need to vary from site to site?

If the sample is the fundamental unit of the analysis, then we will need to run CTM based on initial conditions until the start of our sea-level rise / geochronology record to get the initial state. Then run the merged MEM and CTM in annual time steps. Then calculate depth integrated weighted means of the key variables. This could get really complicated.

If the core is the fundamental unit of the analysis then we can maybe use Jim’s simplified ‘Michaelis-Menten’ curve from his MEM carbon burial rate sensitivity analysis. If this is the case then MM curves have a couple shape parameters we’d need. Also a few of the parameters and input variables could be simplified.



## Parameters for Spatial Model

Idea: have a level of hierarchy that elaborates on the MEM type model and allows spatial autocorrelation in how much the actual data vary from the modeled values. This could incorporate a lot of site specific and geomorphic variation. Would help in mapping stage if we ever get to it. Would show higher uncertainty further away from current calibration points. 

Concern: The model is already very overloaded. Would this model take years or decades to converge?

## Parameters for Observation Model


* Age-Depth Models
  + Age-depth models from BACON previously determined uncertainty
  + Issues needed to address uncertainty ahead of BACON include:
    * 137Cs ages getting more uncertain in areas with low fallout and as time gets further away from 1963
    * 210Pb CRS uncertainty and bias
      + Higher bias with wider depth intervals
      + Higher uncertainty with lower detection limit
* Organic Matter Fraction
  + Could integrate a lab specific random effect, normally distributed
  + Process model could be beta distribution, Observation model could be normal
* Bulk Density
  + Process model could by gamma, observation model could be lognormal to acknowledge the fact that you’re most likely to squish samples as you sample them
  + Could model heteroskedasticity, increasing uncertainty and bias w/ more organic soils
* Need covariance between OM and BD

## Derived Quantities
* Converting OM burial to OC burial, propagate model uncertainty - lab/local effects from the in Holmquist et al. equation
* Subtracting stable C from high turnover C - calculate carbon sequestration rate as a derived quantity
* Normalizing over relevant time period.
  + What’s the right time frame to make predictions or hindcasts over?
  + This could be a whole paper on its own
  + I suggest 18.6 year lunar nodal cycles, the same time frames tidal datums are typically updated over
* Predicting out of sample geospatial data 
   + Past and future lunar nodal cycle?
  + IDK if this is anywhere near possible given computational constraints.
* Pushing forward in time?




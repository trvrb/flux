## Example XMLs for BMDS cartographic models in BEAST

### Data specification

Titer data should be specified in the following fashion:

```
virusIsolate	virusStrain	virusYear	serumIsolate	serumStrain	serumYear	titer
A/Arizona/14/78	A/Arizona/14/1978	1978	A/Arizona/14/78	A/Arizona/14/1978	1978	640
A/Brazil/11/78	A/Brazil/11/1978	1978	A/Arizona/14/78	A/Arizona/14/1978	1978	160
A/Lackland/3/78	A/Lackland/3/1978	1978	A/Arizona/14/78	A/Arizona/14/1978	1978	160
...
```

where `virusIsolate` and `serumIsolate` are the unique virus and serum isolates (traditionally rows and columns in an HI table) and `virusStrain` and `serumStrain` are the strains from which these isolates derive.

### Basic model

The file `H1N1_mds_nodrift_noeffects_notree.xml` gives the most basic cartographic model, corresponding to model #2 in Table 1 of the manuscript.  

The antigenic model is specified by:

```xml
<antigenicLikelihood id="antigenicLikelihood" 
							fileName="H1N1_HI_data.txt"
							mdsDimension="2"
							intervalWidth="1.0">
	<virusLocations>
		<matrixParameter id="virusLocations"/>
	</virusLocations>	
	<serumLocations>
		<matrixParameter id="serumLocations"/>
	</serumLocations>				
	<mdsPrecision>
		<parameter id="mds.precision" value="1.0" lower="0.0"/>
	</mdsPrecision>
</antigenicLikelihood>
```

Here `mdsDimension` specifies the number of dimensions to use in the BMDS.  `intervalWidth` is an optional parameter, that when specified uses interval likelihoods (eq. 7 in the manuscript) rather than point likelihoods (eq. 5 in manuscript).  Additionally, there is an optional parameter `mergeSerumIsolates`, that when set to `"true"` takes locations based on `serumStrain` rather than `serumIsolate`.

MCMC proposals on `virusLocations`, `serumLocations` and `mds.precision` follow:

```xml
<operators id="operators" optimizationSchedule="log">
			
	<randomWalkOperator windowSize="1.0" weight="1000">
		<parameter idref="virusLocations"/>
	</randomWalkOperator>

	<randomWalkOperator windowSize="1.0" weight="1000">
		<parameter idref="serumLocations"/>
	</randomWalkOperator>

	<scaleOperator scaleFactor="0.99" weight="1">
		<parameter idref="mds.precision"/>
	</scaleOperator>	
				
</operators>
```

Priors on virus and serum locations follow a diffuse normal distribution and prior on `mds.precision` follows a diffuse gamma distribution:

```xml
<prior id="prior">
			
	<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
		<parameter idref="mds.precision"/>
	</gammaPrior>
	
	<uniformPrior lower="-100" upper="100">
		<parameter idref="virusLocations"/>
	</uniformPrior>
	
	<uniformPrior lower="-100" upper="100">
		<parameter idref="serumLocations"/>
	</uniformPrior>	
															
</prior>
```

Virus and serum locations are logged as normal vector parameters:

```xml
<log id="fileLog1" logEvery="200000" fileName="H1N1_mds.log">
	<posterior idref="posterior"/>
	<prior idref="prior"/>
	<likelihood idref="likelihood"/>
	<antigenicLikelihood idref="antigenicLikelihood"/>		
	<parameter idref="mds.precision"/>		
</log>

<log id="fileLog2" logEvery="200000" fileName="H1N1_mds.virusLocs.log">
	<parameter idref="virusLocations"/>
</log>

<log id="fileLog3" logEvery="200000" fileName="H1N1_mds.serumLocs.log">
	<parameter idref="serumLocations"/>
</log>		
```

### Drift

A more complicated model that models antigenic drift is specified in `H1N1_mds_drift_noeffects_notree.xml`.  This corresponds to model 6 in Table 1 of the manuscript.

This differs from the basic model by specifying a positive `location.drift` and specifying `virusOffsets` and `serumOffsets`:

```xml
<antigenicLikelihood id="antigenicLikelihood" 
							fileName="H1N1_HI_data.txt"
							mdsDimension="2"
							intervalWidth="1.0">
	<virusLocations>
		<matrixParameter id="virusLocations"/>
	</virusLocations>	
	<serumLocations>
		<matrixParameter id="serumLocations"/>
	</serumLocations>				
	<mdsPrecision>
		<parameter id="mds.precision" value="1.0" lower="0.0"/>
	</mdsPrecision>
	<locationDrift>
		<parameter id="location.drift" value="0.5" lower="0.0"/>
	</locationDrift>
	<virusOffsets>
		<parameter id="virusOffsets"/>
	</virusOffsets>
	<serumOffsets>
		<parameter id="serumOffsets"/>
	</serumOffsets>
</antigenicLikelihood>
```

This also requires the addition of:

```xml
<driftedLocationsStatistic id="driftedVirusLocations">
	<locations>
		<matrixParameter idref="virusLocations"/>
	</locations>	
	<offsets>
		<parameter idref="virusOffsets"/>
	</offsets>			
	<locationDrift>
		<parameter idref="location.drift"/>
	</locationDrift>		
</driftedLocationsStatistic>

<driftedLocationsStatistic id="driftedSerumLocations">
	<locations>
		<matrixParameter idref="serumLocations"/>
	</locations>	
	<offsets>
		<parameter idref="serumOffsets"/>
	</offsets>			
	<locationDrift>
		<parameter idref="location.drift"/>
	</locationDrift>		
</driftedLocationsStatistic>	

<distributionLikelihood id="virusLocations.hpm">
	<data>
		<parameter idref="virusLocations"/>
	</data>
	<distribution>
		<normalDistributionModel>
			<mean>
				<parameter id="virus.mean" value="0.0" lower="-Infinity" upper="Infinity"/>
			</mean>
			<precision>
				<parameter id="virus.precision" value="1.0" lower="0.0" upper="Infinity"/>
			</precision>
		</normalDistributionModel>
	</distribution>
</distributionLikelihood>	
	
<distributionLikelihood id="serumLocations.hpm">
	<data>
		<parameter idref="serumLocations"/>
	</data>
	<distribution>
		<normalDistributionModel>
			<mean>
				<parameter id="serum.mean" value="0.0" lower="-Infinity" upper="Infinity"/>
			</mean>
			<precision>
				<parameter id="serum.precision" value="1.0" lower="0.0" upper="Infinity"/>
			</precision>
		</normalDistributionModel>
	</distribution>
</distributionLikelihood>	
```

Operators include the addition of:

```xml
<scaleOperator scaleFactor="0.99" weight="1000">
	<parameter idref="location.drift"/>
</scaleOperator>
				
<scaleOperator scaleFactor="0.99" weight="10">
	<parameter idref="virus.precision"/>
</scaleOperator>			

<scaleOperator scaleFactor="0.99" weight="10">
	<parameter idref="serum.precision"/>
</scaleOperator>
```

The diffuse normal prior on virus and serum locations is replaced by hierarchical normal priors:

```xml									
<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
	<parameter idref="virus.precision"/>
</gammaPrior>

<distributionLikelihood idref="virusLocations.hpm"/>		
		
<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
	<parameter idref="serum.precision"/>
</gammaPrior>			
					
<distributionLikelihood idref="serumLocations.hpm"/>	
```

And a diffuse gamma prior is included for drift rate:

```xml
<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
	<parameter idref="location.drift"/>
</gammaPrior>
```

Locations are logged using the *drifted* locations instead of raw locations:

```xml
<log id="fileLog2" logEvery="200000" fileName="H1N1_mds.virusLocs.log">
	<statistic idref="driftedVirusLocations"/>
</log>

<log id="fileLog3" logEvery="200000" fileName="H1N1_mds.serumLocs.log">
	<statistic idref="driftedSerumLocations"/>
</log>		
```

### Estimating virus avidity and serum potency

A more complicated model that includes virus avidity and estimates (rather than fixing) serum potency is specified in `H1N1_mds_drift_effects_notree.xml`.

This has the addition of:

```xml
<virusAvidities>
	<parameter id="virusAvidities"/>
</virusAvidities>
<serumPotencies>
	<parameter id="serumPotencies"/>
</serumPotencies>	
```

in the `antigenicLikelihood` block.

And the addition of hierarchical `distributionLikelihoods` on `virusAvidities` and `serumPotencies`:

```xml
<distributionLikelihood id="virusAvidities.hpm">
	<data>
		<parameter idref="virusAvidities"/>
	</data>
	<distribution>
		<normalDistributionModel>
			<mean>
				<parameter id="virusAvidities.mean" value="0.0"/>
			</mean>
			<precision>
				<parameter id="virusAvidities.precision" value="1.0" lower="0.0"/>
			</precision>
		</normalDistributionModel>
	</distribution>
</distributionLikelihood>	

<distributionLikelihood id="serumPotencies.hpm">
	<data>
		<parameter idref="serumPotencies"/>
	</data>
	<distribution>
		<normalDistributionModel>
			<mean>
				<parameter id="serumPotencies.mean" value="10.0"/>
			</mean>
			<precision>
				<parameter id="serumPotencies.precision" value="1.0" lower="0.0"/>
			</precision>
		</normalDistributionModel>
	</distribution>
</distributionLikelihood>	
```

MCMC proposals now include changes to avidity and potency vectors as well as proposals to the hierarchical priors:

```xml
<randomWalkOperator windowSize="1.0" weight="100">
	<parameter idref="virusAvidities"/>
</randomWalkOperator>	

<randomWalkOperator windowSize="1.0" weight="100">
	<parameter idref="serumPotencies"/>
</randomWalkOperator>

<scaleOperator scaleFactor="0.99" weight="10">
	<parameter idref="virusAvidities.precision"/>
</scaleOperator>					

<randomWalkOperator windowSize="1.0" weight="10">
	<parameter idref="serumPotencies.mean"/>
</randomWalkOperator>	

<scaleOperator scaleFactor="0.99" weight="10">
	<parameter idref="serumPotencies.precision"/>
</scaleOperator>	
```

Notice there is no proposal for `virusAvidities.mean`.  This stays fixed at `0.0` for reasons of identifiability.

Hierarchical priors are included for both virus avidities and serum potencies:

```xml
<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
	<parameter idref="virusAvidities.precision"/>
</gammaPrior>								

<distributionLikelihood idref="virusAvidities.hpm"/>				

<normalPrior mean="0.0" stdev="100.0">
	<parameter idref="serumPotencies.mean"/>
</normalPrior>				

<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
	<parameter idref="serumPotencies.precision"/>
</gammaPrior>								

<distributionLikelihood idref="serumPotencies.hpm"/>	
```

Hierarchical parameters are included in the main log file:

```xml
<parameter idref="virusAvidities.mean"/>			
<parameter idref="virusAvidities.precision"/>			
<parameter idref="serumPotencies.mean"/>
<parameter idref="serumPotencies.precision"/>	
```

and avidities and potencies are recorded in separate log files:

```xml
<log id="fileLog4" logEvery="200000" fileName="H1N1_mds.virusAvidities.log">
	<parameter idref="virusAvidities"/>
</log>

<log id="fileLog5" logEvery="200000" fileName="H1N1_mds.serumPotencies.log">
	<parameter idref="serumPotencies"/>
</log>
```

### Phylogenetic diffusion

A diffusion model for changes in antigenic phenotype across a viral phylogeny is given in `H1N1_mds_drift_effects_tree.xml`.  This corresponds to model 10 in Table 1 of the manuscript.

This requires the addition of a `taxa` block specifying the year of isolation of a virus and a placeholder for its antigenic location:

```xml
<taxa id="taxa">
	<taxon id="A/Annecy/2013/2009">
		<date value="2009" direction="forwards" units="years"/>
		<attr name="antigenic">1.0 1.0</attr>
	</taxon>
	<taxon id="A/Arizona/14/1978">
		<date value="1978" direction="forwards" units="years"/>
		<attr name="antigenic">1.0 1.0</attr>
	</taxon>
	...
</taxa>
```

A sequence `alignment` and a `treeLikelihood` model can be included to sample the phylogeny.  However, here I'm the simpler approach of running BEAST to generate trees and then using these trees in a downstream BMDS analysis:

```xml
<empiricalTreeDistributionModel id="treeModel" fileName="H1N1_sample.trees">
	<taxa idref="taxa"/>
</empiricalTreeDistributionModel>

<statistic id="treeModel.currentTree" name="Current Tree">
	<empiricalTreeDistributionModel idref="treeModel"/>
</statistic>
```

The multivariate diffusion is specified with:

```xml
<multivariateDiffusionModel id="diffusionModel">
	<precisionMatrix>
		<matrixParameter id="precisionMatrix">
			<compoundParameter>
				<parameter id="diffusion.precision" value="1.0"/>
				<parameter id="diffusion.corr" value="0.0"/>
			</compoundParameter>	
			<compoundParameter>
				<parameter idref="diffusion.corr"/>
				<parameter idref="diffusion.precision"/>	
			</compoundParameter>	
		</matrixParameter>	
	</precisionMatrix>
</multivariateDiffusionModel>

<multivariateTraitLikelihood id="traitLikelihood" traitName="antigenic" 
							 useTreeLength="true" scaleByTime="false" 
							 reportAsMultivariate="true" 
							 reciprocalRates="true" 
							 integrateInternalTraits="true"
							 cacheBranches="true">
	<multivariateDiffusionModel idref="diffusionModel"/>
	<treeModel idref="treeModel"/>
	<traitParameter>
		<parameter id="leaf.antigenic"/>
	</traitParameter>
	<conjugateRootPrior>
		<meanParameter>
			<parameter value="0.0 0.0"/>
		</meanParameter>
		<priorSampleSize>
			<parameter value="1"/>
		</priorSampleSize>
	</conjugateRootPrior>
</multivariateTraitLikelihood>	
```

The `<distributionLikelihood id="virusLocations.hpm">` block is dropped.

Operators include:

```xml
<empiricalTreeDistributionOperator weight="1">
	<empiricalTreeDistributionModel idref="treeModel"/>
</empiricalTreeDistributionOperator>
```

and the operator on `virus.precision` is replaced with:

```xml
<scaleOperator scaleFactor="0.99" weight="10">
	<parameter idref="diffusion.precision"/>
</scaleOperator>
```

Priors include:

```xml
<gammaPrior shape="0.001" scale="1000.0" offset="0.0">
	<parameter idref="diffusion.precision"/>
</gammaPrior>

<multivariateTraitLikelihood idref="traitLikelihood"/>	
```

The location-tagged phylogeny is logged with:

```xml
<logTree id="treeFileLog" logEvery="200000" nexusFormat="true" fileName="H1N1_mds.trees" sortTranslationTable="true">
	<treeModel idref="treeModel"/>
	<posterior idref="posterior"/>
	<driftedTraits>
		<multivariateTraitLikelihood idref="traitLikelihood"/> 
		<parameter idref="location.drift"/>
	</driftedTraits>
</logTree>
```
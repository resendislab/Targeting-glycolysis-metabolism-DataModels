
## Aalize the HaCaT Normal Cell line model.

We read the Recon2.2 of biomodels, if you whant to download uncomment the second line.
```python
import cobra
#!wget http://www.ebi.ac.uk/biomodels-main/download?mid=MODEL1603150001 -O recon2.2.xml
Recon2 = cobra.io.read_sbml_model("Models/recon2.2.xml")
```

Reading the default model
```python
hacat_model=cobra.io.read_sbml_model("Models/hgu133APlus2_hacat_model_0418_DEMEM6429_n5.sbml")

```

Calculating growth default growth_rate and duplication time
```python
import math
growth_rate = math.log(2)/22.5
print(growth_rate*1.20)
#duplication time
print(math.log(2)/0.008)
```
    0.03696784962986375
    86.64339756999316

Define the default bounds for the defined DMEM 6429 medium

```python
#Biomass bounds
for rxex in hacat_model.exchanges:
    rxex.bounds=(-0.01,1)
    
### DMEM 6429 medium
#### AminoAcids

hacat_model.reactions.EX_ala_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_arg_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_asn_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_asp_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_cys_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_gln_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_glu_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_gly_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_his_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_ile_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_leu_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_lys_L_LPAREN_e_RPAREN_.lower_bound=-1 
hacat_model.reactions.EX_met_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_phe_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_pro_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_ser_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_thr_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_trp_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_tyr_L_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_val_L_LPAREN_e_RPAREN_.lower_bound=-1

### DMEM 6429 medium
#Carbon Sources
hacat_model.reactions.EX_glc_LPAREN_e_RPAREN_.lower_bound=-4.5
hacat_model.reactions.EX_pyr_LPAREN_e_RPAREN_.lower_bound= -1
### DMEM 6429 medium
#Minerals, vitamins, other
hacat_model.reactions.EX_chol_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_fe2_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_fol_LPAREN_e_RPAREN_.lower_single_gene_deletionbound=-1
hacat_model.reactions.EX_h2o_LPAREN_e_RPAREN_.lower_bound=-10
hacat_model.reactions.EX_inost_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_pi_LPAREN_e_RPAREN_.lower_bound=-10
hacat_model.reactions.EX_ribflv_LPAREN_e_RPAREN_.lower_bound=-1
hacat_model.reactions.EX_so4_LPAREN_e_RPAREN_.lower_bound=-1
#EXTRA
hacat_model.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(-1000,0)
hacat_model.reactions.biomass_reaction.lower_bound=0.008
hacat_model.reactions.biomass_reaction.upper_bound=0.037


fba_solution = hacat_model.optimize()
hacat_model.optimize()
```

Calculate FBA and parsimonius FBA
```python
fba_solution.fluxes.to_csv('hacat_fba_fluxes_221117_DEMEM6429_n5.csv')

pfba_solution = cobra.flux_analysis.pfba(hacat_model)
```

Save result to file

```python
pfba_solution.fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5.csv')
```

Calculate the Flux Variability analisis
```python
hacat_model.summary(fva=1.00)
```


Create a single deletation analisis using the hacat_model
```python
import pandas
from time import time

import cobra.test

from cobra.flux_analysis import (
    single_gene_deletion, single_reaction_deletion, double_gene_deletion,
    double_reaction_deletion)

deletion_results=single_reaction_deletion(hacat_model, hacat_model.reactions)
```

Print the deletation results which are distint than optimal

```python
deletion_results[deletion_results.status != 'optimal']
```

Ref: Doubling time of human cancerous cell lines 16.2 Hrs 
http://bionumbers.hms.harvard.edu/bionumber.aspx?&id=100685&ver=21


Analisis of model exchanges.
We close all fluxes except the once necesary to analize the main type of metabilism
test for max ATP hydrolysis flux from only o2 and the defined carbon

```python
closedModel=hacat_model.copy()
closedModel.add_reaction(Recon2.reactions.DM_atp_c_)
closedModel.objective="DM_atp_c_"

for rx in closedModel.exchanges:
    rx.upper_bound= 100
    rx.lower_bound= -0.00000001
    

#Biomass
closedModel.reactions.biomass_other.lower_bound=0
closedModel.reactions.biomass_carbohydrate.lower_bound=0
closedModel.reactions.biomass_DNA.lower_bound=0
closedModel.reactions.biomass_lipid.lower_bound=0
closedModel.reactions.biomass_protein.lower_bound=0
closedModel.reactions.biomass_RNA.lower_bound=0
closedModel.reactions.biomass_reaction.lower_bound=0
closedModel.reactions.biomass_reaction.upper_bound=0


#closedModel.reactions.r1147.bounds=(0,0)
#closedModel.reactions.r2375.bounds=(0,0)
# test for max ATP hydrolysis flux from only o2 and the defined carbon
# source
# glucose aerobic

closedModel.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(-1000,0.0)
closedModel.reactions.EX_h2o_LPAREN_e_RPAREN_.bounds=(-1000,1000)
closedModel.reactions.EX_co2_LPAREN_e_RPAREN_.bounds=(-0.0,1000)
closedModel.reactions.EX_glc_LPAREN_e_RPAREN_.bounds=(-1,-1)
closedModel.optimize()
```


 Analisis of parcimonius FBA of the closed model with different carbon sources.

parsimonius FBA for glutamine anaerobic medium

```python
cobra.flux_analysis.pfba(closedModel).fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5_glucose_aerobic.csv')
```


```python
closedModel=hacat_model.copy()
closedModel.add_reaction(Recon2.reactions.DM_atp_c_)
closedModel.objective="DM_atp_c_"

for rx in closedModel.exchanges:
    rx.upper_bound= 100
    rx.lower_bound= -0.00000001
    

#Biomass
closedModel.reactions.biomass_other.lower_bound=0
closedModel.reactions.biomass_carbohydrate.lower_bound=0
closedModel.reactions.biomass_DNA.lower_bound=0
closedModel.reactions.biomass_lipid.lower_bound=0
closedModel.reactions.biomass_protein.lower_bound=0
closedModel.reactions.biomass_RNA.lower_bound=0
closedModel.reactions.biomass_reaction.lower_bound=0
closedModel.reactions.biomass_reaction.upper_bound=0

# test for max ATP hydrolysis flux from only o2 and the defined carbon
# source
# glucose anaerobic

closedModel.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(0,0)
closedModel.reactions.EX_h2o_LPAREN_e_RPAREN_.bounds=(-1000,1000)
closedModel.reactions.EX_co2_LPAREN_e_RPAREN_.bounds=(0.0,1000)
closedModel.reactions.EX_glc_LPAREN_e_RPAREN_.bounds=(-1,-1)
closedModel.optimize()
```


```python
cobra.flux_analysis.pfba(closedModel).fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5_glucose_anaerobic.csv')
```

parsimonius FBA for glutamine aerobic medium

```python
closedModel=hacat_model.copy()
closedModel.add_reaction(Recon2.reactions.DM_atp_c_)
closedModel.objective="DM_atp_c_"

for rx in closedModel.exchanges:
    rx.upper_bound= 100
    rx.lower_bound= -0.00000001
    

#Biomass
closedModel.reactions.biomass_other.lower_bound=0
closedModel.reactions.biomass_carbohydrate.lower_bound=0
closedModel.reactions.biomass_DNA.lower_bound=0
closedModel.reactions.biomass_lipid.lower_bound=0
closedModel.reactions.biomass_protein.lower_bound=0
closedModel.reactions.biomass_RNA.lower_bound=0
closedModel.reactions.biomass_reaction.lower_bound=0
closedModel.reactions.biomass_reaction.upper_bound=0

# glutamine aerobic
closedModel.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(-1000,0)
closedModel.reactions.EX_h2o_LPAREN_e_RPAREN_.bounds=(-1000,1000)
closedModel.reactions.EX_co2_LPAREN_e_RPAREN_.bounds=(0,1000)
closedModel.reactions.EX_gln_L_LPAREN_e_RPAREN_.bounds=(-1,-1)

closedModel.optimize()
```

```python
cobra.flux_analysis.pfba(closedModel).fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5_glutamine_aerobic.csv')
```

parsimonius FBA for glutamine anaerobic medium

```python
closedModel=hacat_model.copy()
closedModel.add_reaction(Recon2.reactions.DM_atp_c_)
closedModel.objective="DM_atp_c_"

for rx in closedModel.exchanges:
    rx.upper_bound= 100
    rx.lower_bound= -0.00000001 #-0.00474 #this is the minimum value
    

#Biomass
closedModel.reactions.biomass_other.lower_bound=0
closedModel.reactions.biomass_carbohydrate.lower_bound=0
closedModel.reactions.biomass_DNA.lower_bound=0
closedModel.reactions.biomass_lipid.lower_bound=0
closedModel.reactions.biomass_protein.lower_bound=0
closedModel.reactions.biomass_RNA.lower_bound=0
closedModel.reactions.biomass_reaction.lower_bound=0
closedModel.reactions.biomass_reaction.upper_bound=0

# glutamine anaerobic
closedModel.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(0,0)
closedModel.reactions.EX_h2o_LPAREN_e_RPAREN_.bounds=(-1000,1000)
closedModel.reactions.EX_co2_LPAREN_e_RPAREN_.bounds=(0,1000)
closedModel.reactions.EX_gln_L_LPAREN_e_RPAREN_.bounds=(-1,-1)

closedModel.optimize()
```


```python
cobra.flux_analysis.pfba(closedModel).fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5_glutamine_anaerobic.csv')
```


```python
closedModel=hacat_model.copy()
closedModel.add_reaction(Recon2.reactions.DM_atp_c_)
closedModel.objective="DM_atp_c_"

for rx in closedModel.exchanges:
    rx.upper_bound= 100
    rx.lower_bound= -0.00000001 #-0.00474
    
    
#Biomass
closedModel.reactions.biomass_other.lower_bound=0
closedModel.reactions.biomass_carbohydrate.lower_bound=0
closedModel.reactions.biomass_DNA.lower_bound=0
closedModel.reactions.biomass_lipid.lower_bound=0
closedModel.reactions.biomass_protein.lower_bound=0
closedModel.reactions.biomass_RNA.lower_bound=0
closedModel.reactions.biomass_reaction.lower_bound=0
closedModel.reactions.biomass_reaction.upper_bound=0


## fru aerobic
closedModel.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(-1000,0)
closedModel.reactions.EX_h2o_LPAREN_e_RPAREN_.bounds=(-1000,1000)
closedModel.reactions.EX_co2_LPAREN_e_RPAREN_.bounds=(0,1000)
closedModel.reactions.EX_fru_LPAREN_e_RPAREN_.bounds=(-1,-1)

closedModel.optimize()
```


```python
cobra.flux_analysis.pfba(closedModel).fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5_fructose_aerobic.csv')
```


```python
closedModel=hacat_model.copy()
closedModel.add_reaction(Recon2.reactions.DM_atp_c_)
closedModel.objective="DM_atp_c_"

for rx in closedModel.exchanges:
    rx.upper_bound= 100
    rx.lower_bound= -0.00000001
    
    
#Biomass
closedModel.reactions.biomass_other.lower_bound=0
closedModel.reactions.biomass_carbohydrate.lower_bound=0
closedModel.reactions.biomass_DNA.lower_bound=0
closedModel.reactions.biomass_lipid.lower_bound=0
closedModel.reactions.biomass_protein.lower_bound=0
closedModel.reactions.biomass_RNA.lower_bound=0
closedModel.reactions.biomass_reaction.lower_bound=0
closedModel.reactions.biomass_reaction.upper_bound=0


## fru aerobic
closedModel.reactions.EX_o2_LPAREN_e_RPAREN_.bounds=(0,0)
closedModel.reactions.EX_h2o_LPAREN_e_RPAREN_.bounds=(-1000,1000)
closedModel.reactions.EX_co2_LPAREN_e_RPAREN_.bounds=(0,1000)
closedModel.reactions.EX_fru_LPAREN_e_RPAREN_.bounds=(-1,-1)

closedModel.optimize()
```


```python
cobra.flux_analysis.pfba(closedModel).fluxes.to_csv('hacat_pfba_fluxes_221117_DEMEM6429_n5_fructose_anaerobic.csv')
```


```python
closedModel=hacat_model.copy()
for rx in closedModel.reactions:
    rx.bounds=(-0.001,0.001)
    fba=closedModel.optimize()
    fba2=hacat_model.optimize()
    if(fba.f != fba2.f/2 ):
        print(rx.id)
```



# fme_PythonCaller
<https://dataflow.center/webinar/index.html>
Every code example is in python3
Collection of Python scripts to manipulate features in FME
Here you can read more abour Python and FME. 

Here I collect some useful scripts to manipulate features using Python in FME.

## 1. csv attribut into fme list
```
import fme
import fmeobjects
# Template Function interface:
# When using this function, make sure its name is set as the value of
# the 'Class or Function to Process Features' transformer parameter
def processFeature(feature):

    string1 = feature.getAttribute('CSV_COLLECTION')
    b = string1.split(',')
    
    for i, j in enumerate(b):
        feature.setAttribute('list{'+str(i)+'}',j)
```
## 2. access *.py file from within fme
source : <https://community.safe.com/s/question/0D54Q000080hSCgSAM/general-class-in-python-where-to-place>
```
import fmeobjects
import sys
sys.path.append('$(FME_SHAREDRESOURCE_DATA)python')
import my_module
class FeatureProcessor(object):
    ... blablabla
```
## 3. compare two fme lists
```
import fme
import fmeobjects

# attributes relevant for BEDOM_INTERVALL
RELEVANT_FIELDS = ['1','2', '3','4','5', '6', '7'] 
"""
    if at least one element in updated == at least one element in RELEVANT_FIELDS:
        force compute and update for BEDOM_INTERVALL field
        status_bedom_intervall = 'recompute'
"""

def processFeature(feature):
    updated = feature.getAttribute('list{}.attributeName')
    
    if updated:
        OVERLAP = (set(updated) & set(RELEVANT_FIELDS))    
        if OVERLAP:
            feature.setAttribute("CHANGE_BI_OPERATION", 'change')
``` 

## 3. zip two fme lists
```
import fme
import fmeobjects

def processFeature(feature):
    
    guid = feature.getAttribute('GlobalID')
    attrnamn = [i for i in feature.getAttribute('list{}.attributeName')]
    orig = [i for i in feature.getAttribute('list{}.originalValue')]
    revised = [i for i in feature.getAttribute('list{}.revisedValue')]
    ziped = (attrnamn, orig, revised)
    #print(ziped)
    subzip = zip(ziped[0],ziped[1],ziped[2])
    subzip_lst = [', '.join(i) for i in subzip]
    print('\n'.join(subzip_lst))
    #changes = ','.join(subzip_lst)
    #print(changes)
``` 

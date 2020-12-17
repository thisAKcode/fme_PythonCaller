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


## interpolate
``` 
import fme
import fmeobjects
# Template Function interface:
# When using this function, make sure its name is set as the value of
# the 'Class or Function to Process Features' transformer parameter
def processFeature(feature):
    pass

# Template Class Interface:
# When using this class, make sure its name is set as the value of
# the 'Class or Function to Process Features' transformer parameter
class FeatureProcessor(object):
    def __init__(self):
        pass
    def input(self,feature):
        a = feature.getAttribute('_indices{}.z')
        a = list(map(int,a))
        for idx, val in enumerate(a):
            if val == -99999:
                s=(a[idx-1]+a[idx+1])/2
                a[idx] = s
                feature.setAttribute('_indices{'+str(idx)+'}.z',a[idx])
        self.pyoutput(feature)
    def close(self):
        pass

``` 

# nice solution for route problem by Nariman E.
``` 
import fme
import fmeobjects
import math

# Avståndsfunktion, används för att beräkna avstånd mellan två punkter,
def getDistance(f1, f2):
    g1 = f1.getGeometry().getXYZ()
    g2 = f2.getGeometry().getXYZ()
    x = g1[0]-g2[0]
    y = g1[1]-g2[1]
    dist = math.sqrt(x*x+y*y)
    return dist

# Template Class Interface:
# When using this class, make sure its name is set as the value of
# the 'Class or Function to Process Features' transformer parameter
class FeatureProcessor(object):
    def __init__(self):
        self.features = []
        pass
    def input(self,feature):
        self.features.append(feature)
        #self.pyoutput(feature)
    def close(self): #instance method takes self as arg
        i = 0
        p0 = []
        p2 = []
        # Sortera punkterna, start och slutpunkter i varsin vektor.
        while i < len(self.features):
            if self.features[i].getAttribute('SUBTYP') == 0:
                p0.append(self.features[i])
            elif self.features[i].getAttribute('SUBTYP') == 2:
                p2.append(self.features[i])
            i += 1
        # Beräkna avståndet mellan alla startpunkter till alla slutpunkter
        len_p0ap2a = getDistance(p0[0], p2[0])
        len_p0ap2b = getDistance(p0[0], p2[1])
        len_p0bp2a = getDistance(p0[1], p2[0])
        len_p0bp2b = getDistance(p0[1], p2[1])
        
        # Summera alla avstånd, detta används för att beräkna hur stor andel av den totala sträckan som varje avstånd utgör.
        len_tot = len_p0ap2a + len_p0ap2b + len_p0bp2a + len_p0bp2b
        
        kvot1 = len_p0ap2a / len_tot
        kvot2 = len_p0ap2b / len_tot
        kvot3 = len_p0bp2a / len_tot
        kvot4 = len_p0bp2b / len_tot
        
        #threshold, gränsvärde
        t = 0.01
        
        # Om ett avstånd mellan start (start1) och slutpunkt (slut1) är väldigt liten (se threshold): 
        #  - dessa två punkter kan rimligtvis inte utgöra ett mätområde
        #  -> mätområdet bör i så fall bestå av denna startpunkten (start1) och den andra slutpunkten (slut2)
        #  -> det andra mätområdet är det motsatta, dvs (start2) - (slut2)
        # kvot 1 och 4 samt 2 och 3 medför egentligen samma sak.
        if kvot1 < t:
            p0[0].setAttribute('_new_group', '1')
            p2[1].setAttribute('_new_group', '1')
            p0[1].setAttribute('_new_group', '2')
            p2[0].setAttribute('_new_group', '2')
        elif kvot2 < t:
            p0[0].setAttribute('_new_group', '1')
            p2[0].setAttribute('_new_group', '1')
            p0[1].setAttribute('_new_group', '2')
            p2[1].setAttribute('_new_group', '2')
        elif kvot3 < t:
            p0[1].setAttribute('_new_group', '1')
            p2[1].setAttribute('_new_group', '1')
            p0[0].setAttribute('_new_group', '2')
            p2[0].setAttribute('_new_group', '2')
        elif kvot4 < t:
            p0[0].setAttribute('_new_group', '1')
            p2[1].setAttribute('_new_group', '1')
            p0[1].setAttribute('_new_group', '2')
            p2[0].setAttribute('_new_group', '2')
			# instead of return you make pyoutput
        self.pyoutput(p0[0])
        self.pyoutput(p0[1])
        self.pyoutput(p2[0])
        self.pyoutput(p2[1])
``` 

# Enable Adjacent Feature Attributes

``` 
PrevSampleDateTime = feature[-1].sampleDateTime
``` 

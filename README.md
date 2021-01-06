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

def processFeature(feature):

    string1 = feature.getAttribute('CSV_COLLECTION')
    b = string1.split(',')
    
    for i, j in enumerate(b):
        feature.setAttribute('list{'+str(i)+'}',j)
```
Or as per this article> <https://community.safe.com/s/question/0D54Q000080hTgYSAU/set-a-list-attribute-in-python>
```
for i in range(numberFeatures):
    feature.setAttribute("lstGrouped{%d}.STRATA_UNIT_AREA" % i, str(oneStrataUnitArea))
```
Here you see how to access fme list in pythoncaller. 
Use curly braces when using getAttribute.

```
list = feature.getAttribute("lstGrouped{}.STRATA_UNIT_AREA")
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

def processFeature(feature):
    pass


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


# Create an attribute that gives you path to google street view at some particular location
``` 
http://maps.google.com/?cbll=@Value(GPS_koordinat)&cbp=12,20.09,,0,5&layer=c?hl=sv 

http://maps.google.com/?cbll=59.6670184,12.9099721&cbp=12,20.09,,0,5&layer=c?hl=sv 
  
59.6670184,12.9099721 
58.36219232550405, 12.458045063716435
  
  
  
cbll= “cbll” is latitude,longitude for Street View. 
cbp=“cbp” Street View window that accepts 5 parameters: 
  
1) Street View/map arrangement, 12=mostly Street View with corner map 
2) Rotation angle/bearing (in degrees) 
3) Tilt angle, -90 (straight up) to 90 (straight down) 
``` 
# parse json body returned as plain string

``` 
import fme
import fmeobjects
import json


def processFeature(feature):
    data = '[' + feature.getAttribute('_response_body') + ']'
    info = json.loads(data)     # {'distance': 166842, 'title': 'Oslo', 'location_type': 'City', 'woeid': 862592, 'latt_long': '59.912281,10.749980'}
    num_items = len(info)
    
    for i in range(num_items):
        distance = info[i]['distance']
        feature.setAttribute("grannar", json.dumps(distance))
``` 

# convert dictionary string to dictionary

``` 
import fme
import fmeobjects
import json

def processFeature(feature):
    data = feature.getAttribute('body')
    
    # using json.loads() 
    # convert dictionary string to dictionary 

    grannar = json.loads(data)
    print("success")
    
    for k,v in grannar.items():
        print(k, v)

``` 
# textwrap79: immense teststring to fit som label width or something
``` 
import fme
import fmeobjects
from textwrap import wrap, fill

STRING_WIDTH = 79
def processFeature(feature):
    var = """Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."""
    var2 = 'Comment: ' +  var
    foo_raw = fill(var2, width= STRING_WIDTH)
    feature.setAttribute('KOMMENT1', foo_raw)
``` 
# compute iqr 

``` 
import fme
import fmeobjects
import numpy as np

# Template Function interface:
# When using this function, make sure its name is set as the value of
# the 'Class or Function to Process Features' transformer parameter

work_attribute = 'SCI300'
            
class FeatureProcessor(object):
    def __init__(self):
        pass

    def input(self,feature):
        
        range_n_numerical = feature.getAttribute('range_n_numerical')
        list_of_tuples = [(i.split(',')[0], i.split(',')[1]) for i in range_n_numerical.split(';')]
        
        range_list = [range(int(element[0]), int(element[1])) for element in list_of_tuples]
        
        buckit1, buckit2, buckit3, buckit4 = list_of_tuples
        
        buckit1_0 = int(buckit1[0])
        buckit1_1 = int(buckit1[1])
        
        buckit2_0 = int(buckit2[0])
        buckit2_1 = int(buckit2[1])
        
        buckit3_0 = int(buckit3[0])
        buckit3_1 = int(buckit3[1])
        
        buckit4_0 = int(buckit4[0])
        buckit4_1 = int(buckit4[1])

        
        actual = feature.getAttribute(f'{work_attribute}')
        
        
        if buckit1_0 <= int(float(actual)) < buckit1_1:
            range_n = f'{buckit1_0}_{buckit1_1}'
        elif buckit2_0 <= int(float(actual)) < buckit2_1:
            range_n = f'{buckit2_0}_{buckit2_1}'
        elif buckit3_0 <= int(float(actual)) < buckit3_1:
            range_n = f'{buckit3_0}_{buckit3_1}'
        elif buckit4_0 <= int(float(actual)) <= buckit4_1:
            range_n = f'{buckit4_0}_{buckit4_1}'
        else:
            range_n = 'no idea'
        
       
        feature.setAttribute('range_n', range_n)
        self.pyoutput(feature)
    def close(self):
        pass
``` 
# sql left join

```sql 
select GDI_VAGDATA.[VAGTRUMMA_AGGREGAT].GlobalID as GlobalID_Ag, GDI_VAGDATA.[VAGTRUMMA_AGGREGAT].GDB_FROM_DATE as FD_Ag, GDI_VAGDATA.[TRUMINVENTERING_V5].GDB_FROM_DATE as FD_Mast, GDI_VAGDATA.[TRUMINVENTERING_V5].GDB_TO_DATE as ToD_Mast
from GDI_VAGDATA.[TRUMINVENTERING_V5]
left join GDI_VAGDATA.[VAGTRUMMA_AGGREGAT] on GDI_VAGDATA.[VAGTRUMMA_AGGREGAT].GlobalID = GDI_VAGDATA.[TRUMINVENTERING_V5].[GlobalID]
``` 


# collections.Counter
``` 
import fme
import fmeobjects
import collections
# Template Function interface:
# When using this function, make sure its name is set as the value of
# the 'Class or Function to Process Features' transformer parameter
def concat_list(feature):
    some_var = feature.getAttribute('_list{}.REDORG')
    cnt_elmnts = collections.Counter(some_var).most_common()
    some_var2 = feature.getAttribute('_list{}.REDDATUM')
    cnt_elmnts2 = collections.Counter(some_var2).most_common()
    feature.setAttribute('result_invorg', str(cnt_elmnts))
    feature.setAttribute('result_invdate', str(cnt_elmnts2))
``` 

# counter 2
```
import fme
import fmeobjects
import collections

def concat_list(feature):
    # logger = fmeobjects.FMELogFile()
    # logger.logMessageString("Hello, I am Logging Now")
    some_var = feature.getAttribute('_list{}')
    cnt_elmnts = collections.Counter(some_var)
    feature.setAttribute('result', str(cnt_elmnts))

```

# FMEFeature.performFunction to compute length

source: <https://community.safe.com/s/question/0D54Q000080hL4iSAE/calculating-the-length-of-a-line-using-python>

```
# Use the FMEFeature.performFunction to call the @Length() function
len2 = float(feature.performFunction('@Length(2)')) # 2D Length
len3 = float(feature.performFunction('@Length(3)')) # 3D Length
```


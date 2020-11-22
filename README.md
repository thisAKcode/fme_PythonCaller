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


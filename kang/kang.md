---
layout: page
title: RP - COVID-19 Healthcare Accessibility in Chicago
---


**Reproduction of**
# Reproduction of COVID-19 Healthcare Accessibility in Chicago

Replication Materials Available at: [RP-Malcomb forked repository](https://github.com/stevenmontilla/RP-Kang)

Created: `26 May 2021`
Revised: `26 May 2021`

## Introduction
Kang et al. (2020) developed a replicable methodology to assess accessibility to hospital beds and ventilators in the city of Chicago. Their motivation was the rapid growth of the COVID-19 pandemic. They emphasize the need for rapidly diagnosing access to healthcare, and developed a methodology that generates this information rather quickly. Moreover, they publicly published their methodology through a Pyhton Notebook that outlines and provides almost everything needed for analysis.

## Materials and Methods

The reproduction data was accessed through the cybergisx repository provided by the original study authors, and fixed for bugs by Prof. Holler.

**Data and sources**
- Road network: accessed through an Open Street Map python library called OSMNX
- Population: American Community Survey
- Hospital information: Homeland Infrastructure Foundation-Level Data.
  - Specifically, ICU beds and ventilator counts.
- Covid-19 confirmed cases: this dataset was provided through the Illinois Department of Public Health

The study was focused on accessibility in Chicago, IL, but the methodology is meant to be applicable in different geographies and even in different contexts outside of healthcare.

In general terms, they calculated accessibility by increasing the accessibility score of a population if it was within the 30 minute hospital service catchments areas based on driving times along the road network. This approach is called a two-step floating catchment area, but it does not account for friction of distance. Therefore, a population that is 30 minutes away from the hospital will be considered to have the same access as one that is 5 minutes away. To account for friction of distance, they implemented the enhanced two-step catchment area (E2SFCA) method, and defined service catchments based on 10, 20, and 30 minute drives.

One factor that this methodology does not take into account is the gravity of the hospitals based on their size (number of beds or ventilators). Therefore, all hospital are considered to provide the same access regardless of size.

## Modifications to Kang et al. (2020)

Our class identified possible sources of error in the analysis related to edge effects. The model takes into account hospitals outside of Chicago for analysis. However, the road network used did not reach these hospitals. In order to calculate driving times from the hospitals, they hospital points needed to be reassigned to the closest valid node in the road network. Thus, hospitals outside of Chicago were assigned to nodes at the edges of the city, likely inflating the estimates of accessibility at the edges of the city. To address this problem, our class determined that the best way to go about it would be to increase the size of the road network. 

Increasing the road network generated some unexpected errors that Maja Cannavo was able to solve by modifying the original script from:

```python
def network_setting(network):
    _nodes_removed = len([n for (n, deg) in network.out_degree() if deg ==0])
    network.remove_nodes_from([n for (n, deg) in network.out_degree() if deg ==0])
    for component in list(nx.strongly_connected_components(network)):
        if len(component)<10:
            for node in component:
                _nodes_removed+=1
                network.remove_node(node)
    for u, v, k, data in tqdm(G.edges(data=True, keys=True),position=0):
        if 'maxspeed' in data.keys():
            speed_type = type(data['maxspeed'])
            if (speed_type==str):
                if len(data['maxspeed'].split(','))==2:
                    data['maxspeed']=float(data['maxspeed'].split(',')[0])               
                elif data['maxspeed']=='signals':
                    data['maxspeed']=35.0 # drive speed setting as 35 miles
                else:
                    data['maxspeed']=float(data['maxspeed'].split()[0])
            else:
                data['maxspeed']=float(data['maxspeed'][0].split()[0])
        else:
            data['maxspeed']= 35.0 #miles
        data['maxspeed_meters'] = data['maxspeed']*26.8223 # convert mile to meter
        data['time'] = float(data['length'])/ data['maxspeed_meters']
    print("Removed {} nodes ({:2.4f}%) from the OSMNX network".format(_nodes_removed, _nodes_removed/float(network.number_of_nodes())))
    print("Number of nodes: {}".format(network.number_of_nodes()))
    print("Number of edges: {}".format(network.number_of_edges()))    
    return(network)
```
to
**note the addition of the TRY and EXCEPT clauses**
```python
def network_setting(network):
    _nodes_removed = len([n for (n, deg) in network.out_degree() if deg ==0])
    network.remove_nodes_from([n for (n, deg) in network.out_degree() if deg ==0])
    for component in list(nx.strongly_connected_components(network)):
        if len(component)<10:
            for node in component:
                _nodes_removed+=1
                network.remove_node(node)
    for u, v, k, data in tqdm(G.edges(data=True, keys=True),position=0):
        if 'maxspeed' in data.keys():
            speed_type = type(data['maxspeed'])
            if (speed_type==str):
                try:
                    if len(data['maxspeed'].split(','))==2:
                        data['maxspeed']=float(data['maxspeed'].split(',')[0])                  
                    elif data['maxspeed']=='signals':
                        data['maxspeed']= 35.0 # drive speed setting as 35 miles
                    else:
                        data['maxspeed']= float(data['maxspeed'].split()[0])
                except:
                        data['maxspeed']= 35.0 #miles                                             
            else:
                try:
                    data['maxspeed']=float(data['maxspeed'][0].split()[0])
                except:
                    data['maxspeed']= 35.0                                                   
        else:
            data['maxspeed']= 35.0 #miles
        data['maxspeed_meters'] = data['maxspeed']*26.8223 # convert mile to meter
        data['time'] = float(data['length'])/ data['maxspeed_meters']
    print("Removed {} nodes ({:2.4f}%) from the OSMNX network".format(_nodes_removed, _nodes_removed/float(network.number_of_nodes())))
    print("Number of nodes: {}".format(network.number_of_nodes()))
    print("Number of edges: {}".format(network.number_of_edges()))    
    return(network)
```


## Results and Discussion

![kang result](img/ChicagoResult.png)


![reproduction result](img/modified_script_result.png)

![improved figure](img/chicahgohospitals_improved_25_classes.png)


- need to include preprocessing of hospital data to match what they wrote in the paper.

## Acknowledgements

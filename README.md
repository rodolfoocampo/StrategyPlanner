# Strategy Planner

## Healthy Country AI Strategy Planner: an AI-enabled decision support system for prioritising invasive weed actions on Indigenous lands

## Overview

This is an extension of the [Healthy Country AI system](https://github.com/microsoft/HealthyCountryAI).  The Healthy Country AI system is an AI-enabled system that collects, classifies, stores and visualises high-quality data on invasive weeds in Kakadu according to indigenous governance protocols. This system builds upon that work to help turn that data into actions by allowing rangers to make informed weed control prioritisation decisions.

[Code](https://github.com/rodolfoocampo/StrategyPlanner/blob/main/StrategyPlanner.ipynb)

![Healthy Country AI](./Photos/before.png)

![Healthy Country AI + Strategy Planner](./Photos/after.png)

## Requirements 


## **Methodology**

Our methodology has four steps: data clustering, generation and ranking of strategies, spread simulation and interactive visualisation.  We implemented this functionality as an open-source Python package strategyplanner, integrated into the Healthy Country AI pipeline and an interactive dashboard. The strategyplanner package can be used in other similar data pipelines as shown in Appendix 1. 
Our frameworks build on more than a decade of work in the integration of indigenous knowledge in conservation planning and prioritisation in Australia (Robinson et al. 2012; Adams et al 2015a; Adams et al 2015b). 
Data
Our data comes from photos collected by drones and labeled by a computer vision algorithm at a resolution of 5 meters in 3 different sites within Kakadu National Park (Robinson et al. in review). This application case will focus on one of the sites on the Nardab floodplain, which has dimensions of 700x425m, or 140x85 grid cells, for a total of 11,900 cells. The photos were captured on 2019-10-29 corresponding to the Kunumeleng season. There are 10 possible labels for each cell: water, water lillies, bare ground, tree, other grass, burnt othergrass, dead paragrass, recovering paragrass, paragrass, dense paragrass. The Healthy Country AI Strategy Planner connects to this data remotely through the Open Database Connectivity (ODBC) interface and the Python package pyodbc (https://github.com/mkleehammer/pyodbc). 
We define cost and gain metrics for each of the cells depending on the density of grass. In terms of cost, we reflect monetary cost management proportional to the grass density. In terms of gain, we define it in terms of amount of grass killed, and we set it as proportional to grass density as well.
Data clustering
Our data has a spatial resolution of 5x5 meters. This is too granular for weed management and planning in the Kakadu wetlands, which is done at the larger patch scales. Therefore, the first step is to automatically group the data into these patches. We use the k-means unsupervised clustering algorithm using scikit-learn (Pedregosa 2012). K-means requires a number of clusters passed as a parameter. We assume an approximate average patch size of 3,000 m2, thus we end up with 10 clusters. These patches will be our planning units. For each one, we sum the cost and gains of individual cells.

Strategy generation and ranking
We define strategies as a combination of patches that are scheduled for chemical treatment. Initially, we tested the stochastic optimisation algorithms to generate solutions used in other prioritisation approaches (i.e. Possingham, 2009). These methods are useful when the search space is large. However, we are working at a scale with few planning units thus the search space is not large enough to grant the use of complex optimisation models. Instead, we can generate all possible combinations and rank them deterministically.
Our algorithm tries all the combinations of n patches, where n >= 1 and n < P, and P is total patches. There are 1023 possible such combinations. We select only those that have a total cost below the given budget. These leaves us with 155 possible solutions. We then select the top 20% of these in terms of total gain. 

Spread simulation
For each of the top 20% management strategies, we simulate subsequent spread for one year using a cellular automata model developed by Adams et al (2015). The spread model works as follows: each infested cell can spread if it has reached reproductive stage, that is, if it has been infested for more than a given wait time, W. Each cell that spreads produces a given number of offspring, r. For each of the r offspring, we move from the spreader cell in direction d for a distance D given in number of cells. Upon landing on a cell, a spread event occurs according to a probability of establishment defined for each cell label.

The results of the previous step populate an SQL database, which in turn feeds a dashboard where the user can compare strategies and derive insights to inform on the ground weed management. The dashboard is divided in three sections: Introduction page, Strategy Metrics Overview and Strategy Visualiser. 
The purpose of the introduction page is providing clarity about what the tool is doing, and inviting the user to engage with the tool as an informative device rather than a descriptive one. 
In the strategy metrics the user can see, for each of the top 20% strategies: the cost, the percentage of grass after management, and the percentage of grass after spread. The user can rank the according to each one of this metrics, and go to the next page to visualise in a map each specific strategy implementation.
In this page, the user can see to the left a map of the infestation grouped by patches. The user can select a strategy based on the previous page rankings and see which patches are managed in that strategy. The user can then see how the infestation map would look after management, as well as an estimation of the spread after one year from the strategy's implementation.

#### Insert image of dashboard here

## How to run this code

At the core of Strategy Planner is the strategyPlanner function. 

```python
strategyPlanner(label_grid, invasive_number_label, cost_grid, benefit_grid, max_cost, n_patches)
```

**Parameters** 2D numpy array
        
        label_grid: 2D numpy array of integers
            This is the infestation data. It needs to be a 2d array where each cell is an observation with a label attached. The array must contain a number in each position. Each different number represents a different label. In Healthy Country AI, each entry in the array represents a 5x5m photo with a numeric label representing the following labels. 
            
             - 'water' = 1
             - 'water lillies' = 2
             - 'bare ground' = 3
             - 'tree' = 4
             - 'other grass' = 5
             - 'burnt othergrass' = 6
             - 'dead paragrass' = 7
             - 'recovering paragrass' = 8
             - 'paragrass' = 9
             - 'dense paragrass' = 10





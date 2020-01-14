
# Community Detection - Lab

## Introduction

In this lab, you'll once again work to cluster a network dataset. This time, you'll be investigating a meetup dataset for events occurring in Nashville.

## Objectives

You will be able to:
- Cluster a social network into subgroups


```python
import pandas as pd
import networkx as nx
import warnings
warnings.simplefilter("ignore")
import matplotlib.pyplot as plt

```

## Load the Dataset

To start, load the dataset `'nashville-meetup/group-edges.csv'` as a pandas Dataframe. 


```python
df = pd.read_csv('nashville-meetup/group-edges.csv', index_col=0)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>group1</th>
      <th>group2</th>
      <th>weight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>19292162</td>
      <td>535553</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>19292162</td>
      <td>19194894</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>19292162</td>
      <td>19728145</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>19292162</td>
      <td>18850080</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>19292162</td>
      <td>1728035</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



To add some descriptive data, import the file `'nashville-meetup/meta-groups.csv'`.


```python
des_df = pd.read_csv('nashville-meetup/meta-groups.csv')
des_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>group_id</th>
      <th>group_name</th>
      <th>num_members</th>
      <th>category_id</th>
      <th>category_name</th>
      <th>organizer_id</th>
      <th>group_urlname</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>339011</td>
      <td>Nashville Hiking Meetup</td>
      <td>15838</td>
      <td>23</td>
      <td>Outdoors &amp; Adventure</td>
      <td>4353803</td>
      <td>nashville-hiking</td>
    </tr>
    <tr>
      <th>1</th>
      <td>19728145</td>
      <td>Stepping Out Social Dance Meetup</td>
      <td>1778</td>
      <td>5</td>
      <td>Dancing</td>
      <td>118484462</td>
      <td>steppingoutsocialdance</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6335372</td>
      <td>Nashville soccer</td>
      <td>2869</td>
      <td>32</td>
      <td>Sports &amp; Recreation</td>
      <td>108448302</td>
      <td>Nashville-soccer</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10016242</td>
      <td>NashJS</td>
      <td>1975</td>
      <td>34</td>
      <td>Tech</td>
      <td>8111102</td>
      <td>nashjs</td>
    </tr>
    <tr>
      <th>4</th>
      <td>21174496</td>
      <td>20's &amp; 30's Women looking for girlfriends</td>
      <td>2782</td>
      <td>31</td>
      <td>Socializing</td>
      <td>184580248</td>
      <td>new-friends-in-Nashville</td>
    </tr>
  </tbody>
</table>
</div>



## Transform to a Network Representation

Take the Pandas DataFrame and transform it into a graph representation via NetworkX. Make nodes the names of the various groups within the Nashville area. The edges should be the weights between these groups.


```python
group_dict = dict(zip(des_df.group_id, des_df.group_name))
G = nx.Graph()
for row in df.index:
    g1 = group_dict[df.group1[row]]
    g2 = group_dict[df.group2[row]]
    weight = df.weight[row]
    G.add_edge(g1, g2, weight=weight)
```

## Visualize the Network


```python
nx.draw(G, pos=nx.spring_layout(G, k=2, seed=5), alpha=.8, node_color="#32cefe")
```


![png](index_files/index_9_0.png)


## Refine the Visual

As you should see, the initial visualization is a globular mess! Refine the visualization to better picture the center core of the  network.


```python
for i in range(0,100,5):
    print("{}th percentile: {}".format(i,df.weight.quantile(q=i/100)))
```

    0th percentile: 1.0
    5th percentile: 1.0
    10th percentile: 1.0
    15th percentile: 1.0
    20th percentile: 1.0
    25th percentile: 1.0
    30th percentile: 1.0
    35th percentile: 1.0
    40th percentile: 1.0
    45th percentile: 1.0
    50th percentile: 1.0
    55th percentile: 1.0
    60th percentile: 1.0
    65th percentile: 1.0
    70th percentile: 2.0
    75th percentile: 2.0
    80th percentile: 2.0
    85th percentile: 3.0
    90th percentile: 4.0
    95th percentile: 7.0
    


```python
threshold = 5
G_subset = nx.Graph()
for row in df.index:
    g1 = group_dict[df.group1[row]]
    g2 = group_dict[df.group2[row]]
    weight = df.weight[row]
    if weight > threshold:
        G_subset.add_edge(g1, g2, weight=weight)
plt.figure(figsize=(30,20))
nx.draw(G_subset, pos=nx.spring_layout(G_subset, k=2, seed=5),
        alpha=.8, node_color="#32cefe", node_size=5000,
        with_labels=True, font_size=12, font_weight="bold")
```


![png](index_files/index_12_0.png)



```python
len(G_subset)
```




    146



## Try Clustering the Network

Now, that the dataset is a little more manageable, try clustering the remaining group nodes.


```python
gn_clusters = list(nx.algorithms.community.centrality.girvan_newman(G_subset))
for n, clusters in enumerate(gn_clusters):
    print("After removing {} edges, there are {} clusters.".format(n, len(clusters)))
```

    After removing 0 edges, there are 7 clusters.
    After removing 1 edges, there are 8 clusters.
    After removing 2 edges, there are 9 clusters.
    After removing 3 edges, there are 10 clusters.
    After removing 4 edges, there are 11 clusters.
    After removing 5 edges, there are 12 clusters.
    After removing 6 edges, there are 13 clusters.
    After removing 7 edges, there are 14 clusters.
    After removing 8 edges, there are 15 clusters.
    After removing 9 edges, there are 16 clusters.
    After removing 10 edges, there are 17 clusters.
    After removing 11 edges, there are 18 clusters.
    After removing 12 edges, there are 19 clusters.
    After removing 13 edges, there are 20 clusters.
    After removing 14 edges, there are 21 clusters.
    After removing 15 edges, there are 22 clusters.
    After removing 16 edges, there are 23 clusters.
    After removing 17 edges, there are 24 clusters.
    After removing 18 edges, there are 25 clusters.
    After removing 19 edges, there are 26 clusters.
    After removing 20 edges, there are 27 clusters.
    After removing 21 edges, there are 28 clusters.
    After removing 22 edges, there are 29 clusters.
    After removing 23 edges, there are 30 clusters.
    After removing 24 edges, there are 31 clusters.
    After removing 25 edges, there are 32 clusters.
    After removing 26 edges, there are 33 clusters.
    After removing 27 edges, there are 34 clusters.
    After removing 28 edges, there are 35 clusters.
    After removing 29 edges, there are 36 clusters.
    After removing 30 edges, there are 37 clusters.
    After removing 31 edges, there are 38 clusters.
    After removing 32 edges, there are 39 clusters.
    After removing 33 edges, there are 40 clusters.
    After removing 34 edges, there are 41 clusters.
    After removing 35 edges, there are 42 clusters.
    After removing 36 edges, there are 43 clusters.
    After removing 37 edges, there are 44 clusters.
    After removing 38 edges, there are 45 clusters.
    After removing 39 edges, there are 46 clusters.
    After removing 40 edges, there are 47 clusters.
    After removing 41 edges, there are 48 clusters.
    After removing 42 edges, there are 49 clusters.
    After removing 43 edges, there are 50 clusters.
    After removing 44 edges, there are 51 clusters.
    After removing 45 edges, there are 52 clusters.
    After removing 46 edges, there are 53 clusters.
    After removing 47 edges, there are 54 clusters.
    After removing 48 edges, there are 55 clusters.
    After removing 49 edges, there are 56 clusters.
    After removing 50 edges, there are 57 clusters.
    After removing 51 edges, there are 58 clusters.
    After removing 52 edges, there are 59 clusters.
    After removing 53 edges, there are 60 clusters.
    After removing 54 edges, there are 61 clusters.
    After removing 55 edges, there are 62 clusters.
    After removing 56 edges, there are 63 clusters.
    After removing 57 edges, there are 64 clusters.
    After removing 58 edges, there are 65 clusters.
    After removing 59 edges, there are 66 clusters.
    After removing 60 edges, there are 67 clusters.
    After removing 61 edges, there are 68 clusters.
    After removing 62 edges, there are 69 clusters.
    After removing 63 edges, there are 70 clusters.
    After removing 64 edges, there are 71 clusters.
    After removing 65 edges, there are 72 clusters.
    After removing 66 edges, there are 73 clusters.
    After removing 67 edges, there are 74 clusters.
    After removing 68 edges, there are 75 clusters.
    After removing 69 edges, there are 76 clusters.
    After removing 70 edges, there are 77 clusters.
    After removing 71 edges, there are 78 clusters.
    After removing 72 edges, there are 79 clusters.
    After removing 73 edges, there are 80 clusters.
    After removing 74 edges, there are 81 clusters.
    After removing 75 edges, there are 82 clusters.
    After removing 76 edges, there are 83 clusters.
    After removing 77 edges, there are 84 clusters.
    After removing 78 edges, there are 85 clusters.
    After removing 79 edges, there are 86 clusters.
    After removing 80 edges, there are 87 clusters.
    After removing 81 edges, there are 88 clusters.
    After removing 82 edges, there are 89 clusters.
    After removing 83 edges, there are 90 clusters.
    After removing 84 edges, there are 91 clusters.
    After removing 85 edges, there are 92 clusters.
    After removing 86 edges, there are 93 clusters.
    After removing 87 edges, there are 94 clusters.
    After removing 88 edges, there are 95 clusters.
    After removing 89 edges, there are 96 clusters.
    After removing 90 edges, there are 97 clusters.
    After removing 91 edges, there are 98 clusters.
    After removing 92 edges, there are 99 clusters.
    After removing 93 edges, there are 100 clusters.
    After removing 94 edges, there are 101 clusters.
    After removing 95 edges, there are 102 clusters.
    After removing 96 edges, there are 103 clusters.
    After removing 97 edges, there are 104 clusters.
    After removing 98 edges, there are 105 clusters.
    After removing 99 edges, there are 106 clusters.
    After removing 100 edges, there are 107 clusters.
    After removing 101 edges, there are 108 clusters.
    After removing 102 edges, there are 109 clusters.
    After removing 103 edges, there are 110 clusters.
    After removing 104 edges, there are 111 clusters.
    After removing 105 edges, there are 112 clusters.
    After removing 106 edges, there are 113 clusters.
    After removing 107 edges, there are 114 clusters.
    After removing 108 edges, there are 115 clusters.
    After removing 109 edges, there are 116 clusters.
    After removing 110 edges, there are 117 clusters.
    After removing 111 edges, there are 118 clusters.
    After removing 112 edges, there are 119 clusters.
    After removing 113 edges, there are 120 clusters.
    After removing 114 edges, there are 121 clusters.
    After removing 115 edges, there are 122 clusters.
    After removing 116 edges, there are 123 clusters.
    After removing 117 edges, there are 124 clusters.
    After removing 118 edges, there are 125 clusters.
    After removing 119 edges, there are 126 clusters.
    After removing 120 edges, there are 127 clusters.
    After removing 121 edges, there are 128 clusters.
    After removing 122 edges, there are 129 clusters.
    After removing 123 edges, there are 130 clusters.
    After removing 124 edges, there are 131 clusters.
    After removing 125 edges, there are 132 clusters.
    After removing 126 edges, there are 133 clusters.
    After removing 127 edges, there are 134 clusters.
    After removing 128 edges, there are 135 clusters.
    After removing 129 edges, there are 136 clusters.
    After removing 130 edges, there are 137 clusters.
    After removing 131 edges, there are 138 clusters.
    After removing 132 edges, there are 139 clusters.
    After removing 133 edges, there are 140 clusters.
    After removing 134 edges, there are 141 clusters.
    After removing 135 edges, there are 142 clusters.
    After removing 136 edges, there are 143 clusters.
    After removing 137 edges, there are 144 clusters.
    After removing 138 edges, there are 145 clusters.
    After removing 139 edges, there are 146 clusters.
    

## Determine An Optimal Clustering Schema

Finally, determine a final clustering organization.


```python
import numpy as np
import seaborn as sns
sns.set_style('darkgrid')
colors = ["#1cf0c7","#ffd43d","#00b3e6","#32cefe","#efefef",
          "#1443ff","#a6cee3","#1f78b4","#b2df8a","#33a02c","#fb9a99",
          "#e31a1c","#fdbf6f","#ff7f00","#cab2d6","#6a3d9a","#ffff99",
          "#b15928","#8dd3c7","#ffffb3","#bebada","#fb8072","#80b1d3",
          "#fdb462","#b3de69","#fccde5","#d9d9d9","#bc80bd","#ccebc5",
          "#ffed6f","#bf812d","#dfc27d","#f6e8c3","#f5f5f5","#c7eae5",
          "#80cdc1", "#35978f", "#01665e", "#003c30"]

fig = plt.figure(figsize=(12,10))

x = [n+1 for n in range(len(gn_clusters))]

max_cluster_size = [max([len(c) for c in cluster]) for cluster in gn_clusters]
plt.plot(x,max_cluster_size, color=colors[0], label='Max Cluster Size')

min_cluster_size = [min([len(c) for c in cluster]) for cluster in gn_clusters]
plt.plot(x,min_cluster_size, color=colors[1], label='Minimum Cluster Size')

mean_cluster_size = [np.mean([len(c) for c in cluster]) for cluster in gn_clusters]
plt.plot(x,mean_cluster_size, color=colors[2], label='Mean Cluster Size')

median_cluster_size = [np.median([len(c) for c in cluster]) for cluster in gn_clusters]
plt.plot(x,median_cluster_size, color=colors[3], label='Median Cluster Size')

single_node_clusters = [sum([1 if len(c)==1 else 0 for c in cluster]) for cluster in gn_clusters]
plt.plot(x,single_node_clusters, color=colors[6], label='Number of Single Node Clusters')

small_clusters = [sum([1 if len(c)<=5 else 0 for c in cluster ]) for cluster in gn_clusters]
plt.plot(x,small_clusters, color=colors[5], label='Number of Small Clusters (5 or less nodes)')

plt.legend(loc=(1.01,.75), fontsize=14)
plt.title('Cluster Size Metrics versus Number of Edges Removed', fontsize=14)
plt.xlabel('Number of Edges Removed', fontsize=14)
plt.ylabel('Cluster Metric')
plt.ylim(0,80)
plt.yticks(ticks=list(range(0,80,5)))
plt.show()
```


![png](index_files/index_17_0.png)


## Visualize the Clusters

Analyze the output of your clustering schema. Do any clusters of groups stand out to you?


```python
warnings.simplefilter("ignore")
def plot_girvan_newman(G, clusters):
    #Your code here
    fig = plt.figure(figsize=(35,20))
    colors = ["#1cf0c7","#ffd43d","#00b3e6","#32cefe","#efefef",
          "#1443ff","#a6cee3","#1f78b4","#b2df8a","#33a02c","#fb9a99",
          "#e31a1c","#fdbf6f","#ff7f00","#cab2d6","#6a3d9a","#ffff99",
          "#b15928","#8dd3c7","#ffffb3","#bebada","#fb8072","#80b1d3",
          "#fdb462","#b3de69","#fccde5","#d9d9d9","#bc80bd","#ccebc5",
          "#ffed6f","#bf812d","#dfc27d","#f6e8c3","#f5f5f5","#c7eae5",
          "#80cdc1", "#35978f", "#01665e", "#003c30"]
    for n , c in enumerate(clusters):
        ci = G.subgraph(c)
        nx.draw(ci, pos=nx.spring_layout(G_subset, k=3.6, seed=3), with_labels=True, node_color=colors[n],
                alpha=.8, node_size=20000, font_weight="bold", font_size=20)
    plt.show()
sns.set_style('white')
plot_girvan_newman(G_subset, gn_clusters[20])
```


![png](index_files/index_19_0.png)


## Summary

In this lab, you got another chance to practice your network clustering skills, investigating a meetup dataset of events and groups surrounding the Nashville area. If you're up for the challenge, feel free to continue investigating the dataset and trying to cluster users or events!                                                                                                                                                               


```python
!jupyter nbconvert --to markdown index.ipynb
!m
```

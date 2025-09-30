---
layout: page
title: Trajectory Clustering with $(K,\ell)$-Means
description: An implementation of trajectory-clustering algorithms for 2D GPS data.
img: /assets/img/trajectory_clustering/thumbnail.png
importance: 1
category: Personal
related_publications: true
---

### Table of Contents

- **[Background](#background)**
    - *[Polygonal Curves](#polygonal-curves)*
    - *[Fréchet Distance](#fréchet-distance)*
    - *[$\ell$-Simplification](#-simplification)*

# **Introduction**

Clustering algorithms have proven to be one of the most general and useful classes of problems in unsupervised learning. Their value, both as a preprocessing step (e.g. to learn a finite $K$-size representation of high-dimensional continuous data) and as an auditing technique for visualization and interpretability has been emphasized throughout the course. However, all of the methods covered thus far require that data be organized as a series of points in $\mathbb{R}^d$. In this paper, I examine an extension of the clustering problem for the more general task of clustering variable-length trajectories. 

To motivate this approach, consider two examples. First, say that one collected GPS data from hikers departing from a common trailhead. It could be useful to extract from these data a set of characteristic paths reflecting common hikes---either to add these routes to maps, or identify social trails which may pose erosion risks. However, hikers travel at different speeds, pause at variable locations to rest, and almost certainly travel slightly different paths due to the stochastic nature of a trail. The result is a noisy set of data which, if plotted in high dimensional space (or projected onto a common subspace), is unlikely to have obvious shared structure.

Likewise, consider an experiment where a human is equipped with probes that, at regular intervals, record electrical activity from firing neurons. As they see a common stimulus, a rapid ordered series of signals propagate through the brain, and these signals form a trajectory in $\mathbb{R}^3$. Existing approaches measure correlations between the stimulus and individual nodes to capture the likely path of the signal, but it is not obvious how to aggregate these signals into global behavior. It may be useful to identify where these signals travel on average through repeated experiments to encode the pathways that the stimulus excites. However, once again we run into issues using traditional clustering techniques, for the same reasons as above.

In this post, I present an approach to the trajectory clustering problem which generalizes the $K$-means algorithm to trajectory data, and demonstrate results from a compute-efficient implementation of one solution on both real and synthetic datasets.

## **Background**
    
Because we are working with slightly different structures than in the rest of the course, I begin by introducing some of the basic elements of our problem, as well as their nearest analogue in the familiar $K$-means domain, if applicable.
    
### Polygonal Curves

In our problem setting, our data are no longer points in $d$-dimensional space. Instead, each looks like a variable-length list of $d$-dimensional points
    
$$
\tau_i = 
\left[
(x_1^{(1)}, x_2^{(1)}, \dots, x_d^{(1)}), 
\dots,
(x_1^{(k_i)}, x_2^{(k_i)}, \dots, x_d^{(k_i)})
\right]
$$
    
for some set of curves $$\left\{ \tau_i \right\}_{i=1}^N$$ of lengths $$\left\{ k_i \right\}_{i=1}^N$$. Still, there exists some underlyting structure between points not present in an ordinary matrix. Namely, each corresponds to a *polygonal curve* in $\mathbb{R}^d$ (Agarwal et al., 2005):
    
---

#### **Definition (Polygonal Curve):**

A *Polygonal Curve* associated with $\tau_i$ is a continuous mapping $P : [0,n] \to \mathbb{R}^d$ such that $P$ is piecewise linear along each segment $[i, i+1]$, $i \in \{0, \dots, k_i \}$, and $P(j) = \tau_{ij} \: \forall j \in \{0, \dots, k_i \}.$

---



Intuitively, $P$ is the (unique) curve in space connecting each point in $\tau_i$ via a straight line. When clear from context, we use $\tau$ and $P$ interchangeably, so $\tau$ corresponds both to the points of the polygonal curve and the function itself.

     
    
### Fréchet Distance 
    
Given this new structure, it is important to have some measure of distance between curves.
In standard $k$-means, we commonly employ a Euclidean metric, where the distance between two points is given by 

$$
d(x,y) = ||x-y|| = \sqrt{(x_1 - y_1)^2 + \dots + (x_n - y_n)^2}.
$$

In the trajectory setting, however, this metric is no longer a reasonable choice. For one, different trajectories $\tau$ and $\sigma$ may have different lengths, yielding a category error. Moreover, even if we can simplify each path to a common length $n$, there is no guarantee that corresponding points must be in similar regions in space. For instance, say that two people walk a path in opposite directions---their curves may look identical, but performing a pointwise distance operation will yield a huge distance.

To accommodate this, we instead use the *Fréchet Distance* between two curves as our metric (Agarwal et al., 2005):


---

#### **Definition (Fréchet distance):**

Let $\Phi(n)$ be the set of all continuous functions $\phi : [0,1] \to [0,n]$, and let $$\norm{\cdot}$$ be the ordinary Euclidean metric. Then the *Fréchet distance* between two polygonal curves $\tau$ and $\sigma$ of lengths $$n_\tau$$ and $$n_\sigma$$ is given by

$$
d_F(\tau, \sigma)
=
\underset{\substack{\varphi \in \Phi(n_\tau) \\ \psi \in \Phi(n_\sigma)}}{\text{inf}} \:
\underset{t \in [0,1]}{\text{max}}
\norm{
(\tau \circ
\varphi)(t)
-
(\sigma \circ \psi) (t)}.
$$

That is, we take the distance to be the maximum Euclidean distance between the curves, minimized under reparametrization. 

---

<div class="row justify-content-sm-center">
<div class="col-sm-8 mt-3 mt-md-0">
{% include figure.liquid loading="eager" path="assets/img/trajectory_clustering/frechet_ex.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
</div>
</div>
<div class="caption">
A visualization of the Fréchet distance between $\tau$ and $\sigma$. The blue region is formed by tracing a circle of radius $$d_F(\tau, \sigma)$$ by its center along $\tau$. We see that this is the smallest such circle that contains all of $\sigma$.
</div>


Intuitively, it defines a distance such that, for every point on one curve, there exists a point on the other curve at most $d_F(\tau, \sigma)$ away. It thus bounds the maximum distance between curves, like the $\ell_\infty$ norm. 

For the sake of time, we do not verify that this constitutes a valid metric, though the proof is not too difficult. Other metrics are occasionally used for trajectory clustering (in particular Dynamic Time-Warping (DTW) distance), but we only consider Fréchet distance for the time being, as it is used most commonly in practice.

Unfortunately, computing this quanitity is very difficult in general. The authors of (Buchin et al., 2019b) use an algorithm developed by Bringmann et al. (Bringmann et al. 2019), which approximates the Fréchet distance between two curves in nearly $O(n^2)$ time. This method is quite complex (perhaps warranting a final project in an algorithms course on its own), and so I do not cover it in this post. Also, though the authors mention that this method is generalizable to greater than two dimensions, they restrict their work to curves in $\mathbb{R}^2$. Since this was, to my knowledge, the only such implementation at the time of the $(k,\ell)$-clustering paper's publishing, they too only consider curves in two dimensions, though nothing in principal stops these tools from being used on higher-dimensional data.

### $\ell$-Simplification

We also require certain regularity conditions of our cluster centers, as otherwise we could consider an infinite number of trajectories during optimization. In particular, we enforce the constraint that cluster centers have exactly $\ell$ points.
This is accomplished via an algorithm for $\ell$-simplification---that is, given any input curve $\tau$, we must return a curve composed of $\ell$ points which is as close to the original curve as possible. Formally: 


---

#### **Definition ($\ell$-Simplification):**

Let $$\mathcal{C}_{\ell}$$ be the set of all $\ell$-length polygonal curves. 
The \textit{$\ell$-Simplification} of some curve $\tau$ is an $\ell$-length polygonal curve $$\tau^{(\ell)} \in \mathcal{C}_{\ell}$$ such that

$$
\tau^{(\ell)} =
\underset{\sigma \in \mathcal{C}_{\ell}}{\text{argmin}} \; d_F(\tau, \sigma).
$$

---

<div class="row justify-content-sm-center">
<div class="col-sm-8 mt-3 mt-md-0">
{% include figure.liquid loading="eager" path="assets/img/trajectory_clustering/path_simplifications_exact.png" title="example image" class="img-fluid rounded z-depth-1" %}
</div>
</div>
<div class="caption">
Example $\ell$-simplifications for a circular path. Increasing the number of allowed points yields a higher fidelity to the original curve.
</div>



Formally, there are no constraints on where this curve lies. However, in order for this simplification to be computationally feasible, most methods constrain the points of $\tau^{(\ell)}$ to be an ordered subset of the points of $\tau$. The paper that I am examining does this via an approximate algorithm by (Agarwal et al. 2005). Like computing $d_F$, this too is quite a complex procedure, so I leave the technical details to the original authors. For examples of $\ell$-simplifications, see below.


\subsection*{$(K,\ell)$-Means Clustering}

With the background out of the way, we have the vocabulary to formally state our problem:

\begin{definitionbox}{klmeans-clustering} 
Given a dataset of trajectories $\{\tau_i\}_{i=1}^N$ of lengths $\{ k_i\}_{i=1}^N$, we define a \emph{$(K,\ell)$-Means Clustering} as a set of $K$ centers $\mathcal{C} = \{ c_1, \dots, c_K\} \subset \mathcal{C}_\ell$, each a polygonal curve of length $\ell$, and cluster assignments $\mathcal{Z} =  z_1, \dots, z_N \in \{1,2, \dots, K\}$ such that
$$
\mathcal{C}, \mathcal{Z}
=
\underset{\substack{c_1, \dots, c_K \in \mathcal{C}_{\ell} \\ z_1, \dots, z_N \in \{1,\dots, K\}}}{\text{argmin}}
\sum_{i=1}^N
\sum_{k=1}^K
{\mathbb{I}[z_i = k]}
d_F(\tau_i, c_k).
$$
\end{definitionbox}

\section*{Algorithm}

I use the algorithm from Buchin et al. (2019) \cite{buchin2019klcluster}, which is a practical (compute-efficient) approximation of the algorithm from the same lab's paper published earlier that year \cite{buchin2019approximating}.

\subsection*{Initialization}

The authors emphasize that, like $K$-means, the initial location of the cluster centers has a large impact on the result. To account for this this, the authors initialize the clusters using an altered form of \emph{Gonzalez's Algorithm} \cite{gonzalez1985clustering}, which is a greedy algorithm known to initialize centers to good regions in space for $K$-means clustering.
The authors alter Gonzalez's algorithm as follows for a trajectory dataset $\{\tau_i\}_{i=1}^N$:

\begin{enumerate}
\item Intialize an (empty) set of centers $\mathcal C$.
\item Randomly choose some element of the dataset $\tau_i$, and append its $\ell$-simplification $\tau_i^{(\ell)}$ to $\mathcal C$.
\item for iterations $1, \dots, k-1$:
\begin{enumerate}
\item Find the point $\tau_i$ farthest from all of $\mathcal C$ by $d_F$, and append its $\ell$-simplification to $\mathcal C$.
\end{enumerate}
\end{enumerate}

This initialization places clusters far apart and thus encourages the eventual clustering to capture diverse centers. After this initialization, the algorithm is analogous to Lloyd's algorithm, and we alternate between computing cluster centers and assigning clusters which minimize $d_F$. However, computing cluster means in this setting is nontrivial, as there is not a natural way to define the ``mean" of a set of polygonal curves. Thus, \cite{buchin2019klcluster} propose a process they call \emph{Fr\'{e}chet Centering}.

\subsection*{Fr\'{e}chet Centering}

I have included a very helpful figure from the original authors for visual intuition (Fig. \ref{fig:compute-center}). 

\begin{figure}[h]
\centering
\includegraphics[trim={0cm 0.cm 0cm 0cm},clip, width=0.8\textwidth]{compute_centers.png}
\caption{A visualization of the Fr\'{e}chet centering step from \cite{buchin2019klcluster}.}
\label{fig:compute-center}
\end{figure}

After initial cluster centers have been chosen, and the dataset has been partitioned across the centers, computing $d_F$ yields a mapping from each point of the center to the remaining points in the cluster---namely, the points which correspond to each vertex in the minimizing parametrization of the $d_F$ computation. This associates with each vertex of each cluster a set of points $x_1, \dots, x_{N_k}$.

We then may calculate the smallest circle containing the points by taking

$$
r = 
\underset{i,j}{\text{max}}
|| x_i - x_j ||
$$

and find the circle's center at the midway point between the two maximizing points $x_i$ and $x_j$. The authors then take each of these $\ell$ ordered centers to define the vertices of the cluster mean for the next iteration. Doing this for all $K$ clusters yields the next set of means.

\subsection*{Full Algorithm}

With each component defined, we describe the complete algorithm pseudocode below (Algorithm \ref{alg:klmeans-algo}). The result is essentially Lloyd's algorithm, though the individual components are more complex to accommodate the new problem setting.


\begin{algorithm}%[ht!]
\caption{($K,\ell$)-Means Clustering}\label{alg:klmeans-algo}
\begin{algorithmic}
\STATE {\bfseries Input:} A dataset $\mathcal{D} = \left\{ \tau_i \right\}_{i=1}^N$ of trajectories, a desired cluster count $K$, and a number of points per cluster $\ell$.
\STATE 
\STATE Initialize centers $\mathcal{C}$ via \texttt{Gonzalez's Algorithm}.
\FOR{$i = 0, 1, \dots T$}
\STATE Compute cluster assignments $z_1, \dots, z_N$ such that
$$
z_i = \underset{j \in \{1,\dots, K\}}{\text{argmin}} d_F(\tau_i, c_j).
$$
\STATE Generate new cluster means $\{c_1, \dots, c_k\}$ by \texttt{Fr\'{e}chet Centering}.
\ENDFOR

\STATE \textbf{return} Centers $\mathcal{C}$ and Cluster Assignments $\mathcal{Z}$.
\end{algorithmic}
\label{alg:kmeans-algo}
\end{algorithm}

\section*{Experiments}

To test Algorithm \ref{alg:klmeans-algo}, I used Fred-Frechet \cite{Rohde_2023}, a lightweight Python package that implements the paper's major algorithms, as well as curve simplification and fast discrete and continuous Fr\'{e}chet distance in C for fast computation. To collect GPS trajectory data for my real-world data, I used the MyTracks iPhone app \cite{Stichling} to collect GPS data. 

\subsection*{Example 1: Synthetic Data}

When first learning about this method, I was immediately curious about how clustering performance would be impacted by the distance between and variance within trajectories. To test these results in an organized way, I first evaluated on a synthetic dataset collected as follows: I randomly sampled 100 quadratic curves from the random polynomial $f(x) = Ax^2 + Bx + C$, for $A \sim \mathcal{N} (1/2, \sigma^2)$, $B \sim \mathcal{N} (0, \sigma^2)$, and $C \sim \mathcal{N} (0,\sigma^2)$. I then rotate half of the curves counterclockwise by $\theta$, and take 200 evenly spaced points in $[-2, 2]$ to create each realized polygonal curve. I compute clusterings for data under different $\theta$ and $\sigma^2$, shown in Fig. \ref{fig:exp2}.

\begin{figure}[h]
\centering
\includegraphics[trim={13cm 7.cm 10cm 0cm},clip, width=1.\textwidth]{experiment_2_2_20.png}
\caption{Results of $(K,\ell)$-Means clustering with varying curve variance $\sigma^2$ and true distance $\theta$.}
\label{fig:exp2}
\end{figure}

We see that, in the low-noise regime, the clustering performs quite well even when the rotation is small, with the clustering performing essentially perfectly and recovering reasonable cluster means. However, in the moderate noise regime, clustering performance suffers even when curves are reasonably dissimilar. This is because, like regular $K$-means, $(K,\ell)$-means is highly sensitive to outliers, so if there exists a curve with low or negative curvature (a parabola with $A \leq 0$), the cluster will be forced to a strange region in space to accommodate it. This is exacerbated by the fact that $d_F$ takes a maximum over two curves' distances, so a single outlier on one curve corresponds to a large penalty to distance.

Still, it is worth noting that, across all methods, the recovered centers seem to retain parabolic structure. This is quite curious, and I wonder if it is an intrinsic feature of Fr\'{e}chet centering that the polynomial structure of the input is maintained for some reason.

Finally, I include in the appendix the same experiment with varying values of $\ell$ (5 and 10, Figs. \ref{fig:exp2b}, \ref{fig:exp2c}). We see that the hyperparameter choice does not have a large impact on the cluster assignments, but it does result in qualitatively different results. With the fast $\ell$-simplification and Fr\'{e}chet distance algorithms, varying $\ell$ by this amount does not result in a noticeably slower compute time.


\section*{Example 2: Clustering GPS Data}

Because I am ultimately interested in recovering characteristic trajectories from GPS data like hikes, I examine next a dataset composed of real trajectories. Due to time constraints, rather than collecting many hike samples from Alltrails or some other database, I simply collected 40 trajectories consisting of walks around Harvard Yard.

I considered four locations in and around the Yard:
Caf\'{e} Gato Rojo, Widener Library, Lamont Library, and the Science Center. Each was chosen due to their varying proximities and the complex shared substructures between connecting paths. To collect my dataset, I generated a 40-length random walk around the graph of the four buildings by the following scheme:

\begin{enumerate}
\item Randomly begin at one of the four buildings.
\item 40 times:
\begin{enumerate}
\item Randomly sample one of the remaining buildings..
\item Walk to that building and record the trajectory.
\end{enumerate}
\end{enumerate}

\begin{figure}[h]
\centering
\includegraphics[trim={0cm 0.cm 0cm 0cm},clip, width=0.6\textwidth]{data_map.png}
\caption{The 40 trajectories collected for my real-world dataset.}
\label{fig:data-map}
\end{figure}

The result was a set of trajectories shown in Fig. \ref{fig:data-map}. I also included one outlier (the rightmost trajectory around the yard) to see how it would impact clustering. The only preprocessing step that I applied was to scale the data to lie in $[0,1]^2$ and (optionally) remove the outlier. I evaluated the clusters visually for varying $K$ and $\ell$, both without the added outlier (Fig. \ref{fig:exp1}, appendix) and with it (Fig. \ref{fig:exp1b}, appendix). I also took the true $K=6$ result with $\ell = 30$ and visualized each individual cluster and mean, as shown in Fig. \ref{fig:exp1-decomp} (no outlier) and Fig. \ref{fig:exp1b-decomp} (with outlier).

\begin{figure}[h]
\centering
\includegraphics[trim={13cm 20.cm 10cm 18cm},clip, width=1.\textwidth]{experiment_1_decomp_NO_OUTLIER.png}
\caption{Clustering for $K=6$, $\ell = 30$ on the GPS data with no outlier, decomposed into each cluster.}
\label{fig:exp1-decomp}
\end{figure}

\begin{figure}[h]
\centering
\includegraphics[trim={13cm 20.cm 10cm 18cm},clip, width=1.\textwidth]{experiment_1_decomp_YES_OUTLIER.png}
\caption{Clustering for $K=6$, $\ell = 30$ on GPS data with an outlier added on the right, decomposed into each cluster.}
\label{fig:exp1b-decomp}
\end{figure}

We see that, regardless of whether the dataset has an outlier, the overarching structure of the walks does seem to be recovered. In particular, the routes that are very separated in space are sorted into different clusters, and the recovered cluster centers have relatively high fidelity to the original paths. Still, in regions where there is high spread between the paths, like the lower walks from Caf\'{e} Gato Rojo to Widener and Lamont, the method struggles across different $K$ settings to distinguish the two curves. 

The clustering also seems to struggle with shared subtrajectories. For example, we see that clusters 3 and 4 in the no-outlier (2 and 3 with outlier) both contain a mix of members of the Science Center-Lamont and Science Center-Widener paths. This is also possibly due to variance, because I purposefully walked different paths through the yard between Memorial Church and widener, yielding high spread in the middle of those paths. Interestingly, these high-variance examples are correctly identified and placed in a single cluster both with and without the outlier (clusters 4 and 2, respectively).

Curiously, the recovered center diagram seems slightly better in the case where the outlier is added, though the individual clusters look very similar. This is most likely due to the relative number of curves represented from each sub-trajectory, though I am not completely positive.

Finally, we see in Figs. \ref{fig:exp1} and \ref{fig:exp1b} that varying $\ell$ from 10 to 30 does not seem to yield a significant change in cluster location, though varying $K$ does cause the discovery of additional subtrajectories, especially in the long line connecting the Science Center to Lamont.



\section*{Strengths and Limitations}

Unfortunately, $(k,\ell)$-means inherits many of the limitations of standard $k$-means clustering. In particular, as \cite{buchin2019klcluster} mention, the quality of the clustering is highly sensitive to initial conditions, even with Gonzalez's algorithm. Likewise, you still must choose the number of clusters $K$, as well as a complexity $\ell$ for each cluster. This is potentially an even greater problem than in the $K$-means case, because trajectories may share common sub-structure whose clustering is ambiguous, such as the SC-Lamont and SC-Widener paths. Other algorithms, such as TRACLUS \cite{lee2007trajectory}, attempt to address this ambiguity by first partitioning the data into common segments, and then using those segments to hierarchically determine larger clusters, but $(K,\ell)$-means has no way to address this problem directly.

It also requires that data be distributed continuously in space, so it may not perform well in discrete state spaces, such as in many reinforcement learning problems. Finally, it relies on a series of approximations---for $\ell$-simplification, Fr\'{e}chet distance, and mean computation---many of which are still not fully solved problems in the general case for large datasets in high state dimensions. 

Still, it is reassuringly similar to the original $K$-means algorithm, and thus benefits from a shared intuition and highly interpretable cluster means compared to other methods. It shares the additional benefit that it returns means which can be used to cluster new trajectories zero-shot. Finally, it is by far the most compute-efficient mainstream algorithm for clustering trajectory data, and many of its approximations are technically optional, and can thus be substituted with slower, higher-quality algorithms which better approximate (or exactly compute) the values of interest.


\section*{Conclusion}

This concludes my paper on $(K,\ell)$-means clustering and the trajectory clustering problem. Thank you so much for reading, and for offering such an entertaining and well-taught class this year. I had an absolutely wonderful time.
\\\\
\noindent{All the best,}
\\\\
\noindent{Thomas}


\bibliographystyle{plain} % We choose the "plain" reference style
\bibliography{refs} % Entries are in the refs.bib file

\newpage

\section*{Additional Figures}


\begin{figure}[h]
\centering
\includegraphics[trim={13cm 7.cm 10cm 0cm},clip, width=1.\textwidth]{experiment_2_2_5.png}
\caption{Results of $(K,\ell)$-Means clustering with varying curve variance $\sigma^2$ and true distance $\theta$.}
\label{fig:exp2b}
\end{figure}

\begin{figure}[h]
\centering
\includegraphics[trim={13cm 7.cm 10cm 0cm},clip, width=1.\textwidth]{experiment_2_2_10.png}
\caption{Results of $(K,\ell)$-Means clustering with varying curve variance $\sigma^2$ and true distance $\theta$.}
\label{fig:exp2c}
\end{figure}

\begin{figure}[h]
\centering
\includegraphics[trim={13cm 4.cm 10cm 0cm},clip, width=1.\textwidth]{experiment_1_NO_OUTLIER.png}
\caption{Clustering results on GPS data with no outlier. The column corresponding to the correct number of clusters (6) is highlighted in gray.}
\label{fig:exp1}
\end{figure}


\begin{figure}[h]
\centering
\includegraphics[trim={13cm 4.cm 10cm 0cm},clip, width=1.\textwidth]{experiment_1_YES_OUTLIER.png}
\caption{Clustering results on GPS data with an outlier added on the right. The column corresponding to the correct number of clusters (6) is highlighted in gray.}
\label{fig:exp1b}
\end{figure}

# References + Further Reading

- [Agarwal et al., 2005. Near-Linear Time Approximation Algorithms for Curve Simplification.](https://link.springer.com/article/10.1007/s00453-005-1165-y)

- [Alt and Godau, 1995. Computing the Fréchet Distance Between Two Polygonal Curves.](https://www.worldscientific.com/doi/abs/10.1142/S0218195995000064)

- [Bringmann et al., 2019. Walking the Dog Fast in Practice: Algorithm Engineering of the Fréchet Distance](https://arxiv.org/abs/1901.01504)

- [Buchin et al., 2019a. Approximating $(k, \ell)$-Center Clustering for Curves](https://epubs.siam.org/doi/abs/10.1137/1.9781611975482.181)

- [Buchin et al., 2019b. klcluster: Center-based Clustering of Trajectories.](https://dl.acm.org/doi/abs/10.1145/3347146.3359111?casa_token=g_Zv1N2Rl44AAAAA:aE0Z50FvARw6oSo1SxznQqY3HUZscZs6mEzc--wrmshD16ONqsCs5aegD46vbORds5GAqnVcB0w)

- [Gonzalez, 1985. Clustering to minimize the maximum intercluster distance.](https://www.sciencedirect.com/science/article/pii/0304397585902245)

- [Lee et al., 2007. Trajectory Clustering: A Partition-and-Group Framework.](https://dl.acm.org/doi/abs/10.1145/1247480.1247546?casa_token=k6yQ3O8i7m4AAAAA:iXZsTf0LqpXuxvkYwsqNi6PFEA25KOb_WK2M71ZvAkE1r-tXOQrZf2ckH2u8qeiPKoW4eRlfvW4)

- [Fred-frechet Polygonal Curve Python Library](https://pypi.org/project/Fred-Frechet/)

- [MyTracks GPS Tracker](https://www.mytracks4mac.info/en)

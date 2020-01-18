---
layout: post
comments: true
title: Latex tricks for paper publication 
feature-img: "assets/img/pexels/book-glass.jpeg"    # Add a feature-image to the post
#thumbnail: "assets/img/thumbnail/desk-messy.jpeg"   # Add a thumbnail image on blog view
categories: [research]
tags: [latex, publication]
#series: "Blogging with Jekyll"
# Note: include series header in page body {% include series.html %}
---


For the first time I got these errors from EDAS paper submission platform for conferences. I could get the paper checking passed and note the tricks in this post.

```
1. pdf	columns	The paper has 1 column, but should have 2.	-
2. pdf	gutter	The gutter between columns is 0.01 inches wide (on page 3), but should be at least 0.12 inches.	-
```

Number 1 is wierd because the paper obviously has 2 columns. But fixing the 2. error will also solve it.

The 2. error is propably causes by a figure across 2 column as suggested by [1](https://www.cnblogs.com/quinn-yann/p/11279801.html). It is solved by adding a minipage. My previous submissions did not have any problem. 

{% raw %}
````
\begin{figure*}[th!]
  \centering
  \begin{minipage}[t]{\linewidth}    <----------- Add this
    \includegraphics[width=0.90\textwidth,height=5.9cm]{images/arche_overview}
  \caption{Multi-domain network slicing management architectures: federated (left), brokering (right). The dotted arrows show stakeholder-neutral communication and the solid arrows show operator specific network protocols}
\label{fig:multi_domain_arche}
%\parbox{6.5cm}{\small \hspace{1.5cm} }
\end{minipage}                  <---------- Add this
\end{figure*}
````
{% endraw %}

After fixing the figure, the 2. error still existed but at another place (as expected!), which was most likly caused by a table on the right column. It seemed the table's hline(s) expanded to the column space. Fortunately the table fited in a single colums so centering it solved the issues. 

``` latex
\begin{table}
\centering      <------------- Add this
\tiny
\resizebox{0.47\textwidth}{!}{%
\begin{tabular}{llll}
\hline
\textbf{QoS} & \textbf{Functional} & \textbf{Non-Functional} & \textbf{Other} \\
\hline
Bandwidth    & Input               & Cost                    & Timeliness     \\
Latency      & Output              & Availability            & Efficiency     \\
Throughput   & Precondition        & Responsiveness          & Geo-location    \\
Jitter       & Effect              & Reliability             &               \\
Capacity     &                     & Security                &               \\
\hline
\end{tabular}%
}
\caption{Some attributes}
\label{tab:attributes}
\end{table}
``` 

And the same gutter error at another place!. This time the long equation on the left column seemed to expand to the column space. The math spacing tricks from [2](https://www.egr.msu.edu/~renjian/latex.htm) helped.

``` latex
%%% Math Space
%\arraycolsep=5pt                  % distance between 2 columns
\thinmuskip=2mu                   % space between ordinary and operator atoms
\medmuskip=3mu plus 2mu minus 4mu % space between ordinary and binary atoms
\thickmuskip=4mu plus 5mu         % space between ordinary and relation atoms
%%%

```

After all the 2. error still persisted. Explicitly defining column separation space of 0.13 inch worked. Giving it 0.12 inch as required didn't work. 

``` latex
\setlength{\columnsep}{0.13in}
```

Following tricks are also useful to check the final document.

``` latex
%%% Debug
%\setlength{\columnseprule}{0.12in}
%\usepackage{showframe}
%%
```

Conclusion: 

Sometimes EDAS or some other pdf checking tools give strange errors. Most answers on the Internet suggest to ignore them. Some editors just trust the warning without checking the papers. Sometime obviously distorted papers still pass the check!. But closely following the warning and identify possible source of errors (mostly figures, tables, equations) will lead to solutions. 

## References 

1. https://www.cnblogs.com/quinn-yann/p/11279801.html
2. https://www.egr.msu.edu/~renjian/latex.htm

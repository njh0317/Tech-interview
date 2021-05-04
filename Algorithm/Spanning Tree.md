# Minimum Spanning Tree
## Spanning Tree
N개의 노드들의 connected graph 에서 얻을 수 있다.

### 특징

- N개의 node, N-1 개의 edges 로 된 Tree
- cycle은 그래프로부터 제거 - 한번 방문한 노드는 다시 방문 하지 않기 때문에 cycle 생기지 않음
- Depth-first traversal, Breath-first traversal 을 통해 확인

한 개 이상의 spanning tree는 같은 그래프에서 구할 수 있음

그래프가 connected 일 때, DFS 나 BFS는 암묵적으로 그래프 G의 edge 를 두 개의 집합으로 분리

- T(for tree edges) : edge 의 집합으로 사용되거나 search 도중 방문
- N(for nontree edges) : 남아있는 edge의 집합
- T의 edge는 G의 모든 vertex 들을 포함하는 트리를 형성

### 요약

- Spanning Tree는 G의 edge로만 구성되며, G안의 모든 vertex를 포함하는 트리
- 다시말해, 정점집합 v 를 그대로 두고(각 점들을 움직이지 않고) 간선(link)를 v-1개 (v개의 link 로 연결하면 cycle이 생김)만 남겨 트리가 되도록 만든 것

*주의
1. 사이클이 없는 연결 그래프
2. 정점의 개수가 V개이고, 간선의 개수가 V-1개

1은 트리가 맞지만 2는 트리가 아닐 수도 있다.

### 장점

- spanning tree에 nontree edge를 추가한다면 cycle이 됨
- spanning tree는 V(G') = V(G) 이고 G'는 connected 와 같은 G의 minimum 그래프 G'임
- |E(G')| = n-1 이라면 |V(G)|=n 임
    - edge를 최소한으로 사용하므로 minimum cost spanning tree이다.

## Minimum Spanning Tree
총 weighted가 최소인 spanning tree

- N개의 노드들의 weighted connected 그래프에서 얻을 수 있음
- unique 가 아님(여러개 일 수 있음)

**weighted undirected 그래프의 spanning tree 비용**

- spanning tree안에 있는 edge의 cost(weight)의 합
- 최소 비용의 spanning tree
- Kruskal, Prim, Sollin 알고리즘이 대표
- greedy method : 단계적으로 최적의 해결책을 구성하고, 어떤 기준을 사용하여 각 단계에서 최적의 결정을 내림

**spanning tree는 최소 비용을 기준으로 함**

- 그래프 안에 있는 edge만을 사용
- 정확하게 n-1개의 edge를 사용
- cycle을 유발할 수 있는 edge는 사용하지 않을 수 있음

**종류**

- Prim 알고리즘
- Kruskal 알고리즘
- 그래프 내의 간선의 숫자가 적은 희소 그래프(Sparse Graph)의 경우 Kruskal 알고리즘이 유리하고, 그래프 내의 간선의 숫자가 많은 밀집 그래프(Dense Graph)의 경우 Prim 알고리즘이 유리하다.

## Prim 알고리즘
**vertex 중심**

**시작 정점에서 부터 출발하여 신장트리 집합을 단계적으로 확장 해나가는 방법**

- 정점 선택을 기반으로 하는 알고리즘이다.
- 이전 단계에서 만들어진 신장 트리를 확장하는 방법이다.

### 순서

어느 한 점을 선택해 해당 점에서 갈 수 있는 최소의 edge를 결정한 다음 잇는다. 이은 다음 두 점에서 갈 수 있는 최소의 edge를 다시 결정해 잇고 이러한 방식으로 모든 vertex를 이을 때 까지(트리가 N-1 개의 간선을 가질 때 까지) 한다.

1. 그래프의 node 에서 시작
2. 모든 인접한 node들을 계산
3. 최소한의 edge의 weight를 선택
4. 선택된 인접한 노드에서 계속 시작
    - 선택되지 않은 모든 edge를 계산
    - 새로 인접한 모든 edge들을 계산
    - 최소한의 weight를 가진 edge를 선택, 단 cycle은 피함
5. 그래프의 모든 node 들을 포함할 때까지 진행하고 모든 노드는 1번만 방문하도록

### 구현방법&시간복잡도
알고리즘을 구현하는 방법에는 2가지 방법이 있다.
1. 배열을 사용, T와 각 노드를 연결하는 최소 간선 가중치를 찾는 방법 
    + 시간 복잡도 **`O(V^2)`**
2. 힙(min heap)을 사용해 최소의 간선 가중치를 찾는 방법
    + 시간 복잡도 **`O(ElogE)`**, 즉 **`O(ElogV)`** 이다.

* V = 정점 개수 , E = 간선의 개수

## Kruskal 알고리즘

**탐욕적인 방법(Greedy)을 이용하여 가중치 그래프의 모든 정점을 최소 비용으로 연결하는 방법**

- MST가 `최소 비용의 간선으로 구성됨`, `사이클을 포함하지 않음` 의 조건에 근거하여 각 단계에서 사이클을 이루지 않는 최소 비용 간선을 선택한다.
- 간선 선택을 기반으로하는 알고리즘이다.
- 이전 단계에서 만들어진 신장 트리와는 상관없이 무조건 최소 간선만 선택하는 방법이다.

prim 알고리즘과 다르게 어느 한 vertex를 선택하지 않고 전체 link 의 edge중 가장 작은 edge를 찾아 잇는다. 그 다음 가장 작은 edge를 이어가는 방식, 단 cycle 을 이루지 않는 안전 edge를 선택 해야한다.

### 순서

1. 그래프의 간선들을 가중치의 오름차순으로 정렬한다.
2. 정렬된 간선 리스트에서 순서대로 사이클을 형성하지 않는 간선을 선택한다.
    - 즉, 가장 낮은 가중치를 먼저 선택한다.
    - 사이클을 형성하는 간선을 제외한다.
    - 해당 간선을 현재의 MST집합에 추가한다.

### 특징

- 간선을 추가할 때 사이클이 생성되는지 체크해야함
    - 새로운 간선이 이미 다른 경로에 의해 연결되어 있는 정점들을 연결할 때 사이클을 형성함
    - 즉, 추가할 때 새로운 간선의 양끝 정점이 같은 집합에 속해 있으면 사이클이 형성됨
- Union-Find 알고리즘을 이용하여 사이클 생성 여부를 확인할 수 있다.

**union-find 자료구조를 사용한다.**

find - 부모 노드를 return하는 함수

```python
def find(v): # v 의 최상위 정점을 찾는다.
    if(parent[v]!=v):
        parent[v] = find(parent[v])
    return parent[v]
```

union - find 로 두 개의 노드를 알아낸 다음, 이 두 값이 다르다면 둘의 부모를 같도록 만듦(두 트리를 하나로 합쳐줌)

두 값이 같다면 cycle 을 형성한다는 의미이므로 연결하지 않음

```python
def union(v, u):#v, u 연결
    root1 = find(v)
    root2 = find(u)

    if(root1 != root2):
        #짧은 트리의 루트가 긴 트리의 루트를 가리키도록
        if(rank[root1] > rank[root2]):
            parent[root2] = root1
        else:
            parent[root1] = root2
            if rank[root1] == rank[root2]:
                rank[root2] +=1
```

disjoint set : **서로 중복되지 않는 부분** 집합들로 나눠진 원소들에 대한 정보를 저장하고 조작하는 자료구조

- 즉, 공통 원소가 없는, 즉 "상호 배타적"인 부분 집합들로 나눠진 원소들에 대한 자료구조
- disjoint set = 서로소 집합 자료구조

### 프림과 차이점
프림은 원리상 노드를 중심으로 간선을 탐색

크루스칼은 간선을 중심으로 탐색하면서 노드를 확인용으로만 사용한다.

### 시간복잡도
+ 운이 좋지 않으면 모든 edge를 다 봐야 하기 때문에 O(M) 이고 findSet은 O(N)이 걸리기 때문에 최악에 상황에서 **`O(MN)`** 이 될 수 있다.

+ Union-Find알고리즘을 사용하면 Kruskal 알고리즘의 시간 복잡도는 간선을 정렬하는 시간에 좌우된다. 즉, 간선 E개를 효율적으로 정렬한다면 **`O(ElogE)`**
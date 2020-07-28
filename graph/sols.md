# Solutions
Below are all the solutions (some code omitted), with explanation, to the graph questions that I have completeted from CSES.

## [Counting Rooms](https://cses.fi/problemset/task/1192)
Given an *nxm* map of a building consisting of walls (#) and floor tiles (.), we need to count the number of rooms in the building. A room is nothing more than adjacent
floor tiles surrounded by walls. You can think of the rooms as islands in an ocean. Counting the rooms is easy, simply iterate over the grid, stopping when we find a floor tile.
From this position in the grid, we launch a Depth-First Seach. Why? Because we need to see if there are any floor tiles adjacent to us. These floor tiles, obviously, belong to the same
room, and we must "remove" them from the grid so we don't count them during our iteration later on. After repeating this process for the rest of the grid, the total number of rooms
is simply the number of times we call our DFS algorithm.

### Code
```C++
void fillRoom(vector<vector<char> >& board, int r, int c) {
	if(r < 0 || r >= board.size() || c < 0 || c >= board[r].size()) return;
	if(board[r][c] == '#') return;
	board[r][c] = '#';
	fillRoom(board, r-1, c); // up
	fillRoom(board, r, c+1); // right
	fillRoom(board, r+1, c); // down
	fillRoom(board, r, c-1); // left
}

ll countRooms(vector<vector<char> >& board) {
	ll rooms = 0;
	for(int r = 0; r < board.size(); r++) {
		for(int c = 0; c < board[r].size(); c++) {
			if(board[r][c] == '.') {
				rooms++;
				fillRoom(board, r, c);
			}
		}
	}
	return rooms;
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);

	ll n, m;
	cin >> n >> m;
	vector<vector<char> > board(n, vector<char>(m));
	for(int i = 0; i < n; i++) {
		for(int j = 0; j < m; j++) {
			cin >> board[i][j];
		}
	}
	
	ll ans = countRooms(board);
	cout << ans << endl;	
	return 0;
}	
```
In the code above, the `main` method is used to read in the input to the problem, the `countRooms` method iterates over the board to count the rooms, and the `fillRoom` method
is our DFS method described above.

## [Labyrinth](https://cses.fi/problemset/task/1193)
Again, we are given an *nxm* map consisting of walls and floor tiles, however, this time, we need to find the shortest path, if one exists, from the starting point A, to the end point B and print out such a path.
Since we are interested in the *shortest* path, you should immediately be thinking Breadth-First Search.

### Code
I have omitted the code for this problem because this is a rather straightforward BFS search. The tricky part of this problem is recovering the path. If you are having trouble with this, I would reccommend reading over [benq's solution](https://github.com/thcy/CSES-Solutions/blob/master/Graph%20Algorithms/Labyrinth%20(1193)/Benq.cpp), who creates another *nxm* table consisting of pairs of values. The first value being another pair consisting of integers to store the "parent's" board position, and the second value being a character describing the direction the parent took to reach the current board position. Also, depending on your implemenation, you may need to reverse your final path.

## [Building Roads](https://cses.fi/problemset/task/1666)
Given *n* cities and *m* roads connecting these cities, we need to find the smallest number of roads such that there is a road between any two cities. Note that roads are bi-directitonal
and that any solution is valid.

To help break this problem down, consider the following example. Suppose you have six cities (labeled 1 through 6) and you have the edge relation given by 
Roads = {(1,2), (3,4), (5,6)} (recall that the graph is undirected). Currently, this graph is *disconnected*. If we want to be able to reach any city from any other city, we
simply need to make the graph *connected*. To do this, we can add the edges (2,3) and (4,5) or we can add (1,6) and (6,3), etc. But how many roads do we need? Looking at our example,
there are three connected components {{1,2}, {3,4}, {5,6}} and we need two edges to connect the graph. In general, if our graph has *k* connected components, we need exactly *k-1* roads to make our graph connected.

Our solution will count the number of roads and create the roads simultaneously. Let `k = 0` denote the number of roads we will need. Following the example above, we will count the number of connected components
in our graph by performing (B/D)FS from each unvisited city, incrementing `k` each time we call our (B/D)FS. By the end of our algorithm, the number of roads we will need is `k-1`. 
To find the necessary roads, we need to keep track of just one city from one of the components. We denote this city as the `head` city. Whenever we perform (B/D)FS from a new city *i*,
we will create the road (head, i), save it to a string, and then set `head = i`.

### Code
```C++
void dfs(vector<int> graph[], int source, vector<int>& visited) {
	visited[source] = true;
	for(auto it = graph[source].begin(); it != graph[source].end(); it++) {
		int v = *it;
		if(!visited[v]) {
			dfs(graph, v, visited);
		}
	}
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);
	
	// initialize data structures
	int n, m, k = 0;
	cin >> n >> m;
	vector<int> *graph = new vector<int>[n+1];
	vector<int> visited(n+1, 0);

	// read input	
	for(int i = 0; i < m; i++) {
		int a, b;
		cin >> a >> b;
		graph[a].push_back(b);
		graph[b].push_back(a);
	}

	int head = -1;
	string roads = "";
	for(int i = 1; i <= n; i++) {
		if(!visited[i]) {
			k++;
			if(head != -1) {
				roads += to_string(head)+" "+to_string(i)+"\n";
			}
			dfs(graph, i, visited);
			head = i;
		}
	}	

	cout << (k-1) << endl;
	cout << roads;
	return 0;
}
```
Following up on our example, this code would create the roads (1,3) and (3,5). Again, please note that this solution is not unique.

## [Message Routes](https://cses.fi/problemset/task/1667)
Given *n* computers labeled 1 through *n*, we need to determine if there is a path from computer 1 to computer *n*. If such a path exists, we need to print out the 
shortest path.

This is again nothing more than a simple Breadth-First Search. And, at the risk of contradicting myself, the code is provided below. In my solution, I keep track 
of three auxiliary arrays. *seen* is used to keep track of vertices that have already been visited (duh), *dist* is used to keep track of all the distances to each
of the vertices (duh), and *prev* which is used to store parent vertex for each vertex. Initially, all distances are set to 0, and the parent of the source node (in our case node 1) is -1 as it has no parent. If we find a valid path, we simply iterate over the *prev* array staring from node *n*.

## Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);

	const int v = 100001;
	vector<int> graph[v];
	int seen[v] = {0};
	int dist[v] = {0};
	int prev[v] = {0};
	queue<int> frontier;

	int V, E;
	cin >> V >> E;	
	for(int i = 0; i < E; i++) {
		int a, b;
		cin >> a >> b;
		graph[a].push_back(b);
		graph[b].push_back(a);
	}
	
	frontier.push(1);
	prev[1] = -1;
	while(!frontier.empty()) {
		int u = frontier.front();
		frontier.pop();
		seen[u] = 1;
		if(u == V) break;
		for(auto& v: graph[u]) {
			if(!seen[v]) {
				prev[v] = u; // update prev
				dist[v] = dist[u] + 1; // update dist
				seen[v] = 1; // update seen
				frontier.push(v);
			}
		}
	}

	if(!seen[V]) cout << "IMPOSSIBLE" << endl;
	else {
		int d = dist[V];
		vector<int> path;
		path.push_back(V);
		while(V != 1) {
			path.push_back(prev[V]);
			V = prev[V];
		}
		reverse(all(path));
		cout << d+1 << endl;
		for(auto& e: path) cout << e << " ";
	}


	return 0;
}
```


## [Round Trip](https://cses.fi/problemset/task/1669)
Given *n* cities and *m* roads between them, our task is to design a round trip that begins in a city, goes through two or more other cities, and finally returns to the starting city. Every intermediate city on the route has to be distinct. Any valid solution is acceptable.

This problem reduces to finding a cycle in an undirected graph. We can do this simply by performing a depth-first search. Staring from some node *u*, recursively call our DFS method on the univisited neighbors. If we reach some neighbor *v* that has already been visited and *v* is *not* the parent of *u*, we have a cycle. Recovering the cycle is easy, by keeping two variables, `cycleStart` and `cycleEnd`, we simply iterate backwards from `cycleEnd` (we can do this by keeping track of the parent nodes) until we reach `cycleStart`.

## Code
```C++
int cycleStart = -1;
int cycleEnd = -1;

bool hasCycle(vector<int> graph[], int visited[], int prev[], int source, int parent) {
	visited[source] = true;
	for(auto& v: graph[source]) {
		if(!visited[v]) {
			prev[v] = source;
			if(hasCycle(graph, visited, prev, v, source)) return true;
		}
		else if(v != parent) {
			cycleStart = v;
			cycleEnd = source;
			return true;
		}
	}
	return false;
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);
	
	const int v = 100001;
	vector<int> graph[v];
	int visited[v] = {0};
	int prev[v] = {0};

	int V, E;
	cin >> V >> E;
	for(int i = 0; i < E; i++) {
		int a, b;
		cin >> a >> b;
		graph[a].push_back(b);
		graph[b].push_back(a);
	}

	for(int i = 1; i <= V; i++) {
		if(!visited[i]) {
			if(hasCycle(graph, visited, prev, i, -1)) break;
		}
	}
	
	if(cycleStart == -1) cout << "IMPOSSIBLE" << endl;
	else {
		vector<int> path;
		path.push_back(cycleEnd);
		while(cycleEnd != cycleStart) {
			path.push_back(prev[cycleEnd]);
			cycleEnd = prev[cycleEnd];
		}
		reverse(all(path));
		path.push_back(cycleStart);
		cout << path.size() << endl;
		for(auto& e: path) cout << e << " ";
	}
	return 0;
}	
```

## [Monsters](https://cses.fi/problemset/task/1194/)
Given an *n*x*m* grid consisting of walls (#), floor tiles (.) and monsters (M), our goal is to find a path out of the labyrinth such that we never share a square with a monster. This problem is not so simple, and I've made my solution verbose on purpose. The idea for this question is keep a track of all the monsters and perform a Breadth-First Search from all of them simultaneously, keeping track of the minimum number of steps necessary for any one monster to reach a given location in the grid. In my solution, these distances are maintained in `monsterSteps[mxD][mxD]`. After we've considered all possible locations the monsters can travel, we perform a Breadth-First Search starting from the source location (marked as 'A' in the grid). This BFS search should be familiar; we consider the current position, check if we are at the goal node, and, if we are not, we add all valid neighbors. This time, however, a neighbor is valid if and only if it is not a wall, and we can reach the neighbor before any monster can. Once we find a valid path, we can simply backtrack as we've done so many times before.

### Code
```C++
const int mxD = 1000;
char grid[mxD][mxD]; // holds the maze itself
ll monsterSteps[mxD][mxD]; // will hold the monster's steps
pair<pii, char> parent[mxD][mxD]; // will hold the parent of each square the user takes
ll dist[mxD][mxD];
int dx[4] = {-1, 0, 1, 0};
int dy[4] = {0, 1, 0, -1};
string dir = "URDL";

bool isValid(int r, int c, int N, int M) {
	return (r >= 0 && r < N && c >= 0 && c < M);
}

bool isGoal(int r, int c, int N, int M) {
	return (r == 0 || c == 0 || r == N-1 || c == M-1);
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	ll N, M;
	pii source, goal;
	queue<pii> frontier, monsterFrontier;
	cin >> N >> M;

	// read in the maze
	for(int i = 0; i < N; i++) {
		for(int j = 0; j < M; j++) {
			cin >> grid[i][j];
			dist[i][j] = INF;
			monsterSteps[i][j] = INF;
			if(grid[i][j] == 'A') {
				dist[i][j] = 0;
				source = {i,j};
				frontier.push({i,j});
			}
			else if(grid[i][j] == 'M') {
				monsterFrontier.push({i,j});
			}
		}
	}

	// perform BFS from each of the monsters
	// keep track of the minimum time to get to a square
	int currentMonsterStep = 0;
	while(!monsterFrontier.empty()) {
		int monsterSize = monsterFrontier.size();
		for(int k = 0; k < monsterSize; k++) {
			pii currentMonster = monsterFrontier.front();
			monsterFrontier.pop();
			int monsterRow = currentMonster.first;
			int monsterCol = currentMonster.second;
			
			// check if we can reach square in better time
			// if we can't do NOT add neighbors
			if(monsterSteps[monsterRow][monsterCol] <= currentMonsterStep) {
				continue;
			}
			monsterSteps[monsterRow][monsterCol] = currentMonsterStep;

			// add valid neighbors
			for(int i = 0; i < 4; i++) {
				int newRow = monsterRow + dx[i];
				int newCol = monsterCol + dy[i];

				if(isValid(newRow, newCol, N, M) && grid[newRow][newCol] != '#') {
					monsterFrontier.push({newRow, newCol});
				}
			}
		}
		// increment step
		currentMonsterStep++;
	}

	// now perform BFS from person
	// moving only to positions where currentPersonStep < monsterStep
	int currentPersonStep = 0;
	bool hasPath = false;
	while(!frontier.empty()) {
		int personSize = frontier.size();
		for(int k = 0; k < personSize; k++) {
			pii currentPerson = frontier.front();
			frontier.pop();
			int personRow = currentPerson.first;
			int personCol = currentPerson.second;

			// check if we are at a goal state
			if(isGoal(personRow, personCol, N, M)) {
				hasPath = true;
				goal = currentPerson;
				break;
			}

			// if not at goal, add all valid neighbors
			for(int i = 0; i < 4; i++) {
				int newRow = personRow + dx[i];
				int newCol = personCol + dy[i];

				// determine if position is valid
				if(isValid(newRow, newCol, N, M) && grid[newRow][newCol] != '#' && dist[newRow][newCol] == INF) {
					// don't go to positions where a monster WILL be
					if(monsterSteps[newRow][newCol] > currentPersonStep+1) {
						// update data structures
						parent[newRow][newCol] = {currentPerson, dir[i]};
						dist[newRow][newCol] = dist[personRow][personCol] + 1;
						frontier.push({newRow, newCol});
					}
				}
			}
		}
		// increment step
		currentPersonStep++;
	}

	// now check if we have a valid path
	if(!hasPath) cout << "NO" << endl;
	else {
		cout << "YES" << "\n" << dist[goal.first][goal.second] << endl;
		// recover the path
		vector<char> path;
		while(goal != source) {
			path.push_back(parent[goal.first][goal.second].second);
			goal = parent[goal.first][goal.second].first;
		}
		// reverse the path
		reverse(all(path));
		for(auto& e: path) {
			cout << e;
		}
	}
	return 0;
}
```

## [Shortest Routes I](https://cses.fi/problemset/task/1671)
Given a graph consisting of *n* nodes, labeled from 1 to *n*, and *m* edges, our goal is to find the shortest path from node 1 to all other nodes.

This problem can be solved by using [Dijkstra's Algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm). We present the algorithm here.

### Dijkstra's Algorithm
Dijkstra's Algorithm can be used on both undirected and directed graph. If all the edge weights are one, this algorithm reduces to a traditional Breadth-First Search. An important note about this algorithm is that it does **NOT** work on graphs whose edges weights are negative! Though the algorithm will terminate if negative edge weights are present, it is not guaranteed that the results will be accurate.

## Code
Note that in our implementation, after we relax an edge, we push the pair {a,b} onto the priority queue, where a is the distance to node b, and b is the node.
Moreover, we push the distance *-a* onto the queue because C++ uses a max-heap.

```C++
vector<ll> dijkstra(vector<pair<int, int> > adj[], int source, int V) {
	vector<ll> distances(V+1, 0); // vector to hold final distances
	vector<bool> processed(V+1, false); // vector to keep track of seen nodes
	// initialize all distances to INF
	for(int i = 1; i <= V; i++) {
		distances[i] = LONG_MAX;
	}
	distances[source] = 0; // distance of source is 0

	// maintain priority queue to perform dijkstra
	priority_queue<pair<ll ,ll> > pq;
	pq.push({0,source}); // push source node with distance 0
	while(!pq.empty()) {
		int u = pq.top().second; // get node at top of pq
		pq.pop();
		if(processed[u]) continue;
		processed[u] = true;
		for(auto& pair: adj[u]) {
			int v = pair.first; // get node incident to u
			int w = pair.second; // get cost of traveling to node v
			if(distances[u] + w < distances[v]) {
				distances[v] = distances[u] + w; // update distance to v
				pq.push({-distances[v], v}); // push v with new distance onto pq
				// we push negative distances because by default, c++ uses a max
				// priority queue while we need a min priority queue
			}
		}
	}
	return distances;
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);
	
	int V, E;
	cin >> V >> E;
	vector<pair<int, int> > *adj = new vector<pair<int, int> >[V+1];
	for(int i = 0; i < E; i++) {
		int a, b, c;
		cin >> a >> b >> c;
		adj[a].push_back({b,c});
	}
	vector<ll> distances = dijkstra(adj, 1, V);
	for(int i = 1; i <= V; i++) cout << distances[i] << " ";	
	return 0;
}	
```

## [Shortest Routes II](https://cses.fi/problemset/task/1672)
Similar to the previous question, we are given *n* nodes labeled from 1 to *n* and *m* edges connected these nodes. Our task is to process *q* queries of the form
*a b* where you have to determine the shortest path between nodes *a* and *b*.

Dijkstra's Algorithm is commonly known as a *Single Source Shortest Path Algorithm.* This time, we will need something a little more stronger. Fortuneatlly for us, there exists an *All Pairs Shortest Path Algorithm*, more commonly known as the [Floyd-Warshall Algorithm](https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm). We present the algorithm here.

### Floyd-Warshall
The Floyd-Warshall Algorithm is both spectacular and incredibly easy to implement, however, its time complexity is `O(n^3)`, which makes it rather inefficient for large graphs. It also forces us to use an adjacency matrix. You may wish to keep these notes in consideration when looking to implement this algorithm!

## Code 
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);

	// create graph
	const int v = 501;
	ll graph[v][v];

	// read in input
	ll V, E, q, a, b, c;
	cin >> V >> E >> q;
	for(int i = 1; i <= V; i++) {
		for(int j = 1; j <= V; j++) {
			graph[i][j] = INF;
		}
		graph[i][i] = 0;
	}

	for(int i = 0; i < E; i++) {
		cin >> a >> b >> c;
		graph[a][b] = min(graph[a][b], c);
		graph[b][a] = min(graph[b][a], c);
	}

	// floyd-warshall
	for(int k = 1; k <= V; k++) {
		for(int i = 1; i <= V; i++) {
			for(int j = 1; j <= V; j++) {
				graph[i][j] = min(graph[i][j], graph[i][k] + graph[k][j]);
			}
		}
	}

	for(int i = 0; i < q; i++) {	
		cin >> a >> b;
		cout << ((graph[a][b] == INF) ? -1: graph[a][b]) << endl;
	}
	return 0;
}	
```

## [Cycle Finding](https://cses.fi/problemset/task/1197)
Given a directed graph, your task is to find out if it contains a negative cycle, and also give an example of such a cycle.

This problem is a direct application of the [Bellman Ford Algorithm](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm).

## Code
See [this](https://cp-algorithms.com/graph/finding-negative-cycle-in-graph.html) for the code.

## [Round Trip II](https://cses.fi/problemset/task/1678)
As was the case with Round Trips, our task is to design a round trip that begins in a city, goes through one or more other cities, and finally returns to the starting city. Every intermediate city on the route has to be distinct. The difference between this question and its predecessor is that now our graph is *directed*.

### Finding Cycles in a Directed Graph
Though finding cycles in a directed graph is no harder than that of unidrected graphs, it is slightly more subtle, and deserves some explanation.
Consider a graph *G* consisting of *n* nodes and *m* edges. To look for a cycle, we will use three colors to indicate the state of a node. Color 0 (white) means the node has been univisted (this is the initial state of all the nodes), color 1 (grey) means the node has been discovered, and color 2 (black) means the node has been completely explored. We start by performing the ususal DFS from any unvisited node *u*. We label the color of *u*, which was 0 to start, to 1, and we look at *u's* neighbors. We recursively call our DFS methtod for all neighbors *v* of *u* such that `color[v] == 0` (that is, we recur over all unvisited neighbors). If we have some neighbor *v* of *u* such that `color[v] == 1`, then we have a cycle. This is commonly known as a *back edge* in a directed graph. You can learn more about this algorithm [here](https://www.youtube.com/watch?v=AK7BuT5MgU0).

Now our job is simple. Look for a directed cycle using the steps above. If we have a cycle, recover it as we've done before.

## Code
```C++
int cycleStart = -1;
int cycleEnd = -1;
bool dfs(vector<int> graph[], int color[], int prev[], int u) {
	color[u] = 1;
	for(auto& v: graph[u]) {
		if(color[v] == 0) {
			prev[v] = u;
			if(dfs(graph, color, prev, v)) return true;
		}
		else if(color[v] == 1) {
			cycleStart = v;
			cycleEnd = u;
			return true;
		}
	}
	color[u] = 2;
	return false;
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	const int mxN = 10e5+1;
	static vector<int> graph[mxN];
	static int color[mxN] = {0};
	static int prev[mxN] = {0};

	int V, E, a, b;
	cin >> V >> E;
	for(int i = 0; i < E; i++) {
		cin >> a >> b;
		graph[a].push_back(b);
	}

	// perform dfs
	for(int i = 1; i <= V; i++) {
		if(color[i] == 0) {
			prev[i] = -1;
			if(dfs(graph, color, prev, i)) break;
		}
	}	

	if(cycleStart == -1) cout << "IMPOSSIBLE" << endl;
	else {
		////cout << "cycle starting at " << cycleStart << " and ending at " << cycleEnd << endl;
		vector<int> path;
		path.push_back(cycleEnd);
		while(cycleEnd != cycleStart) {
			path.push_back(prev[cycleEnd]);
			cycleEnd = prev[cycleEnd];
		}
		reverse(all(path));
		path.push_back(cycleStart);
		cout << path.size() << endl;
		for(auto& e: path) cout << e << " ";
	}
	return 0;
}	
```

## [Course Schedule](https://cses.fi/problemset/task/1679)
Our task is as follows: you have to complete *n* courses. There are *m* requirements of the form "course a has to be completed before course b". Your task is to find an order in which you can complete the courses.

### Topological Sort
Our task reduces to finding a [topological ordering](https://en.wikipedia.org/wiki/Topological_sorting) of the courses. It can be proven that every Directed-Acyclic Graph (DAG) has a topological ordering of its vertices. Our algorithm must do two things: make sure that our graph does not contain a cycle, and find an ordering of the nodes. In the previous problem, we learned how to check for directed cycles. As it turns out, we can actually find a topological ordering of the nodes by adding all nodes to a list as soon as they get colored black. This produces a *reverse topological ordering*.
## Code
Note that our code for this problem is almost identical to the previous question!
```C++
bool cycle = false;
void dfs(vector<int> graph[], int color[], vector<int>& top, int u) {
	color[u] = 1;
	for(auto& v: graph[u]) {
		if(color[v] == 0) {
			dfs(graph, color, top, v);
		}
		else if(color[v] == 1) {
			cycle = true;
			return;
		}
	}
	top.push_back(u);
	color[u] = 2;
}


int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	const int mxN = 10e5+1;
	static vector<int> graph[mxN];
	static int color[mxN] = {0};
	vector<int> top;

	int V, E, a, b;
	cin >> V >> E;
	for(int i = 0; i < E; i++) {
		cin >> a >> b;
		graph[a].push_back(b);
	}

	// perform dfs
	for(int i = 1; i <= V; i++) {
		if(color[i] == 0) {
			dfs(graph, color, top, i);
		}
		if(cycle) break;
	}

	if(cycle) cout << "IMPOSSIBLE" << endl;
	else {
		reverse(all(top));
		for(auto& e: top) cout << e << " ";
	}
	
	return 0;
}
```

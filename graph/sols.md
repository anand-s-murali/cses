# Solutions
Below are all the solutions (some code omitted), with explanation, to the graph questions that I have completeted from CSES.

## [Counting Rooms](https://cses.fi/problemset/task/1192)

### Explanation
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
Give *n* computers labeled 1 through *n*, we need to determine if there is a path from computer 1 to computer *n*. If such a path exists, we need to print out the 
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

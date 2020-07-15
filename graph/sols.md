# Solutions
Below are all the solutions, with explanation, to the graph questions from CSES.

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
Since we are interested in the *shortest* path, you should immediately be thinking Breadth-First Search. That's exactly what we will do. Simply perform BFS from A and attempt to reach B.
If we can reach B, we have, by definition, found the shoretst path. If we cannot, no such path exists. Now we need to recover our path. To do this, we will maintain another *nxm* grid
which will hold a *pair* of values. Our pair will consists of a (another) pair of integers to hold the previous board position, and a character to tell us which direction we traveled to get the the current position.
In C++ we can model this grid as such `pair<pair<int,int>, char> prev[n][m].` Now, once we find our path, we simply need to look at our table for the direction we took and the board position we came from, until we get back to A.

### Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false);
    cin.tie(NULL);

	// initialize data structures
	pair<int,int> source, goal;
	char grid[1000][1000];
	int dist[1000][1000];	
	pair<pii, char> prev[1000][1000];
	int dx[4] = {-1,0,1,0};
	int dy[4] = {0, 1, 0, -1};
	string ds = "URDL";
	queue<pii> frontier;

	// read in grid	
	int N, M;
	cin >> N >> M;
	for(int i = 0; i < N; i++) {
		for(int j = 0; j < M; j++) {
			dist[i][j] = -1;
			cin >> grid[i][j];
			if(grid[i][j] == 'A') source = {i,j};
			if(grid[i][j] == 'B') goal = {i,j};
		}
	}

	dist[source.first][source.second] = 0; // distance of source is trivially 0

	// begin BFS from source
	frontier.push(source);
	while(!frontier.empty()) {
		pii curr = frontier.front();
		int x = curr.first, y = curr.second;
		frontier.pop();

		if(curr == goal) break; // check if we have found the goal

		// add neighbors
		for(int i = 0; i < 4; i++) {
			int nx = x + dx[i];
			int ny = y + dy[i];
			// ignore invalid positions
			if(nx < 0 || nx >= N || ny < 0 || ny >= M) continue;
			if(grid[nx][ny] == '#' || dist[nx][ny] != -1) continue;
		
			// update tables and add next position to queue	
			prev[nx][ny] = {curr, ds[i]};
			dist[nx][ny] = dist[x][y] + 1;
			frontier.push({nx,ny});
		}		
	}

	if(dist[goal.first][goal.second] != -1) {
		// recover path
		string path = "";
		int d = dist[goal.first][goal.second];
		while(goal != source) {
			path += prev[goal.first][goal.second].second;
			goal = prev[goal.first][goal.second].first;
		}
		reverse(all(path));
		cout << "YES" << endl;
		cout << d << endl;
		cout << path << endl;
	}
	else cout << "NO" << endl;

	return 0;
}
```
## [Building Roads](https://cses.fi/problemset/task/1666)
Given *n* cities and *m* roads connecting these cities, we need to find the smallest number of roads such that there is a road between any two cities. Note that roads are bi-directitonal
and that any solution is valid.

To help break this problem down, consider the following example. Suppose you have six cities (labeled 1 through 6) and you have the edge relation given by 
Roads = {(1,2), (3,4), (5,6)} (recall that the graph is undirected). Currently, this graph is *disconnected*. If we want to be able to reach any city from any other city, we
simply need to make the graph *connected*. To do this, we can add the edges (2,3) and (4,5) or we can add (1,6) and (6,3), etc. But how many roads do we need? Looking at our example,
there are three connected components {{1,2}, {3,4}, {5,6}} and we need two edges to connect the graph. In general, if our graph has *k* connected components, we need exactly *k-1* roads to make our graph connected.

Our solution will count the number of roads and create the roads simultaneously. Let `k = 0` denote the number of roads we will need. Following the example above, we will count the number of connected components
in our graph by performing (B/D)FS from each unvisited city, incrementing `k` each time we call our (B/D)FS. By the end of our algorithm, the number of roads we will need is `k-1`. 
To find the necessary roads, we need to keep track of just one city from one of the components. We denote this city as the `head`. Whenever we perform (B/D)FS from a new city *i*,
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

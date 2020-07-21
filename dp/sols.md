# Solutions
Below are all the solutions, with explanation, to the DP questions that I have completeted from CSES.

## [Dice Combinations](https://cses.fi/problemset/task/1633)
### Problem Statement
Your task is to count the number of ways to construct sum n by throwing a dice one or more times. Each throw produces an outcome between 1 and 6.

### Explanation

### Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	ll n, mod = 1e9+7;
	cin >> n;
	vector<ll> dp(n+1, 0);
	dp[0] = 1;

	for(int i = 1; i <= n; i++) {
		for(int j = 1; j <= 6 && i >= j; j++) {
			dp[i] = (dp[i]%mod + dp[i-j]%mod)%mod;
		}
	}
	cout << dp[n] << endl;
	return 0;
}	
```

## [Minimizing Coins](https://cses.fi/problemset/task/1634)
### Problem Statement
Consider a money system consisting of n coins. Each coin has a positive integer value. Your task is to produce a sum of money x using the available coins in such a way that the number of coins is minimal.
For example, if the coins are {1,5,7} and the desired sum is 11, an optimal solution is 5+5+1 which requires 3 coins.

### Explanation

### Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	ll n, x;
	cin >> x >> n;
	vector<ll> coins(x), dp(n+1, INF-1);
	
	for(int i = 0; i < x; i++) {
		cin >> coins[i];
	}

	dp[0] = 0; // base case: 0 ways to make sum 0
	for(int i = 1; i <= n; i++) {
		for(auto c: coins) {
			if(i >= c) {
				dp[i] = min(dp[i], dp[i-c]); // only update dp if we can make valid difference
			}
		}
		dp[i] += 1; // we add one to symbolize actually adding a coin
	}

	cout << ((dp[n] == INF) ? -1: dp[n]) << endl;
	return 0;
}	
```

## [Grid Paths](https://cses.fi/problemset/task/1638)
### Problem Statement
Consider an *n×n* grid whose squares may have traps. It is not allowed to move to a square with a trap.
Your task is to calculate the number of paths from the upper-left square to the lower-right square where you only can move right or down.

### Explanation

### Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	ll n, mod = 1e9+7;
	cin >> n;
	vector<vector<char> > grid(n, vector<char>(n));
	vector<vector<ll> > dp(n, vector<ll>(n,0));
	for(int i = 0; i < n; i++) {
		for(int j = 0; j < n; j++) {
			cin >> grid[i][j];
		}
	}

	// first row
	bool ob = false;
	for(int c = 0; c < n; c++) {
		if(grid[0][c] == '*') {
			ob = true;
			dp[0][c] = 0;
		}
		else if(ob) dp[0][c] = 0;
		else dp[0][c] = 1;
	}

	// first column
	ob = false;
	for(int r = 0; r < n; r++) {
		if(grid[r][0] == '*') {
			ob = true;
			dp[r][0] = 0;
		}
		else if(ob) dp[r][0] = 0;
		else dp[r][0] = 1;
	}	

	// rest of grid
	for(int r = 1; r < n; r++) {
		for(int c = 1; c < n; c++) {
			dp[r][c] = (grid[r][c] == '*') ? 0: (dp[r-1][c]%mod + dp[r][c-1]%mod) % mod;
		}
	}
	cout << dp[n-1][n-1] << endl;

	return 0;
}
```

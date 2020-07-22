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

## [Coin Combinations I](https://cses.fi/problemset/task/1635/)
### Problem Statement
Consider a money system consisting of *n* coins. Each coin has a positive integer value. Your task is to calculate the number of distinct ways you can produce a money sum *x* using the available coins.

### Explanation
This questiton is exactly the same as Dice Combinations, only this time, we iterate over the available denominations.

### Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	ll x, n, m = 1e9+7;
	cin >> x >> n;
	vector<ll> coins(x), dp(n+1, 0);
	for(int i = 0; i < x; i++) {
		cin >> coins[i];
	}

	dp[0] = 1; // there is exactly one way to construct the empty set!
	for(int i = 1; i <= n; i++) {
		for(int j = 0; j < (int)coins.size(); j++) {
			ll c = coins[j];
			if(i >= c) {
				dp[i] = (dp[i] + dp[i-c]) % m;
			}
		}	
	}
	
	cout << dp[n] << endl;

	return 0;
}
```

## [Coin Combinations II](https://cses.fi/problemset/task/1636/)
### Problem Statement
Consider a money system consisting of n coins. Each coin has a positive integer value. Your task is to calculate the number of *distinct ordered ways* you can produce a money sum x using the available coins.

### Explanation

### Code
```C++
int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	int x, n, m = 1e9+7;
	cin >> n >> x;
	vector<int> coins(n);
	for(int& v: coins) cin >> v;

	vector<vector<int> > dp(n+1, vector<int>(x+1,0));
	dp[0][0] = 1;
	for(int i = 1; i <= n; i++) {
		for(int j = 0; j <= x; j++) {
			// we can either use the current coin, or not
			// if we don't use the current coin
			dp[i][j] += dp[i-1][j];
			
			// if we do use the current coin look at difference
			int left = j - coins[i-1];
			if(left >= 0) {
				dp[i][j] += dp[i][left];
			}
			// take modulo
			dp[i][j] %= m;
		}
	}

	cout << dp[n][x] << endl;
	return 0;
}
```

## [Removing Digits](https://cses.fi/problemset/task/1637/)
### Problem Statement
You are given an integer *n*. On each step, you may substract from it any one-digit number that appears in it.

### Explanation
This problem is exactly the same as Minimizing Coins, however, now, our denominations are digits of our number!

### Code
```C++
vector<ll> getDigits(ll n) {
	vector<ll> digits;
	while(n > 0) {
		digits.push_back(n%10);
		n /= 10;
	}
	return digits;
}

int main() {
	// makes input/output faster
	ios_base::sync_with_stdio(false); cin.tie(NULL);

	ll n;
	cin >> n;
	vector<ll> dp(n+1, INF);

	if(n == 0) {
		cout << 0 << endl;
	}
	else if(n < 10) {
		cout << 1 << endl;
	}
	else {
		// base cases
		dp[0] = 0; // 0 steps to get to 0
		
		for(int i = 1; i <= n; i++) {
			vector<ll> digits = getDigits(i);
			for(auto d: digits) {
				if(d != 0) {
					dp[i] = min(dp[i], dp[i-d]);
				}
			}
			dp[i] += 1;
		}
		cout << dp[n] << endl;
	}
	return 0;
}	
```

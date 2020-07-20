# Solutions
Below are all the solutions, with explanation, to the DP questions that I have completeted from CSES.

## [Dice Combinations](https://cses.fi/problemset/task/1633)
Our task for this question is to count the number of ways to construct sum *n* by throwing a dice one or more times. Each throw produces an outcome between 1 and 6.

## Code
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

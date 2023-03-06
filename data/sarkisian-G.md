# StakedTokenLock

Finding: Unnecessary state variable: The depositedBalance state variable is not required as its value can be easily calculated from the balance of the contract. This adds unnecessary gas costs to the contract.

Recommendation: Remove the depositedBalance state variable and use the balance of the contract instead to save gas.
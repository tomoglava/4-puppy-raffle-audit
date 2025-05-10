# ok first steps
- check all functions in contract - use Solidity: Metrics and scrool to end or: `forge inspect PuppyRaffle methods`




# informational
`PuppyRaffle::entranceFee` is immutable and should be different syntax (`i_entranceFee`)


# notes
- function enterRaffle should check if duplicate and then push player to array if not duplicate. That way there is only 1 n2 loop rather than n + n2
- 



- after we removed // q from code and have just @audit, we should:
    - check slither / aderyn
    - code quality/tests



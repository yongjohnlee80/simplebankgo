## What is a db transaction.

### A single unit of work

### Often made up of multiple db operations.

### Example of a simple bank db tansaction:
1. Create a transfer record with amount 10
2. Create an account entry for account A with amount -10
3. Create an account entry for account B with amount +10
4. Subtract 10 from the balance of account A
5. Add 10 to the balance of account B

### Two objectives of db transactions:
1. To provide a reliable and consistent unit of work, even in case of system failure
2. To provide isolation between programs that access the db concurrently.

## ACID property
1. Atomicity(A) - Either all operations complete successfully or the transaction fails and the db is unchanged.
2. Consistency(C) - The db state must be valid after the transaction. All constraints must be satisfied.
3. Isolation(I) - Concurrent transactions must not affect each other.
4. Durability(D) - Data written by a successful transaction must be recorded in persistent storage.

## SQL TX(transaction) in GOLANG 
* Each query only do 1 operation on 1 specific table, so [Queries](../db/sqlc/db.go) struct doesn't support transaction.
* Thus, it is preferred to extend its functionality by embedding in the [store.go](../db/sqlc/store.go) struct.
* Composition is the preferred Golang pattern to extend struct functionality instead of inheritance.
* Default isolation level for transaction is "read-committed" in case of Postgres.
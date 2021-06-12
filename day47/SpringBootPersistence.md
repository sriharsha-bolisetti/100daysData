#### How to handle OptimisticLockException?

- The JPA `OptimisticLockException` prevents lost updates, and you shouldn't ignore it.
- You can catch it in a common exception handler and `redirect the user to the starting point of the currently excecuting workflow`, indicating that the `flow has to be restarted` since it was operating it with stale data.
- You might think that an auto-retry against a fresh entity database snapshot will fix the problem, but you will end up with the same optimistic locking exception since the load-time version is still lower than the current entity version in the DB.
- Another options is using Pessimistic locking, (eg. pessimistic_write or pessimistic_read) so that, once a row-level lock is acquired, no other transaction can modify the locked record.
- JPA requires running the Persistence Context code inside a transaction, and if our Transaction Manager catches a `RuntimeException`, it initiates the rollback process.
- This makes the Persistence Context unusable since we should discard it along with the rolled-back transactions.
- Therefore, it's safer to retry the business logic operation when we are not within a running transaction.

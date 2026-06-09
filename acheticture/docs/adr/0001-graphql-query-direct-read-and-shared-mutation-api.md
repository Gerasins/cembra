# Use direct read paths for GraphQL queries and backend APIs for shared mutation logic

In the shared GraphQL BFF layer, query paths may read from approved database read models or replicas directly, including `IQueryable`-based execution, while mutations must call a separate backend API when the mutation logic needs to be shared outside GraphQL. We chose this split because it keeps read paths efficient and scalable for GraphQL while preserving one canonical home for reusable business mutation logic in the underlying domain system.

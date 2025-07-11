## Title: Operation logs are disabled automatically if the parent directory does not exist.

## Description
Operation logging is automatically disabled when the parent directory, which is named after the hive session ID, is deleted or does not exist. This can occur if the operation log directory (e.g., /tmp) is purged by the operating system at configured intervals. When a query is executed from that session, it results in an `IOException` indicating that the operation log file cannot be created due to the absence of the specified directory.

## StackTrace
```
    java.io.IOException: No such file or directory,
        at java.io.UnixFileSystem.createFileExclusively(Native Method),
        at java.io.File.createNewFile(File.java:1012),
        at org.apache.hive.service.cli.operation.Operation.createOperationLog(Operation.java:195),
        at org.apache.hive.service.cli.operation.Operation.beforeRun(Operation.java:237),
        at org.apache.hive.service.cli.operation.Operation.run(Operation.java:255)
```

## RootCause
The root cause of the issue is an `IOException` triggered by the absence of the operation log directory, which is expected to be created based on the session ID. The `createOperationLog` method in the `Operation` class fails to create a log file because the directory returned by `parentSession.getOperationLogSessionDir()` does not exist.

## Suggestions
Ensure that the operation log directory is correctly configured and exists before executing queries. Implement a check in the `createOperationLog` method to create the directory if it does not exist. 

## ProblemLocation
```
{
    class: Operation.java,
    methods: [createOperationLog, beforeRun]
}
```

## PossibleFix
Modify the `createOperationLog` method to check if the directory exists and create it if it does not. Example code snippet:
```java
if (!operationLogFile.getParentFile().exists()) {
    operationLogFile.getParentFile().mkdirs();
}
```
# EventLog Analyzer
### Author: Viraj Ranaware

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#background">Background</a>
    </li>
    <li>
      <a href="#application-overview">Application Overview</a>
    </li>
    <li><a href="how-to-use-it">How to use it?</a></li>
    <li><a href="#roadmap">Technical details</a></li>
    <li><a href="#sample-reports">Sample Reports</a></li>
    <li><a href="#recommendations">Recommendations</a></li>
  </ol>
</details>

### Background
There are several pipelines on the platform which execute across distributed clusters. It has been observed that many pipelines do not use resources effectively. Spark generates extensive event logs that capture crucial runtime information, which includes task execution times, utilization of resources, partitioning, etc. Analyzing these logs provides invaluable insights into the working of applications and help in understanding bottlenecks, performance issues and inefficiencies in the application. A typical Spark application has several jobs which are actions on the dataset. A job has several stages (transformations to achieve the action) and each stage has several tasks (a computation performed in parallel).
The figure below illustrates the execution flow of a Spark Application. 

![spark-execution-flow.PNG](resources%2Fspark-execution-flow.PNG)

##### _Figure 1. Execution flow of Spark Application_

### Application Overview
The EventLog Analyzer application aims to streamline the process of analyzing and interpreting the Spark event logs. The application aims to achieve the following goals:
1. Parse the event logs and **filter out irrelevant data**.
2. **Identify loss** due to task failures, executor failures and garbage collection overhead.
3. Identify **unbalanced partitions.**
4. **Aggregate results** at various levels.
5. Generate intuitive reports with simple scores and metrics illustrating application effectiveness and potential candidates for optimization.

### How to use it?
For setting up project and executing it locally, follow these simple steps:
* Prerequisites:
  * JDK:
    The Application needs JDK version 1.8 and above to build and run.
  * SSH Key: SSH key pair configured for the user. Use [these steps](https://docs.gitlab.com/ee/user/ssh.html).
  * Maven installed and added to path
* Installation:
  * Clone the repo: 
  ```
  git clone <repo URL>
  ```
  * Build the application
  ```
  mvn clean install
  ```
  * Run the Java Application: Run the application with below two command line arguments:
    * A path of event log file
    * Number of cores available

### Technical details
The application analyzes all the events in the given event log file. It then filters out irrelevant events and starts with analyzing events for all the tasks. Initial report is built at Stage Attempt level using information gathered from these events. The report is then aggregated at Stage, Job and Application level in reports named StageReport, JobReport and ApplicationReport respectively. The diagram below depicts aggregation of these reports.

![eventlog-analyzer-report-generation.PNG](resources%2Feventlog-analyzer-report-generation.PNG)

##### _Figure 2. Report aggregation process of Eventlog Analyzer application_


The application assigns scores at each level of the report. These score indicates how efficient the application is in that aspects. Currently, below three scores are assigned at StageAttempt level and aggregated in all the reports. These scores have values ranging from 0 to 100. All scores are calculated in a way such that closer the score value to 100, better the application in that aspect. 

**1. taskScore:** This score indicates how well application utilizes all the cores available for computation. 

**2. executionScore:** This score tells efficiency of all tasks by comparing time taken by executor for computation against total time taken by task to report the result back to the driver.

**3. gcScore:** This score indicates time lost by application for Garbage Collection.

Below table summarizes all the metrics captured by the Eventlog Analyzer application.

| ApplicationReport                                 | JobReport                                          | StageReport                                         | StageAttemptReport                              |
| ------------------------------------------------- | -------------------------------------------------- |-----------------------------------------------------|-------------------------------------------------|
| appID                                             | jobID                                              | stageID                                             | stageAttemptUID                                 |
| fileName                                          | appID                                              |                                                     | stageID                                         |
|                                                   |                                                    |                                                     | stageAttemptID                                  |
| status                                            | status                                             | status                                              | status                                          |
| totalApplicationDuration                          | totalJobDuration                                   | totalStageDuration                                  | timeTaken                                       |
| totalDurationOfAllTasks                           | totalDurationOfAllTasks                            | totalDurationOfAllTasks                             | totalDurationOfAllTasks                         |
| actualExecutionDurationOfAllTasks                 | actualExecutionDurationOfAllTasks                  | actualExecutionDurationOfAllTasks                   | actualExecutionDurationOfAllTasks               |
| totalFailedJobsDuration                           | totalFailedStagesDuration                          | totalFailedAttemptsDuration                         | totalFailedTasksDuration                        |
| totalGCDuration                                   | totalGCDuration                                    | totalGCDuration                                     | totalGCDuration                                 |
| maxJobDuration                                    | maxStageDuration                                   | maxAttemptTime                                      | maxTaskTime                                     |
| minJobDuration                                    | minStageDuration                                   | minAttemptTime                                      | minTaskTime                                     |
| averageJobDuration                                | averageStageDuration                               | averageAttemptTime                                  | averageTaskTime                                 |
| medianJobDuration                                 | medianStageDuration                                | medianAttemptTime                                   | medianTaskTime                                  |
| numberOfJobs                                      | numberOfStages                                     | numberOfAttempts                                    | numberOfTasks                                   |
| numberOfFailedJobs                                | numberOfFailedStages                               | numberOfFailedAttempts                              | numberOfFailedTasks                             |
| numberOfAvailableCores                            | numberOfAvailableCores                             | numberOfAvailableCores                              | numberOfAvailableCores                          |
|                                                   |                                                    | numberOfStageRetries                                | numberOfTaskRetries                             |
| taskScore                                         | taskScore                                          | taskScore                                           | taskScore                                       |
| executionScore                                    | executionScore                                     | executionScore                                      | executionScore                                  |
| gcScore                                           | gcScore                                            | gcScore                                             | gcScore                                         |
| Map<String, FailureMetric> tasksFailureMetrics    | Map<String, FailureMetric> tasksFailureMetrics     | Map<String, FailureMetric> tasksFailureMetrics      | Map<String, FailureMetric> tasksFailureMetrics  |
| List\<JobReport> jobReportList                     | List\<StageReport> stageReportList                | List\<StageAttemptReport> stageAttemptReportList    |                                                 |

**_Note:_** List of failure reasons captured in tasksFailureMetrics: 
- ExceptionFailure
- ExecutorLostFailure
- FetchFailed
- TaskCommitDenied
- TaskKilled

### Sample Reports

**- Sample Report for an Efficient Application:**

The report below shows sample report for an application performing well.

**Application Report:**
```json
{
  "appID": "eventlog-simple",
  "status": "SUCCESS",
  "totalApplicationDuration": 6406487,
  "totalDurationOfAllTasks": 12807592,
  "actualExecutionDurationOfAllTasks": 12802609,
  "totalFailedJobsDuration": 0,
  "totalGCDuration": 31,
  "maxJobDuration": 6406487,
  "minJobDuration": 6406487,
  "averageJobDuration": 6406487,
  "medianJobDuration": 6406487,
  "numberOfJobs": 1,
  "numberOfFailedJobs": 0,
  "numberOfAvailableCores": 2,
  "taskScore": 99,
  "executionScore": 99,
  "gcScore": 99,
  "tasksFailureMetrics": {
    
  },
  "jobReportList": [
    {
      "jobID": "0",
      "appID": "eventlog-simple",
      "status": "SUCCESS",
      "totalJobDuration": 6406487,
      "totalDurationOfAllTasks": 12807592,
      "actualExecutionDurationOfAllTasks": 12802609,
      "totalFailedStagesDuration": 0,
      "totalGCDuration": 31,
      "maxStageDuration": 6406487,
      "minStageDuration": 6406487,
      "averageStageDuration": 6406487,
      "medianStageDuration": 6406487,
      "numberOfStages": 1,
      "numberOfFailedStages": 0,
      "numberOfAvailableCores": 2,
      "taskScore": 99,
      "executionScore": 99,
      "gcScore": 99,
      "tasksFailureMetrics": {
        
      },
      "stageReportList": [
        {
          "stageID": "0",
          "status": "SUCCESS",
          "totalStageDuration": 6406487,
          "totalDurationOfAllTasks": 12807592,
          "actualExecutionDurationOfAllTasks": 12802609,
          "totalFailedAttemptsDuration": 0,
          "totalGCDuration": 31,
          "numberOfStageRetries": 0,
          "maxAttemptTime": 6406487,
          "minAttemptTime": 6406487,
          "averageAttemptTime": 6406487,
          "medianAttemptTime": 6406487,
          "numberOfAttempts": 1,
          "numberOfFailedAttempts": 0,
          "numberOfAvailableCores": 2,
          "taskScore": 99,
          "executionScore": 99,
          "gcScore": 99,
          "tasksFailureMetrics": {
            
          },
          "stageAttemptReportList": [
            {
              "stageAttemptUID": "0-0",
              "stageID": "0",
              "stageAttemptID": "0",
              "status": "SUCCESS",
              "timeTaken": 6406487,
              "totalDurationOfAllTasks": 12807592,
              "actualExecutionDurationOfAllTasks": 12802609,
              "totalFailedTasksDuration": 0,
              "totalGCDuration": 31,
              "numberOfTaskRetries": 0,
              "maxTaskTime": 50057,
              "minTaskTime": 50007,
              "averageTaskTime": 50010,
              "medianTaskTime": 50009,
              "numberOfTasks": 256,
              "numberOfFailedTasks": 0,
              "numberOfAvailableCores": 2,
              "taskScore": 99,
              "executionScore": 99,
              "gcScore": 99,
              "tasksFailureMetrics": {
                
              }
            }
          ]
        }
      ]
    }
  ]
}
```

The above report we can see that:
- All three score values as 99, indicating application is very efficient. 
- The min, max, average and median values have very little deviation, data has partitioned well.

**- Sample Report for an Inefficient Application:**

The report below shows sample report for an inefficient application with areas for optimization.


**Application Report Snippet:**
```json
{
  "appID": "eventlog-overpartitioned",
  "status": "SUCCESS",
  "totalApplicationDuration": 49647175,
  "totalDurationOfAllTasks": 1565897161,
  "actualExecutionDurationOfAllTasks": 1493966338,
  "totalFailedJobsDuration": 0,
  "totalGCDuration": 583850998,
  "maxJobDuration": 48369854,
  "minJobDuration": 33,
  "averageJobDuration": 2758176,
  "medianJobDuration": 1666,
  "numberOfJobs": 18,
  "numberOfFailedJobs": 0,
  "numberOfAvailableCores": 160,
  "taskScore": 19,
  "executionScore": 95,
  "gcScore": 62,
  "tasksFailureMetrics": {
    "ExecutorLostFailure": {
      "count": 10
    },
    "ExceptionFailure": {
      "count": 36
    },
    "FetchFailed": {
      "count": 19
    }
  },
  "jobReportList": [
  ]
}
```
From the above report we can conclude below:
- The taskScore value is 19, indicating application does not utilize all the computational resources available. We can go further in job, stage and attempt levels to find out where the issue is.
- The gcScore also has value 62, meaning application is spending a lot of time for Garbage Collection.
- The tasksFailureMetrics shows all the different task failures occurred and their count.

For full report, refer this file: [eventlog-overpartitioned-report.json](resources%2Feventlog-overpartitioned-report.json)


### Recommendations
Currently, the EventLog Analyzer application sets clear outline for processing event logs, filter them and generating reports. It aggregates the reports at various levels and presents some metrics with three scores. The application can be extended in various ways. Below are some of the recommended next steps for the success of the project. 
1. At present, application only supports processing of a single file at a time, it needs to be extended for **parallel processing of multiple files**.
2. Application needs to adapted for **execution on the platform** where it utilizes cluster for execution and can identify files from AWS S3 bucket for reports generation.
3. Due to distributed nature of the Spark, reports will be spread across multipart files and need to **aggregate the output report files.**
4. For easy access of the report, we need to **link the Application Report with the input event log file.**
5. Currently, application supports only three scores, and it can be extended to **add more scores** covering other aspects of measuring the application effectiveness. The application has outline and flow setup for aggregation. Hence, it can be easily extended.


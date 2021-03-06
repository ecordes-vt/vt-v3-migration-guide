
# V3 Migration Documentation

Sections:

[creating a job](#creating-a-job)
  
  - [transcription job](#transcription)
  
  - [speaker separation job](#speaker-separation)
  
  - [translation job](#translation)

[checking job status](#checking-job-status)

[retrieving job output](#retrieving-job-output)

[notificationUris](#job-notifications)

## API Flow

1. `createJob` - Create a cognition job.
2. `checkJobStatus` - Check the status of a job. (optional)
3. `retrieveEngineOutput` - Retrieve the output of a completed job.


## Migrating to V3

### Update Request Header

V3 allows application and user level tracking for API calls.

Setting a header "x-veritone-application":"org:orgGuid" allows Veritone to monitor API usage and provides better tracking for debugging purposes.  

Example: `"x-veritone-application": "org:your_org_guid"`

Please set `x-veritone-application` header on all API requests. 

## Example V3 API Calls

# Creating a job

In V3 the createJob mutations have more definition than the previous V2 and Iron frameworks in order to make them more flexible. 

Once you have your template there are a few things you will need for each job.

- `tdoId` Use when reprocessing a file that is already in the Veritone system.

- `sourceUrl` Use a signedUrl to your file when creating a new job.

- `engineId` Provide the id of your desired cognition engine.  Example uses an English transcription engine.

For a more in-depth look at Job creation, visit the official [Veritone documentation](https://docs.veritone.com/#/quickstart/jobs/?id=working-with-jobs).

# Transcription

Transcription engineId's and payload types ( Speechmatics uses different engineId's instead of payloads )

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 Microsoft Transcription | 1fe73773-edcb-4610-9c6e-cbe39eecec30 | "language"
 Google Transcription    | d12e8f6d-9285-4f7d-aa2e-9e6151206277 | "languageCode"
 Speechmatics English    | c0e55cde-340b-44d7-bb42-2e0d65e98255 | n/a
 Speechmatics French     | ac20a648-5ed4-486c-a31d-d553f4209e32 | n/a
 Speechmatics Russian    | e02e7daa-5920-4119-bcc4-e0470ae12680 | n/a
 Speechmatics Spanish    | 71ab1ba9-e0b8-4215-b4f9-0fc1a1d2b255 | n/a
 Speechmatics Arabic     | 52bb5731-7365-4179-b520-1bd9fdd37f62 | n/a
 Speechmatics Korean     | 718bce7f-cddb-44f1-b838-dfb9b25b8485 | n/a
 
 To use a different transcription engine, change the engineId and add a payload field (if necessary).
 
 #### Example engine payload
 
 ```
 payload: {
  "language": "en-US"
 }
 ```
 
### transcription template

```
mutation createCognitionJob {
    createJob(input: {
      target: { status: "downloaded" } # no longer need to provide start/end time
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio",
            url: "link to your media file"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" } # change this value to < 60 if using google engine
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          payload: {}
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      id
      targetId
}}
```

# Speaker Separation

Speaker separation should be run against a tdo with a transcription asset.

 Engine Name             | engineId                             | payload
 ----------------------- | ------------------------------------ | --------------------------
 Speechmatics Speaker Separation | 06c3f1d7-7424-407b-a3b5-6ef61154fc0b | n/a
 
### speakerSeparation template

```
mutation createCognitionJob {
    createJob(input: {
      targetId: tdo_id
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          engineId: "06c3f1d7-7424-407b-a3b5-6ef61154fc0b"
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      id
}}
```
# Translation

Engine Name             | engineId                             | payload
 ----------------------- | ------------------------------------ | --------------------------
 Google Translate V3 | a7a16a08-a2f7-4f4f-94ea-e75e74cc8252 | target: "French:fr"
 Microsoft Translate V3 | 477b1aba-8ac4-4526-aef3-359c14ea416a | target: "fr"
 Amazon Translate V3 | 1fc4d3d4-54ab-42d1-882c-cfc9df42f386 | sourceLanguageCode: "", target: ""


### textTranslation Template 

```
mutation createTranslationJob{
  createJob(input: {
    target: { status: "downloaded" } # no longer need to provide start/end time
    clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
    tasks: [
      {
        engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"  
        payload:{ ffmpegTemplate: "rawchunk", url: "link to your text file" }
        ioFolders: [
          { referenceId: "chunkInputFolder", mode: stream, type: input },
          { referenceId: "chunkOutputFolder", mode: chunk, type: output }
        ],
        executionPreferences: { parentCompleteBeforeStarting: true, priority: -20 }
      }
      {
        engineId: "ENGINE_ID"
        payload: {
          target: "LANG_CODE"
        }
        ioFolders: [
          { referenceId: "engineInputFolder", mode: chunk, type: input },
          { referenceId: "engineOutputFolder", mode: chunk, type: output }
        ],
        executionPreferences: {	parentCompleteBeforeStarting: true, priority: -20 }
      }
      {
        engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"  
        ioFolders: [
          { referenceId: "owInputFolder", mode: chunk, type: input }
        ],
        executionPreferences: {	parentCompleteBeforeStarting: true, priority: -20 }
      }
    ]
    routes: [
      {
        parentIoFolderReferenceId: "chunkOutputFolder"
        childIoFolderReferenceId: "engineInputFolder"
        options: {}
      }
      {
        parentIoFolderReferenceId: "engineOutputFolder"
        childIoFolderReferenceId: "owInputFolder"
        options: {}
      } 
    ]
  }) {
    targetId
    id
  }
}
```

# Checking job status

You can poll for job status updates or use `notificationUris` detailed below.

```
query queryJobStatus {
  job(id:"JOB_ID") {
    id
    status
    targetId
    tasks{
      records{
        id
        status
        engine{
          id
          name
        }
        # taskOutput - include this field for a more verbose output
      }
    }
  }
}
```
# Retrieving job output

Include the TDO ID and Engine ID to retrieve transcription output in JSON format.

```
query getEngineOutput {
  engineResults(tdoId: "TDO_ID",
    engineIds: ["ENGINE_ID"]) { 
    records {
      tdoId
      engineId
      startOffsetMs
      stopOffsetMs
      jsondata
      assetId
      userEdited
}}}
```
# Job Notifications

`notificationUris` allow you to link to a custom endpoint(s).  This removes the need to poll for job completion, you will instead be notified when the job completes at the endpoint provided.

This example notifies you when the output is completed:

```
{
  engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
  executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
  ioFolders: [
    { referenceId: "owInputFolder", mode: chunk, type: input }
  ]
  notificationUris: ["https://example.net/dump"]
}
```

This example notifies you as each task is completed:

```
mutation createCognitionJob {
    createJob(input: {
      target: { status: "downloaded" } # no longer need to provide start/end time
      notificationUris: [ "https://example.net/dump" ] # This endpoint will be set for each task
      clusterId :"rt-1cdc1d6d-a500-467a-bc46-d3c5bf3d6901"
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "audio"
            customFFMPEGProperties: { chunkSizeInSeconds: "300" }
          }
          ioFolders: [
            { referenceId: "siInputFolder", mode: stream, type: input }
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          payload: {}
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      id
}}
```

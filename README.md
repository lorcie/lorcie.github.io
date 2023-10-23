
# Forge AI Backlog Issues Helper

This app demonstrates a Forge apps for **Codegeist Challenge AI**. It also delegates AI processing to an asynchronous task such that it may be able to benefit from a longer runtime timeout and therefore be more resilient to expensive AI API requests.

If this is all a bit new to you, you may also like these other example apps and tutorials:
* [Getting started: creating a hello world app in Forge](https://developer.atlassian.com/platform/forge/getting-started/)
* [The basics of building a Forge AI app](https://blog.developer.atlassian.com/forge-ai-basics/)

## App overview

The app provides functionalities to help analyze, suggest storypoints, translate the project's backlog issues.

The app uses the `jira:projectPage` extension point to render a user interface (UI) that allows the user to analyze and enhance Backlog issues. The title of the project page is "AI Backlog Issues Helper". The app uses AI to evaluate the issues and suggests storypoint, based on Team Velocity Average calculated using OpenAI special Prompts. All current project Backlog Issues are returned with suggested storypoints

The Backlog issues with suggested storypoints are returned to the UI which then presents them to the end user. The UI also has a text field allowing ther user to change the Team Velocity Average. The UI enables user to choose the language for Issues summary/description.

## Implementation overview

The Full Stack (frontend and backend) Application has all been coded in Typescript .ts / .tsx , using @Atlaskit (button, css-reset, form, select, spinner, textfield,..)  dependencies + @forge/bridge on frontend and several @forge modules (events, resolver, storage) at backend side.

The UI is implemented with [Custom UI](https://developer.atlassian.com/platform/forge/custom-ui/iframe/). The UI sends a `LaunchEnhancement` [invoke](https://developer.atlassian.com/platform/forge/custom-ui-bridge/invoke/#invoke) request to the backend to initiate the AI operation. This operation generates an ID to represent the job and delegates the job to another function using the [async event API](https://developer.atlassian.com/platform/forge/runtime-reference/async-events-api/#async-events-api). At this point, the job ID is returned to the UI and the function invocation ends. 

The asynchronous function invocation is where the AI call occurs. The result of the call is stored in [Forge Storage](https://developer.atlassian.com/platform/forge/runtime-reference/storage-api/#storage-api) in a record with a key based on the job ID and Backlog Issue Key.

Whilst the back end is waiting for the AI API to return a result, the app's front end periodically polls it's backend until the result is available. This is done by sending a `pollJobResult` [invoke](https://developer.atlassian.com/platform/forge/custom-ui-bridge/invoke/#invoke) request. The implementation of this request simply involves attempting to retrieve the results from [Forge Storage](https://developer.atlassian.com/platform/forge/runtime-reference/storage-api/#storage-api) with query to match all items (Issues) whose key starts with jobId. The UI passes the job ID so the back end knows the key of the storage records to match with. 

This pattern is illustrated by the following sequence diagram:

![Forge async job pattern sequence diagram](/forge-async-job-pattern.png "Forge async job pattern sequence diagram")

## Limitations and improvements

* The app doesn't actually create or modify sprints since it doesn't contribute to the goal of the app which is is to explain how to manage long running AI tasks.
* For simplicity, the app mainly implements "happy paths". A commercial quality app should ensure error conditions are properly handled.
* One particular known error condition involves the AI API returning a response that is not JSON formatted. This seems to occur from time to time and could be reduced by improving the prompt.

## Setup

Run `forge variables set --encrypt OPEN_API_KEY {your-open-ai-key}` and deploy with `yarn deploy` before

## Prompt design

This section defines the prompt used to request the issue storypoints suggestion. The prompt and the AI process may need to be optimised.

### Current prompt format

The following is a JSON formatted list of open issues in a Jira backlog where the field "key" is a unique identifier of the issue and the field "summary" is the summary of the issue and the field "storypoints" is an estimate of the relative effort to implement the issue:

[{
  "key: "PS-1",
  "summary: "Form to capture new types of products",
  "storypoints": 3
}, {
  "key: "PS-2",
  "summary: "Define the initial database schema",
  "storypoints": 5
}, {
  "key: "PS-3",
  "summary: "Research what type of database the Pet Shop application should use",
  "storypoints": 1
}, {
  "key: "PS-4",
  "summary: "Research what the main use cases are for the Pet Shop application",
  "storypoints": 3
}, {
  "key: "PS-5",
  "summary: "Design the API for web client and server interactions",
  "storypoints": 5
}, {
  "key: "PS-6",
  "summary: "Replace the mock API implementation with a real implementation",
  "storypoints": 3
}, {
  "key: "PS-7",
  "summary: "Create the API with a mock implementation",
  "storypoints": 1
}, {
  "key: "PS-8",
  "summary: "Implement the home page",
  "storypoints": 5
}, {
  "key: "PS-9",
  "summary: "Design the home page",
  "storypoints": 3
}, {
  "key: "PS-10",
  "summary: "Define the different kinds of user groups needed",
  "storypoints": 1
}, {
  "key: "PS-11",
  "summary: "Add a search capability to the user experience",
  "storypoints": 3
}, {
  "key: "PS-12",
  "summary: "Enhance the API with a search function",
  "storypoints": 3
}, {
  "key: "PS-13",
  "summary: "Define the types of entities and the relationships between them that will need to be stored in the database",
  "storypoints": 1
}, {
  "key: "PS-14",
  "summary: "Define the scopes required for the API",
  "storypoints": 5
}, {
  "key: "PS-15",
  "summary: "Research whether REST or GraphQL would be a better fit for the type of API",
  "storypoints": 3
}, {
  "key: "PS-16",
  "summary: "Create a regression test for the search functionality",
  "storypoints": 1
}, {
  "key: "PS-17",
  "summary: "Demonstrate the API capabilities to stakeholders",
  "storypoints": 1
}, {
  "key: "PS-18",
  "summary: "Demonstrate the initial user experience to stakeholders",
  "storypoints": 3
}]

Return JSON formatted string representing an array of issue objects and suggest storypoints based on team velocity average (example 10) and issue summary description and/or goal. Each issue object contains a field named "key" identifying the issue key. Do not return any other text other than the JSON.

## Screenshots

Architecture >

![Combogeist Architecture](./screenshots/combogeist-architecture.png)

Forge cli commands >

![Forge cli commands](./screenshots/combogeist-forge-cli-commands.png)

Combogeist Application Codebase >

![Application Codebase](./screenshots/combogeist-ts-tsx-codeset.png)

Combogeist Application production >

![Application app production](./screenshots/combogeist-app-production.png)

Combogeist Backlog Issues Helper Run on Atlassian JIRA Cloud >

![Backlog Issues Helper Run on Atlassian JIRA Cloud](./screenshots/combogeist-backlog-issues-helper-jira-cloud-app-run.png)


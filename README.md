# Buildkite Agentic Pipelines

AI-powered automation for Linear issues and Buildkite builds. Uses Claude AI agents to analyze support requests, implement features, and fix failing builds.

## Features

- **Analyze Requests**: Triages Linear support issues, provides analysis for complex problems, and implements fixes for simple ones
- **Complete Tasks**: Implements Linear tasks labeled `buildsworth-analysis` and creates draft pull requests
- **Fix Builds**: Automatically fixes failing CI builds triggered by GitHub or Buildkite webhooks
- **Fix Bugsnag Errors**: Automatically analyzes and fixes production errors when Bugsnag detects error spikes

## Dependencies

It'd be good to provide instructions on how/where to get these.

### External Systems
- **Linear**: Project management and issue tracking
- **GitHub**: Code repository, PR management, CI status checks
- **Bugsnag**: Error monitoring and spike detection

### API Keys
- **Linear API Token**: "Read" permissions
- **GitHub Token**: Fine-grained PAT with `repo` and `pull_request` read/write scope
- **Buildkite API Token**: `read_builds`, `read_build_logs`, and `read_pipelines` (anything else?)
- **Bugsnag API Token**: Just create it. (It doesn't have a scope.)
- Optionally, Anthropic API token, if you aren't using hosted agents?

### Infrastructure
- **Buildkite Pipeline**: With Docker Compose plugin support (this is baked in, right?)
- **Buildkite Agent**: With access to Hosted Models and sufficient resources (this is also built-in, right?)

Should we instead say "Buildkite hosted agents and hosted models? Or if that isn't a requirement, mention that hosted agents does this automatically?

### Webhook Configuration

Trigger IDs are configured as environment variables in `.buildkite/pipeline.yml`.

^ We should clarify that you have to do this in the UI yourself, by copying the values from the URL and pasting them into the YAML. (This is also a bit of a weird thing to have to do, but I get it, since the pipeline handles multiple webhook callbacks. Can we use the names instead? If we could give them default names, and fetch those names, it'd make the setup work much less onerous.)

#### Buildkite Pipeline Triggers
Create webhook triggers in Buildkite Pipeline Settings with trigger IDs matching the environment variables above. You can customize the trigger IDs by modifying the environment variables in `.buildkite/pipeline.yml`.

#### External Webhook Sources
Configure webhooks in external services to POST to your Buildkite pipeline webhook URL.

These should have step-by-step setup instructions.

**Linear**:
- **Events**: Under Data change events, choose "Issues". (You can't specify individual issue event types, only "Issues" — it's all or nothing.)

**GitHub**:
- **Events**: Choose "Pull requests". That's all that's needed. (We'll be looking specifically for the `pull_request.labeled` event and for a label named `fix-build`.)

**Bugsnag**:
- **Events**: `projectSpiking` (error spike detection)
- **Payload fields**: `project.name`, `error.errorId`, `error.url`, `error.exceptionClass`, `error.message`, `error.context`, `error.app.releaseStage`, `error.app.version`, `error.stacktrace`
- **Target trigger**: Use `FIX_BUG_TRIGGER_ID`

## Installation

1. Create a new Buildkite pipeline using this repository
2. Configure the following secrets in Buildkite Secrets:
   - `LINEAR_API_TOKEN`: Linear API token with read and comment permissions
   - `GITHUB_TOKEN`: GitHub Fine Grained Personal Access Token with `repo` and `pull_request` read/write scope
   - `API_TOKEN_BUILDKITE`: Buildkite API token for build access
   - `BUGSNAG_API_TOKEN`: (Optional) Bugsnag API token for error analysis and commenting
3. Create Pipeline Triggers in Project Settings

## Usage

The agent automatically processes events based on trigger IDs:
- Linear issues with the `first-draft` label trigger task completion or analysis
- Failed Buildkite PR builds with the `fix-build` label trigger automatic fix attempts
- Bugsnag error spikes trigger automatic error analysis and fix attempts

# [DEPRECATED] MCP Task Manager

⚠️ **DEPRECATION NOTICE** ⚠️

This package has been renamed and moved to a new repository. Please use the new package instead:

```bash
npm install taskqueue-mcp
```

New repository: [taskqueue-mcp](https://github.com/chriscarrollsmith/taskqueue-mcp)

This repository is no longer maintained. All future updates and improvements will be made to the new package.

---

A Model Context Protocol (MCP) server for AI task management. This tool helps AI assistants handle multi-step tasks in a structured way, with user approval checkpoints.

## Features

- Task planning with multiple steps
- Progress tracking
- User approval of completed tasks
- Project completion approval
- Task details visualization
- Task status state management
- Enhanced CLI for task inspection and management

## Structure

The codebase has been refactored into a modular structure:

```
src/
├── index.ts              # Main entry point
├── cli.ts                # CLI for task approval and listing
├── server/
│   └── TaskManagerServer.ts   # Core server functionality
└── types/
    ├── index.ts          # Type definitions and schemas
    └── tools.ts          # MCP tool definitions
```

## Data Schema and Storage

The task manager stores data in a JSON file with platform-specific default locations:

- **Default locations**: 
  - **Linux**: `~/.local/share/mcp-taskmanager/tasks.json` (following XDG Base Directory specification)
  - **macOS**: `~/Library/Application Support/mcp-taskmanager/tasks.json`
  - **Windows**: `%APPDATA%\mcp-taskmanager\tasks.json` (typically `C:\Users\<username>\AppData\Roaming\mcp-taskmanager\tasks.json`)
- **Custom location**: Set via `TASK_MANAGER_FILE_PATH` environment variable

```bash
# Example of setting custom storage location
TASK_MANAGER_FILE_PATH=/path/to/custom/tasks.json npm start
```

The data schema is organized as follows:

```
TaskManagerFile
├── projects: Project[]
    ├── projectId: string            # Format: "proj-{number}"
    ├── initialPrompt: string        # Original user request text
    ├── projectPlan: string          # Additional project details
    ├── completed: boolean           # Project completion status
    └── tasks: Task[]                # Array of tasks
        ├── id: string               # Format: "task-{number}"
        ├── title: string            # Short task title
        ├── description: string      # Detailed task description
        ├── status: string           # Task status: "not started", "in progress", or "done"
        ├── approved: boolean        # Task approval status
        ├── completedDetails: string # Completion information (required when status is "done")
        ├── toolRecommendations: string # Suggested tools that might be helpful for this task
        └── ruleRecommendations: string # Suggested rules/guidelines to follow for this task
```

The system persists this structure to the JSON file after each operation.

**Explanation of Task Properties:**

- `id`: A unique identifier for the task
- `title`: A short, descriptive title for the task
- `description`: A more detailed explanation of the task
- `status`: The current status of the task (`not started`, `in progress`, or `done`)
- `approved`: Indicates whether the task has been approved by the user
- `completedDetails`: Provides details about the task completion (required when `status` is `done`)
- `toolRecommendations`: A string containing suggested tools (by name or identifier) that might be helpful for completing this task. The LLM can use this to prioritize which tools to consider.
- `ruleRecommendations`: A string containing suggested rules or guidelines that should be followed while working on this task. This can include things like "ensure all code is commented," "follow accessibility guidelines," or "use the company style guide". The LLM uses these to improve the quality of its work.

## Installation

```bash
npm install @chriscarrollsmith/mcp-taskmanager
```

You can also install globally:

```bash
npm install -g @chriscarrollsmith/mcp-taskmanager
```

## Usage

This command will start the MCP server:

```bash
npx -y @chriscarrollsmith/mcp-taskmanager

# Or, with a custom tasks.json path:
TASK_MANAGER_FILE_PATH=/path/to/tasks.json npx -y @chriscarrollsmith/mcp-taskmanager
```

However, usually you will set the tool configuration in Claude Desktop, Cursor, or another MCP client as follows:

```json
{
  "tools": {
    "taskmanager": {
      "command": "npx",
      "args": ["-y", "@chriscarrollsmith/mcp-taskmanager"]
    }
  }
}
```

To use the CLI utility, you can use the following command:

```bash
npx task-manager-cli --help
```

This will show the available commands and options.

## License

MIT

<a href="https://glama.ai/mcp/servers/bdjh7kx05h"><img width="380" height="200" src="https://glama.ai/mcp/servers/bdjh7kx05h/badge" alt="@kazuph/mcp-taskmanager MCP server" /></a>

## Quick Start (For Users)

### Prerequisites
- Node.js 18+ (install via `brew install node`)
- Claude Desktop (install from https://claude.ai/desktop)

### Configuration

1. Open your Claude Desktop configuration file at:
`~/Library/Application Support/Claude/claude_desktop_config.json`

You can find this through the Claude Desktop menu:
1. Open Claude Desktop
2. Click Claude on the Mac menu bar
3. Click "Settings"
4. Click "Developer"

2. Add the following to your configuration:

```json
{
  "tools": {
    "taskmanager": {
      "command": "npx",
      "args": ["-y", "@chriscarrollsmith/mcp-taskmanager"]
    }
  }
}
```

## For Developers

### Prerequisites
- Node.js 18+ (install via `brew install node`)
- Claude Desktop (install from https://claude.ai/desktop)
- tsx (install via `npm install -g tsx`)

### Installation

```bash
git clone https://github.com/chriscarrollsmith/mcp-taskmanager.git
cd mcp-taskmanager
npm install
npm run build
```

### Development Configuration

1. Make sure Claude Desktop is installed and running.

2. Install tsx globally if you haven't:
```bash
npm install -g tsx
# or
pnpm add -g tsx
```

3. Modify your Claude Desktop config located at:
`~/Library/Application Support/Claude/claude_desktop_config.json`

Add the following to your MCP client's configuration:

```json
{
  "tools": {
    "taskmanager": {
      "args": ["tsx", "/path/to/mcp-taskmanager/index.ts"]
    }
  }
}
```

## Available Operations

The TaskManager now uses a direct tools interface with specific, purpose-built tools for each operation:

### Project Management Tools

- `list_projects`: Lists all projects in the system
- `read_project`: Gets details about a specific project
- `create_project`: Creates a new project with initial tasks
- `delete_project`: Removes a project
- `add_tasks_to_project`: Adds new tasks to an existing project
- `finalize_project`: Finalizes a project after all tasks are done

### Task Management Tools

- `list_tasks`: Lists all tasks for a specific project
- `read_task`: Gets details of a specific task
- `create_task`: Creates a new task in a project
- `update_task`: Modifies a task's properties (title, description, status)
- `delete_task`: Removes a task from a project
- `approve_task`: Approves a completed task
- `get_next_task`: Gets the next pending task in a project
- `mark_task_done`: Marks a task as completed with details

### Task Status and Workflows

Tasks have a status field that can be one of:
- `not started`: Task has not been started yet
- `in progress`: Task is currently being worked on
- `done`: Task has been completed (requires `completedDetails`)

#### Status Transition Rules

The system enforces the following rules for task status transitions:
- Tasks follow a specific workflow with defined valid transitions:
  - From `not started`: Can only move to `in progress`
  - From `in progress`: Can move to either `done` or back to `not started`
  - From `done`: Can move back to `in progress` if additional work is needed
- When a task is marked as "done", the `completedDetails` field must be provided to document what was completed
- Approved tasks cannot be modified
- A project can only be approved when all tasks are both done and approved

These rules help maintain the integrity of task progress and ensure proper documentation of completed work.

### Usage Workflow

A typical workflow for an LLM using this task manager would be:

1. `create_project`: Start a project with initial tasks
2. `get_next_task`: Get the first pending task
3. Work on the task
4. `mark_task_done`: Mark the task as complete with details
5. Wait for approval (user must call `approve_task` through the CLI)
6. `get_next_task`: Get the next pending task
7. Repeat steps 3-6 until all tasks are complete
8. `finalize_project`: Complete the project (requires user approval)

### CLI Commands

#### Task Approval

Task approval is controlled exclusively by the human user through the CLI command:

```bash
npm run approve-task -- <projectId> <taskId>
```

Options:
- `-f, --force`: Force approval even if the task is not marked as done

Note: Tasks must be marked as "done" with completed details before they can be approved (unless using --force).

#### Listing Tasks and Projects

The CLI provides a command to list all projects and tasks:

```bash
npm run list-tasks
```

To view details of a specific project:

```bash
npm run list-tasks -- -p <projectId>
```

This command displays information about all projects in the system or a specific project, including:
- Project ID and initial prompt
- Completion status
- Task details (title, description, status, approval)
- Progress metrics (approved/completed/total tasks)

## Example Usage

### Creating a Project with Tasks

```javascript
// Example of how an LLM would use the create_project tool
{
  'create_project': {
    'initialPrompt': "Create a website for a small business",
    'projectPlan': "We'll create a responsive website with Home, About, Services, and Contact pages",
    'tasks': [
      { 
        'title': "Set up project structure", 
        'description': "Create repository and initialize with basic HTML/CSS/JS files",
        'toolRecommendations': "create_directory, create_file, git_init",
        'ruleRecommendations': "Use consistent file naming, Initialize git repository"
      },
      { 
        'title': "Design homepage", 
        'description': "Create responsive homepage with navigation and hero section",
        'toolRecommendations': "html_editor, css_editor, image_optimizer",
        'ruleRecommendations': "Follow accessibility guidelines (WCAG), Optimize for mobile-first"
      },
      { 
        'title': "Implement about page", 
        'description': "Create about page with company history and team section",
        'toolRecommendations': "html_editor, css_editor",
        'ruleRecommendations': "Use clear and concise language, Include team member photos"
      }
  ]
}
}

// Response will include:
// {
//   status: "planned",
//   projectId: "proj-1234",
//   totalTasks: 3,
//   tasks: [
//     { id: "task-1", title: "Set up structure", ..., toolRecommendations: "...", ruleRecommendations: "..." },
//     { id: "task-2", title: "Design homepage", ..., toolRecommendations: "...", ruleRecommendations: "..." },
//     { id: "task-3", title: "Implement about page", ..., toolRecommendations: "...", ruleRecommendations: "..." }
//   ],
//   message: "Project created with 3 tasks"
// }
```

### Getting the Next Task

```javascript
// Example of how an LLM would use the get_next_task tool
{
  'get_next_task': {
    'projectId': "proj-1234"
  }
}

// Response will include:
// {
//   status: "next_task",
//   task: {
//     id: "task-1",
//     title: "Set up project structure",
//     description: "Create repository and initialize with basic HTML/CSS/JS files",
//     status: "not started",
//     approved: false
//   },
//   message: "Retrieved next task"
// }
```

### Marking a Task as Done

```javascript
// Example of how an LLM would use the mark_task_done tool
{
  'mark_task_done': {
    'projectId': "proj-1234",
    'taskId': "task-1",
    'completedDetails': "Created repository at github.com/example/business-site and initialized with HTML5 boilerplate, CSS reset, and basic JS structure."  // Required when marking as done
  }
}

// Response will include:
// {
//   status: "task_marked_done",
//   task: {
//     id: "task-1",
//     title: "Set up project structure",
//     status: "done",
//     approved: false,
//     completedDetails: "Created repository at github.com/example/business-site and initialized with HTML5 boilerplate, CSS reset, and basic JS structure."
//   },
//   message: "Task marked as done"
// }
```

### Approving a Task (CLI-only operation)

This operation can only be performed by the user through the CLI:

```bash
npm run approve-task -- proj-1234 task-1
```

After approval, the LLM can check the task status using `read_task` or get the next task using `get_next_task`.

### Finalizing a Project

```javascript
// Example of how an LLM would use the finalize_project tool
// (Called after all tasks are done and approved)
{
  'finalize_project': {
    'projectId': "proj-1234"
  }
}

// Response will include:
// {
//   status: "project_finalized",
//   projectId: "proj-1234",
//   message: "Project has been finalized"
// }
```

## Status Codes and Responses

All operations return a status code and message in their response:

### Project Tool Statuses
- `projects_listed`: Successfully listed all projects
- `planned`: Successfully created a new project
- `project_deleted`: Successfully deleted a project
- `tasks_added`: Successfully added tasks to a project
- `project_finalized`: Successfully finalized a project
- `error`: An error occurred (with error message)

### Task Tool Statuses
- `task_details`: Successfully retrieved task details
- `task_updated`: Successfully updated a task
- `task_deleted`: Successfully deleted a task
- `task_not_found`: Task not found
- `error`: An error occurred (with error message)

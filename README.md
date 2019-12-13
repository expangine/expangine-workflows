# expangine-workflows
A service and UI for creating and running user-designed workflows.

**Workflow:**
- `start`: **Step**
- `steps`: **Step[]**
- `logs`: **Workflow Log[]**
- `infos`: **Workflow Info[]**
- `watchers`: the watchers run concurrently for conditions outside the normal workflow
- `input`: the (readonly) input type given when the workflow was created
- `temp`: the temporary type used while the workflow is in a non-FINISHED status
- `output`: the output type

**Workflow Info:** (ex: priority, due)
- `workflow`: **Workflow**
- `label`: the label of the info
- `order`: the order of the info
- `visibility`: where is this info displayed
- `getData`: Expression

**Workflow Instance:**
- `type`: **Workflow**
- `status`: one of WAITING, RUNNING, PAUSED, FINISHED, ERROR, FAILED
- `current`: **Step**
- `input`: the input data given when the workflow was created
- `temp`: the temporary data used while the workflow is in a non-FINISHED status
- `output`: the output data
- `cache`: cached data used by steps
- `updated`: when workflow last ran
- `created`: when workflow was created
- `scheduled`: the next time this workflow should run

**Step:**
- `type`: **Step Type**
- `name`: the name of the step
- `icon`: an icon of the step
- `color`: the color of the step
- `parent`?: **Step**
- `parent_outcome`?: the outcome of the parent
- `outcomes`: one or more outcomes

**Step Type:**
- `name`: the name of the step type
- `icon`: the default icon
- `color`: the default color
- `isAsync (workflow: Workflow, step: Step): boolean`
- `getOutcomes (workflow: Workflow, step: Step): string[]`
- `getOutcomeScope (workflow: Workflow, step: Step): Type`
- `getExpressions (workflow: Workflow, step: Step): Expression[]`
- `execute (workflow: Workflow, step: Step, instance: WorkflowInstance): Promise<string>`

**Watcher:**
- `schedule`: how often the watcher runs
- `step`: the step to run each time a watcher is ran

**Actor:**
- `name`: the name of the actor
- `type`: the actor type (user, role, group, workflow, system, watcher)

**Workflow Log:**
- `workflow`: **Workflow Instance**
- `index`: the index of this log in the list of things done to the workflow
- `actor`: **Actor**
- `from`: **Step**
- `to`: **Step**
- `created`: when the log was created

## Step Types

**Execute:** (run a transactional program)
- `program`: Expression (Any)
- `outcomes`: [success, error] 
- `outcomeScope`: result

**Email:** (send an email)
- `protocol`: **Email Protocol**
- `to`: Expression (Text, Text[])
- `cc`: Expression (Text, Text[])
- `bcc`: Expression (Text, Text[])
- `subject`: Expression (text)
- `body`: Expression (Text, Element)
- `attachments`: Expression (File, File[])
- `outcomes`: [sent, error]

**API call:** (call another service over HTTP(s))
- `url`: Expression (Text)
- `method`: Expression (Enum: GET/POST/DELETE/PUT)
- `query`: Expression (Object)
- `data`: Expression (Any)
- `headers`: Expression (Object)
- `timeout`: Expression (Number)
- `outcomes`: [success, error]
- `outcomeScope`: response, status, responseHeaders, timedout

**Branch:** (evaluate state and return possible outcomes)
- `program`: Expression (Enum)
- `outcomes`: *determined by program*

**Pause:** (wait until workflow evaluates further)
- `resume`: Expression (Date)
- `outcomes`: [elapsed]

**Finish:** (stop workflow)
- `outcomes`: []

**Goto:** (go to step)
- `step`: Expression (Enum: steps)
- `outcomes`: []

**Throttle:** (don't execute too often)
- `frequency`: Expression (Number)
- `max`: Expression (Number?)
- `getNextTime`: Expression (Date?)
- `outcomes`: [run, throttled]

**Fork:** (create a new workflow)
- `workflow`: Expression (Enum: workflows)
- `getInput`: Expression
- `outcomes`: [created, error]

**Sub:** (create a workflow and wait for it to finish)
- `workflow`: Expression (Enum: workflows)
- `getInput`: Expression
- `outcomes`: [success, error]
- `outcomeScope`: output

**User Decision:** (user must click a button)
- `due`: Expression (Date)
- `outcomes`: [...user-defined, elapsed]

**User Input:** (user must enter data)
- `due`: Expression (Date)
- `input`: Type
- `submitted`: Expression
- `outcomes`: [submitted, elapsed]
- `outcomeScope`: result

**Access:** (who can see or act on workflow instance)
- `actor`: Expression (List of Enum: user/role/group)
- `rights`: Expression (Enum: View, Act, Pause, Resume, Stop, Delete)
- `type`: ADD, REMOVE, REPLACE
- `outcomes`: [granted]

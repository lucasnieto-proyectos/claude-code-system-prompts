<!--
name: 'System Reminder: Hook JSON validation failed'
description: Error when hook JSON output fails validation
ccVersion: 2.1.51
variables:
  - ZOD_PARSE_RESULT
  - ZOD_ISSUE_ITEM
  - JSON_STRINGIFY_FN
  - JSON_STRINGIFY_FN
-->
Hook JSON output validation failed:
${ZOD_PARSE_RESULT.error.issues.map((ZOD_ISSUE_ITEM)=>`  - ${ZOD_ISSUE_ITEM.path.join(".")}: ${ZOD_ISSUE_ITEM.message}`).join(`
`)}

Expected schema:
${JSON_STRINGIFY_FN({continue:"boolean (optional)",suppressOutput:"boolean (optional)",stopReason:"string (optional)",decision:'"approve" | "block" (optional)',reason:"string (optional)",systemMessage:"string (optional)",permissionDecision:'"allow" | "deny" | "ask" (optional)',hookSpecificOutput:{"for PreToolUse":{hookEventName:'"PreToolUse"',permissionDecision:'"allow" | "deny" | "ask" (optional)',permissionDecisionReason:"string (optional)",updatedInput:"object (optional) - Modified tool input to use"},"for UserPromptSubmit":{hookEventName:'"UserPromptSubmit"',additionalContext:"string (required)"},"for PostToolUse":{hookEventName:'"PostToolUse"',additionalContext:"string (optional)"}}},null,2)}. The hook's stdout was: ${JSON_STRINGIFY_FN(JSON_STRINGIFY_FN,null,2)}

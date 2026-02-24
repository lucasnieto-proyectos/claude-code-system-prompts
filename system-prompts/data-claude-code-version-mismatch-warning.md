<!--
name: 'Data: Claude Code version mismatch warning'
description: Warning shown when Claude Code version is outdated
ccVersion: 2.1.51
variables:
  - VERSION_CHECK_RESULT
-->

It looks like your version of Claude Code (${{ISSUES_EXPLAINER:"report the issue at https://github.com/anthropics/claude-code/issues",PACKAGE_URL:"@anthropic-ai/claude-code",README_URL:"https://code.claude.com/docs/en/overview",VERSION:"<<CCVERSION>>",FEEDBACK_CHANNEL:"https://github.com/anthropics/claude-code/issues",BUILD_TIME:"<<BUILD_TIME>>"}.VERSION}) needs an update.
A newer version (${VERSION_CHECK_RESULT.minVersion} or higher) is required to continue.

To update, please run:
    claude update

This will ensure you have access to the latest features and improvements.

# see-claude-code
inspecting and evaluating claude code

![see_claude_code](https://github.com/user-attachments/assets/8eb610eb-7062-4536-bdee-55d9aa0cfec5)

## Claude Code Feedback Commands for Open Coding Analysis

### [feedback_simple_open_coding.md](.claude/commands/feedback_simple_open_coding.md)  
Simplified feedback command focused purely on capturing user feedback through open coding methodology. Provides a streamlined way to record qualitative insights and learnings without git tracking complexity.

**Example:**
![CleanShot 2025-06-15 at 20 11 55@2x](https://github.com/user-attachments/assets/f698ece7-c1e4-4888-9494-ca70ff42cee7)


**Key features:**
- Clean, minimal interface for feedback capture
- Direct integration with CLAUDE.md for persistent learning
- Focus on qualitative analysis and pattern recognition

### [feedback_with_git_tracking.md](.claude/commands/feedback_with_git_tracking.md)
🚧 [in progress] 🚧 Advanced feedback command that captures user feedback using open coding techniques while automatically tracking the complete git state of the conversation. Records all changes from conversation start to current state, including commits made during the session and uncommitted working directory changes. Uses flexible git detection logic to work across any repository without hardcoded commit references.

**Key features:**
- Automatic conversation start detection using flexible git fallback logic
- Complete diff tracking from session start to current state  
- Captures both committed and uncommitted changes
- Writes learnings to CLAUDE.md with full git context


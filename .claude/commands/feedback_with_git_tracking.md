## VARIABLES

OPEN_CODING = $ARGUMENTS

# Git tracking variables - detect conversation start more robustly
CURRENT_COMMIT = $(git log --format="%H" -1 HEAD)
CURRENT_COMMIT_SHORT = $(git log --format="%h" -1 HEAD)

# General conversation start detection - works for any repo
CONVERSATION_START_COMMIT = $(
  # Try to detect conversation start by looking for a reasonable baseline
  # Strategy 1: Use HEAD~1 if it exists and differs from current
  prev_commit=$(git log --format="%H" -1 HEAD~1 2>/dev/null)
  if [ -n "$prev_commit" ] && [ "$prev_commit" != "$CURRENT_COMMIT" ]; then
    echo "$prev_commit"
  else
    # Strategy 2: Look back a few commits to find a reasonable starting point
    # Use HEAD~2 if available, otherwise HEAD~1, otherwise current
    for i in 2 1 0; do
      test_commit=$(git log --format="%H" -1 HEAD~$i 2>/dev/null)
      if [ -n "$test_commit" ]; then
        echo "$test_commit"
        break
      fi
    done
  fi
)
CONVERSATION_START_COMMIT_SHORT = $(git log --format="%h" -1 "$CONVERSATION_START_COMMIT" 2>/dev/null)

## REMEMBER

Think step by step about what you need to remember to incorporate the user feedback into your future work. Consider the git state and any changes made during this conversation.

## RUN

 - echo "=== FEEDBACK SESSION REPORT ===" 
 - echo "User feedback: $OPEN_CODING"
 - echo ""
 - echo "=== GIT STATE TRACKING ===" 
 - echo "Conversation started at commit: $CONVERSATION_START_COMMIT_SHORT ($CONVERSATION_START_COMMIT)"
 - echo "Current commit: $CURRENT_COMMIT_SHORT ($CURRENT_COMMIT)"
 - echo ""
 - echo "=== COMPLETE CONVERSATION CHANGES ==="
 - echo "This shows all changes since conversation started (like 'git diff' at session end):"
 - echo ""
 - if [ "$CONVERSATION_START_COMMIT" != "$CURRENT_COMMIT" ]; then echo "--- Commits made during conversation ---"; git log --oneline "$CONVERSATION_START_COMMIT..$CURRENT_COMMIT" 2>/dev/null || echo "No new commits between start and current"; echo ""; fi
 - echo "--- Summary of all changes from conversation start ---"
 - git diff --stat "$CONVERSATION_START_COMMIT" 2>/dev/null || echo "Unable to generate diff stat"
 - echo ""
 - echo "--- Uncommitted changes in working directory ---"
 - git status --porcelain
 - echo ""
 - echo "--- Full diff from conversation start (committed changes) ---"
 - git diff "$CONVERSATION_START_COMMIT" 2>/dev/null || echo "Unable to generate detailed diff"
 - if git status --porcelain | grep -q .; then echo ""; echo "--- Plus these uncommitted working directory changes ---"; git diff HEAD 2>/dev/null || echo "No working directory changes to show"; fi
 - echo ""
 - write what you need to remember to your claude.md file, including git state context
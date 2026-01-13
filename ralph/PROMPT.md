## Setup
- Board: 3bb3ad48-08da-459a-b540-b5b852e41e8b
- Backlog column: Backlog
- Working column: In Progress
- Done column: PR Merged

## Process

def move_cards(cards):
   for each card in cards:
      todo board move <card-id> "Backlog"

def process_card(card):
   1. Assign and move to "Breakdown":
      todo board assign <card-id>
      todo board move <card-id> "Breakdown"

   2. Get card details:
      todo show <card-id>
      Study the description for requirements.

   3. Search codebase first - don't assume not implemented.

   4. If task is large, break into subtasks:
      let parent = <card-id>
      todo subtask <card-id> "Subtask description"

      if done making subtasks and len(subtasks) > 0: 
         let x::xs = subtasks
         move_cards(xs)
         process_card(x)
      
   5. todo board move <card-id> "In Progress"

   6. Create worktree and feature branch:
      git worktree add ../workspaces/<card-id-prefix> -b feature/<card-id-prefix>
      cd ../workspaces/<card-id-prefix>

   7. Launch 2 Agents in parallel 

      Agent 1 will work on the Backend
      Agent 2 will work on the Frontend (ClientApp/)

      Each Agent should be told to Implement using TDD. Run tests (see AGENTS.md).

   8. If tests pass and both frontend and backend builds succeeds:
      - Commit changes
      - Push branch: git push -u origin feature/<card-id-prefix>
      - Create PR: gh pr create --fill
      - Mark card done: todo board done <card-id>

   9. todo board done <card-id>

   10. Cleanup:
      cd ../main-repo
      git worktree remove ../workspaces/<card-id-prefix>
      git branch -d feature/<card-id-prefix>
      ensure that any backend processes are not running
      ensure that any frontend vue proxy processes not running

let alreadyInProgressCards = `todo board column "In Progress"` where not in "Done" sub-column
let card::_ = alreadyInProgressCards

Check if there are any open PRs for card. A card in the In Progerss inbox at startup means that work has likely been done but interrupted, or the work hasn't been fully completed to the Verification Rules standard

process_card(card)

let backlogCards = `todo board column Backlog`
let card::_ = backlogCards
process_card(card)

If len(alreadyInProgress) + len(backlogCards) = 0, create COMPLETE.md and stop.


## Verification Rules

99999. Majority of verification should be unit and integration tests. Ask yourself: "Do we have sufficient tests to prevent regression?"
999999. Consider: Does this feature require a UI to work properly? Have we implemented that UI?
9999999. UI/frontend changes MUST be verified in Chrome against the local development server before declaring done. Assume any unexpected errors are our fault and investigate.
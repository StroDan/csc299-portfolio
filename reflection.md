# Reflection

My instructor said the round rules don't apply to Tiny Blacksmith. So for the questions below I use my other two projects, duesday and one-word-study, which I built with problem.md, fresh sessions and tests.

## 1. What did a fresh session get wrong or ask about that an earlier round had already settled? What was missing from the repository that would have prevented it?

In duesday, I asked a fresh session to build the menu (R7–R9). It stopped and didn’t write any code. The reason was that these requirements were only in my conversation with the assistant, not in problem.md, and CLAUDE.md told the session that problem.md is the source of truth, so it had nothing to build from. After I wrote R7 and R8 into problem.md (commit 9c1985b), the next session built them without asking any questions. Now, before I start a new session, I always write new requirements into problem.md and commit them first.

## 2. In the round that changed an earlier requirement, what broke? Did the tests catch it, and does the decision record explain the change to someone who was not there?

In one-word-study, I changed R4 to make the card rules stricter and added R7 (commit 5dd292c), because when I tried a real PDF from my economics class, I got bad cards: a time like “Mondays 2:30-5:00 pm” was split at the colon into a card with the term “Mondays 2”, and long definitions were cut off at the end of the line. Some old tests described the old rules on purpose (“a colon without a space is a card”, “wrapped lines are ignored”, “Chapter 3: Cells is a card”), so I updated them first, and 15 tests failed before the fix (commit 88fea67), but something also broke that I did not expect: the duplicate-card test used the lowercase term “cell”, which the new R4 rejects, so R7 glued that line onto the previous card instead. The tests did catch it, and the duplicate-card test was updated to use “CELL” so it tests duplicates again instead of the new rules (commit 6640c91). There is no decision record for this change, because one-word-study has no docs/decisions/ folder; the reason is in spec.md, which marks the rules “(changed with R4)”, and in the commit messages. Someone who was not there would have to put the story together from those places, so next time I would write a short decision record saying what changed, why, and which tests had to change.

## 3. Name one test that caught a real problem, or one test the assistant wrote that checked nothing useful. Quote it.

In duesday, `test_R8_cleared_once_per_menu_screen` in `tests/test_r8_clear.py`:

```python
menu("2", "", "4", "", "3", "1", "", "1", "b", "", "", "q")
assert menu.console.clears == 5  # first screen + one return after each of 4 actions
```

It was supposed to run four actions (list, month, mark task #1 done, and add a task titled “b” with no due date) and check that the screen was cleared 5 times: once at the start and once after each action. After I added R9, typing “b” means “go back”, so the fourth action never added a task; it went straight back to the menu, and the two extra “” answers were treated as wrong menu choices, but the count still happened to be 5, so the test kept passing even though it no longer did what its comment says. A fresh session noticed this and changed the title from “b” to “Essay” in commit d13283e, with the commit message saying the test “only passed by accident” instead of adding a task. I learned that a passing test is not proof that the code works: this one passed for the wrong reason, so when a requirement changes, I should check that each related test would still fail if the code it tests were wrong.

## 4. Choose one commit you would now write differently, and say how.

In one-word-study, commit 39b6f90 has the message “Save every upload as a set, give each browser a current set, and add the history list with Study and confirmed Delete (R10, R11)”. The problem is that it does four things at once, which you can see from the “and”s in the message, while the rules say one change per commit; if one part had a bug, it would be hard to review that part or undo it without undoing the others. It also never says why: the commit has only this subject line and the co-author line, with no explanation in the body. Now I would split it into four commits, each saying what changed and why:

1. “Save every uploaded PDF as its own card set (R10) so old cards are not lost”
2. “Give each browser a current set (R11) so the cards and quiz pages know which saved set to show”
3. “Show the saved sets on the home page with a Study button (R11) so the student can return to cards from an earlier PDF”
4. “Add Delete to the history list with a confirm step (R11) so a set isn’t removed by accident”

## 5. Choose one function in the project you developed further that you did not read while building it. Without an assistant open, read it and write what you think it does, in two or three sentences. Then ask an assistant, and say what you got right and wrong.

I chose `month_marks` in `duesday/tasks.py`, which I did not read while building the project.

What I thought it did (before asking): It starts with an empty dictionary called marks. It checks every task: whether the task has no due date, and whether the due date is in a different month or year. After that there is a part I don't understand, but I know for sure it creates something new and empty. It also adds 1 to the day's count. If a task is overdue, it marks it true. It returns the marks for the days.

After asking an assistant: I understood most of it correctly: it goes through all tasks, skips the ones with no due date or with a due date in another month or year, counts how many tasks are due on each day, marks a day as overdue if any of its tasks is overdue, and returns a dictionary from day number to its mark. The part I only half understood was setdefault. I saw that it creates a new empty mark, but I did not notice that it does this only the first time a day appears; after that it reuses the same mark, so two tasks due on the same day share one mark and the count goes up to 2. I also did not notice that done tasks are counted too, because nothing in the function skips them.


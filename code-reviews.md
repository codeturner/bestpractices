# Best Practices for a Code Reviewer

Google engineering published their best practices on how to do a code review (see <https://google.github.io/eng-practices/review/reviewer>). It addresses many of the hang-ups we've encountered in our own code review process here at Ident, and should be required reading for all software engineers at Ident.

The following highlights some of the key points that I liked from this guide on how to review code.

## Every PR should have an intrinsic higher goal to improve general code health

Yes, the code should implement the required features and such, but code that reduces the overall health of the code should never be accepted in non-emergency situations.

## Review with courtesy and love

Be honest about flaws, but also be sure to applaud the good engineering that occurred. Build confidence from within the team.

## Be an enabler of speedy reviews

Block out time our of every day to review your peers' code. We prioritize the efficiency of the team over the efficiency of a single engineer. Providing timely feedback is essential to enabling team members to complete tasks as a team. This means reviewers need to respond to code reviews within the next business day.

However, be conscious of the importance of maintaining momentum. Current tasks shouldn't be interrupted, but should instead wait for a natural breaks in the work.

## Take the time to educate and teach

As part of our "always learning, always teaching" culture at Ident, reviews become a great medium for sharing knowledge with other engineers, and is essential to improving the software health over time. Just be sure that if your comment is educational and should not be addressed as required for approval, prefix your comment with "Nit:" to indicate its optional nature.

## Prioritize readability over brevity

While minimizing lines of code is often the correct approach, do not do so at the expense of being readable to a reviewer. The primary purpose of a programming language is to instruct the computer to perform a task using instructions that express the programmer's intent in clear language. It the reviewer cannot quickly understand this intent, then the code needs to be refactored. If the reviewer has to ask what the code is doing, this should prompt a rewrite of that code.

## Comments should be used to explain the why of the code, or to document complexities that cannot be simplified

Avoid comments that describe simple things, but also comments are needed when it is unclear why it was done a certain way. As a reviewer, insist on code clarity.

## Don't argue in comments

If a quick resolution is not happening in the code review tool, pull the conversation out to a different medium, then come back to the review and comment on how the issue was resolved.

## Break up large PRs into several smaller PRs

Large change sets become a burden to the team, and inhibit overall team speed. Instead, create a PR plan beforehand on how your work can be split up for an effective code review. In particular, if your work includes large scale work such as interface expansion or code reformatting, isolate those activities into individual PRs.

## Point out problems, but don't solve them

The code review process is a forum for receiving feedback. If the reviewer tries to explicit solve found problems for the author, then a learning opportunity will be missed. Be willing to offer advice if solicited, but avoid providing direct guidance to the author.

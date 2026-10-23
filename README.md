# memex-gerrit

A [memex] adapter for the `discussion` role in Gerrit. The role is
specified in memex's [discussion.md].

## Reaching Gerrit

ADDR is the base URL of the server.

The adapter uses the REST API. With credentials, from GERRIT_USER and
GERRIT_PASSWORD (the HTTP password) or else from `~/.netrc` for ADDR's
host, it calls the `/a/` endpoints; without them, only reading works,
and PERSON `me` is the account the working copy's `ssh://` remote on
ADDR's host logs in as.
It strips `)]}'` from every reply.


## Commands

    locate COMMIT   GET /changes/?q=commit:COMMIT
    change CHANGE   GET /changes/CHANGE?o=ALL_REVISIONS&o=ALL_COMMITS
                    &o=DETAILED_ACCOUNTS&o=DETAILED_LABELS
    series CHANGE   GET /changes/CHANGE/revisions/current/related,
                    reversed
    threads CHANGE  GET /changes/CHANGE/comments
    drafts CHANGE   GET /changes/CHANGE/drafts
    draft CHANGE    PUT /changes/CHANGE/revisions/REVISION/drafts
    discard CHANGE ID
                    DELETE /changes/CHANGE/revisions/REVISION/drafts/ID
    publish CHANGE  POST /changes/CHANGE/revisions/current/review, with
                    "drafts": "PUBLISH_ALL_REVISIONS"

All capabilities are true.

## Change

- CHANGE is the change number.
- REVISION is the patch set number. `fetch` is
  `refs/changes/NN/CHANGE/PS`, NN being the last two digits of CHANGE.
- `status`: NEW is open, MERGED merged, ABANDONED abandoned.
- `votes`: every non-zero vote in each label's `all`.
- `links`: `change` and `change-id`, the Change-Id.
- PERSON `me` is true for the account of `/accounts/self`.

## Thread

A thread is a comment and the comments that reply to it, followed through
`in_reply_to` to the root.

- `revision` is the comment's `patch_set`.
- `resolved` is the opposite of the last comment's `unresolved`.
- `basis` is the current patch set and the id of the last comment.

The ANCHOR comes from the comment's `path`, `line`, `range` and `side`:

- path `/PATCHSET_LEVEL`: `change`
- path `/COMMIT_MSG`: `message`, or `lines` in it
- no `line` and no `range`: `file`
- `line` only: `lines` from `line` to `line`
- `range`: `lines` from `start_line` to `end_line`, `start_col` is
  `start_character` + 1, `end_col` is `end_character` + 1
- `side` PARENT: `base`; otherwise `revision`

## Open questions

- `/COMMIT_MSG` lines count Gerrit's header above the message: keep
  them, or count from the message's first line?
- Comments on a merge commit's second parent: map to `base` and drop
  `parent`?

[memex]: https://github.com/roolebo/memex
[discussion.md]: https://github.com/roolebo/memex/blob/main/discussion.md

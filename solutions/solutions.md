
# Admission Controllers Always

Do all state changes in an admission controller before persisting them.

pros:
 - solves everything

cons:
 - loss of state/data if something gets in a bad state between acting and persistence. 
 - duplicates everything into the apiserver
 
 This gets  us out of the very restricted  set of API Server operations.  It 
also makes state changes  synchronous.

This jumps the state of the broker ahead of what the the apiserver
knows and has persisted until it is done and ready to be committed.
Anything in progress that fails is dropped on the floor. It could be
put into apiserver and through to etcd in a bad state so that users
have feedback on why something has failed. In a bad state, only allow
through updates that succeed.

Update Lock: Try to do the update, if it fails, reject the spec
change.

Delete Rollback: Try to do the delete, if it fails, reject the
request, do not set the deletion timestamp.

# Queuing/Journaling/Append-Only Log

filesystem/database inspired "how do I make something reliable on top
of something unreliable?"

For every write we want to do, stage it first, and then act on it in
order. Queue up changes, then act on them.

Every state change is attributable directly to someone.

Difficult for end users to see what the final state is.  Work required
to simulate the possible final state.

Requires a policy on what to do when an update fails, move on, or
halt. If halt, irritating to fix. If move on, requested work is
dropped.

Work might "back up" and take a while to clear.

# 

# converting admission controller

Can we have an admission controller convert the delete into an update and update the delete-field?


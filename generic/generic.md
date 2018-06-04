
# Problem Statement

"KubeAuth -> State is persisted -> ExternalAuth" flow.

Sync between KubeAuth and ExternalAuth can either not exist, or not be
up-to-date at all times.

Points:
- Kubernetes assumes it is the source of truth and the single
  arbitrator of what is a valid action to take
- Kube doesn't allow for a 3rd party to be the source of truth
- Kube doesn't allow for "reverse synchronization" - meaning the true
  state being outside of kube and then resource.Status being updated
  to match it. E.g. someone updates an Instance via the dashboard,
  that needs to be reflected back in Kube. But, in Spec or Status? If
  just status, it'll keep trying to make Spec be reality
- We need to track each change with the user that made the request
- We need to then pass along that user+change to the 3rd party and
  they need to be able to say "no" to that one change.

DELETE really exposes these issues. The Broker can say "no" for a
variety of reasons and when that happens the OSB API spec implies that
things are returned to the pre-action state. Kube does not allow
for that.

Some worst case scenarios:
 - on DELETE, broker rejects after a few hours with unauthorized
 - 

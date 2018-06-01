
# DELETE 

 - User A has permissions to the broker.
 - User B does not have permissions to the broker, but can still see and modify the k8s objects.

User A creates a [ServiceInstance] and binds it by creating a [ServiceBinding].
![provision](provision.jpg)

User B issues DELETE on the ServiceBinding. This is persisted as
[setting the DeletionTimestamp] on the resource. Service-catalog will
try to make the delete real by talking to the broker. Because this is
a different user, it will have a different Originating-Identity
Header. This will cause the broker to reject the request with a 403.

User A will later want to update the ServiceBinding, and be unable to,
as it will be in the deleting state waiting for the finalizer to
run. This state for User A is unacceptable.

![stuck delete](delete.jpg)

### tl;dr

 - User B deletes a kube resource, which initializes the deletionTimestamp
 - This causes the finalizer to run
 - User B is not allowed to delete the backing object
 - Therefore the kube object should be rolled-back/un-deleted.
 - This is impossible with api-machinery
 - The finalizer cannot ever finish
 - the object is in a bad state forever

### FAQ

#### Why is user B allowed to do that action? You should set up RBAC better.

When is any system ever coherent? This is a possible state. We need to
handle it.

#### You should be pre-authorizing the action before you persist it with an admission controller.

Still can get out of sync in between intial auth and action requiring auth.

### Possible solutions

#### add a declarative/imperative deprovision/unbind DELETE equivalent
 - un-kube like
 - must do an update to do a delete

#### hack the apiserver to remove this convention
```
Once the deletionTimestamp is set, this value may not be unset or
be set further into the future, although it may be shortened or the
resource may be deleted prior to this time.
```

#### imperative/declarative delete through UPDATE

 - disable user access to DELETE with RBAC
 - add a field that indicates we should go through the delete flow
   - update that field, have platform do the delete on the broker
   - have controller call DELETE on resource if broker sucessfully deletes

#### [Admission Controllers Always]

Do all state changes in an admission controller before persisting them.
 
pros:
 - solves everything, we can do anything we want.

cons:
 - loss of state/data if something gets in a bad state between acting and persistence. 
 - duplicates everything in the apiserver.

#### queue/journal/append-only-log

 - doesn't solve delete case directly. still need something else as workaround

[Admission Controllers Always]: ../solutions/solutions.md#admission-controllers-always
[ServiceInstance]: https://github.com/kubernetes-incubator/service-catalog/blob/v0.1.20/pkg/apis/servicecatalog/types.go#L670-L680
[ServiceBinding]: https://github.com/kubernetes-incubator/service-catalog/blob/v0.1.20/pkg/apis/servicecatalog/types.go#L964-L975
[setting the DeletionTimestamp]: https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md#metadata

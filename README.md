# rancher-vcenter-rbac

RBAC permissions for Rancher in vCenter

This took a few to figure out and ensure we were able to restrict Rancher access to specific objects (resource pools, datastores, port groups) in vCenter. Following the [documentation from Rancher v2.7](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/launch-kubernetes-with-rancher/use-new-nodes-in-an-infra-provider/vsphere/create-credentials) says to create a global role in vCenter with the following permissions.

| Privilege Group | Operations                                 |
| --------------- | ------------------------------------------ |
| Cns Privileges  | Searchable                                 |
| Datastore       | AllocateSpace                              |
|                 | Browse                                     |
|                 | FileManagement (Low level file operations) |
|                 | UpdateVirtualMachineFiles                  |
|                 | UpdateVirtualMachineMetadata               |
| Global          | Set custom attribute                       |
| Network         | Assign                                     |
| Resource        | AssignVMToPool                             |
| Virtual Machine | Config (All)                               |
|                 | GuestOperations (All)                      |
|                 | Interact (All)                             |
|                 | Inventory (All)                            |
|                 | Provisioning (All)                         |
| vSphere Tagging | Assign or Unassign vSphere Tag             |
|                 | Assign or Unassign vSphere Tag on Object   |

Instead, do the following...

1. Identify the resources you want Rancher to access
2. Create a new vCenter role, I used `RancherRole` and assign the permissions above as directed
3. Clone the vCenter `Read-only`
  1. Rename the cloned role to `RancherCnsSearchable`
  2. Add the permission `Cns.Searchable`
4. Create new user account in vCenter, I used `rancher-user@vsphere.local`
5. Using the list created in step 1, assign the role `RancherRole` to each object on the list using propagation where needed.
6. Assign the `RancherCnsSearchable` role to the user on the vCenter object, and disable propagation.

Looking in vCenter under `Administration | Access Control | Roles` you should see the following...

| Role                 | Defined in    | User/Group                 | Propagate |
| -------------------- | ------------- | -------------------------- | --------- |
| RancherCnsSearchable | vCenter       | VSPHERE.LOCAL\rancher-user | false     |
| RancherRole          | ResourcePools | VSPHERE.LOCAL\rancher-user | true      |
|                      | Folders       | VSPHERE.LOCAL\rancher-user | true      |
|                      | Datastores    | VSPHERE.LOCAL\rancher-user | true      |
|                      | Port Groups   | VSPHERE.LOCAL\rancher-user | true      |

At this point, you can add the user account to Rancher and start testing and using without allowing Rancher access to objects it doesn't need.

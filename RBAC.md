# Introduction

Contrail RBAC provides API (operation) along with object/resource level access control. Previously with multi-tenancy, only resource/object level access control was available that was modeled after unix style permissions for user, group (role) and others (world).

RBAC currently works in conjunction with Keystone as it works on user credentials obtained from keystone from token present in the API request. Credentials include user, role, tenant and domain names and corresponding UUIDs.

RBAC is controlled by list of rules. Attachment point for the rules is domain and project.

# API Level RBAC
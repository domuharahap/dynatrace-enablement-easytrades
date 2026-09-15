# Cleanup

Once you have completed all workshop use cases, follow these steps to remove all resources from your cluster and avoid unnecessary costs.

---

### Delete Codespaces Instance

!!! tip "Deleting the codespace from inside the container"
    We like to make your life easier, for convenience there is a function loaded in the shell of the Codespace for deleting the codespace, just type `deleteCodespace`. This will trigger the deletion of the codespace.

Another way to do this is by going to [https://github.com/codespaces](https://github.com/codespaces){target=_blank} and delete the codespace.

You may also want to deactivate or delete the API token needed for this lab.

### Revoke the Dynatrace API token

1. Log in to your Dynatrace tenant.
2. Navigate to **Settings > Access tokens**.
3. Find the token you created for this workshop.
4. Click **Revoke**.

!!! warning "Token cleanup is important"
    API tokens with broad scopes (especially `settings.write`) should be revoked after use. Do not leave workshop tokens active in production tenants.

---

<div class="grid cards" markdown>
- [References :octicons-arrow-right-24:](references.md)
</div>

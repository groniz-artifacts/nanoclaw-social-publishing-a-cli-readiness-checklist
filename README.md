# NanoClaw Social Publishing: A CLI Readiness Checklist

Groniz documents a CLI path for NanoClaw. Before using it, verify that the actual agent execution environment can run the CLI, authenticate through an approved credential route, and access the intended media. A CLI installed on the host may still be unavailable inside the agent container.

This checklist covers the prerequisites for running the CLI. We have not verified a complete NanoClaw integration or tested a delivery inside it. Groniz's remote MCP route is not confirmed for NanoClaw's documented stdio-only MCP path. Check the CLI prerequisites below before attempting delivery.

The [general setup guide](https://groniz.com/blog/ai-agent-social-media-publishing-setup-a-client-by-client-guide) can help you select an alternative client when you need a documented configuration that is ready to use.

## Identify where the command will execute

Current NanoClaw upstream documentation describes per-session agent containers and credential handling through a OneCLI proxy. The host and agent container can therefore differ in which executables, credentials, and files they can access. [NanoClaw repository](https://github.com/nanocoai/nanoclaw)

The exact Groniz-to-OneCLI credential mapping has not been verified for this guide. An operator still needs to resolve that mapping for your installation. Until then, this guide cannot supply a verified proxy configuration, secret mount, or container installation recipe.

Start by naming the environment that will execute the write. "On this machine" is not specific enough when the planning process, terminal, and agent session may run with different filesystem and credential access.

## Complete the readiness table

Fill out the host and agent columns separately. Repeat the checks inside the agent environment before filling in its column.

| Requirement | Host evidence | Agent execution evidence | Passing condition |
| --- | --- | --- | --- |
| CLI executable | Result of `command -v groniz` | Same check in the approved agent environment | Executor can find Groniz |
| Authentication | Sanitized `groniz whoami` result | Identity result through the approved credential route | Executor reaches the intended account |
| Destination | Selected integration ID | Read-only discovery from the executor | Intended integration is available |
| Local media | File exists and is readable | Approved file is available to the executor | Upload can read the intended bytes |
| Uploaded media | Returned Groniz `.path` if uploaded | Same reference included in the review packet | Post uses the uploaded reference |
| Delivery ownership | Named operator or agent | Matching owner in the task record | One executor owns submission |

Use the table to record observations; its rows are not configuration options. Resolve empty cells before publishing. If you are blocked, leave a precise note such as "CLI found; approved proxy authentication unresolved" so the next operator knows what to investigate.

## Check the CLI inside the execution environment

Run these commands in the environment approved for execution:

```bash
command -v groniz
groniz whoami
```

`command -v` checks whether the environment can find Groniz; `whoami` checks the authenticated identity. Avoid printing keys as part of diagnosis; record the environment, credential source category, and sanitized result instead.

If you cannot verify the approved proxy credential route, leave execution with an operator until it is resolved. Do not remove isolation or improvise a secret transfer to make the check pass. Continue only after the operator establishes a supported route, or use the host handoff described below.

Once identity succeeds in the intended environment, discover accounts and inspect the chosen integration:

```bash
groniz integrations:list
groniz integrations:settings INTEGRATION_ID
```

Use the returned integration ID and current schema. Similar account labels can refer to different destinations, and platform requirements differ. Required settings and content limits belong in the review record before the executor prepares a create request.

## Resolve media before reviewing the delivery

A filename in a conversation is not evidence that the executor can read the file. Verify access to the actual approved asset. If the asset must move between environments, use the mechanism approved for that deployment and check that the resulting file is the intended version.

Groniz's upload step comes before post creation:

```bash
groniz upload file
```

Replace `file` with the approved file path available to that executor. Use only the returned `.path` in the post's media data. Keep the local file path and uploaded reference as separate fields in your notes. The [media upload guide](https://groniz.com/blog/how-media-uploads-work-in-multi-platform-social-publishing) explains why the distinction matters across destinations.

## Use an authorized host handoff when necessary

If the agent environment is not ready, use NanoClaw to prepare a review packet for an authorized host operator. Include the exact final body, destination integration ID, approved media reference, provider settings, ISO timestamp with timezone, and the approval scope.

The host operator should verify identity and destination independently, then compare the packet with the current schema. The packet specifies what to publish. Keep credentials in the operator's approved environment. Identify the host as the sole executor so a later NanoClaw retry cannot create a second post.

After submission, preserve the returned post ID and observed state. Record the queued state, then confirm publication after delivery is due. If any earlier attempt has an uncertain result, reconcile remote records before the host creates anything.

To prepare the destination while runtime questions are being resolved, [connect the account in Groniz](https://groniz.com/console/connectors). Keep the readiness table with the task so the next session can see exactly what has been verified.

# AzShimRemove

The antithesis of [Blumira's AzShim](https://github.com/Blumira/AzShim). Removes the Azure resources that `AzShim.azcli -c` creates for the Blumira Event Hubs integration.

Unofficial and not affiliated with or supported by Blumira.

## What AzShim creates (and what this removes)

| Resource | Name |
|---|---|
| Resource group | `blumira-rg-<epoch>` |
| Event Hubs namespace | `blumiraeventhubns-<region>-<epoch>` |
| Event hub | `blumiraeventhub-<region>-<epoch>` |
| Listen SAS rule | `LogsToBlumira-<epoch>` |
| Subscription diagnostic setting | `BlumiraDiagSetting-<epoch>` |

Deleting the resource group removes the namespace, event hub and SAS rule together.

It also finds diagnostic settings (subscription, Entra ID tenant, and with `-d` resource level) that stream to the namespaces being removed, even if they were created manually under another name.

Order of operations: diagnostic settings first, then resource groups, so nothing is left pointing at a deleted event hub.

## Pre-requisites

1. Owner, or Contributor plus permission to delete diagnostic settings, on the subscription
2. Global Administrator or Security Administrator to remove Entra ID diagnostic settings (otherwise the script tells you to do that step manually)
3. Azure Cloud Shell in **Bash**, or a local bash 4+ shell with Azure CLI and `az login` completed

## Running it

Paste into Azure Cloud Shell (Bash):

```
git clone https://github.com/m-timm/AzShimRemove.git
cd ./AzShimRemove
chmod +x ./AzShimRemove.azcli
./AzShimRemove.azcli -n
```

`-n` is a dry run and changes nothing. When the output looks right:

```
./AzShimRemove.azcli -r
```

Run it once per subscription AzShim was deployed to.

## Options

| Flag | Meaning |
|---|---|
| `-r` | Run the removal (interactive) |
| `-n` | Dry run, shows what would be removed (implies `-r`) |
| `-y` | Assume yes. Resource groups holding anything other than Event Hubs namespaces are still skipped |
| `-d` | Deep scan of every resource's diagnostic settings (slow) |
| `-g <rg>` | Target a specific resource group instead of matching `blumira-rg-<epoch>` |
| `-h` | Help |

## Not removable via Azure CLI

- Blumira: Settings > Cloud Connectors, remove the Azure Event Hubs connector(s)
- Defender XDR: Streaming API settings targeting the removed event hub
- Intune: diagnostic settings targeting the removed event hub
- The `Microsoft.Insights` provider registration is intentionally left in place

## Warning

This permanently deletes resource groups. Use `-n` first and read the output.

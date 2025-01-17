# Developing hcloud-cli

## Generated files

This repository contains generated files, mainly for testing purposes. These files are generated by running

```sh
go generate ./...
```

in the root directory of this repository. Make sure to keep generated files up-to-date
when making changes to the code.

## Unit tests

Unit tests are located in the `internal` directory. Run them with:

```sh
go test ./...
```

## Build

To build the binary, run:

```sh
go build -o hcloud-cli ./cmd/hcloud
```

To include version information in the resulting binary and build for all targets, use GoReleaser:

```sh
goreleaser --snapshot --skip-publish --rm-dist
```

## Conventions

### Subcommand groups

Cobra offers the functionality to group subcommands. The conventions on when and how to group commands are as follows:

1. Use the following Categories:
    - **General** (CRUD operations such as `create`, `delete`, `describe`, `list`, `update` + Label
      commands `add-`/`remove-label`)
    - Groups based on actions (e.g. `enable`, `disable`, `attach`, `detach`), for example `enable-`/`disable-protection`
      or `poweron`/`poweroff`/`shutdown`/`reset`/`reboot`
    - **Additional commands** that don't fit into the other categories (`Group.ID` is empty). These should be
      utility commands, for example ones that perform client-side actions.
2. Groups are only needed if more than the "General" group commands are present.
3. `Group.ID` formatting:
    1. If the `ID` is a noun, it should be singular.
    2. If the `ID` is a verb, it should be in the infinitive form.
    3. The `ID` should be in kebab-case.
4. `Group.Title` formatting:
    1. Should be the `ID` but capitalized and with spaces instead of dashes.
    2. If a single resource is managed, the `Title` should be singular. Otherwise, it should be plural.

       Example: Multiple network can be attached to server -> `Networks`, but only one ISO can be attached to a server -> `ISO`

Here is how to create a group:

```go
// General
util.AddGroup(cmd, "general", "General",
   SomeGeneralCommand.CobraCommand(s),
   // ...
)

// Additional commands
cmd.AddCommand(
   SomeUtilCommand.CobraCommand(s),
   // ...
)
```

Example of the `hcloud server` command groups:

```
General:
  add-label                   Add a label to a server
  change-type                 Change type of a server
  create                      Create a server
  create-image                Create an image from a server
  delete                      Delete a server
  describe                    Describe a server
  list                        List Servers
  rebuild                     Rebuild a server
  remove-label                Remove a label from a server
  update                      Update a Server

Protection:
  disable-protection          Disable resource protection for a server
  enable-protection           Enable resource protection for a server

Rescue:
  disable-rescue              Disable rescue for a server
  enable-rescue               Enable rescue for a server

Power/Reboot:
  poweroff                    Poweroff a server
  poweron                     Poweron a server
  reboot                      Reboot a server
  reset                       Reset a server
  shutdown                    Shutdown a server

Networks:
  attach-to-network           Attach a server to a network
  change-alias-ips            Change a server's alias IPs in a network
  detach-from-network         Detach a server from a network

ISO:
  attach-iso                  Attach an ISO to a server
  detach-iso                  Detach an ISO from a server

Placement Groups:
  add-to-placement-group      Add a server to a placement group
  remove-from-placement-group Removes a server from a placement group

Backup:
  disable-backup              Disable backup for a server
  enable-backup               Enable backup for a server

Additional Commands:
  ip                          Print a server's IP address
  metrics                     [ALPHA] Metrics from a Server
  request-console             Request a WebSocket VNC console for a server
  reset-password              Reset the root password of a server
  set-rdns                    Change reverse DNS of a Server
  ssh                         Spawn an SSH connection for the server
```

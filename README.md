<!--
SPDX-FileCopyrightText: © 2026 cloudquarter.io
SPDX-License-Identifier: Apache-2.0

File:    README.md
Project: cqdcp-tools
Author:  Raul Saura <raul@cloudquarter.io>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# cqdcp-tools

Ready-to-use binaries of the client tools for the cloudquarter.io
distributed control plane:

- **`cq`**, the command-line interface;
- **`terraform-provider-cqdcp`**, the Terraform provider, source address
  `cloudquarter/cqdcp`;
- the **Guided Exercises' files**: the Terraform configuration and the YAML
  the chapters of the guide *Guided Exercises for Consumers* build from.

This repository holds **release files only**. The source code is not here,
and nothing in this repository needs to be built. Each binary is distributed
under the licence of its own project: `cq` under the Apache License 2.0, the
Terraform provider under the Mozilla Public License 2.0. See
[Licences](#licences). The Guided Exercises' files are cloudquarter.io's,
under the notice each file carries.

## Releases

Each tool is released under its own tag prefix, with one GitHub Release per
tag:

| Tool | Tag | Release title | Files |
|---|---|---|---|
| `cq` | `cq/vX.Y.Z` | `cq X.Y.Z` | `cq-vX.Y.Z-<os>-<arch>` (`.exe` on Windows), `SHA256SUMS`, `LICENSE`, `THIRD_PARTY_NOTICES` |
| Terraform provider | `terraform-provider-cqdcp/vX.Y.Z` | `terraform-provider-cqdcp X.Y.Z` | `terraform-provider-cqdcp_X.Y.Z_<os>_<arch>.zip`, `terraform-provider-cqdcp_X.Y.Z_SHA256SUMS`, `LICENSE`, `THIRD_PARTY_NOTICES` |
| Guided Exercises' files | `guided-exercises/vX.Y.Z` | `guided-exercises X.Y.Z` | `guided-exercises-X.Y.Z.tar.gz`, `SHA256SUMS` |

Platforms: `darwin_amd64`, `darwin_arm64` (Apple silicon), `linux_amd64`,
`linux_arm64` and `windows_amd64`.

A tool's version follows the API version of the control plane it was made
for. Install the release whose version matches the API version of the region
you use, or the newest release below it; your operator tells you which
version the region runs. `cq` compares its version with the region's API
version: it prints a warning when the minor version or only the patch version
differs, and runs the command; it refuses to run when the major version
differs, unless you pass `--skip-version-check` or set
`CQDCP_SKIP_VERSION_CHECK=1`. The binaries are not code-signed.

The Guided Exercises' files follow the guide instead: install the version its
laptop setup names.

## Install `cq`

### macOS and Linux

Set the version, your operating system (`darwin` or `linux`) and your
architecture (`arm64` or `amd64`), then download the binary and the checksum
file:

```sh
V=v2.329.0; OS=darwin; ARCH=arm64
BASE="https://github.com/cloudquarter-io/cqdcp-tools/releases/download/cq/$V"
curl -fLO "$BASE/cq-$V-$OS-$ARCH"
curl -fLO "$BASE/SHA256SUMS"
```

Check the download. The line for your file must say `OK`:

```sh
shasum -a 256 -c --ignore-missing SHA256SUMS     # Linux: sha256sum -c --ignore-missing SHA256SUMS
```

On macOS, a file downloaded with a web browser is quarantined, and macOS
refuses to open the unsigned binary. A download with `curl` is not
quarantined. If you used a browser, remove the quarantine flag from the
downloaded file before you install it:

```sh
xattr -d com.apple.quarantine "cq-$V-$OS-$ARCH"
```

Put it on your `PATH` as `cq`, and check that it runs:

```sh
sudo install -m 0755 "cq-$V-$OS-$ARCH" /usr/local/bin/cq   # or any directory on your PATH
cq --version
```

### Windows (PowerShell)

```powershell
$V = "v2.329.0"
$Base = "https://github.com/cloudquarter-io/cqdcp-tools/releases/download/cq/$V"
Invoke-WebRequest "$Base/cq-$V-windows-amd64.exe" -OutFile cq.exe
Invoke-WebRequest "$Base/SHA256SUMS" -OutFile SHA256SUMS
(Get-FileHash cq.exe -Algorithm SHA256).Hash.ToLower()
Select-String "cq-$V-windows-amd64.exe" SHA256SUMS
```

The two hashes must be the same. Move `cq.exe` to a directory on your `PATH`
and run `cq --version`. Windows SmartScreen may warn about the unsigned
binary.

## Install the Terraform provider

The provider is not in the public Terraform Registry. Terraform installs it
from a **filesystem mirror**, a local directory that holds the release zips.
Your configurations keep the source address `cloudquarter/cqdcp`, and
Terraform still downloads every other provider from the public registry.

### 1. Put the zip in the mirror

Terraform looks in a few local mirror directories without any configuration.
On macOS and Linux, one of them is `~/.terraform.d/plugins`; on Windows, it is
`%APPDATA%\terraform.d\plugins`. The provider's zip goes, as downloaded and
**not unzipped**, in this directory under that one:

```
registry.terraform.io/cloudquarter/cqdcp/terraform-provider-cqdcp_X.Y.Z_<os>_<arch>.zip
```

macOS and Linux (`P` is your platform: `darwin_arm64`, `darwin_amd64`,
`linux_amd64` or `linux_arm64`):

```sh
V=2.317.0; P=darwin_arm64
M="$HOME/.terraform.d/plugins/registry.terraform.io/cloudquarter/cqdcp"
BASE="https://github.com/cloudquarter-io/cqdcp-tools/releases/download/terraform-provider-cqdcp/v$V"
mkdir -p "$M" && cd "$M"
curl -fLO "$BASE/terraform-provider-cqdcp_${V}_${P}.zip"
curl -fLO "$BASE/terraform-provider-cqdcp_${V}_SHA256SUMS"
shasum -a 256 -c --ignore-missing "terraform-provider-cqdcp_${V}_SHA256SUMS"   # Linux: sha256sum -c --ignore-missing ...
```

Windows (PowerShell):

```powershell
$V = "2.317.0"
$M = "$env:APPDATA\terraform.d\plugins\registry.terraform.io\cloudquarter\cqdcp"
$Base = "https://github.com/cloudquarter-io/cqdcp-tools/releases/download/terraform-provider-cqdcp/v$V"
New-Item -ItemType Directory -Force $M | Out-Null
Invoke-WebRequest "$Base/terraform-provider-cqdcp_${V}_windows_amd64.zip" -OutFile "$M\terraform-provider-cqdcp_${V}_windows_amd64.zip"
Invoke-WebRequest "$Base/terraform-provider-cqdcp_${V}_SHA256SUMS" -OutFile "$M\terraform-provider-cqdcp_${V}_SHA256SUMS"
(Get-FileHash "$M\terraform-provider-cqdcp_${V}_windows_amd64.zip" -Algorithm SHA256).Hash.ToLower()
Select-String "windows_amd64.zip" "$M\terraform-provider-cqdcp_${V}_SHA256SUMS"
```

Terraform also accepts the zip's contents unpacked, in
`registry.terraform.io/cloudquarter/cqdcp/X.Y.Z/<os>_<arch>/` under the same
mirror directory. Use one layout or the other.

### 2. Require the provider

```hcl
terraform {
  required_providers {
    cqdcp = {
      source  = "cloudquarter/cqdcp"
      version = "2.317.0"
    }
  }
}
```

`terraform init` then installs `cloudquarter/cqdcp` from the mirror. When you
have no `provider_installation` block in your Terraform CLI configuration,
Terraform excludes every provider it finds in these mirror directories from
the registry by itself, so nothing else is needed.

### Alternative: an explicit `provider_installation` block

If your Terraform CLI configuration already has a `provider_installation`
block, Terraform no longer looks in the mirror directories by itself, and you
add the mirror to that block. The CLI configuration is `~/.terraformrc` on
macOS and Linux, and `%APPDATA%\terraform.rc` on Windows; the
`TF_CLI_CONFIG_FILE` environment variable can name another file. Give the
mirror's full path; Terraform does not expand `~` in it:

```hcl
provider_installation {
  filesystem_mirror {
    # Linux:   "/home/<you>/.terraform.d/plugins"
    # Windows: "C:/Users/<you>/AppData/Roaming/terraform.d/plugins"
    path    = "/Users/<you>/.terraform.d/plugins"   # macOS
    include = ["registry.terraform.io/cloudquarter/cqdcp"]
  }
  direct {
    exclude = ["registry.terraform.io/cloudquarter/cqdcp"]
  }
}
```

The mirror directory can be anywhere with this block, for example a shared
directory with the zips of every platform your team uses. Remove any
`dev_overrides` entry for `cloudquarter/cqdcp` left from a build from source.

### Lock file across operating systems

`terraform init` records in `.terraform.lock.hcl` the checksum of your own
platform's zip only. If your team shares the lock file across operating
systems, put the zip of every platform in one mirror directory and record them
all:

```sh
terraform providers lock -fs-mirror="$HOME/.terraform.d/plugins" \
  -platform=darwin_arm64 -platform=darwin_amd64 \
  -platform=linux_amd64 -platform=windows_amd64 \
  registry.terraform.io/cloudquarter/cqdcp
```

## Licences

`cq` is distributed under the Apache License 2.0, and the Terraform provider
under the Mozilla Public License 2.0. Every release carries, beside its
binaries:

- `LICENSE`, the full text of the tool's licence;
- `THIRD_PARTY_NOTICES`, the Go standard library and each open-source Go
  module compiled into the binaries of that release, with its version, the
  name of its licence, where the licence is published and its full text,
  followed by any NOTICE file the module carries.

The binaries include the Go modules unmodified. For each module under the
Mozilla Public License 2.0, such as several in the Terraform provider,
`THIRD_PARTY_NOTICES` also gives where its source code is available: the
module's public repository at that version, and the module's archive on the
Go module proxy, `https://proxy.golang.org/<module>/@v/<version>.zip`.

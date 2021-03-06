[![Build Status](https://travis-ci.org/brainrape/nixform.svg?branch=master)](https://travis-ci.org/brainrape/nixform)

# nixform

define terraform infrastructure in Nix

Terraform by Hashicorp is a great tool with a meh language. Fixed.

## Installation

First, [install nix](https://nixos.org/nix/download.html). Then,

```
# install nixform (and deps)
nix-env -i -f "https://github.com/brainrape/nixform/archive/master.tar.gz"
```

## Usage

Create a file like [example.tf.nix](example.tf.nix) containing a nix expression that evaluates to a set that looks like terraform's json format:

```nix
let
  lib = (import <nixpkgs> {}).lib;
  tpls = lib.mapAttrs (name: lib.recursiveUpdate {
    tags.name = name;
    ami = "ami-0d729a60";
    instance_type = "t2.nano";
  });
in {
  provider.aws.region = "us-east-1";
  resource.aws_instance = tpls {
    one = {};
    two = {
      tags.description = "Second!";
      instance_type = "t2.micro";
    };
  };
}
```

Then use `nixform` instead of `terraform`.

```
nixform apply
```


## How?

Before running `terraform` with the given arguments, `*.tf.nix` is evaluated strictly and output in terraform json format. The above example will produce an [example.tf.json](example.tf.json) like this:
```json
{
    "provider": {
        "aws": {
            "region": "us-east-1"
        }
    },
    "resource": {
        "aws_instance": {
            "one": {
                "ami": "ami-0d729a60",
                "instance_type": "t2.nano",
                "tags": {
                    "name": "one"
                }
            },
            "two": {
                "ami": "ami-0d729a60",
                "instance_type": "t2.micro",
                "tags": {
                    "description": "Second!",
                    "name": "two"
                }
            }
        }
    }
}
```
